---
title: Kokoros Project Context
summary: Self-hosted Kokoro TTS service on Hetzner, protected by Cloudflare and consumed through Supabase.
type: project-context
status: canonical
updated: 2026-05-03
---

# Kokoros — Project Context

## Product Role

- Kokoros provides self-hosted text-to-speech for Yawnly.
- Mobile clients must not call Kokoros directly. Yawnly should call Supabase Edge Functions, and Edge Functions call Kokoros.
- The service exposes an OpenAI-compatible TTS API.

## Repo

- Path: `/Users/eloelo/Downloads/Kokoros`
- Fork remote: `https://github.com/omkardharmesh/kokoros-tts.git`
- Upstream remote: `https://github.com/lucasjinreal/Kokoros.git`
- Deployment source: `deployment-spec/`

## Runtime Stack

- **Language/runtime:** Rust binary packaged in upstream Docker image.
- **Image:** `ghcr.io/lucasjinreal/kokoros:main`
- **Server mode:** `koko openai`
- **Instances:** `--instances 2`
- **Container port:** `3000`
- **Host binding:** `127.0.0.1:3000:3000` only. Do not expose port `3000` publicly.
- **Reverse proxy:** Caddy.
- **Cloud edge:** Cloudflare proxied DNS + WAF header rule.
- **VPS:** Hetzner CX23, Ubuntu 24.04.3 LTS, IPv4 `46.225.151.61`.
- **Hostname:** `https://tts-hetzner.yawnly.org`

## Live Request Path

```text
Yawnly app
  -> Supabase Edge Function
  -> Cloudflare WAF
  -> Caddy on Hetzner
  -> Kokoros Docker container on 127.0.0.1:3000
```

## API Surface

- `GET /` returns `OK`.
- `GET /v1/models` returns OpenAI-style model list.
- `POST /v1/audio/speech` generates audio.
- Known model IDs include `tts-1`, `tts-1-hd`, `kokoro`, and `gpt-4o-mini-tts`.
- Known voice examples include `af_bella`, `af_sky`, Hindi voices `hf_alpha`, `hf_beta`, `hm_omega`, and `hm_psi`.

## Streaming Usage

Kokoros supports OpenAI-compatible streaming when `"stream": true` is included in the JSON body. The response body is **raw 24 kHz, mono, signed 16-bit little-endian PCM**, sent chunked at sentence boundaries. There is no MP3 streaming variant — streaming responses are always PCM. The non-streaming path (no `stream` flag) returns MP3 by default.

### Request body

```json
{
  "model": "tts-1",
  "voice": "af_bella",
  "input": "Multi-sentence text. The first chunk arrives within ~700ms via Cloudflare. Each chunk is one sentence.",
  "stream": true
}
```

### Response stream contract

- Content-Type: `audio/pcm`
- Transfer-Encoding: chunked
- Sample rate: 24000 Hz
- Channels: 1 (mono)
- Sample format: s16le (signed 16-bit little-endian)
- Bytes per second of audio: `24000 * 2 = 48000`
- Total audio duration in seconds = `byte_count / 48000`

Every chunk is one sentence (Kokoros splits text on punctuation). The first chunk's bytes flush as soon as the first sentence has been generated. Audio bytes can be played as they arrive — there is no header to wait for.

### Curl example (raw PCM to file)

```shell
curl -sS -N -X POST https://tts-hetzner.yawnly.org/v1/audio/speech \
  -H "Content-Type: application/json" \
  -H "X-Yawnly-TTS-Key: $HETZNER_TTS_PROXY_KEY" \
  -d '{"model":"tts-1","voice":"af_bella","input":"Streaming demo.","stream":true}' \
  -o stream.pcm \
  -w 'ttfb=%{time_starttransfer}s total=%{time_total}s size=%{size_download}\n'
```

`-N` disables curl's output buffering so you see streaming bytes land. The header secret is not stored in MyWiki — load it from Supabase secrets / shell env.

### Live playback while streaming (ffplay)

```shell
curl -sS -N -X POST https://tts-hetzner.yawnly.org/v1/audio/speech \
  -H "Content-Type: application/json" \
  -H "X-Yawnly-TTS-Key: $HETZNER_TTS_PROXY_KEY" \
  -d '{"model":"tts-1","voice":"af_bella","input":"Live streaming demo.","stream":true}' \
  | ffplay -f s16le -ar 24000 -ch_layout mono -nodisp -autoexit -loglevel warning pipe:0
```

If `ffplay` rejects `-ch_layout`, fall back to the non-streaming pattern: pipe to a `.pcm` file first, then play with the same flags.

### Mobile / app integration notes

- The mobile client must not call `tts-hetzner.yawnly.org` directly. Yawnly hits a Supabase Edge Function which attaches the `X-Yawnly-TTS-Key` header before forwarding to Kokoros.
- For full-length stories, prefer streaming PCM end-to-end so the user starts hearing audio in <1 second. Buffer ~250 ms of PCM (~12 KB) before starting playback to absorb network jitter.
- For short prompts (single sentence) streaming offers little benefit — the chunk generation time dominates. Non-streaming MP3 is fine and gives a smaller payload.
- PCM is uncompressed: ~48 KB/s. For very long stories or metered networks, the Edge Function can transcode the streamed PCM to MP3 server-side before sending to the device.

### Performance reference (uint8 model, CX23, `--instances 1`, 2026-05-03)

| call | ttfb | total | audio | RTF |
|------|------|-------|-------|-----|
| short stream (single sentence) | 0.08s | 1.99s | 2.52s | 0.79x |
| multi-sentence stream (5 sentences) | 0.08s | 8.56s | 11.30s | 0.76x |
| multi-sentence via Cloudflare | 0.73s | 8.78s | 11.30s | 0.78x |

RTF below 1.0 means generation is faster than playback — streaming can run live without buffer underruns.

## Security Contract

- Cloudflare blocks requests that do not include the required secret header.
- Required header name: `X-Yawnly-TTS-Key`.
- Supabase secret name: `HETZNER_TTS_PROXY_KEY`.
- Do not store the raw header value in MyWiki, app code, git, or chat.
- Mobile clients must never receive the Kokoros header secret.

## Operational Commands

On the VPS:

```shell
cd /opt/kokoros
docker compose ps
docker compose logs --tail=100 kokoros
docker compose pull
docker compose up -d
docker stats
systemctl status caddy --no-pager
ufw status verbose
```

## Hard Constraints

- Do not publish Kokoros directly with `0.0.0.0:3000:3000`.
- Do not remove Cloudflare WAF protection unless replacing it with an equivalent auth layer.
- Do not put raw secrets into docs or source control.
- Do not let the mobile app call Kokoros directly.
- Treat `ghcr.io/lucasjinreal/kokoros:main` as mutable. Pin a digest or controlled fork before relying on this for production stability.

## Current Release Status

See `current-state.md`.
