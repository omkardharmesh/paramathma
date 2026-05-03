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
