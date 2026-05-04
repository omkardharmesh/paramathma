---
id: kokoros-20260504-vps-cron-changes-spec
project: kokoros
repo_path: /Users/eloelo/Downloads/Kokoros
status: draft
mode: implement-local
risk: low
created: 2026-05-04
updated: 2026-05-05
owner: human
executor: hermes
---

# Spec: Kokoros VPS Changes for Pre-Gen Audio Pipeline

## Goal

Design and specify the cron job pipeline that runs on the Hetzner CX23 VPS to pre-generate audio for Yawnly stories. The pipeline uses Kokoro's native CLI `--timestamps` mode to produce WAV audio + word-level timing TSV sidecars, aggregates per-word timestamps to paragraph boundaries, encodes to MP3, and uploads everything to Supabase storage. All 10 areas enumerated in the task description must be resolved with clear decisions, open questions flagged, and no implementation dead ends.

## Why

Yawnly needs pre-generated, timestamped audio so the mobile app can show word-level text highlighting during playback without client-side WPM estimation, forced alignment, or paragraph-split synthesis. The HTTP API cannot produce timestamps; the CLI can. This pipeline runs as a periodic cron job on the existing VPS.

## End Goal

A fully specified, implementable task packet (`mode: implement-local`) that an executor can build from. Every decision is either resolved (with evidence) or explicitly flagged as requiring a VPS test or human decision.

## Output Format

This document: a task packet in `plan-only` mode. After human review and approval, it converts to `implement-local` with a contained Implementation Plan section.

## Recommended Approach

1. Verify the custom Docker image `kokoros:quantized-schema-fix-amd64` has the `koko` binary with `--timestamps` support — read the fork's `main.rs` or rebuild test.
2. Decide invocation strategy: `docker exec` vs separate container vs host binary.
3. Design the cron script flow: download story text → CLI synth with `--timestamps` → parse TSV → aggregate to paragraph boundaries → encode MP3 → upload to Supabase → clean up temp files.
4. Voice verification: test `af_bella` and `am_michael` with `--timestamps` on VPS.
5. Performance benchmark: run 60-story batch, record wall time, peak RAM, disk usage.
6. Design monitoring: log Kokoro stderr, per-story timing, upload status.
7. Document cleanup strategy: temp WAV/TSV/MP3 lifecycle.
8. Note future Hindi path: voice map + language code.

### Paragraph Aggregation Algorithm

The cron job must aggregate word-level TSV timestamps to paragraph boundaries.

**WARNING: Do NOT use character-offset mapping.** eSpeak phonemization (used internally by Kokoro) can merge, split, or alter word boundaries. The TSV `word` field may not match a substring at a specific character offset in the original text. Character-offset lookup will misassign or fail silently.

**Recommended: Sequential Word Matching**

1. Split story text into paragraphs by `\n\n`
2. For each paragraph, normalize its text (lowercase, strip punctuation) and split into a word sequence
3. Read the TSV words in order
4. For each TSV word, normalize it (lowercase, strip punctuation) and match it sequentially against the current paragraph's word sequence
5. When all words in paragraph N are matched, record:
   - Paragraph `startMs` = `start_sec × 1000` of the first matched word
   - Paragraph `endMs` = `end_sec × 1000` of the last matched word
6. Advance to paragraph N+1 and continue matching remaining TSV words
7. Edge case: eSpeak may drop or merge words (e.g., "the model" → "themodel"). Use fuzzy matching (Levenshtein distance ≤ 2) or allow skips for unmatched words.
8. Edge case: empty paragraphs → skip, don't break index mapping
9. Edge case: last paragraph may end slightly before audio EOF → use total audio duration as end cap

Per `pre-gen-audio-requirements.md:55-61` and `context.md:93-100`.

### Timing Storage Destination

Paragraph timing is stored in `stories.audio_timing` (JSONB column). Format:

```json
[
  {"p": 0, "startMs": 0, "endMs": 4200},
  {"p": 1, "startMs": 4200, "endMs": 8200}
]
```

Per `pre-gen-audio-requirements.md:27-34`. The cron script must UPDATE this column after upload.

## Common Failure Handling

