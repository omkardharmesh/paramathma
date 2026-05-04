---
title: Yawnly Pricing Tiers — Locked (₹33 / ₹99 + Add-Ons)
summary: Decision proposal for locked two-tier pricing (₹33 text+teaser audio, ₹99 full audio), 3-day trial, and one-time add-on packs. Supersedes Free/Family/Premium ladder from text-first thesis.
type: proposal
status: approved
owner: hermes-proposal
created: 2026-05-04
applies_to: [yawnly]
---

# Yawnly — Pricing Tiers Locked (D009 Proposal)

Supersedes the tier ladder in `inbox/claude-proposals/yawnly-20260504-text-first-tier-thesis.md` (which proposed Free/Story/Audio/Family/Premium). This proposal locks a **two-tier model** with a trial funnel and add-on packs.

## Decision

Lock Yawnly's pricing to two subscription tiers + one-time add-on packs. No permanent free tier. No family/premium ladder at launch.

### Trial (not a tier)

- **3-day trial**, 10-15 personalised stories per account
- **3 audio teaser credits** included (enough to hear the magic, not enough to be satisfied)
- No credit card required at trial start
- After trial: hard paywall → must choose ₹33 or ₹99
- Purpose: let parent + kid experience the product, then convert

### ₹33 / month — Text + Audio Teaser

- Unlimited personalised text stories (soft rate-limit: 10/day)
- **10 audio credits/month** (pre-generated, server-scheduled)
- Audio credits are the conversion lever: parent hears kid's name in story, kid loves it, runs out mid-month → upgrade to ₹99

**Why 10 audio, not 5 or 0:**
- 0 audio = parent pays ₹33 for "fancy PDF reader" — product magic is the name-in-story audio, not text alone
- 5 audio = runs out in 5 days, parent barely formed the habit, feels stingy
- 10 audio = runs out ~day 10, kid is emotionally hooked by then, 20 days of silence creates natural upgrade pressure from the child ("mummy where's my story?")
- TTS cost: 10 × ₹0.13 = ₹1.30/month — rounding error at any scale

### ₹99 / month — Full Audio Plan

- Everything in ₹33, plus:
- **60 pre-generated audio stories** (2/day: 1 profile-based + 1 mythology, server cron at 2am IST)
- **30 on-demand TTS requests/month** (parent-triggered, async queue)
- Daily soft guardrail: 3 on-demand/day max (prevents queue abuse)
- **Total audio: ~90/month** (60 pre-gen + 30 on-demand)

### One-Time Add-On Packs (above ₹99's 30 on-demand)

When a ₹99 user exhausts their 30 on-demand requests:

- **+10 requests → ₹29** (cost ₹1.30, margin 95.5%)
- **+25 requests → ₹59** (cost ₹3.25, margin 94.5%)
- **+50 requests → ₹99** (cost ₹6.50, margin 93.4%)

Add-on packs do not expire. Simple UPI-friendly pricing. No subscription creep.

## Upsell Mechanism — ₹33 → ₹99 (Google IAP)

Google Play handles subscription upgrades natively via `SubscriptionUpdateParams` with proration.

### Technical Flow

1. User is on ₹33 monthly plan (active, auto-renewing)
2. App tracks audio credit usage server-side
3. When trigger conditions met → show upgrade prompt
4. User taps "Upgrade to ₹99"
5. App calls `launchBillingFlow` with:
   - `SubscriptionUpdateParams` pointing from ₹33 → ₹99
   - Proration mode: `IMMEDIATE_WITH_TIME_PRORATION`
6. Google calculates: ₹99 minus unused portion of ₹33 (pro-rated by remaining days)
7. User pays the difference today, gets ₹99 features instantly
8. Next billing cycle: full ₹99 from original billing date

### Trigger Conditions (when to show the prompt)

**Primary triggers:**
- Credit 8 or 10 of 10 audio credits used → bottom sheet prompt
- User opens app on day 15+ with 0 remaining credits

**Prompt copy (example):**
- *"Want unlimited bedtime audio? Upgrade to ₹99 — you'll only pay ₹[difference] today"*
- *"Your bedtime audio stories are all used up. [Kid's name] misses them — upgrade to keep the magic going"*

**Anti-annoyance:**
- Show max 1 prompt per session
- Dismissed prompt → don't show again for 48 hours
- After 3 dismissals in a month → stop prompting until next credit cycle

## Why This Model

1. Two tiers = simple decision. Parent picks ₹33 or ₹99. No analysis paralysis.
2. Trial → ₹33 → ₹99 is a natural ladder. Trial hooks the kid. ₹33 builds the habit. ₹99 is where the parent stops reading aloud.
3. Audio is the upsell engine. 10 credits in ₹33 are not revenue — they're conversion fuel. The kid sells ₹99 to the parent, not you.
4. Add-ons capture power users. No queue hogging, no plan upgrade needed.
5. No free tier = no freeloaders. Trial is generous enough to validate, always ends at a paywall.

