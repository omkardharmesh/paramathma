---
title: Kokoros Current State
summary: Live deployment status, verification, and remaining integration work for the Kokoros TTS service.
type: current-state
status: canonical
updated: 2026-05-03
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
- Kokoros starts with `--instances 2 openai`.
- Kokoros is bound to `127.0.0.1:3000`, not public port `3000`.
- Caddy proxies `https://tts-hetzner.yawnly.org` to Kokoros.
- Cloudflare proxy is enabled.
- Cloudflare WAF blocks requests without `X-Yawnly-TTS-Key`.

## Verified

- `GET /` over protected HTTPS returns `OK` when the header is present.
- `GET /v1/models` returns OpenAI-style model IDs.
- `POST /v1/audio/speech` generated a valid MP3 through public HTTPS.
- Requests without the header return Cloudflare `403`.

## Pending

- Store the raw header value in Supabase as `HETZNER_TTS_PROXY_KEY`.
- Update the relevant Supabase Edge Function to call `https://tts-hetzner.yawnly.org/v1/audio/speech` with `X-Yawnly-TTS-Key`.
- Verify Yawnly app flow through Supabase, not direct mobile-to-Kokoros calls.
- Decide whether to pin the Docker image digest for production stability.
- Add off-server backup for `/opt/kokoros/docker-compose.yml` and any future operational notes if they drift from git.

## Risks

- `ghcr.io/lucasjinreal/kokoros:main` is mutable.
- The service depends on Cloudflare WAF for access control.
- CX23 is suitable for light use; sustained high concurrency may need a larger server.
- Kokoros has no built-in application-level authentication.