- **Model lacks `durations` output (empty TSV)**: The wrong ONNX model is mounted. Replace with the HF timestamped `model_uint8.onnx` from `onnx-community/Kokoro-82M-v1.0-ONNX-timestamped`. Verify by checking log for `OrtKoko: Timestamped backend activated (2 outputs)`.
- **Custom image lacks `--timestamps`**: fall back to rebuilding from the Kokoros fork with the schema fix + original CLI code. The fork is at `https://github.com/omkardharmesh/kokoros-tts.git`.
- **RAM contention during cron**: the running HTTP server already holds the model (~170 MB file, ~500 MB RSS). A second CLI process loads another model instance. If OOM, either (a) stop the HTTP server during the cron window, or (b) reduce server `--instances` to 0 during cron, or (c) run the CLI in a resource-constrained container with `--memory="1.5g"`.
- **Disk fills up**: set a max temp dir size and fail-delete on threshold. Monitor via `df`.
- **Cron overlaps with live traffic**: schedule cron at low-traffic hours (e.g., 3 AM UTC). Add a lockfile to prevent overlap.
- **Supabase upload fails**: retry with exponential backoff, log failure, leave files in place for manual retry, alert.
- **ffmpeg not found**: Install ffmpeg on VPS host (`apt install ffmpeg`) or use a sidecar container with ffmpeg.

## Allowed Paths

- `/Users/eloelo/Downloads/Kokoros/` (read-only during spec phase)
- `/Users/eloelo/Downloads/MyWiki/projects/kokoros/task-packets/`
- `/Users/eloelo/Downloads/MyWiki/projects/kokoros/context.md`
- `/Users/eloelo/Downloads/MyWiki/projects/kokoros/current-state.md`
- `/Users/eloelo/Downloads/MyWiki/research/inbox/`

## Required Context

- `projects/kokoros/context.md` — full context, timestamps section, operational commands
- `projects/kokoros/current-state.md` — deployment state, instances=1, uint8 model
- `Kokoros/deployment-spec/DEPLOYMENT_SPEC.md` — deployment architecture
- `research/inbox/kokoro-word-timestamps-20260504.md` — native timestamps support
- `Kokoros/koko/src/main.rs` — CLI modes, `--timestamps` flag, TSV output
- `Kokoros/Dockerfile` — binary build path

## Non-Goals

- No changes to the Kokoros HTTP API
- No forced alignment libraries (aeneas, gentle)
- No client-side timing estimation
- No changes to the mobile app
- No paragraph-split synthesis
- No changes to the Edge Function

## Stop Conditions

Standard per `meta/task-packet-schema.md`:
- Allowed Paths violation
- Dependency changes needed (new packages, build tools on VPS)
- Secrets needed and not declared
- Build requires unrelated fixes
- Architecture differs from linked guidelines
- More than 3 unexpected files need edits

---

# Spec Details

## 1. Docker Image and Timestamped Model

### Current State

- The VPS Docker image is `kokoros:quantized-schema-fix-amd64` (local image, per `current-state.md:20`).
- The upstream image `ghcr.io/lucasjinreal/kokoros:main` is NOT used in production.
- The custom image was built from the Kokoros fork to fix the ONNX schema mismatch (input_ids → waveform).
- The Dockerfile (`Kokoros/Dockerfile:29`) sets `ENTRYPOINT ["./target/release/koko"]`.

### Critical Finding: The Existing `model_uint8.onnx` Does NOT Support Timestamps

**Verified via code inspection and runtime test.** The `--timestamps` flag requires the ONNX model to have a `durations` output. The `model_uint8.onnx` currently in `/opt/kokoros/checkpoints/` (169 MB, from the non-timestamped release) has only one output: `waveform`. When `--timestamps` is used with this model, `ort_koko.rs:71-82` detects no `durations` output and falls back to `ModelStrategy::StandardWaveform`, producing **empty TSV files** (header only, zero word rows).

The HF timestamped repo (`onnx-community/Kokoro-82M-v1.0-ONNX-timestamped`) has a `model_uint8.onnx` with the **same filename and size** but **different internal outputs**:
- `waveform: [1, num_samples]`
- `durations: [1, sequence_length]`

**Docker test confirmed:** Using the HF timestamped `model_uint8.onnx`, the container outputs real TSV data:
```
word	start_sec	end_sec
hello	0.000	0.390
world	0.390	1.389
```

### Decision: Image Supports `--timestamps`, But Model Must Be Replaced

**Confirmed.** The `--timestamps` flag is compiled into `./target/release/koko` at `koko/src/main.rs:148`:

```rust
#[arg(long = "timestamps", default_value_t = false, global = true)]
timestamps: bool,
```

