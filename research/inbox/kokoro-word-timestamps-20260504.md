---
title: Kokoro Word-Level Timestamps Are Natively Supported
topic: kokoro-tts-timing
status: inbox
created: 2026-05-04
projects: [kokoros, yawnly]
tags: [tts, timestamps, word-alignment, highlighting]
---

# Research: Kokoro Word-Level Timestamps Are Natively Supported

## Question

Does Kokoro produce word-level timing data during synthesis, so per-paragraph audio timing metadata can be derived without paragraph-split synthesis, forced alignment, or WPM estimation?

## Keep

- **`tts_timestamped_raw_audio()` exists** in `kokoros/src/tts/koko.rs:744`. Returns `Option<(Vec<f32>, Vec<WordAlignment>)>` — audio samples + word alignments.
- **`WordAlignment` struct** (`koko.rs:27`): `{ word: String, start_sec: f32, end_sec: f32 }`. Produced during ONNX inference, not post-processed.
- **CLI `--timestamps` flag** (`koko/src/main.rs:148`): when set, outputs a TSV sidecar (`word\tstart_sec\tend_sec`) alongside the WAV file. Works with `Text` and `File` modes.
- **NOT wired to HTTP API**: the OpenAI-compatible server (`handle_tts` in `kokoros-openai/src/lib.rs:617`) calls `tts_raw_audio()`, which discards timestamps (`.unwrap().0`). The data is produced but dropped.
- **Cron job path**: call Kokoro CLI directly on the VPS with `--timestamps`, parse TSV output, aggregate word timestamps to paragraph boundaries by matching character offsets to paragraph text ranges. One Kokoro call per story, zero audio splicing, 100% accurate.

## Decision

Paragraph-level timing metadata for text highlighting should be derived from Kokoro's native word-level timestamps, accessed via the CLI `--timestamps` flag on the VPS, not through the HTTP API. No paragraph-split synthesis, forced alignment, or estimation is needed.

## Rejected

- Adding timestamps to the HTTP API (requires Rust changes in the Kokoros fork, adds API surface complexity)
- Paragraph-split synthesis + MP3 concatenation (audio artifacts, timing discontinuities)
- Client-side char-proportional allocation (accuracy loss)
- Forced alignment via aeneas/gentle (extra dependency, adds processing time when-native timestamps exist)

## Applies To

- Yawnly pre-gen audio cron job
- Kokoros deployment on Hetzner CX23
- Future: any downstream consumer that needs per-word or per-paragraph timing from Kokoro

## Next Action

- Update `projects/kokoros/context.md` — add Timestamps section
- Update `projects/kokoros/current-state.md` — add to Working
- Create task packet for cron job implementation (Kokoros CLI + TSV parsing + paragraph aggregation + Supabase storage upload)

## Sources

- `/Users/eloelo/Downloads/Kokoros/kokoros/src/tts/koko.rs` — `WordAlignment` struct (line 27), `tts_timestamped_raw_audio` (line 744), `process_internal` with `ExecutionMode::Batch` and `Aligned` variant
- `/Users/eloelo/Downloads/Kokoros/koko/src/main.rs` — `--timestamps` CLI flag (line 148), TSV output (line 302-306)
- `/Users/eloelo/Downloads/Kokoros/kokoros-openai/src/lib.rs` — `handle_tts` calls `tts_raw_audio()` at line 618, discards timestamps
