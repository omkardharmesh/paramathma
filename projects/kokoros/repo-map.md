---
title: Kokoros Repo Map
summary: High-level map of the Kokoros repo files relevant to deployment and integration.
type: repo-map
status: canonical
updated: 2026-05-03
---

# Kokoros — Repo Map

## Important Paths

- `README.md` — upstream Kokoros overview, build instructions, CLI usage, and OpenAI-compatible server examples.
- `deployment-spec/DEPLOYMENT_SPEC.md` — deployment architecture and runbook for Hetzner/Coolify-era plan; still useful for image, command, port, security, and verification facts.
- `deployment-spec/LOCAL_SETUP.md` — local Docker smoke-test guide.
- `deployment-spec/docker-compose.yml` — tracked Compose source for Kokoros service.
- `testRun/LOCAL_SETUP_RUNBOOK.md` — local build/test notes and voice benchmark observations.
- `koko/` — Rust application source.
- `target/` — generated Rust build output; do not bulk-read or commit.
- `checkpoints/` and `data/` — model/voice artifact locations; avoid storing large artifacts in MyWiki.

## Current Deployment Layout

The live VPS uses `/opt/kokoros/docker-compose.yml`, based on the deployment spec but adjusted for Caddy:

```yaml
services:
  kokoros:
    image: ghcr.io/lucasjinreal/kokoros:main
    command: ["--instances", "2", "openai"]
    restart: unless-stopped
    mem_limit: 3g
    cpus: "1.7"
    environment:
      - RUST_LOG=info
    ports:
      - "127.0.0.1:3000:3000"
```

## Integration Boundary

Yawnly integration should happen through Supabase Edge Functions. The app calls Supabase; Supabase calls Kokoros with the Cloudflare header secret.