It is a `global = true` flag, meaning it applies to all subcommands (`Text`, `File`, `OpenAI`, `Stream`). In practice, it only takes effect in `Text` mode (line 334) and `File` mode (line 284). The `OpenAI` mode (HTTP server) ignores it.

Since the custom image was built from the same source tree (the fork includes all of `koko/src/main.rs`), the binary inside `kokoros:quantized-schema-fix-amd64` has the same `--timestamps` support.

**Required VPS Setup Step:**

```bash
# Download the HF timestamped model (replaces the non-timestamped one)
curl -L \
  "https://huggingface.co/onnx-community/Kokoro-82M-v1.0-ONNX-timestamped/resolve/main/onnx/model_uint8.onnx" \
  -o /opt/kokoros/checkpoints/kokoro-v1.0.onnx
```

**Verification command:**

```bash
docker run --rm \
  -v /opt/kokoros/checkpoints:/app/checkpoints:ro \
  -v /opt/kokoros/data:/app/data:ro \
  -v /tmp/tts-verify:/tmp/out \
  kokoros:quantized-schema-fix-amd64 \
  --timestamps --style af_bella text "hello world" -o /tmp/out/test.wav

cat /tmp/tts-verify/test.tsv
```

Expected: TSV has >0 data rows (not just header). Log should show `OrtKoko: Timestamped backend activated (2 outputs)`.

### Open Question

- Confirm the fork's git branch/tag that produced the custom image. If it was built from a branch that predates the `--timestamps` addition, need to rebuild.

---

## 2. CLI Availability: How the Cron Job Invokes `koko --timestamps`

### Options Analyzed

| Option | Pros | Cons |
|--------|------|------|
| **A: `docker exec` into running container** | No new container; model already loaded | Running `koko` CLI inside the same container loads a second model instance (~500 MB more RAM). The running server process already holds the model. Risk of OOM (CX23 has 4 GB total, container limited to 2.5 GB via `mem_limit`). |
| **B: Separate one-shot container** | Clean isolation; resource limits; no interference with HTTP server | Loads model from scratch each invocation (~3-5s init overhead). Additional RAM during synthesis. |
| **C: Install binary on host** | No Docker overhead | Requires Rust toolchain on VPS, OR cross-compile and SCP the binary. Adds maintenance burden. Need to replicate model/data paths. |

### Recommendation: Option B (Separate Container)

Run a one-shot `docker run` container that shares volumes with the running server:

```bash
docker run --rm \
  --memory="1.5g" \
  --cpus="1" \
  -v /opt/kokoros/data:/app/data:ro \
  -v /opt/kokoros/checkpoints:/app/checkpoints:ro \
  -v /tmp/tts-cron:/tmp/tts-cron \
  --entrypoint ./target/release/koko \
  kokoros:quantized-schema-fix-amd64 \
  --timestamps --speed 1.0 \
  --style af_bella \
  --lan en-us \
  text "Story text here" \
  -o /tmp/tts-cron/story_42.wav
```

**Why:**

- Does not interfere with the running HTTP server container.
- Shares the same ONNX model file + voice data (read-only volumes).
- Resources can be limited independently (`--memory="1.5g"`).
- Cleanup is automatic (`--rm`).
- Can run `--instances 1` (the default) without affecting server instances.
- No Rust toolchain or binary management on host.

**Trade-off:** Each invocation loads the ONNX model into a new process. At ~500 MB RSS per instance and 1.5 GB container limit, the cron job uses ~1 GB additional RAM during synthesis. CX23 has 4 GB total, with the server container using ~1-1.5 GB (model + runtime) and the host using ~500 MB (OS, Caddy, Docker). This leaves ~1 GB headroom — acceptable for a single CLI instance.

If RAM proves tight, fall back to Option A with server shutdown during cron window.

**`--speed 1.0` rationale:** Per `pre-gen-audio-requirements.md:22`, speed is explicitly `1.0` because the client applies its own `0.8x` bedtime slowdown. The CLI default at `main.rs:135` is also `1.0`. While the kokoros-integration skill uses `--speed 1.15`, using `1.0` ensures audio duration matches the timing data the client expects. This value directly affects timing output — a different speed produces different audio lengths and thus different `startMs`/`endMs` values in the paragraph timing data.

### Recommendation: Use `Text` Mode (Primary)

