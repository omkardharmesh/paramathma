---
title: Daily Catholic Spiritual Companion
summary: Text-first Catholic spiritual companion where users talk about their day and receive story-first reflections grounded in Scripture and Catechism citations.
status: shortlisted
source: Saarthi golden chats + Yawnly app/paywall reuse + Bible AI ideation
build_time: 4-6 week validation sprint
est_mrr: $500+ early target, validate pricing at $5.99/mo or $39.99/yr
updated: 2026-05-14
---

# Daily Catholic Spiritual Companion - Product Plan

## One-Liner

A quiet Catholic companion for daily life: users talk through their day, and the app helps them see that moment through Scripture, prayer, and the Catechism.

## Core Thesis

This is not a Bible reader with a chatbot attached. It is a daily spiritual companion.

The wedge is the Saarthi interaction pattern:

1. User shares a life moment.
2. The system detects the emotional or spiritual shape.
3. It places the user inside a relevant sacred scene.
4. It reflects from that scene before giving advice.
5. It anchors serious claims in exact Scripture or Catechism citations.
6. It gives one small next step: reflection question, prayer, examen prompt, act of repair, or safe redirect.

Example:

User: "I snapped at my mom today and feel bad."

Companion: Peter also knew the pain of failing someone he loved. After denying Jesus, he did not repair himself with perfect words. He met Christ again, and Christ asked him, "Do you love me?"

Then cite John 21, ask what one small act of repair could look like tonight, and optionally suggest a short prayer.

## Positioning

Use:

> Talk through your day with a Catholic companion that reflects back through Scripture, prayer, and the Catechism.

Avoid:

- "AI priest"
- "AI therapist"
- "Bible chatbot"
- "Ask anything about Christianity"
- "Generic self-care with Bible verses"

The promise is not "the app solves your problems." The promise is "the app helps you bring your day into a spiritual journey."

## MVP Scope

### Free

- One daily check-in
- Daily Scripture scene and short reflection
- Limited reflection chat, for example 3 free messages/day
- Exact citations shown for any Scripture used
- Clear disclaimers around priesthood, confession, crisis, medical, and legal boundaries

### Paid

- Unlimited or generously capped reflection chat
- 7-day or 30-day journey history
- Weekly spiritual recap
- Session continuity within the current reflection
- Deeper Catechism-backed answers where legally safe

### Explicitly Cut From V1

- Full Bible reader
- Full Catechism browser
- Images
- Audio
- Community
- Prayer streaks
- Long onboarding
- Saved journal beyond simple journey notes
- Heavy personalization
- Multi-denomination mode

## Why Cut The Bible Reader

YouVersion owns Bible reading. A reader creates legal and UX work without proving the core wedge.

For V1, the app should show the cited passage it used, or link out where appropriate. The differentiated behavior is scene-placement reflection, not Bible navigation.

## Product Loop

1. App opens with: "How was your day?"
2. User talks or types.
3. AI identifies the spiritual shape: stress, guilt, grief, fear, anger, gratitude, doubt, dryness, forgiveness, hope.
4. AI chooses a Biblical scene.
5. AI responds:
   - scene first
   - user mapping second
   - citation third
   - one small practice last
6. App saves a simple journey note.
7. At the end of the week, the app summarizes recurring themes and cited passages.

Example weekly recap:

- Recurring theme: trust
- Moments reflected on: work anxiety, guilt after an argument, gratitude after family time
- Scriptures: Matthew 6, John 21, Philippians 4
- Suggested practice: 5-minute evening examen for the next 3 days

## Golden Chat Direction

Before building ingestion, create `BIBLE_COMPANION_GOLDEN_CHATS_v1.md`.

Required cases:

- Anxiety at work -> Martha / Philippians 4 / Gethsemane
- Guilt after hurting someone -> Peter after denial / Prodigal Son
- Grief -> Lazarus / Mary at the Cross / Mary Magdalene at the tomb
- Doubt -> Thomas
- Anger -> Jesus cleansing the temple, with nuance
- Forgiveness -> 70x7 / Joseph and brothers
- Spiritual dryness -> Psalms / Gethsemane / Catholic mystic references where legally safe
- Gratitude -> Magnificat / healed leper returning
- Confession boundary -> refuse absolution, point to sacrament
- Self-harm crisis -> hard safety redirect
- Medical/legal/political questions -> safe refusal or limited spiritual reflection
- Doctrinal probes -> Eucharist, Mary, papacy, purgatory, confession, mortal sin

Voice rules:

