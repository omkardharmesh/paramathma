---
title: Yawnly Marketing Project Context
summary: Post-launch marketing research, launch plans, creative assets, and store listing strategy for Yawnly.
type: project-context
status: draft
updated: 2026-05-14
---

# Yawnly Marketing — Project Context

## Product Role

- Yawnly Marketing is the **marketing brain** for the Yawnly bedtime story app.
- Contains post-launch research (10 X threads analyzed), a 30-day India launch plan, creative asset bank, store listing strategy, completion audit, and a generic solo-dev playbook.
- The playbook (`guidelines/marketing/solo-dev-app-launch-playbook.md`) is app-agnostic and reusable for any future app — Yawnly Marketing contains the Yawnly-specific execution of that playbook.

## Repo

- Path: `/Users/eloelo/Downloads/Marketing Research`
- Source of truth for Yawnly marketing docs. Product decisions (pricing, architecture, locked decisions) live in `MyWiki/projects/yawnly/`.

## Directory Structure

```
Marketing Research/
├── README.md                                    — Entry point, pointers to active files
├── yawnly-x-links-research.md                   — 10 X threads analyzed → Yawnly actions (288 lines)
├── yawnly-creative-assets.md                    — Hook bank, scripts, paid ad ideas, video prompts (702 lines)
├── yawnly-creative-test-matrix.md               — 25+ concrete creative concepts with kill rules (124 lines)
├── yawnly-india-launch-marketing-plan.md        — 30-day India launch plan (507 lines)
├── yawnly-store-listing-plan.md                 — Play Store title/subtitle/screenshots/keywords (324 lines)
├── yawnly-marketing-completion-audit.md         — Source-to-plan traceability map (245 lines)
├── yawnly-marketing-research-supplement.md      — Additional research depth (478 lines)
├── yawnly-marketing-plan-agent-prompt.md        — The prompt another agent used to generate the plans (71 lines)
├── solo-dev-app-launch-playbook.md              — Original copy (copied to MyWiki guidelines)
├── solo-dev-app-launch-playbook.html            — HTML version of the generic playbook
├── yawnly-marketing-plan.html                   — HTML version of the Yawnly plan
├── techmotok-prototype/                         — Internal Growth OS tool (React/TS/Vite)
│   ├── src/App.tsx                              — Portfolio, Intake, Prioritization, Growth Pack, Metrics screens
│   ├── src/data.ts                              — Yawnly-seeded country strategies & thresholds
│   └── dist/index.html                          — Built version
├── archive/                                     — Stale AppTok/TechmoTok specs (do not treat as current)
│   └── 2026-05-13-md-cleanup/
├── .spec-workflow/                              — Spec workflow state
├── .claude/                                      — Claude session artifacts
└── skills/                                       — Skills directory
```

## Key Files (Always Read)

| File | What's in it | When to read |
|------|-------------|-------------|
| `yawnly-x-links-research.md` | Core demand signals, non-negotiables | Any marketing work |
| `yawnly-creative-assets.md` | Hook bank, scripts, prompt corpus | Creative production |
| `yawnly-india-launch-marketing-plan.md` | 30-day sprint, channel strategy, budget | Launch execution |
| `yawnly-store-listing-plan.md` | Screenshot sequence, keywords, locales | Store updates |
| `yawnly-marketing-completion-audit.md` | Source-to-plan traceability | Audit/verification |

## Hard Constraints

- **README.md says**: `archive/2026-05-13-md-cleanup/` contains stale AppTok/TechmoTok specs. Do not treat as current source of truth.
- **MyWiki is the source of truth** for pricing (D009), architecture, and locked product decisions — not the Marketing Research directory.
- **Do not modify** `yawnly-marketing-plan-agent-prompt.md` — it's the historical prompt that generated the plans, kept for traceability.
- **Do not delete** `yawnly-marketing-plan.html` or `solo-dev-app-launch-playbook.html` — they're automated exports.
- **No TikTok India plan** — TikTok is not a domestic India launch channel (documented in `yawnly-x-links-research.md`).
- **No paid ad spend** until screenshots, onboarding, and post-purchase path are fixed (non-negotiable from X research).

## External Services

- **Meta** (FB + IG) — primary paid channel for Yawnly India
- **YouTube Shorts** — secondary paid channel
- **Instagram Reels** — primary organic channel
- **Moj/ShareChat/Josh** — tier-2/3 India organic (if relevant)
- **Google Play Console** — store listing, A/B tests, install reports
- **RevenueCat** — subscription data referenced in research benchmarks
- **Airship** — push notification benchmarks referenced in research

## TechmoTok Prototype

- **Stack:** React 19 + TypeScript + Vite
- **Build:** `cd techmotok-prototype && npm run build` (produces `dist/index.html`)
- **Dev:** `npm run dev`
- **Purpose:** Internal Growth OS — country portfolio scoring + Intake → Prioritization → Growth Pack → Metrics pipeline. Seeded with Yawnly data. Has a "Second app slot" placeholder for future apps.
- **Not ready for production** — localStorage-based persistence only, no backend.

## Current Release Status

See `current-state.md`.