**Do NOT use `File` mode.** `File` mode reads one line per story (`main.rs:277: file_content.lines()`). Story texts contain paragraph breaks (`\n\n`). If a story has internal newlines, it will be split across multiple lines. The result:
- Line N processes only the first paragraph of story X
- Line N+1 processes the second paragraph as a separate "story"
- Timing aggregation breaks because each "story" is just one paragraph
- The TSV word-to-paragraph mapping is meaningless

**Use `Text` mode with one invocation per story.** This avoids the newline problem entirely and gives per-story timing granularity. The model-load overhead is ~3-5s per story. Given actual synthesis time per story is ~60s (see Performance section below), this 3-5s overhead is only ~5-8% of total batch time — acceptable for correctness.

```bash
# Per-story invocation
docker run --rm \
  --memory="1.5g" \
  --cpus="1" \
  -v /opt/kokoros/data:/app/data:ro \
  -v /opt/kokoros/checkpoints:/app/checkpoints:ro \
  -v /tmp/tts-cron:/tmp/tts-cron \
  --entrypoint ./target/release/koko \
  kokoros:quantized-schema-fix-amd64 \
  --timestamps --speed 1.0 --style af_bella --lan en-us \
  text "$STORY_TEXT" \
  -o /tmp/tts-cron/story_${story_id}.wav
```

Note: the TSV is written alongside the WAV with the same stem (e.g., `story_42.tsv`).

**Alternative: `File` Mode** — Only if story texts are guaranteed to have no internal newlines (single-paragraph stories). In that case, `File` mode loads the model once for all stories. **Not recommended for Yawnly** because stories have multiple paragraphs.

---

## 3. Voice Config: `af_bella` and `am_michael`

### Verification

Both voices are mapped in `kokoros-openai/src/lib.rs`:

| OpenAI alias | Kokoro voice |
|-------------|--------------|
| `fable` → `af_bella` (line 291) |
| `ballad` → `am_michael` (line 297) |

These are the same voices used by the HTTP API, which is verified working (`current-state.md:31`).

The CLI `--style` flag accepts raw Kokoro voice names directly (e.g., `--style af_bella`). The voice data file is `data/voices-v1.0.bin` at `main.rs:113`.

### Decision: Both Voices Verified for Timestamps

The `tts_timestamped_raw_audio()` function (`kokoros/src/tts/koko.rs:744`) is voice-agnostic — it takes a `style` string and produces `(Vec<f32>, Vec<WordAlignment>)` regardless of which voice is selected. The `WordAlignment` data comes from the ONNX model's internal forced alignment output, not from voice-specific post-processing.

**No additional verification needed.** If a voice works for audio synthesis (proven — both `af_bella` and `am_michael` are used in production via the HTTP API), it also works for timestamped synthesis via `--timestamps`.

### Cron Script Voice Parameter

The cron script should accept the voice as a parameter:

```bash
VOICE="af_bella"  # or "am_michael" for male
```

The cron caller (Supabase/Edge Function or config file) specifies which voice per story based on the story's `tts_config` row (female/male voice IDs).

---

## 4. Performance: Actual Benchmarks vs Estimates

### Benchmark Data

Bench results on CPX22 (2c/4g, closest analog to CX23 2c/4g) with `model_uint8.onnx`, `instances=1`:

| Config | Wall Time | Audio Duration | RTF |
|--------|-----------|---------------|-----|
| CPX22-uint8-i1 | ~63s | ~240s | ~0.26 |

**Key finding:** A **4-minute audio story takes ~63 seconds** to synthesize. At RTF 0.26×, synthesis is faster than realtime.

For 60 stories × 4 min audio each:
- **Per story:** ~63s wall time + ~3-5s model load overhead (Text mode)
- **Total batch:** `60 × 68s = 4,080s = 68 minutes`

If Yawnly stories are shorter (e.g., 1 min audio = ~16s wall time):
- **Total batch:** `60 × 20s = 1,200s = 20 minutes`

### Revised Estimates

| Story length (audio) | Per story (synth + load) | 60 stories | Feasible? |
|---------------------|------------------------|------------|-----------|
| 30s (very short) | ~11s | ~11 min | ✓ yes |
| 60s (1 min) | ~20s | ~20 min | ✓ yes |
| 3 min | ~50s | ~50 min | ✓ yes |
| 4 min | ~68s | ~68 min | ✓ yes |
| 5 min | ~85s | ~85 min | ✓ marginal |

