---
title: Pre-Gen Audio Cron Job — Requirements
summary: Requirements for a VPS cron job that pre-generates story audio via Kokoro CLI, extracts word-level timestamps, aggregates to paragraph boundaries, and stores MP3 + timing metadata in Supabase.
type: requirements
status: draft
created: 2026-05-04
projects: [kokoros, yawnly]
tags: [cron, tts, pre-gen, timestamps, supabase]
---

# Pre-Gen Audio Cron Job — Requirements

## What This Is

A cron job running on the Hetzner CX23 VPS (the same box that hosts Kokoros) that pre-generates bedtime story audio so users don't wait for synthesis at read time.

## Input

- Story text + story ID from Supabase `stories` table
- Voice: `af_bella` (female) or `am_michael` (male), based on `stories.dreamer_profiles.gender`
- Language: `en` only initially (Kokoros Hindi quality unvalidated)
- Speed: `1.0` (normal pace; the client applies its own `0.8x` bedtime slowdown)

## Output

- One MP3 file per story, uploaded to Supabase Storage: `stories/{story_id}/audio.mp3`
- Paragraph-level timing metadata, stored in `stories.audio_timing` (JSONB):
  ```json
  [
    {"p": 0, "startMs": 0, "endMs": 4200},
    {"p": 1, "startMs": 4200, "endMs": 8200},
    ...
  ]
  ```
- Both the MP3 and the timing must be written **atomically** — if either fails, neither should be visible to the client.

## How Timing Works

Kokoro's ONNX engine produces word-level timestamps natively. The cron job must:

1. **Synthesize audio** via Kokoro CLI:
   ```shell
   kokoros text --timestamps --lan en --style af_bella --speed 1.0 \
     -o /tmp/story_{id}.wav -- "{story_text}"
   ```
   → `/tmp/story_{id}.wav` (24kHz PCM)
   → `/tmp/story_{id}.tsv` (word, start_sec, end_sec)

2. **Convert to MP3** with ffmpeg:
   ```shell
   ffmpeg -i /tmp/story_{id}.wav -b:a 48k /tmp/story_{id}.mp3
   ```

3. **Aggregate word timestamps to paragraphs:**
   - Split story text into paragraphs by `\n\n`
   - Compute character offset range of each paragraph in the full text
   - Map each TSV word to its paragraph by checking if the word's character position falls in the paragraph's range
   - Paragraph `startMs` = `start_sec × 1000` of first word in paragraph
   - Paragraph `endMs` = `end_sec × 1000` of last word in paragraph
   - Handle edge case: empty paragraphs → skip, don't break the index mapping
   - Handle edge case: last paragraph may end slightly before audio EOF → use total audio duration as end cap

4. **Store:**
   - Upload MP3 to Supabase Storage: `stories/{story_id}/audio.mp3`
   - UPDATE `stories SET audio_timing = $timing_json WHERE id = $story_id`

5. **Cleanup:** remove temp WAV, TSV, and MP3 from `/tmp/`

## Schedule

- Cron: `0 2 * * *` (2am IST daily)
- Processes **60 stories** per run (₹99 plan: 2 stories/day/user for 30 users)
- Sequential execution (CX23 has 2 vCPU, one Kokoros instance — parallel synthesis hurts RTF)
- Estimated wall time: 60 stories × ~6s/story = ~6 minutes + ffmpeg overhead. Well within the idle window.

## Error Handling

- If Kokoros synthesis fails for a story → log, skip, continue to next story
- If ffmpeg conversion fails → log, skip, do not upload partial data
- If Supabase upload fails → retry 3 times with exponential backoff, then log and skip
- All failures must log enough context for debugging: story_id, error message, timestamp
- The cron run must **never abort mid-batch** — one bad story shouldn't kill the remaining 59

## Constraints

- Runs on Hetzner CX23, same box as Kokoros (`/opt/kokoros/`)
- Kokoros is already running as a Docker container in `openai` mode. The cron job calls the CLI directly or via Docker exec — not through the HTTP API (timestamps are CLI-only)
- Must not conflict with live Kokoros HTTP traffic. If the cron runs while a user triggers on-demand TTS, the CLI call must not steal the Docker container (or: use a separate Kokoros process)
- MP3 bitrate: 48kbps (speech-optimized, same as existing Edge TTS output)
- Temp files must be cleaned even if the script crashes (trap EXIT)
- Secrets: Supabase service role key for Storage uploads — stored on VPS, never in git or MyWiki
- Supabase project ID: `ynjiskzaxsqdovmugiwo`

## What the Client Needs

When a user opens a story that has pre-gen audio:
- `stories.audio_timing` is populated → client reads it on story fetch, feeds into `engine._timingMetadata`
- The existing `activeParagraphIndexAtElapsed()` code path handles highlighting — no client changes needed beyond reading the new field
- If `audio_timing` is NULL → story hasn't been pre-generated yet → fall back to Edge TTS (existing behavior)

## Open Questions (for the spec)

1. Should the cron job write to Supabase directly from the VPS (service role key), or go through an Edge Function?
2. How do we detect which stories need pre-gen? Query `stories` where `audio_timing IS NULL` and `created_at > now() - 24h`? Or a separate `pre_gen_queue` table?
3. What happens if a user opens a story while the cron job is mid-synthesis for that story? Race condition between `audio_timing` being NULL → client falls back to Edge TTS → cron completes.
4. Should pre-gen be scoped to ₹99 users only, or all users? (₹33 users get 10 audio credits — pre-gen wastes TTS compute for them)

## Sources

- Kokoro timestamps: `/Users/eloelo/Downloads/Kokoros/kokoros/src/tts/koko.rs:744` (`tts_timestamped_raw_audio`)
- CLI flag: `/Users/eloelo/Downloads/Kokoros/koko/src/main.rs:148` (`--timestamps`)
- Research note: `research/inbox/kokoro-word-timestamps-20260504.md`
- Pricing: `projects/yawnly/decisions.md` (D009)
