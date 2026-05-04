---
title: Yawnly Pricing & TTS Economics
summary: Two-tier pricing (₹33/₹99) with 3-day trial, add-on packs, and self-hosted Kokoros TTS on CAX21 launch box. D009 locked.
type: research
status: canonical
updated: 2026-05-04
---

# Yawnly — Pricing & TTS Economics

Captured 2026-05-04 from a benchmarking + pricing review session. Companion to `projects/kokoros/current-state.md` (server perf) and `decisions.md` (product strategy).

Updated with bench results from ARM64 sweep on M5 Pro Docker (2026-05-03) and revised pricing model (₹99/mo, auto-gen flow).

## Pricing Model (D009 — locked 2026-05-04)

Two subscription tiers + trial + add-on packs. No permanent free tier.

### Trial (not a tier)
- 3-day trial, 10-15 personalised stories, 3 audio teaser credits
- No credit card, hard paywall after

### ₹33/month — Text + Audio Teaser
- Unlimited text stories (soft limit 10/day)
- 10 audio credits/month (conversion lever, not revenue)
- TTS cost: 10 × ₹0.082 = ₹0.82/month

### ₹99/month — Full Audio Plan
- Everything in ₹33
- 60 pre-generated audio/day (2/day: personalised + mythology, cron 2am IST)
- 30 on-demand TTS requests/month, daily guardrail 3/day
- TTS cost: 90 × ₹0.082 = ₹7.38/month

### Add-On Packs (no expiry)
- +10 → ₹29, +25 → ₹59, +50 → ₹99
- All >95% margin

### Upsell ₹33→₹99
- Google IAP `SubscriptionUpdateParams`, proration `IMMEDIATE_WITH_TIME_PRORATION`
- Trigger: 8-10 credits used or day 15+ with 0 credits

### IAP / Ops
- **IAP cut: 15%** (reduced post-2024 Play Store policy for sub-$10 subs)
- **Tech / infra / ops: ~5% of revenue** (Supabase, Cloudflare, monitoring, support tooling)
- **LLM story-gen: ₹0.20/story** (₹1 per 5 stories)

### Legacy (pre-D009, kept for reference)
Old model was: Free / Standard ₹99 / Family ₹179 / Premium ₹399. Superseded by D009.

## Story Workload Assumptions

- 1 story ≈ 4,000 chars ≈ 240 s of audio
- **30 stories per active user per month** (auto-gen daily, with some skip days)
- Auto-gen flow: server pre-renders 1 personalised story per kid each evening (5-9 pm window)
- No bedtime peak burst — load spread across 4-hour generation window
- Spontaneous on-demand TTS = premium tier only (small minority of paid base)

## Bench Results — ARM64 Sweep (2026-05-03)

Test rig: M5 Pro Docker, `--cpus`/`--memory` capped to mimic Hetzner CPX shapes. Worker-pool routing fix (`kokoros-openai/src/lib.rs`) + uint8 ONNX model.

### CPX32 (4c / 8GB / ₹2,000/mo)

| Config | c=1 RTF | c=3 RTF | c=6 RTF |
|---|---|---|---|
| f32-i2 | 0.31 | 0.65 | 1.01 BAIL |
| **uint8-i2** ✓ | **0.25** | **0.52** | **0.81** |
| uint8-i4 | 0.46 | 0.49 | 0.94 |
| q-i2 | 0.50 | 1.01 BAIL | — |

CPX32 winner: **uint8-i2** — RTF 0.81 at c=6.

### CPX42 (8c / 16GB / ₹2,500/mo)

| Config | c=1 RTF | c=3 RTF | c=6 RTF |
|---|---|---|---|
| f32-i2 | 0.19 | 0.41 | 0.69 |
| **uint8-i4** ✓ | **0.24** | **0.29** | **0.56** |
| uint8-i8 | 0.44 | 0.55 (partial) | — |
| q-i4 | not run | — | — |

CPX42 winner: **uint8-i4** — RTF 0.56 at c=6, best across whole matrix.

Quantized model (`model_quantized.onnx`) is unusable on ARM — slower than f32 and OOMs at concurrency.

## Throughput & TTS Cost per Story

`stories/hr = concurrency × 3600 / wall_sec` at highest workable concurrency.

| VPS / config | ₹/mo | stories/hr | stories/mo | **TTS ₹/story** | Users @ 30/mo |
|---|---|---|---|---|---|
| CPX32-uint8-i2 | 2,000 | 112 | 13,440 | ₹0.149 | ~448 |
| CPX42-f32-i2 | 2,500 | 133 | 15,960 | ₹0.157 | ~532 |
| **CPX42-uint8-i4** | **2,500** | **160** | **19,170** | **₹0.130** | **~639** |

CPX42-uint8-i4 wins on every metric. Use as default deployment.

## Cloud TTS Reality Check

| Provider | ₹/story (4,000 char) | ₹/user/mo @ 30 stories |
|---|---|---|
| Google Chirp 3 HD | ~₹10 | ~₹300 |
| OpenAI TTS-1 | ~₹5 | ~₹150 |
| ElevenLabs | ~₹40+ | ~₹1,200+ |

All exceed ₹99 revenue. Self-host is mandatory.

## Net Unit Economics (D009 — CAX21 @ ₹1,100/mo)

### ₹33 Tier