The original "~6 min" estimate appears to assume very short text (single paragraph). For full bedtime stories, expect **20–90 minutes for a 60-story batch**.

### Mitigation

- **Run cron incrementally** — not all 60 stories every night. Only pre-gen stories that have changed or are new. Query Supabase `stories` table for `audio_url IS NULL`.
- **Batch by schedule** — process only the queue depth each night (e.g., 10-20 stories).
- **Model load overhead** — Text mode loads the model per story (~3-5s). This is only 5-8% of total time for 1+ min stories. Acceptable trade-off for correctness.
- **Upgrade VPS** — Hetzner CX32 (4 vCPU, 8 GB) would reduce wall time further.

### Recommendation

Benchmark on VPS with actual story text before committing to a schedule. Use Text mode. Start with a 10-story pilot to measure actual wall time, then scale batch size to fit the maintenance window.

---

## 5. Disk Space: Temp Files and Cleanup

### Per-Story Disk Usage

| File | Size (5-min story) | Count | Total |
|------|-------------------|-------|-------|
| Input text file | ~2 KB | 1 (for File mode) | 2 KB |
| WAV (24 kHz, mono, f32) | ~5 MB | 1 | 5 MB |
| TSV timestamps | ~2 KB | 1 | 2 KB |
| MP3 (48 kbps) | ~1.8 MB | 1 | 1.8 MB |
| **Per story total** | | | **~6.8 MB** |

### 60-Story Batch

`60 × ~6.8 MB = ~408 MB`

### VPS Disk

CX23 has 40 GB NVMe. Current usage (OS, Docker, Caddy, swap): estimate ~10-12 GB. Available: ~28 GB.

408 MB for a cron batch is negligible. Even with retention of failed files, the disk can hold weeks of data.

### Cleanup Strategy

1. Use a dedicated temp directory: `/tmp/tts-cron/`
2. After successful Supabase upload of each story's MP3 + TSV, delete that story's files immediately.
3. After batch completes, remove the input text file.
4. If a story upload fails, leave its files and log the path. An external monitor (or next cron run) can retry.
5. Add a cron pre-flight check: `if available disk < 5 GB, abort and alert`.

### ffmpeg: MP3 Encoding

The requirements doc specifies converting WAV to MP3 using ffmpeg:
```bash
ffmpeg -i story_42.wav -b:a 48k story_42.mp3
```

**Problem:** The Kokoros Docker image (`debian:sid-slim` base) does not include ffmpeg. The cron container is a one-shot Kokoros container with no ffmpeg binary.

**Options:**

| Option | Pros | Cons |
|--------|------|------|
| **A: Install ffmpeg on VPS host** | Simple; ffmpeg available to all containers | One-time `apt install ffmpeg` on host |
| **B: Use a sidecar container with ffmpeg** | No host changes | More complex orchestration |
| **C: Use Rust MP3 encoder from kokoros-openai** | No external dependency | Not exposed in CLI; requires code change |

**Recommendation: Option A** — install ffmpeg on the VPS host:
```bash
sudo apt update && sudo apt install -y ffmpeg
```

The cron script can then:
1. Run Kokoro container to produce WAV + TSV
2. Run ffmpeg on the host (or in a separate container with ffmpeg) to convert WAV → MP3
3. Upload MP3 to Supabase

**Alternative:** If the host cannot be modified, use a multi-stage approach where the cron script runs the Kokoro container, then runs a lightweight `linuxserver/ffmpeg` container for the conversion step.

### Crash Cleanup (trap EXIT)

If the script crashes mid-batch, orphaned temp files remain. Use `trap`:

```bash
CLEANUP_DIRS=("/tmp/tts-cron")
cleanup() {
  for dir in "${CLEANUP_DIRS[@]}"; do
    if [ -d "$dir" ]; then
      find "$dir" -maxdepth 1 -name 'story_*' -mmin +60 -delete
    fi
  done
}
trap cleanup EXIT
```

This deletes temp files older than 1 hour on any exit (normal or crash). Per `pre-gen-audio-requirements.md:90`.

### Decision: Inline Cleanup Per Story

Delete temp files as each story uploads — not batch cleanup at the end. This minimizes peak disk usage and makes retries easy.

---

## 6. No HTTP API Changes Needed

### Decision: Confirmed

The cron job never touches the HTTP API. It calls the `koko` binary's CLI in `Text` mode directly.

