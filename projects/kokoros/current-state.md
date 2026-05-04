---
title: Kokoros Current State
summary: Live deployment status, verification, and remaining integration work for the Kokoros TTS service. Yawnly Edge Function deployed 2026-05-03; en traffic routed via launch-test bypasses.
type: current-state
status: canonical
updated: 2026-05-04
---

# Kokoros — Current State

## Working

- Hetzner CX23 is provisioned as `kokoros-tts-prod-1`.
- Ubuntu 24.04.3 LTS is updated.
- 4 GiB swap is enabled.
- UFW allows only SSH, HTTP, and HTTPS inbound.
- Docker and Docker Compose are installed and active.
- Caddy is installed and active.
- Kokoros runs from `/opt/kokoros/docker-compose.yml`.
- Kokoros now uses local image `kokoros:quantized-schema-fix-amd64` with the **community uint8 ONNX model** (`onnx/model_uint8.onnx`, ~170 MB) mounted at `/app/checkpoints/kokoro-v1.0.onnx`. The earlier `model_quantized.onnx` (int8 dynamic) was deleted from the VPS on 2026-05-03 because RTF was unacceptable; if a rollback is ever needed, re-download from `onnx-community/Kokoro-82M-v1.0-ONNX`.
- Kokoros starts with `--instances 1 openai` (changed from 2 on 2026-05-03 — see Performance Notes below).
- Kokoros is bound to `127.0.0.1:3000`, not public port `3000`.
- Caddy proxies `https://tts-hetzner.yawnly.org` to Kokoros.
- Cloudflare proxy is enabled.
- Cloudflare WAF blocks requests without `X-Yawnly-TTS-Key`.

## Verified

- `GET /` over protected HTTPS returns `OK` when the header is present.
- `GET /v1/models` returns OpenAI-style model IDs.
- `POST /v1/audio/speech` generated a valid MP3 through public HTTPS.
- Requests without the header return Cloudflare `403`.
- **Word-level timestamps are natively supported** (discovered 2026-05-04). The `--timestamps` CLI flag outputs a TSV sidecar (`word\tstart_sec\tend_sec`) via `tts_timestamped_raw_audio()`. The HTTP API (`/v1/audio/speech`) does NOT expose this — it calls `tts_raw_audio()` which discards the alignments. For pre-gen audio, the cron job should call the CLI directly, parse the TSV, and aggregate word timestamps to paragraph boundaries.

- uint8 deployment verified locally on the VPS: `GET /`, `GET /v1/models`, MP3 generation, and PCM streaming all return `HTTP 200`.
- Patched runtime path detects the schema (`input_ids -> waveform`) for both `model_quantized.onnx` and `model_uint8.onnx` with no code change.
- End-to-end streaming through Cloudflare with uint8 verified 2026-05-03: ttfb 0.73s, total 8.78s for 11.30s audio (**RTF 0.78x — faster than realtime**). Direct VPS loopback uint8 multi-sentence: ttfb 0.08s, total 8.56s for 11.30s audio (RTF 0.76x). Local M-series Docker uint8: ttfb 0.03s, total 1.32s for 11.30s audio (RTF 0.12x, ~8x realtime).

## Performance Notes (2026-05-03)

- **uint8 vs int8-dynamic on x86:** swapping `model_quantized.onnx` (int8 dynamic, 92 MB) for `model_uint8.onnx` (uint8 static, 170 MB) gave ~2.4x throughput on the VPS (multi-sentence stream total 21.0s -> 8.56s, RTF 1.86x -> 0.76x). x86 CPUs lack the kernel optimizations that make int8 dynamic competitive on Apple Silicon; uint8 static fits standard x86 GEMM kernels much better. The 170 MB model also doubles RAM cost (still well within the 3 GB container limit on CX23).
- **`--instances 1`:** CX23 has 2 vCPU. Running `--instances 2` halved per-request CPU and pushed RTF to ~6.3x slower than realtime. With `--instances 1`, both vCPU feed one model.
- **Cloudflare buffering hypothesis was wrong:** the long ttfb under `instances=2` was generation-bound. CF overhead is now ~600 ms ttfb above the VPS loopback; CF is not buffering chunked PCM streams.
- **Chunked streaming:** Kokoros splits text on punctuation. Each chunk generates fully before its bytes flush. Single-sentence inputs therefore have ttfb ≈ first-chunk gen time. Multi-sentence inputs show real streaming benefit (first audio in ~80 ms direct / ~730 ms via CF).
- Apple Silicon (M-series) is still ~6x faster than CX23 amd64 with the same uint8 model. Acceptable for the current Yawnly traffic profile.
- Future levers if RTF needs further improvement: try fp32 base model, set `OMP_NUM_THREADS=2`, upgrade to Hetzner CCX (dedicated AMD EPYC vCPU), or compile Kokoros with the OpenVINO ORT execution provider.