| Component | ₹/user/mo |
|---|---|
| Revenue | 33.00 |
| IAP cut (15%) | -4.95 |
| Tech / infra / ops (5% of revenue) | -1.65 |
| LLM story-gen (~30 × ₹0.04) | -1.20 |
| TTS self-host (10 × ₹0.082) | -0.82 |
| **Net per user** | **~₹24** |

### ₹99 Tier

| Component | ₹/user/mo |
|---|---|
| Revenue | 99.00 |
| IAP cut (15%) | -14.85 |
| Tech / infra / ops (5% of revenue) | -4.95 |
| LLM story-gen (~60 × ₹0.04) | -2.40 |
| TTS self-host (90 × ₹0.082) | -7.38 |
| **Net per user** | **~₹69** |

### Blended (70% ₹33 + 30% ₹99)
- Blended net: ~₹38/user/month
- TTS avg: 34 audio/user/month

### Legacy Net Economics (pre-D009, ₹99-only model)
Old model: ₹99 flat, 30 stories/user/mo, ₹0.13/story on CPX42. Net ~₹69/user. Superseded by D009 tier structure.

## Break-Even & Profit at Scale (D009)

**Launch box — CAX21 (₹1,100/mo):**
- Break-even: ₹1,100 / ₹38 blended = **~30 paying users**
- 100 users → ₹3.8K/mo profit
- 200 users → ₹7.6K/mo profit
- 395 users → ₹15K/mo profit (approaching capacity)

**Scale target — CPX42 (₹2,500/mo):**
- Break-even: ₹2,500 / ₹38 = **~66 paying users**
- 200 users → ₹7.6K/mo profit
- 500 users → ₹19K/mo profit
- 564 users → ₹21.4K/mo profit (capacity ceiling)

Migrate from CAX21 → CPX42 at ~350-400 paying users.

### Legacy Break-Even (pre-D009)
Old model: CPX42 only, ₹2,500/mo, ₹69 net/user, break-even at 36 users. Superseded.

## Why Auto-Gen Flow Unlocks the Math

| Old (on-demand bedtime) | New (auto-gen daily) |
|---|---|
| 50 stories/user/mo | 30 stories/user/mo |
| 80% load in 3hr peak | Spread across 4hr generation window |
| 0.44 stories/hr/user load | 0.25 stories/hr/user load |
| Bursty 5-10× spikes at 7:30/8/8:30 | Smooth continuous queue |
| ~300 user ceiling per CPX42 | **~639 user ceiling per CPX42** |
| Net ~₹95 @ ₹200 | Net ~₹69 @ ₹99 |

Auto-gen does **not** kill the personalisation moat — server uses kid's prefs + listening history to pick themes. Parent does zero input after onboarding. Story is just *there* at 8 pm, like Spotify Discover Weekly.

## Personalisation Moat — How It Survives

Moat = **prompt + content + voice quality**, not real-time TTS delivery.

- **Auto-gen passive personalisation** — kid's name, age, theme history, recent listens drive prompt
- **Word-swap fallback** — render generic base story once, splice hero name via short TTS clip (95% TTS cost savings on cached library content)
- **Voice clone of parent** = strong moat, justifies premium tier

Real-time generation was a delivery assumption, not a product promise. Bedtime app has natural async window — parent ritual fits scheduled gen.

## Recommended Tier Strategy (D009 — locked)

- **Trial:** 3-day, 10-15 stories, 3 audio teaser credits. No credit card. Hard paywall after.
- **₹33/month:** Text stories + 10 audio credits. Conversion tier — audio runs out mid-month, upsell to ₹99.
- **₹99/month:** Full audio (60 pre-gen + 30 on-demand). The real product.
- **Add-on packs:** +10 ₹29, +25 ₹59, +50 ₹99. No expiry. For power users.
- **Launch on CAX21 (₹1,100/mo).** Scale to CPX42 at ~350-400 users.

No free tier. No Family/Premium ladder at launch — revisit after 500 paying users.

## What Still Has to Be True

1. Auto-gen feels personalised (LLM uses prefs/history well — content is the real moat)
2. Default story good enough that <10% of users tap "regenerate" (otherwise breaks 30-stories/mo assumption)
3. Free → paid conversion reasonable (free = limited cached catalog, paid = daily personalised)
4. Retention at ₹99 holds month after month (engagement is the actual bottleneck, not infra)

## Open Decisions

1. ~~Onboarding form depth — how many signals to collect for personalisation~~ (addressed in D009)
2. ~~"Surprise me" behaviour — fall back to cached catalog or live-gen with delay~~ (addressed in D009 — auto-gen flow)
3. ~~Voice-clone tier feasibility~~ — deferred, not feasible now (D009)
4. When to migrate from CAX21 to CPX42 (~350-400 paying users per D009)
5. Cache policy for word-swap personalisation (how aggressive without breaking moat perception)
6. Upsell prompt UI: bottom sheet vs full-screen interstitial vs notification-only
7. Add-on pack purchase flow: in-app sheet or redirect to Play Store
8. On-demand queue priority: FIFO for all, or ₹99 priority over add-on overflow

## Related

- `projects/kokoros/current-state.md` — server perf and deploy state
- `projects/kokoros/quantized-model-handoff-20260503.md` — uint8 model rationale
- `projects/yawnly/decisions.md` — D004 (audio/story flow first), D008 (Library + retention)