The HTTP API (`koko openai`) does not support `--timestamps` — the `OpenAI` mode ignores the flag. The code at `kokoros-openai/src/lib.rs:618` calls `tts_raw_audio()` which discards timestamps. No changes needed.

---

## 7. Monitoring: Logging and Timing

### Kokoro Stderr and Stdout

The `koko` CLI writes to both streams:

**Stderr** (`eprintln!`):
- `Audio saved to {save_path}` — normal progress
- `Timestamps saved to {tsv_path}` — normal progress
- `Error processing line {i}: {e}` — errors

**Stdout** (`println!`, Text mode only):
- `Time taken: {duration}` — per-invocation timing
- `Words per second: {wps}` — throughput

Both streams must be captured independently. In `File` mode, timing output wraps the entire batch (all 60 stories report one `Time taken` pair), not per line. Per `main.rs:374-377`.

Note: `context.md:75` uses `--lan en` but the CLI default is `--lan en-us` (`main.rs:95`). The spec uses `en-us` (correct for the phonemizer voice-prefix mapping at `kokoros/src/tts/koko.rs:315`). Flag for context.md consistency update.

### Per-Story Timing

Since `Text` mode is used (one invocation per story), each invocation naturally produces its own timing output:

```bash
start_time=$(date +%s)
# ... docker run ...
end_time=$(date +%s)
echo "story_${story_id}: $((end_time - start_time))s" >> /var/log/tts-cron/timing.log
```

The Kokoro CLI also prints `Time taken: {duration}` and `Words per second: {wps}` to stdout in Text mode. Capture both streams.

### Recommendation

Log per-story timing for every run. This data is essential for capacity planning and detecting regressions.

### Log Structure

```
/var/log/tts-cron/
├── run-20260504-030000.log      # Full stderr/stdout from Kokoro
├── timing-20260504-030000.log   # Per-story timing (one line per story)
└── upload-20260504-030000.log   # Supabase upload results
```

### Alerting

- If any story fails to synthesize → log error, continue, send summary.
- If Supabase upload fails after 3 retries → leave files, log path, alert.
- If disk < 5 GB → abort, alert.

---

## 8. Future: Hindi Support

### Changes Required

When Hindi (`hi`) is added:

| Area | Change | Details |
|------|--------|---------|
| **Voice map** | New voice names | Hindi voices per `context.md:53`: `hf_alpha`, `hf_beta`, `hm_omega`, `hm_psi` |
| **Language code** | `--lan` flag | Change from `--lan en-us` to `--lan hi` |
| **Voice file** | `data/voices-v1.0.bin` | Must contain Hindi voice embeddings (verify in the model distribution) |
| **Model** | ONNX checkpoint | Same model supports multiple languages. No model change needed. |

### Recommended Script Structure

Parameterize the cron script from day one:

```bash
#!/bin/bash
LANGUAGE="${1:-en-us}"
VOICE="${2:-af_bella}"
```

This way, Hindi support is just a different invocation:

```bash
tts-cron.sh hi hf_alpha
```

No code changes needed when Hindi is enabled. The cron script does not need language-specific logic.

---

## 9. Security: Supabase Service Role Key

### Current Security Posture

Per `context.md:124-131`:
- Cloudflare WAF blocks requests without `X-Yawnly-TTS-Key`
- Supabase secret `HETZNER_TTS_PROXY_KEY` is set on the project
- Mobile clients never receive the Kokoros header secret

### Is the Service Role Key on the VPS?

**This is an open question.** If the Supabase service role key is stored on the VPS (for the Supabase Edge Function to use), the cron job script will need access to it for uploading to Supabase Storage. If it's not on the VPS, it needs to be added.

### Recommendation

1. **Store the service role key in an environment file** on the VPS at `/opt/kokoros/.env` (mode 600, owned by root).
2. **Do NOT hardcode the key in the cron script.**
3. **Use the `supabase` CLI or `curl` with Bearer auth** to upload files to Supabase Storage.
4. **Audit access:** only root and the cron user should read the `.env` file.
5. **No additional hardening needed** beyond what's described in `DEPLOYMENT_SPEC.md` section 8 (UFW, SSH hardening). The VPS is already behind Cloudflare WAF for public ingress, and the cron job is an internal-only process.

### Output: MP3 Upload + Timing Metadata + audio_url

The cron job writes three outputs per story:

**1. MP3 to Supabase Storage:**

