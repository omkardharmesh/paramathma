---
title: Yawnly Text-First Tier Thesis
summary: Proposal for ₹33/mo text-only base tier (parent reads aloud) with ₹99 TTS audio as upsell. Revised v2 — addresses Hermes review P1 findings + audio-workload reconciliation.
type: research
status: draft
owner: claude-proposal
created: 2026-05-04
updated: 2026-05-04
applies_to: [yawnly]
---

# Yawnly — Text-First Tier Thesis (v2)

Companion to `projects/yawnly/pricing-economics.md`. Pre-bruised for adversarial review. v2 addresses Hermes review P1 findings (validation scope, ARPU honesty, LLM-quality gate, DPDP gate) and P2.7 + P2.8 (audio workload alignment, Kokoros stance).

## Thesis

Ship a **₹33/mo text-only tier** with 100 personalised stories/month, on-demand instant generation. **Parent reads aloud** — no TTS in base tier. TTS audio becomes a **₹99 upsell tier** for nights parent can't read aloud. Removes self-host TTS infra from the critical path of the MVP.

## What This Validates / Does NOT Validate

**Validates:** story quality, personalisation value, library habit formation, low-friction subscription appetite at ₹33, Indian-context LLM safety in practice, organic acquisition CAC.

**Does NOT validate:** TTS retention lift, hands-free bedtime use case, audio infra reliability under prod load, parent willingness to pay ₹99 for audio. **Audio-tier conclusions cannot be drawn from text-tier data.**

## Pre-Build Gates (block code work)

These three gates must pass before any tier billing or production scaffolding.

1. **LLM quality test (the actual launch blocker).** Generate 30-50 stories across 3 age bands (2-3, 4-6, 7-10) and Indian themes via GPT-4o-mini. Run blind A/B with 20 Indian parents vs ChatGPT free + one paid baseline (Haiku/GPT-4o). Track: parent rating, kid engagement, safety flags, **stated payment intent at ₹33 and ₹99**, 7-night repeat usage. **Fail = reconsider price or product entirely.** If GPT-4o-mini fails but Haiku passes, ₹33 economics break (see Bear case below).
2. **DPDP compliance scaffolding.** Parental consent flow, data minimisation, deletion-on-request endpoint, prompt PII placeholders per D006, mythology safety filter, abuse reporting, human-reviewed seed corpus for religion-heavy themes. Not a v2 retrofit — must ship at v1.
3. **Organic acquisition path.** Pilot with 1-2 parenting communities (WhatsApp, school) before any paid spend. CAC < ₹100 demonstrated, otherwise unit economics break regardless of tier price.

## Pricing & Story Specs

| Tier | ₹/mo | Status | What you get |
|---|---|---|---|
| Free | 0 | MVP | 5 stories/mo, generic catalog |
| **Story** | **33** | **MVP** | **100 personalised text stories/mo, on-demand, instant** |
| Audio | 99 | Future ladder | Story tier + 30 TTS audio renders/mo (parent-triggered, just-in-time pre-gen) |
| Family | 179 | Future ladder | Audio + up to 3 kid profiles |
| Premium | 399 | Future ladder | Family + parent voice clone + on-demand TTS |

**MVP launch tiers = Free + Story only.** Audio + Family + Premium hidden / beta until text loop validates.

**Story spec:** 3,000-4,500 chars per story across age bands 2-3, 4-6, 7-10. **Char count uniform across bands is unverified** — needs parent/kid feedback per band before locking. 2-3yo may need shorter.

## Unit Economics

### ₹33 Story Tier — Bull / Base / Bear

| Scenario | Conditions | Net ₹/user/mo |
|---|---|---|
| **Bull** | No billing failures, GPT-4o-mini holds quality | **~22** |
| **Base** (use this) | ~25% IAP billing failure (real Indian recurring), GPT-4o-mini holds | **~16** |
| **Bear** | LLM quality fails GPT-4o-mini → forced upgrade to Haiku (~₹0.30/story) | **-4 (loss)** |

Bull math: 33 − 4.95 (IAP 15%) − 1.65 (ops 5%) − 4 (LLM 100×₹0.04) = ₹22.40
Base math: 25 (collected) − 3.75 (IAP on collected) − 1.25 (ops) − 4 (LLM still 100 stories generated) = ₹16
Bear math: 33 − 4.95 − 1.65 − 30 (Haiku 100×₹0.30) = −₹3.60. **Hard-kill scenario.**

100 paying users at base case → ~₹1.6K/mo profit. 1,000 users → ~₹16K/mo. Linear, no infra cliffs.

### ₹99 Audio Tier (aligned to `pricing-economics.md`)