- Lead with a scene, not a category.
- Do not open with therapy-speak.
- Do not open with "The Church teaches..." unless the user asked a direct doctrine question.
- Do not invent citations.
- If no retrieved source supports the answer, say so.
- Crisis protocol overrides voice style.

## Technical Reuse

### From Saarthi

- FastAPI backend
- SSE `/chat` streaming
- RAG pipeline with pgvector, FTS, RRF, embeddings, reranking
- LiteLLM routing
- Golden-chat and response-palette approach
- Docker Compose and Dockerfile

This should be built as a Catholic vertical/fork of Saarthi infrastructure, not as a brand-new backend.

### From Yawnly

- Compose Multiplatform app shell
- Supabase Auth
- Paywall UI
- IAP Edge Functions
- Entitlement flow
- Koin DI
- Navigation
- Shared UI conventions

Use the Yawnly shell only if validation passes or if a TestFlight shell is needed. Do not start by creating a large third standalone app.

## Legal Corpus Strategy

References are safe. Full text is the issue.

Safe starting options:

- Bible: World English Bible Catholic Edition or Douay-Rheims 1899
- Catechism: cite paragraph numbers and link out; use short excerpts only after legal review

Avoid for MVP without licensing:

- NABRE
- RSV-CE
- ESV-CE
- NRSV-CE
- Full Catechism ingestion or redistribution

Non-negotiable: get IP/legal review before public launch if storing or displaying substantial copyrighted religious text.

## Guardrails

Every serious answer must be source-bound.

Rules:

- Scripture citations must come from retrieval, not model memory.
- Catechism claims must cite paragraph numbers.
- If the system cannot retrieve a trustworthy source, it must say it cannot answer confidently.
- The app must not give absolution or replace confession.
- The app must not provide medical, legal, emergency, abuse, or self-harm counseling.
- Crisis and self-harm messages trigger a hard safety redirect.
- Personal moral-gravity questions should suggest speaking with a priest.

## Monetization

Test pricing higher than the original $3.99/month.

Candidate launch prices:

- $5.99/month
- $39.99/year
- Optional test: $9.99/month with 7-day trial

Rationale: Catholic/spiritual buyers are trust-sensitive more than price-sensitive. Too-low pricing can make the product feel lightweight.

Paywall trigger:

- After N free reflection messages/day
- Or after daily check-in plus one deeper follow-up

Do not paywall basic safety redirects.

## Validation Plan

Run 20 Catholic user interviews before writing production code.

Show mock replies, not abstract concepts.

Ask:

- Last time you were stressed, guilty, grieving, or spiritually dry, what did you reach for?
- Do you use a Bible app today? Which one? Why do you open it?
- Have you ever read the Catechism? Where?
- Do you trust AI with Scripture? Why or why not?
- Does this sample reply feel spiritually helpful or fake?
- Which feels better: generic advice, direct doctrine, or Biblical scene reflection?
- Would you pay $5.99/month or $39.99/year for this?
- Would you use it daily, weekly, or only in crisis moments?
- What would make this feel unsafe or inappropriate?
- Would you want this to be explicitly Catholic?

Hard validation test:

Can a Catholic user see one response and say:

> This feels like Scripture meeting my actual life.

## Six-Week Sprint

### Week 1 - Validation

- Build 10-15 golden sample replies
- Interview 20 Catholics
- Test $5.99/month and $39.99/year willingness
- Decide whether this is worth a TestFlight build

### Week 2 - Corpus And Behavior

- Finalize legal-safe Bible source
- Decide Catechism citation strategy
- Draft `BIBLE_COMPANION_GOLDEN_CHATS_v1.md`
- Draft refusal and crisis rules

### Weeks 3-5 - MVP Build

- Reuse Saarthi chat pipeline
- Add Catholic response prompt and golden-chat loading
- Add source verification
- Build minimal CMP text UI if validation passes
- Add paywall/free limit

### Week 6 - Beta

- TestFlight with 20 Catholic beta users
- Track D1/D7 retention
- Track free-limit hit rate
- Track trial intent or paid conversion
- Decide kill, fold into Saarthi, or scale

## Main Risks

- Building this as a third standalone product instead of a Saarthi vertical
- Weak validation masked by founder excitement
- Theological hallucinations
- Hallucinated or misleading citations
- Confession substitution
- Self-harm/crisis mishandling
- Copyright issues with Bible translations or Catechism text
- Generic "AI companion" positioning
- Low retention if daily check-in does not become a habit

## Verdict

Build only the narrow version:

> A text-first Catholic daily spiritual companion that lets users talk through their day and receive story-first, source-bound reflections.

Do not build a full Bible app. Do not build images, audio, community, or a theology debate bot. Prove the daily reflection loop first.