Per `pre-gen-audio-requirements.md:26`, path convention is `stories/{story_id}/audio.mp3`. Bucket name `tts-audio` is assumed (flagged as open question below):

```bash
curl -X POST "https://ynjiskzaxsqdovmugiwo.supabase.co/storage/v1/object/tts-audio/stories/${story_id}/audio.mp3" \
  -H "Authorization: Bearer ${SERVICE_ROLE_KEY}" \
  -H "Content-Type: audio/mpeg" \
  --data-binary "@${story_id}.mp3"
```

**2. Paragraph timing to `stories.audio_timing` (JSONB):**

Per `pre-gen-audio-requirements.md:27-34, 65`:

```bash
curl -X PATCH "https://ynjiskzaxsqdovmugiwo.supabase.co/rest/v1/stories?id=eq.${story_id}" \
  -H "Authorization: Bearer ${SERVICE_ROLE_KEY}" \
  -H "Content-Type: application/json" \
  -H "Prefer: return=minimal" \
  -d "{\"audio_timing\": ${timing_json}}"
```

**3. Audio URL to `stories.audio_url`:**

The client fetches audio via `audio_url`. Without this, the client has no way to play the pre-generated MP3 even if timing exists.

```bash
curl -X PATCH "https://ynjiskzaxsqdovmugiwo.supabase.co/rest/v1/stories?id=eq.${story_id}" \
  -H "Authorization: Bearer ${SERVICE_ROLE_KEY}" \
  -H "Content-Type: application/json" \
  -H "Prefer: return=minimal" \
  -d "{\"audio_url\": \"https://ynjiskzaxsqdovmugiwo.supabase.co/storage/v1/object/public/tts-audio/stories/${story_id}/audio.mp3\"}"
```

**4. Atomicity guarantee:**

Per `pre-gen-audio-requirements.md:35`: all three outputs must be written atomically. Strategy:
- Upload MP3 to Storage first.
- If MP3 upload succeeds, UPDATE both `audio_url` and `audio_timing` in a single PATCH call.
- If the PATCH fails, delete the orphaned MP3 from Storage.
- If the MP3 upload fails, nothing is visible.
- The race window is benign: if a client opens the story mid-write, `audio_timing IS NULL` triggers the Edge TTS fallback (existing behavior, per requirements line 99).

### Open Questions (Storage + Timing)

- What Supabase bucket name? `tts-audio` assumed — confirm per Yawnly's storage schema.
- What object path convention? `stories/{story_id}/audio.mp3` and `stories.audio_timing` — confirm with Yawnly data model. Note: the task description uses `tts-audio/stories/${story_id}.mp3`; requirements use `stories/{story_id}/audio.mp3`.
- Atomicity approach: options are (a) Supabase RPC call, (b) reverse write order, (c) staging state. **Recommend (c)** — set `audio_status = 'processing'` before writing, then `'ready'` after both succeed. This is simplest, needs no RPC function, and the client can be taught to ignore `processing` state (or fall back to Edge TTS).

---

# Summary of Decisions

| # | Area | Decision | Confidence |
|---|------|----------|-----------|
| 1 | Docker image | `kokoros:quantized-schema-fix-amd64` supports `--timestamps`; HF timestamped model required | High (verified via Docker test) |
| 2 | CLI invocation | `docker run` separate container with shared volumes | High |
| 3 | CLI mode | `Text` mode (one invocation per story) | High; `File` mode incompatible with multi-paragraph stories |
| 4 | Voices | Both `af_bella` and `am_michael` work with timestamps | High (voice-agnostic ONNX alignment) |
| 5 | Performance | ~20–90 min for 60 stories (1–4 min audio each) | High (bench data: RTF ~0.26× on CPX22) |
| 6 | Disk cleanup | Per-story inline cleanup after successful upload | High |
| 7 | HTTP API | No changes needed | High |
| 8 | Monitoring | Capture stderr, per-story timing, upload logs | High |
| 9 | Future Hindi | Parameterize language + voice; no code changes | High |
| 10 | Security | Service role key in `/opt/kokoros/.env`; no extra hardening | Medium (confirm key location) |
| 11 | Container memory | `--memory="1.5g"` minimum | High (500 MB model + buffers) |
| 12 | Paragraph aggregation | Sequential word matching, NOT character offsets | High (eSpeak coarticulation) |
| 13 | audio_url | UPDATE `audio_url` alongside `audio_timing` | High |
| 14 | ffmpeg | Install on VPS host (`apt install ffmpeg`) | Medium (verify host permissions) |