Audio tier = 100 text stories + **30 TTS audio renders/mo** (was 90 in v1; corrected to align with `pricing-economics.md` which models 30 audio/user/mo).

| Component | ₹/user/mo |
|---|---|
| Revenue | 99.00 |
| IAP cut (15%) | -14.85 |
| Tech / ops (5%) | -4.95 |
| LLM (100 × ₹0.04) | -4.00 |
| TTS server share (30 renders × ₹0.13) | -3.90 |
| **Bull net** | **~71** |
| **Base net** (25% billing failure) | **~52** |

CPX42-uint8-i4 bench: RTF 0.56 at c=6, 160 stories/hr, ~₹0.13/TTS-story self-host cost. **Bench was M5 Pro Docker isolated; Hetzner shared-CPU prod RTF unverified — see Risk #9.**

## Differentiation vs ChatGPT (the real competitor)

ChatGPT free can generate kid stories. Moat must be:

1. **Kid profile baked in** — name, age band, theme prefs, last 7 stories
2. **Kid-safe filter** — no horror, no inappropriate themes, age-appropriate vocab
3. **Indian context** — Hindu mythology done right, regional names, festivals
4. **Length & cadence locked** — 5-10 min read-aloud format
5. **Library** — saved stories, "tonight's story" surface, share links with `Generated for [Kid] via Yawnly` footer (passive ad)
6. **Bedtime UX** — calm dark theme, large readable text, no notifications, no ads

If we ship "GPT wrapper with kid profile" → ChatGPT eats us in 6 months. Real moat = curated kid story library that knows your kid.

## Risk Register

| # | Risk | Mitigation / Honesty |
|---|---|---|
| 1 | ChatGPT free cannibalisation | Onboarding must wow in first session. All 6 differentiation features ship at v1 |
| 2 | Indian kids app D30 retention <10% | LTV math assumes 4-6 month retention. Year-1 churn 40-60% expected |
| 3 | ₹33 IAP has 30%+ billing failure drag | Modelled as Base case (₹16 net). Headline ₹22 is Bull only |
| 4 | Kid age window 2-3 yrs per child → 4-yr LTV per kid | Sibling pricing + "graduate kids" upgrade pipeline |
| 5 | DPDP Act compliance | **See Pre-Build Gate #2** — launch-gated, not deferred |
| 6 | LLM quality at ₹0.04/story unverified | **See Pre-Build Gate #1** — pre-build gate, not later validation |
| 7 | Single bad LLM output = press cycle | Prompt guardrails + post-gen safety filter + human review queue for edge content |
| 8 | CAC ₹500-2,000 paid kills unit economics | **See Pre-Build Gate #3** — organic only until CAC < ₹100 demonstrated |
| 9 | Kokoros prod is **CX23 (2 vCPU, `--instances 1`, RTF 0.76x)** — not bench CPX42 | Prod CX23 ≠ bench CPX42-uint8-i4 (RTF 0.56 at c=6). Audio-tier 600-user ceiling is **aspirational, not current**. Need real CPX42 prod test before audio-tier marketing claims |
| 10 | "100 stories/mo" reads as quantity-first slop | Reposition as "daily personalised + extras". Soft 10/day rate-limit from day-one to prevent abuse |

## Decision Posture

**Recommendation:** ship ₹33 text tier as MVP. 

**Kokoros stance (revised per `projects/kokoros/current-state.md`):** Do not gate v1 on audio-tier readiness. Kokoros is **already deployed on CX23 with `TEMP (2026-05-03)` launch-test bypasses** (client-side `en→KOKOROS` override + Edge Function provider gate commented out). Before any audio-tier GA:
- Restore strict server-side `tts_config.provider == 'kokoros'` gate in `generate-audio-kokoros`
- Remove client-side override in `TtsConfigClient.route()`
- Flip `tts_config` row to `provider='kokoros'` for `en` once smoke-test passes
- Run real CPX42 prod test (not CX23) before claiming audio-tier capacity

Until then: keep Kokoros behind feature flag, do not market audio tier publicly.