## Unit Economics

### ₹33 Tier

| Component | ₹/user/mo |
|-----------|-----------|
| Revenue | 33.00 |
| IAP cut (15%) | -4.95 |
| Tech/ops (5%) | -1.65 |
| LLM story-gen (~30 stories × ₹0.04) | -1.20 |
| TTS (10 credits × ₹0.082 est.) | -0.82 |
| **Net** | **~₹24** |

### ₹99 Tier

| Component | ₹/user/mo |
|-----------|-----------|
| Revenue | 99.00 |
| IAP cut (15%) | -14.85 |
| Tech/ops (5%) | -4.95 |
| LLM story-gen (~60 stories × ₹0.04) | -2.40 |
| TTS (90 audio × ₹0.082 est.) | -7.38 |
| **Net** | **~₹69** |

### Capacity — Launch Box: Hetzner CAX21 (₹1,100/mo)

- **Spec:** 4 vCPU ARM64 (Ampere Altra), 8 GB RAM, 80 GB NVMe
- **TTS capacity:** ~13,440 audio/month (estimated from CPX32 bench: 112 stories/hr, uint8-i2, RTF ~0.25 at c=1)
- **TTS cost/story:** ~₹0.082 (₹1,100 / 13,440) — better than CPX42's ₹0.13 on pure infra cost
- If all ₹99 users (90 audio each): ~149 users/box
- If 70% ₹33 + 30% ₹99 blend (34 audio avg): **~395 users/box**
- **Break-even at blended: ~30 paying users**

⚠️ **CAX21 capacity is estimated from M5 Pro Docker bench simulating CPX32. Real Ampere Altra performance is unvalidated — needs prod smoke test before marketing claims.**

### Scale Target: Hetzner CPX42 (₹2,500/mo)

When user base outgrows CAX21:

- **Spec:** 8 vCPU x86, 16 GB RAM
- **TTS capacity:** ~19,170 audio/month (uint8-i4, RTF 0.56 at c=6)
- If 70/30 blend (34 audio avg): ~564 users/box
- Break-even: ~66 paying users
- Migrate at ~350-400 paying users (when CAX21 hits ~80% capacity)

### Add-On Pack Economics

- All packs at >95% margin at CAX21 cost (₹0.082/story). Pure upside.
- ₹29 for 10 requests costs you ₹0.82 (97.2% margin)
- ₹59 for 25 requests costs you ₹2.05 (96.5% margin)
- ₹99 for 50 requests costs you ₹4.10 (95.9% margin)
- Add-on revenue is bonus — not factored into break-even.

## Conversion Funnel

- Day 0: Trial starts (10-15 stories + 3 audio teaser credits)
- Day 3: Trial ends → hard paywall
  - Chooses ₹33 (~70% expected) → builds habit, kid hears name in audio 10x
  - Chooses ₹99 (~30% expected) → full audio from day 1
  - Churns (~20-30% expected)
- Day 10-15: ₹33 user runs out of audio credits → kid asks for more → upsell prompt to ₹99 → expected upgrade rate: 15-25% of ₹33 users/month

## Pre-Generation Schedule

- Server cron at **2am IST** nightly
- Generates 2 stories per ₹99 kid profile: 1 personalised + 1 mythology
- Stories stored in Supabase, audio URLs ready by evening
- No bedtime peak burst — load is off-peak

## What This Replaces

This supersedes:
- Free tier from text-first thesis → replaced with 3-day trial
- Family tier (₹179) → deferred post-launch
- Premium tier (₹399) with voice clone → deferred, not feasible now
- The 5-tier ladder → locked to 2 tiers + add-ons

## What Still Has to Be True

1. LLM quality at GPT-4o-mini produces stories Indian parents rate ≥4/5 (Pre-Build Gate #1 from text-first thesis still applies)
2. 10 audio credits in ₹33 are enough to hook the kid but not enough to satisfy — if 10 feels "enough for the month", upsell fails
3. On-demand TTS queue stays under reasonable wait time at peak (measure post-launch)
4. Trial-to-paid conversion ≥ 20% (below that, trial cost is wasted)
5. Google IAP proration works correctly for ₹33→₹99 upgrade (test in closed track)

## Open Decisions (minor, post-approval)

1. Upsell prompt UI: bottom sheet vs full-screen interstitial vs notification-only?
2. Add-on pack purchase flow: in-app sheet or redirect to Play Store?
3. On-demand queue priority: FIFO for all, or ₹99 users get priority over add-on overflow?

## Related

- `inbox/claude-proposals/yawnly-20260504-text-first-tier-thesis.md` — predecessor (text-first thesis with 5-tier ladder)
- `projects/yawnly/pricing-economics.md` — TTS economics and bench data (needs tier table update if approved)
- `projects/yawnly/decisions.md` — D001-D008 (D009 entry to be added if approved)
- `projects/kokoros/current-state.md` — TTS server state