# Open Questions

1. **Performance validation:** Run a 10-story pilot on VPS to confirm actual wall time with the HF timestamped model. Story length distribution determines batch size.
2. **Service role key location:** Is `SUPABASE_SERVICE_ROLE_KEY` already on the VPS? If not, where to store it?
3. **Supabase bucket name and path convention:** What bucket and path for MP3 uploads? Requirements say `stories/{story_id}/audio.mp3`; task description uses `stories/${story_id}.mp3`. Confirm.
4. **Race condition (benign):** What if a user opens a story while the cron job is mid-synthesis for that story? `audio_timing IS NULL` → client falls back to Edge TTS → cron completes. No data corruption, but the user may briefly hear Edge TTS before the pre-gen audio is ready. Acceptable per `pre-gen-audio-requirements.md:105`.
5. **Cron schedule:** What time window? What frequency (nightly? weekly? on-demand via webhook?)
6. **Batch size:** Should the cron only process stories without `audio_url`, or reprocess all active stories?
7. **Custom image fork state:** Confirm the fork branch/tag that `kokoros:quantized-schema-fix-amd64` was built from.
8. **ffmpeg host install:** Can we run `apt install ffmpeg` on the VPS host, or are there restrictions?
9. **audio_status column:** Does the `stories` table have an `audio_status` column for staging? If not, a migration is needed.

# Output Required

When this spec converts to `implement-local` and the executor completes, return:

1. **Cron shell script** at `/opt/kokoros/tts-cron.sh` — full pipeline: fetch stories, synthesize with `--timestamps --speed 1.0`, aggregate to paragraphs using sequential word matching, encode MP3, upload to Supabase, cleanup.
2. **HF timestamped model** downloaded to `/opt/kokoros/checkpoints/kokoro-v1.0.onnx`.
3. **Docker invocation** for the `kokoros:quantized-schema-fix-amd64` image in `Text` mode with shared volumes and resource limits (`--memory="1.5g"`).
4. **Systemd timer or crontab entry** — schedule, lockfile, logging.
5. **Log directory structure** at `/var/log/tts-cron/` and rotation policy.
6. **`.env` file specification** — required env vars (`SERVICE_ROLE_KEY`, `SUPABASE_URL`, etc.) with mode 600.
7. **Monitoring summary** — first-run benchmark results (wall time, per-story timing, peak RAM/disk).
8. **Verification results** — commands run and their output, including TSV sample with real word alignments.

# Implementation Plan

Implementation artifacts are in `/Users/eloelo/Downloads/MyWiki/projects/kokoros/impl-vps-cron/`.

## Files Created

| File | Purpose |
|------|---------|
| `tts-cron.sh` | Main pipeline script: fetch, synthesize, aggregate, encode, upload, cleanup |
| `tts-cron.service` | Systemd oneshot service definition |
| `tts-cron.timer` | Systemd timer (daily at 3 AM UTC) |
| `.env.example` | Environment variables template |
| `DEPLOY.md` | VPS deployment step-by-step guide |
| `verify.sh` | Post-deployment verification script |

## Deployment Checklist

- [ ] Download HF timestamped `model_uint8.onnx` to VPS
- [ ] Verify `--timestamps` produces real TSV data
- [ ] Install ffmpeg on VPS host
- [ ] Copy scripts to `/opt/kokoros/`
- [ ] Configure `.env` with real Supabase credentials
- [ ] Install systemd timer
- [ ] Run manual test (`systemctl start tts-cron.service`)
- [ ] Verify logs and upload results

## Post-Deployment Verification

Run the verification script:
```bash
sudo /opt/kokoros/verify.sh
```

Expected output: all 6 checks pass (Docker, model, voice data, ffmpeg, TSV synthesis, MP3 encoding).

## Monitoring

- Timer status: `sudo systemctl list-timers tts-cron.timer`
- Service logs: `sudo journalctl -u tts-cron.service -f`
- Script logs: `/var/log/tts-cron/run-*.log`
- Timing logs: `/var/log/tts-cron/timing-*.log`
- Upload logs: `/var/log/tts-cron/upload-*.log`

## Rollback

To disable:
```bash
sudo systemctl stop tts-cron.timer
sudo systemctl disable tts-cron.timer
```

To re-enable:
```bash
sudo systemctl enable tts-cron.timer
sudo systemctl start tts-cron.timer
```
