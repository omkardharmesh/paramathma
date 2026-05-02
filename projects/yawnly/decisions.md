---
title: Yawnly Decisions
summary: Project-local decision memory for Yawnly.
type: decision-log
status: canonical
updated: 2026-05-02
---

# Yawnly — Decisions

Each decision uses: ID, Status, Date, Decision, Why, Rejected, Revisit.

---

## D001: Supabase Is Yawnly's Product Backend

Status: accepted
Date: 2026-05-02
Scope: Yawnly

Decision:
Supabase is the product backend for Yawnly. New product APIs go through the Supabase SDK (or through Edge Functions when service-role / third-party secrets are required), with repositories extending `SupabaseBaseRepository`.

Why:
Supabase covers auth, RLS-protected DB, and Edge Functions in one place, which fits a CMP app with a small backend surface. The repo's recent product code already commits to Supabase as the data spine.

Rejected:
- Building a custom backend.
- Routing new product APIs through the legacy Ktor skeleton.

Revisit:
If Supabase pricing or RLS expressiveness becomes a real blocker, or if a feature needs a backend pattern Edge Functions cannot serve.

---

## D002: Ktor / `BaseRepository` Is Legacy Skeleton, Not the Path for New Yawnly APIs

Status: accepted
Date: 2026-05-02
Scope: Yawnly

Decision:
The Ktor `BaseRepository` / `BaseResponseDto` stack is preserved as legacy skeleton code. New Yawnly product APIs must use the Supabase stack (`SupabaseBaseRepository`). The Ktor classes are not deleted but are not extended for new product flows.

Why:
The Ktor scaffold predates the Supabase decision. Mixing two stacks for the same product data leads to drift. Keeping it for skeleton/non-Supabase auxiliary work avoids a large rip-out.

Rejected:
- Deleting the Ktor scaffold today.
- Building new product features on top of `BaseRepository`/`BaseResponseDto`.

Revisit:
If a future feature genuinely needs Ktor (e.g. third-party REST not on Supabase) or if the Ktor scaffold becomes unused entirely.

---

## D003: No Room For Yawnly Product Data

Status: accepted
Date: 2026-05-02
Scope: Yawnly

Decision:
Yawnly product data is online-only. No Room database for product flows. Guest data stays in memory until the user authenticates, after which Supabase becomes the persistence layer.

Why:
Two persistence layers (Room + Supabase) for the same product data is a synchronization tax that does not match Yawnly's read patterns. The app is online-first.

Rejected:
- Adding Room for offline reading of stories.
- Adding Room for guest persistence beyond app session.

Revisit:
When offline reading becomes a real product requirement (e.g. travel mode), and only via a narrow cache layer with explicit sync rules — not a general Room datastore.

---

## D004: Ship Audio / Story Flow Before 3D / Lottie Buddy

Status: accepted
Date: 2026-05-02
Scope: Yawnly

Decision:
Ship the audio + story flow (generation, TTS playback, library) before building the 3D/Lottie buddy character.

Why:
The buddy adds animation and rendering complexity that does not validate the underlying value loop. Validate payment, retention, and story quality first.

Rejected:
- A full 3D animation pipeline before v1.
- The buddy as a launch blocker.

Revisit:
After retention and payment behavior are validated, or around 500 paying users.

---

## D005: Story Generation Uses Structured Prompts / Entropy, Not RAG

Status: accepted
Date: 2026-05-02
Scope: Yawnly

Decision:
Story generation uses structured prompts and controlled entropy (templates, slots, and randomness over a curated story DNA), not Retrieval-Augmented Generation against an external corpus.

Why:
RAG against a custom corpus introduces ingestion, indexing, and freshness work that does not match Yawnly's story-quality control needs. Structured prompts give tighter editorial control.

Rejected:
- A vector DB for story retrieval at launch.
- Free-text prompting without structure.

Revisit:
If structured prompts plateau on quality and a curated retrieval source genuinely improves output without leaking PII.

---

## D006: PII Placeholders In Model-Facing Prompts

Status: accepted
Date: 2026-05-02
Scope: Yawnly

Decision:
Prompts sent to model providers must use placeholders for personally identifying information (child's name, location, etc.). Real PII is substituted client- or server-side just before display, never sent in the prompt text.

Why:
Reduces PII exposure to third-party model providers and protects children's personal data. Also keeps prompts cacheable across users.

Rejected:
- Embedding the child's real name in the prompt sent to the LLM.
- Storing personalization in the prompt text rather than in a substitution layer.

Revisit:
Only with a privacy review. Default direction is to keep prompts PII-free.

---

## D007: Subscription Truth Is Server-Authoritative

Status: accepted
Date: 2026-05-02
Scope: Yawnly

Decision:
Entitlement (subscribed vs not, plan tier) is decided server-side. The Supabase backend (DB + Edge Functions) is the source of truth. The mobile client may cache for UX but reconciles against the server.

Why:
Client-trusted entitlement is bypassable and causes refund/restore drift. Apple/Google receipts are verified server-side via Edge Functions before persisting entitlement.

Rejected:
- Granting entitlement based on a client-side flag.
- Trusting the local IAP receipt without server verification.

Revisit:
Only if a server outage requires a temporary, narrowly scoped client-side fallback — and then with explicit logging and reconciliation on next online session.

---

## D008: Story Library / Retention Work Has Higher Near-Term Product Value Than Heavy Visual Buddy Work

Status: accepted
Date: 2026-05-02
Scope: Yawnly

Decision:
Near-term product effort favors the Story Library and retention features (saved stories, replay, recommendations) over heavy visual buddy work.

Why:
Library + retention tightens the value loop that pays for subscriptions. Heavy visual work without retention means high asset cost without payoff data.

Rejected:
- Buddy 3D rigging or full character animation as a near-term blocker.
- Pausing Library work until visuals ship.

Revisit:
If Library work lands and retention still does not move, OR after 500 paying users when D004's revisit triggers.