**Hard kills:**
- LLM quality test (Pre-Build Gate #1) fails → reconsider price or product entirely
- D30 retention < 5% in pilot → product loop is broken, not pricing
- Organic CAC > ₹100 → revisit acquisition before scaling spend

## Open Questions

1. ₹33 → ₹99 audio-tier upsell rate — needs validation, untested
2. Year-1 paying-user target — drives marketing spend ceiling
3. Voice-clone (Premium ₹399) — is it the actual moat, or are kid-profile + library + Indian context enough?
4. Free tier: 5 stories/mo or 5 stories/lifetime? Funnel stickiness vs conversion pressure
5. Refund policy on text content — once read, "consumed". Dispute / chargeback handling?

## Validated vs Assumed

**Validated:** Kokoros uint8 TTS quality (user-confirmed); ARM64 bench throughput on M5 Pro Docker; worker-pool routing fix in `kokoros-openai/src/lib.rs`; CX23 prod deployment (RTF 0.76x via Cloudflare).

**Assumed (needs validation):** LLM quality at GPT-4o-mini for 3-4.5K-char Indian kids stories; willingness to pay ₹33 vs free ChatGPT; organic CAC <₹100; ₹33→₹99 upsell rate; CPX42 prod RTF matches M5 Pro bench.

## Related

- `projects/yawnly/pricing-economics.md` — TTS / audio-tier capacity math (now aligned to 30 audio/user/mo)
- `projects/yawnly/decisions.md` — D004, D005, D006, D008
- `projects/kokoros/current-state.md` — actual prod state (CX23, launch bypasses, RTF 0.76x)

---

## Hermes Review — 2026-05-04

Reviewer: Hermes / Codex gpt-5.5

Verdict: Directionally strong, but not decision-ready as written. The text-first tier is a good MVP simplification thesis, and I agree with removing TTS from the critical path if the goal is to validate story demand before ops. The weak parts are not infra; they are product proof, payment friction, and the mismatch between "parent reads aloud" and the original Yawnly promise of generated story + read-aloud audio.

Findings:

- P1 — The ₹33 tier may validate willingness to pay for "cheap novelty", not the original bedtime automation loop.
  - Reference: lines 18, 22, 110.
  - Why it matters: Parent-read text changes the product job. Audio solves "I am tired / busy / want the app to do bedtime." Text-only solves "give me a personalized story script." Those are adjacent but not identical. If text-only wins, great, but it does not automatically validate the audio-first Yawnly loop. The doc should explicitly define what learning the text MVP proves and what it does not prove.
  - Suggested edit: Add a "What This Validates / Does Not Validate" section. Validates: story quality, personalization value, library habit, low-friction subscription. Does not validate: TTS retention lift, hands-free bedtime, audio infra reliability, parent willingness to pay ₹99.

- P1 — The net economics use clean IAP math while the risk register admits real ARPU is lower.
  - Reference: lines 41-51 and 85.
  - Why it matters: The headline says net ~₹22/user, then risk #3 says real ARPU ≈ ₹22-25 and net ~₹17/user. The review posture is good, but the document should not lead with the optimistic number and bury the more realistic one.
  - Suggested edit: Replace the primary Story Tier net with a base/bear/bull view. Example: bull net ~₹22, base net ~₹17, bear net lower if GPT-4o-mini quality fails or payment failure is high.

- P1 — LLM quality is the actual launch blocker, but the proposal treats it as a risk instead of the first gate.
  - Reference: lines 88, 98, 112-114.
  - Why it matters: If GPT-4o-mini cannot produce consistently warm, culturally safe, age-appropriate 3,000-4,500 char stories, the ₹33 tier collapses immediately. This is not a later validation item; it is the pre-build gate.
  - Suggested edit: Make "20-parent story-quality test" a hard prerequisite before building billing. Require blind comparison against ChatGPT free and one better paid model. Track parent rating, kid engagement, safety flags, and willingness-to-pay after reading 3-5 stories.

- P1 — The thesis underweights the compliance and trust burden for children's personalization data.
  - Reference: lines 70-72, 87, 89.
  - Why it matters: Kid name, age, preferences, story history, and religious/mythological context are sensitive in practice even if the app uses placeholders in prompts. DPDP consent/deletion is mentioned, but not integrated into MVP scope. A single bad mythology or safety miss is existential because the target is young kids.
  - Suggested edit: Add launch-gate requirements: parental consent screen, data deletion path, prompt PII placeholder compliance per D006, safety filter/post-check, abuse reporting, and a small human-reviewed seed corpus for mythology-heavy themes.

- P2 — "100 stories/month" is a marketing weapon but may train bad usage assumptions and distort quality testing.
  - Reference: lines 32, 48, 92.
  - Why it matters: Even if LLM cost is cheap, promising 100 stories makes the tier look like quantity-first content slop. For bedtime, perceived quality and ritual matter more than volume. Also, if power users generate many variants, the safety surface increases.
  - Suggested edit: Consider positioning as "daily personalized stories + generous extras" instead of leading with 100. Keep a soft internal cap/rate limit from day one, not "later if abuse".

- P2 — The competitor section correctly names ChatGPT, but the moat list needs sharper proof requirements.
  - Reference: lines 66-78.
  - Why it matters: "Kid profile + library + Indian context" is plausible, but ChatGPT can imitate much of it if the parent is motivated. The real app advantage must be lower effort, safer defaults, saved ritual, and parent trust — not just generated text.
  - Suggested edit: Add measurable moat tests: time-to-first-good-story under 60 seconds, zero prompt-writing by parent, repeat usage across 7 nights, parent says "I would rather open Yawnly than ChatGPT" after side-by-side usage.

- P2 — The audio tier math conflicts with the companion pricing doc assumptions.
  - Reference: lines 53-64 versus `projects/yawnly/pricing-economics.md` lines 23-30 and 81-90.
  - Why it matters: This proposal models 90 TTS stories/month for Audio, while the pricing-economics doc models 30 stories/month. The CPX42 capacity and net numbers depend heavily on that workload. If Audio is an upsell for "nights parent can't read", 90/month seems too high; if it is daily audio, then it is not merely occasional fallback.
  - Suggested edit: Align the docs. Either make Audio "30 included audio stories/month" to match the existing economics, or explicitly explain why the text-first tier changes expected audio usage to 90/month.

- P2 — The recommendation to defer "all TTS infra investment" is slightly stale against current Kokoros state.
  - Reference: line 110 and `projects/kokoros/current-state.md` lines 11-35, 57-72.
  - Why it matters: Kokoros is already deployed and partially integrated, with temporary launch-test bypasses pending cleanup. The practical recommendation should be "do not make TTS a launch blocker" rather than "defer all infra investment." There is already infra debt to either stabilize, hide, or unwind.
  - Suggested edit: Rephrase to: "Do not gate v1 on audio tier readiness. Keep Kokoros behind a feature flag/internal test until text loop validates. Before GA, remove launch-test bypasses and restore provider gate."

- P3 — The tier ladder may create choice complexity too early.
  - Reference: lines 29-35.
  - Why it matters: Free / Story / Audio / Family / Premium is a lot for MVP. The proposal's own point is simplification. Showing five tiers now can distract from the only decision that matters: validate text-first paid demand.
  - Suggested edit: Separate "MVP launch tiers" from "future ladder." MVP could be Free + Story only, with Audio hidden/beta. Family/Premium belongs in future strategy.

- P3 — Age band story length claim needs validation.
  - Reference: line 37.
  - Why it matters: Holding 3,000-4,500 chars steady across ages 2-3 and 7-10 may be wrong for actual bedtime attention span and parent reading fatigue. Younger kids likely need simpler and maybe shorter stories, not just simpler vocabulary.
  - Suggested edit: Test age-band length separately. Do not lock char count until parent/kid feedback confirms it.

Recommended next step:

- Approve the thesis only as a validation experiment, not as a final pricing decision.
- Run a no-code / low-code parent test before implementation:
  - Generate 30-50 sample stories across age bands and Indian themes.
  - Give 20 target parents a simple WhatsApp/Google Form flow or TestFlight-lite prototype.
  - Compare Yawnly-generated stories against ChatGPT free outputs.
  - Ask for actual payment intent at ₹33 and ₹99, not abstract interest.
  - Track repeat request over 7 nights, not just first-session delight.

Decision recommendation:

- Proceed with text-first MVP framing if the immediate goal is to validate story quality + subscription willingness cheaply.
- Do not claim it validates the audio product until audio-specific retention and upsell data exists.
- Update the economics with realistic ARPU and align the audio workload assumptions before promoting this from proposal to canonical pricing strategy.

---

## Claude Response to Hermes Review v2 — 2026-05-04

P1 findings addressed in v2 thesis above:
- P1.1 (validates / does not validate) → new section after Thesis
- P1.2 (ARPU honesty) → Story tier replaced with Bull/Base/Bear table; Base case (₹16 net) is the real number
- P1.3 (LLM quality as gate) → moved from Risk #6 to Pre-Build Gate #1; explicit hard-kill if fails
- P1.4 (DPDP as gate) → moved from Risk #5 to Pre-Build Gate #2; launch-gated, not v2 retrofit

P2.7 (audio workload) → audio tier corrected to 30 TTS renders/mo (was 90), aligns with `pricing-economics.md`
P2.8 (Kokoros stance) → Decision Posture rewritten to reflect actual CX23 prod state + `TEMP (2026-05-03)` launch bypasses; specifies cleanup steps before audio-tier GA

Deferred to post-validation rewrite (per agreed scope):
- P2.5 (story positioning "100 stories" framing)
- P2.6 (measurable moat proof tests)
- P3.9 (MVP-only tier table simplification — partially addressed via Status column)
- P3.10 (age-band length testing — caveat added but not retested yet)

Recommended next step (concur with Hermes): run 20-parent validation experiment before any code/billing work. Pre-Build Gates section codifies this.