## Pending

- Verify Yawnly app flow end-to-end against the deployed Edge Function (was deployed 2026-05-03; live test pending).
- Restore the strict server-side provider gate in `generate-audio-kokoros` once `tts_config.provider` is flipped to `kokoros` for `en` (the gate is currently bypassed — see "Launch Test Bypasses" below).
- Remove the client-side `en → KOKOROS` hardcode in `TtsConfigClient.route()` once the DB row is flipped.
- Flip the `tts_config` row to `provider='kokoros'` for `en` once smoke testing confirms Kokoros quality + reliability is acceptable.
- Decide whether to pin the Docker image digest for production stability.
- Add off-server backup for `/opt/kokoros/docker-compose.yml` and any future operational notes if they drift from git.
- Migrate hardcoded voice ids in `generate-audio-kokoros` to `tts_config.kokoros_male_voice_id` / `kokoros_female_voice_id` columns (mirrors the Chirp pattern; SQL flip changes voices without a redeploy).
- Consolidate `generate-audio-chirp` and `generate-audio-kokoros` into a single configurable `generate-audio` Edge Function that dispatches by `tts_config.provider`.

## Yawnly Integration Status (2026-05-03)

- Yawnly client landed: `KokorosTtsClient` + `KokorosTtsPipelinePlayer` (paragraph-batched streaming, mirrors Chirp). Modal references stripped from the routing layer; Modal client/pipeline files left as dead code per request.
- Edge Function: `supabase/functions/generate-audio-kokoros/index.ts` mirrors `generate-audio-chirp` shape — bearer auth, ownership check, hardcoded en voice map (`af_bella` female, `am_michael` male). **Deployed 2026-05-03** to project `ynjiskzaxsqdovmugiwo`.
- Supabase secret: `HETZNER_TTS_PROXY_KEY` set on the project (used by the function to attach the Cloudflare WAF header).
- Languages enabled: `en` only initially. Other locales fall back to Edge TTS at the `TtsEngine` dispatch.
- Voice shortlist source: `testRun/LOCAL_SETUP_RUNBOOK.md` benchmarks (`af_bella`, `am_michael` verified full-length; `bm_lewis` excluded — truncated output).

## Launch Test Bypasses (TEMP — remove before GA)

Two temporary overrides allow `en` traffic to flow to Kokoros without flipping the production `tts_config` row (which would affect all existing users):

1. **Client (`TtsConfigClient.route()`)** — for `language == "en"`, the route returned by `tts-route` is rewritten to `provider = KOKOROS` after fetching. Marked with a `TEMP (2026-05-03)` comment.
2. **Edge Function (`generate-audio-kokoros/index.ts`)** — the strict `tts_config.provider == 'kokoros'` enable gate is commented out. Reference code retained inline so the gate can be restored verbatim. Marked with a `TEMP (2026-05-03)` comment.

Both must be reverted before opening Kokoros to general availability — the gate is the single SQL kill switch and the client override hides config drift.

## Risks

- `ghcr.io/lucasjinreal/kokoros:main` is mutable.
- The service depends on Cloudflare WAF for access control.
- CX23 is suitable for light use; sustained high concurrency may need a larger server.
- Kokoros has no built-in application-level authentication.
