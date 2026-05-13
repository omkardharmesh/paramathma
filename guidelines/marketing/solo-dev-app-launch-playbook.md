---
title: Solo Dev App Launch & Money Playbook
summary: Generic, app-agnostic 30-day launch playbook for solo devs shipping multiple apps. Covers pricing, store listing, 30-day sprint, hard thresholds, retention, upsell, cancel flow, multi-app portfolio strategy.
type: guideline
status: draft
owner: human
applies_to: [global]
created: 2026-05-14
updated: 2026-05-14
last_verified: 2026-05-14
supersedes:
---

# Solo Dev — App Launch & Money Playbook

Generic, app-agnostic 30-day launch playbook for solo devs shipping multiple apps. Distilled from RevenueCat State of Subscription Apps 2026, ASO 2026 benchmarks, Airship push benchmarks 2026, indie-hacker case studies (r/SideProject, r/IndieHackers, r/SaaS top posts, last 12 months), and 10 high-signal X threads on app growth.

Use this as the *template* for every app you launch. Each section has reusable rules, reusable thresholds, and a reusable copy/scaffold.

---

## 0. The brutal truth (read this first, every time)

- **90% of indie hackers fail at distribution, not product.**
- **72% of successful solo founders cite distribution — not product — as the #1 growth driver.**
- **"Find 10 people who genuinely need what you built, then figure out where 10 more like them hang out."** The first 100 users is *finding*, not *marketing*.
- One founder shipped 37 products in 5 years; only 1 went viral. Sticking with one project beats chasing the next hit.
- The median outcome is **6 months to first paying user**, not 6 days. (Refgrow case: 6 months → growing slowly with no ad budget.)
- Hard paywall apps generate **8× revenue per install** at D60 vs freemium ($3.09 vs $0.38) — RevenueCat 2026.
- **Post-launch is 80% marketing, 20% product.** Building is the easy part now.
- **Retention > acquisition. 70% of revenue often comes from existing users.**

If any of the above stings, this playbook is for you.

---

## 1. Pick what to build (the hardest step)

You can build anything fast now. The only question that matters: *who will pay for this in 6 weeks?*

### Demand-first signal hunting

| Source | What you're looking for | Tool / lookup |
|---|---|---|
| App store category leaders | Apps in your candidate niche making >$15K MRR | Sensor Tower, AppTopia (free tiers) |
| Reddit search | Real problem posts, not "wishlist" posts | Reddit search, r/<niche>, sort by top/year |
| TikTok/Reels search | Creator search behavior with 100K+ weekly hits | TikTok Creator Search Insights, Reels search |
| Mobbin | Onboarding + paywall patterns of proven apps | Mobbin (paid) — worth one month subscription |
| Google Trends (5-year) | Demand rising or falling? | trends.google.com |
| YouTube comment scrape | Pain language users actually use | Manual review of top videos in niche |

### The 3 ways to pick a winner

1. **Sharper niche of a proven category.** Rebuild a $15K+ MRR app for a sharper slice (e.g., generic habit tracker → habit tracker for ADHD; generic budget app → budget app for couples). Avoid blind cloning — re-target.
2. **Capability moment.** Ship the day a model or API gets sharply better at a task. (Math solver built in a week, sold for $30K when o4-mini got sharp at math.) Watch model releases.
3. **Anti-position.** Build the explicit opposite of the dominant category leader. (Privacy money tracker vs Mint. Offline note app vs cloud-only.) The anti-position itself becomes the marketing.

### Validation rule (skip at your peril)

Before writing code: **10 real conversations** with people who match your target user. Not "would you use this?" — but *"show me how you currently solve this. Walk me through last week."* If they don't have a workaround, the pain isn't real.

---

## 2. Pre-launch — what to wire before you market

### Tech stack (defaults that compound across apps)

| Layer | Pick | Why |
|---|---|---|
| **Backend** | Supabase | Auth + Postgres + Edge Functions + Storage in one. Free tier good for first 1K users. |
| **IAP / subscriptions** | RevenueCat | One SDK, both stores, free up to $10K MTR. Saves weeks of receipt-verification work. |
| **Paywall UI** | Adapty or Superwall | A/B test paywalls without app updates. Free tier covers first 1K trials. |
| **Push** | OneSignal | Free up to 10K subscribers. Better targeting than native FCM. |
| **Analytics** | PostHog (self-host or cloud free tier) or Mixpanel free tier | Funnels + cohorts without learning SQL. |
| **Email transactional** | Resend | $0 for 3K/month. Trial reminders, win-back emails. |
| **Web (landing + privacy policy)** | Vercel or Cloudflare Pages | $0 hosting. Required surface for store submission. |
| **ASO research** | AppRadar / ApppTweak / ASOMobile | One paid month at launch, drop after. |
| **Competitor screen library** | Mobbin | One month subscription before designing onboarding/paywall. |

This stack works for any app category. Building one app reusing this stack should take 4–8 weeks. The second app: 2–4 weeks because everything is reusable.

### Must-haves before public launch (non-negotiable gate list)

1. **Server-authoritative entitlement.** Subscription truth lives on the server, not the client. Receipts verified server-side.
2. **Day-0 aftercare screen.** Where the trial purchase routes (NOT the home screen — the magic moment).
3. **Confirmation screens.** Show what was saved/purchased. Cuts support tickets ~20%.
4. **Google/Apple sign-in.** Lifts conversion 30–40% vs email-only.
5. **Privacy policy live** at `yourdomain.com/privacy`.
6. **Data Safety form** (Play) / **Privacy Manifest** (App Store) accurate.
7. **Subscription products configured** in both stores with the right trial spec + price points.
8. **Push permission prompt** delayed to after first "aha" moment, never on first launch.
9. **Funnel events instrumented:** `install`, `signup`, `aha_moment_completed`, `trial_started`, `paid_started`, `audio_played` / `feature_used`, `feature_completed`, `cancel_intent`, `cancel_confirmed`.
10. **Analytics dashboard wired** before paid spend. You will need cohort views by day-of-install.

Cannot launch paid spend without 1–10 green. Repeat for every app.

---

## 3. Pricing — the highest-leverage decision

### The 2026 data (RevenueCat, 115K+ apps, $16B+ revenue)

| Decision | 2026 numbers | What to pick |
|---|---|---|
| Free tier vs hard paywall after trial | Hard paywall = **5× freemium D35 conversion** (10.7% vs 2.1%) and **8× revenue per install** at D60 | **Hard paywall after trial** for most categories. |
| Short trial vs long trial | <4 day = **25.5%** convert; 17–32 day = **42.5%** convert | **7-day trial** for most apps. Test 14-day if value takes time to land. |
| Number of tiers | 2 tiers maximum at launch | Avoid analysis paralysis. Add tiers only after data. |
| Annual discount | Industry standard 30–50% off monthly | Offer annual from launch. Annual users churn ~3× less. |
| Add-on packs (no expiry) | Strong overflow lever | Add tiny consumables (10 pack, 25 pack) for power users — keeps the base price honest. |

### Trial design (matters more than the price)

The trial is the demo. Two failure modes:

- **Trial too cheap.** User burns through value before deciding. Trial-to-paid stays low.
- **Trial too rich.** User extracts all value, cancels at Day 2. Trial-to-paid stays low.

Aim for: **enough value to feel the aha clearly, not enough to satisfy the long-term need.** Concrete: if your product's core action repeats daily, trial gives 5–10 of that action, not 50.

### Upsell ladder (template)

Entry tier → Premium tier, plus add-on packs.

- **Entry tier (X/month):** lighter usage cap. Conversion lever — user runs out mid-month and feels the upsell pressure.
- **Premium tier (3–5× entry):** the "real" product. No artificial cap, or generous cap.
- **Add-on packs:** non-expiring usage refills at >95% margin. Saves the cancel flow.

Trigger upsell from entry → premium at:
- **Usage trigger:** 70–80% of cap consumed.
- **Time trigger:** Day 15+ at low usage (re-engagement nudge).

Mechanism (cross-platform):
- **Google Play:** `SubscriptionUpdateParams` with `IMMEDIATE_WITH_TIME_PRORATION`.
- **App Store:** Subscription Group with paid tiers in the same group, automatic proration.

---

## 4. Store listing — the silent conversion lever

### Title and short description

| Field | Rule |
|---|---|
| Title | ≤30 chars. Brand + the highest-volume search keyword. Not "X.ai — AI-powered productivity for the modern professional." |
| Short description | ≤80 chars. One promise, one outcome. Not feature list. |

Avoid in title and short desc: "AI-powered", "intelligent", "smart", "next-gen", "the best." All filtered by user attention.

### Screenshot sequence (8 frames)

Rules locked by 2026 ASO data:

1. **First 3 screenshots gate install.** They appear in search results. The rest only appear if a user opens your listing.
2. **One promise per frame.** 7-second attention span across the whole listing — each frame gets <1 second.
3. **Headline first, app screen second.** Headline = promise, app screen = evidence.
4. **Social proof on screenshot 1** lifts conversion up to **90%.** Add as soon as a real number exists. Never fabricate.
5. **High-contrast palette.** Bright, saturated, dark mode. Pastel and gray underperform.
6. **Top-left placement** for value-prop text. Eye entry point.
7. **Hybrid captions:** text + small visual cue (arrow, glow, hand pointing). Not plain text on screenshot.
8. **Localise.** Submit at least 2 locales — your primary market + your second-largest. Translate the whole listing, not just the title.

### Screenshot template

| Frame | What goes here |
|---|---|
| 1 | The strongest reveal of your app + social proof if you have it |
| 2 | What category of problem you solve, with a visual of the solving moment |
| 3 | The personalisation / customisation moment that makes it feel "for me" |
| 4 | Your moat feature (the one thing competitors don't do) |
| 5 | A proof / outcome screen ("user did X with the app") |
| 6 | Pricing or trial offer card (transparent) |
| 7 | A "save / share / library" feature screen |
| 8 | Compliance / trust frame (no ads / privacy / safety) |

### Feature graphic (Play) / Promo art

- Composition: subject left third, big headline middle, product screen right third.
- Use the same headline as screenshot 1.
- Two language variants minimum.

### Keyword strategy

- **Don't bid for "free X" keywords.** They convert at terrible trial-start rates.
- **Don't position against generic AI.** "AI story generator" loses to ChatGPT in user heads.
- **Do** target keywords your competitors *also* target — that's where demand is proven.
- **Do** mine 1-star and 3-star reviews of competitors. The complaints become your listing copy advantages.

### A/B testing cadence

Most stores support A/B tests on screenshots / icon / short desc. Run **one test per 2 weeks** post-launch. Average lift per cycle: **10–25%.**

---

## 5. The 30-day sprint template

This is the same shape for every app. Adapt only the channels and creative.

### Week 1 (Days 1–7) — Foundation, $0 paid spend

**Primary objective:** ship the listing, validate the trial flow, build the demand board, plant the first organic posts.

| Day | Task |
|---|---|
| 1 | Verify all 10 gate items (§2). Stop launch if any fail. |
| 1 | Build demand board: 20 competitor apps' first 5 screenshots + paywalls + reviews. |
| 2 | Submit store listings in 2 locales. |
| 2 | Wire post-purchase aftercare → first aha moment screen. |
| 3 | Instrument all funnel events. Confirm dashboard reads them. |
| 4 | Record 6–8 raw organic posts (smartphone, no studio). |
| 5 | Publish first 2 posts. Seed 3 niche communities (where your 10 first users hang out). |
| 6 | Publish 2 more posts. Inspect first 24h analytics. |
| 7 | **W1 Gate review.** Trial flow works? Post-purchase routes to aha screen? Entitlement server-side? At least 30 organic installs? At least 5 trial starts? |

### Week 2 (Days 8–14) — Organic volume + paid micro-test

**Primary objective:** find 2–3 hooks that lift organically. Spend $25–100 in paid as *learning*, not scale.

| Day | Task |
|---|---|
| 8 | Launch Meta micro-test: 4 ad sets × $10–25 each. AAA targeting. Optimise for trial-start, not install. |
| 8 | 4 creative variants live: pain angle / reveal angle / outcome angle / specific feature angle. |
| 9–13 | 2 organic posts/day. Inspect paid daily. Comment-bait CTAs ("comment a use case, I'll build the example"). |
| 10 | Cut any ad set under threshold (see §6 thresholds). |
| 12 | Consolidate remaining spend onto top 2 ad sets. |
| 14 | **W2 Gate review.** At least 1 ad set under target CPI + above target CTR? At least 1 organic post with engagement ≥3%? Trial-start from paid ≥25%? |

### Week 3 (Days 15–21) — Scale winning creative

**Primary objective:** concentrate spend on the winning angle with 3 variants. Watch trial-to-paid.

| Day | Task |
|---|---|
| 15 | Move main budget into top ad set. Build 3 variants of winner: same first 2 seconds + same promise, different VO / pain / specific feature. |
| 15 | First trial cohort hits D7+. Read trial-to-paid. |
| 15 | Upsell engine on: trigger at usage threshold. |
| 16–20 | 2 organic posts/day. Cross-post winning post to second channel. |
| 19 | If trial-to-paid <15%, inspect onboarding + aftercare + paywall copy. Iterate. |
| 21 | **W3 Gate review.** Trial-to-paid ≥15%? Cumulative paying users ≥ break-even count? At least one upsell event fired? |

### Week 4 (Days 22–30) — Convert, retain, upsell

**Primary objective:** defend conversion. Drive upsell. Stand up D7 and D14 retention readouts.

| Day | Task |
|---|---|
| 22 | Final paid push on proven angle. Pause weakest variant. |
| 22 | Push notification: daily aha moment reminder at peak-CTR hour for category. |
| 24 | Verify upsell trigger fires reliably. |
| 25 | Repost first user reaction (UGC from comments/DMs) as social proof. |
| 26 | Trial-end cohort reminder push + email. |
| 28 | D7 retention readout on first install cohort. |
| 29 | Upsell-rate health check. <5%? Redesign upsell UI for next iteration. |
| 30 | **30-Day Readout:** CPI · CTR · store CVR · trial-start · trial-to-paid · D7 · D14 · feature-completed · upsell · paying users · net revenue · runway. |

---

## 6. Hard thresholds (kill / scale)

The same shape applies to every app — just adjust currency values for your CPI band.

| Metric | Kill | Inspect | Scale |
|---|---|---|---|
| **Paid CPI** | >2× your target | 1–2× target | ≤target |
| **Ad CTR (Reels / Shorts placement)** | <0.8% | 0.8–1.4% | ≥1.5% |
| **Store CVR (visit → install)** | <15% | 15–24% | ≥25% |
| **Trial-start rate (install → trial)** | <25% | 25–39% | ≥40% |
| **Trial-to-paid** | <10% (industry median 10.7%) | 10–19% | ≥20% |
| **D7 retention** | <15% | 15–24% | ≥25% |
| **D14 retention** | <10% | 10–17% | ≥18% |
| **Aha-moment completion in trial** | <50% | 50–69% | ≥70% |
| **Upsell rate (month 1, of base tier cohort)** | <2% | 2–4% | ≥5% |
| **Organic post engagement / impressions** | <1% | 1–2.9% | ≥3% |
| **Push CTR** | <1.5% | 1.5–2.9% | ≥3% |
| **Cancel-flow save rate** | <15% | 15–24% | ≥25% (industry 25–45%) |

### Hard kill triggers (do not negotiate)

1. **D7:** trial flow broken → halt all paid spend.
2. **D14:** no ad set near target CPI → roll budget into more organic creative; do not start W3 scale.
3. **D21:** trial-to-paid <10% → stop adding W4 budget. Fix funnel first.
4. **Any day:** aha-moment completion in trial <40% → onboarding broken, downstream of a product problem.

### Hard scale triggers

- Single ad set sustains CPI ≤0.6× target + CTR ≥2% + trial-start ≥40% for 72h → double daily budget within cap.
- Organic post: engagement ≥6% + ≥30 saves → boost with $10–25 micro-budget; scale if holds.

---

## 7. Budget allocation template

Solo dev's realistic launch budget: **$0–$500** for the launch month. The template scales linearly.

| Bucket | Amount (small / medium) | When | Purpose |
|---|---|---|---|
| Organic content production | $0 | W1–W4 | Founder time + smartphone |
| Tools (one-month boost) | $0–$50 | W1 | Mobbin + ASO tool one-month subscription |
| Meta micro-test | $50–$100 | W2 | 4 ad sets × small budget |
| Meta scale | $100–$200 | W3 | Concentrate on winning creative |
| Meta defend + W4 push | $100–$200 | W4 | Final spend on proven angle |
| Second-channel reserve | $50 | W4 contingency | YouTube Shorts or TikTok or UAC depending on app |
| **Total committed** | **$300–$500** | | |
| **Reserve (4-week extension if profitable)** | $200–$500 | M2 | Only if 30-day exit gates are met |

### Spend gating

- W1: $0 spent. Spend only after gate green + organic signal.
- W2: micro-budget only after W1 gate.
- W3: scale only after W2 exit gate.
- W4: only if W3 trial-to-paid ≥15%.

If any gate fails, budget rolls forward, not sideways.

---

## 8. Organic content plan

### Cadence

- **Daily**: 1 short-form post on your primary channel + cross-post on your secondary channel.
- **Bi-weekly**: 1 founder build-in-public thread on X.
- **Weekly**: 1 long-form (carousel, deeper post) on your highest-engagement community surface.

### Format mix

| Format | Use | Frequency in 30 days |
|---|---|---|
| Screen recording of the aha moment | Show the magic in <30s | 6 |
| Founder talking to camera about pain | Build trust + audience connection | 6 |
| Before/after split | Compare with status quo / competitor | 4 |
| Carousel of micro-tips relevant to niche | Saves + shares | 4 |
| Reaction / user testimonial UGC | Social proof | 4 |
| Niche-specific deep cut | Where your audience hangs out | 4 |
| Founder build-in-public thread on X | Audience compounding | 2 |

### The comment-mining loop (compound forever)

Every pain-format post must end with **"Comment a [your-niche-specific-thing], I'll [generate/build/show] it."** Replies become next week's content. Repost the generated thing as a reply tagging the commenter. Turns audience into co-creators.

### Community discipline rule

> "Be useful 10 times before you mention your product once."

Founders who promote without participating get filtered out — Reddit shadowbans, X algorithmic suppression, IndieHackers downvotes.

### Launch week timeline (generic)

| Day | Move |
|---|---|
| 1 | Primary platform launch + email blast to existing list |
| 2–3 | Secondary platform submissions (stagger, do not overlap) |
| 4–5 | Targeted shares in 3–5 relevant Slack/Discord communities |
| 6–7 | "Lessons from launch week" thread on IndieHackers + X |

### Product Hunt — yes or no

- **Yes** for consumer apps with a launch-moment angle.
- **No** if your audience does not live on PH (most B2B niches, regional consumer apps).
- Time it for the 2nd week of the sprint, after W1 gate is green.

---

## 9. Paid ad plan

### Channel selection by app type

| App type | Primary | Secondary | Avoid in first month |
|---|---|---|---|
| Consumer mobile app, mass market | Meta (FB + IG) | YouTube Shorts | Snap, X |
| Niche tools / B2B SaaS | Google Search Ads (intent-driven) | LinkedIn | Meta, TikTok |
| Creative / visual product | Pinterest + IG | YouTube Shorts | Snap |
| Regional Indian content | Reels + Shorts + Moj/ShareChat/Josh | Meta | TikTok (banned) |
| Productivity / dev tools | X promoted posts + Reddit ads | Hacker News (organic) | Meta |

### Generic Meta micro-test plan ($50–$100)

- **Objective:** App Promotion / App Events optimised on `trial_started` event. Fallback `first_open` if signal thin.
- **Budget:** 4 ad sets × $10–25 = $50–$100 over 7 days.
- **Audience:** Advantage+ Audience (AAA). Country your app monetises in.
- **Creatives:** 4 angles tested (pain / reveal / outcome / specific feature).
- **Hard kills per ad set:** CTR <0.8% after 1K impressions · CPI >2× target after $15 spend · trial-start <15% of installs after 20 installs.

### Creative variant rules (when scaling)

When a row hits scale signal, build **3 variants** that hold first 2 seconds + promise constant. Change only one of:

1. **VO change** — same line, different narrator.
2. **Specific feature variant** — same promise, swap which feature shows.
3. **Pain variant** — same outcome, different pain entry.

Never change the first 2 seconds across variants. The hook is the leverage point.

### Attribution caveats

- Do not trust the ad platform's in-platform conversion numbers alone. Store install referrer is the source of truth.
- UTM tag every ad URL.
- Reconcile your analytics against platform-reported events daily. Expect 20–30% discrepancy.
- Bot traffic in tier-3 markets: 20–30%. Discount raw install numbers before computing CPI.

---

## 10. Retention design (Day 0 / 1 / 2 / 3)

The single most important industry number: **55% of all 3-day trial cancellations happen on Day 0** (RevenueCat 2026, 115K+ apps). Your defence is product, not push notifications.

### Day 0 (trial-start moment)

1. After signup, **immediately** show the personalised aha moment. No spinner showing "AI thinking."
2. Play / generate / show the magic *before* asking for anything else.
3. After trial purchase, route directly to the first-use screen with copy:
   > "Your first [thing] for [name] is ready. Tap to start."
4. Do **not** drop the user on a generic Home screen. No "what's new" carousel. No settings prompt.

### "Loading personalisation" screen (2026 emerging pattern)

Between the final onboarding question and the trial paywall, insert a 2–4-second loading screen with social proof:

> *"Personalising [your-thing]'s first [output]… [N] [things] generated this week."*

Reasons it works:
- Personalisation feels *earned* (not instant = perceived effort).
- Social proof number lowers trust friction before the paywall ask.
- Buys time for actual server work (story generation, profile setup, etc.).

Use a real number once you have one. Never fabricate.

### Day 1 (≈18–22 hours after trial start)

- Push at **peak-CTR hour for your category** (most consumer: 8–9am or 6–8pm local).
- If D0 aha was completed: nurture push ("[name]'s next [thing] is ready").
- If D0 aha was NOT completed: re-engagement push ("Try [thing] tonight. 60 seconds.").

### Day 2 (trial day 2)

- In-app sticker: "[N] [usage units] left in your trial."
- Auto-generate / pre-render today's content (cron at low-traffic hour) so when user opens, content is *already there*.
- Push at peak-CTR hour: "Today's [thing] is ready."

### Day 3 (trial end)

- 4pm local in-app modal: "Your trial ends tonight. Continue [thing] for [price]/month."
- 9pm local push (only if D2 aha completed): "Don't lose [thing] — continue for [price]."

### Trial close playbook

- **No discount wheel** to lift trial-to-paid. Trust > short-term conversion.
- **No fake countdown.** Timers that reset on app reopen damage trust.
- **Crystal-clear renewal copy:** "You will be charged [price]/month starting [date]. Cancel anytime in [store]."

### Push notification rules (Airship 2026)

- **+51.7% CTR** from sending at the right time vs random.
- Single personalised push in week 1 → **+71% retention** over 2 months.
- One touch wins. Don't stack 3+ pushes in week 1.
- Best CTR hours: 8–9am or 6–8pm.
- Best CTR days: Mon–Tue.
- Worst CTR day: Saturday.
- Worst CTR hour: 4–5pm.
- Kill any push template at <1.5% CTR; healthy floor 3%.

### Aha-moment guardrail

The aha completion rate is the single most important leading indicator of paid conversion. If trial cohort aha-completion <70%, the onboarding is the problem, not the marketing.

---

## 11. Upsell mechanics

### Triggers

- **Usage-based:** user hits 70–80% of their cap.
- **Time-based:** user reaches day 15+ at low usage (re-engagement nudge).
- **Behavioural:** user has used 3+ sessions in 24h (high engagement → ready to upgrade).

### Two upsell paths

| Path | When to show |
|---|---|
| Premium tier upgrade | Engaged user (high aha-completion, multiple sessions/day) |
| Add-on pack | Churn-risk user (low D7, low aha-completion) |

### Upsell UI variants to test in W4

- **A:** Bottom-sheet modal at cap-exhaustion (default).
- **B:** Full-screen interstitial with a personalised pitch.
- **C:** Notification-only ("[name]'s [usage] runs out tomorrow") with no in-app interruption.

Run A as default. Add B and C if A's completion <40%.

### Targets

| Metric | Target |
|---|---|
| Upsell offer impression rate (% of base-tier cohort) | 60% in month 1 |
| Upsell click-through | 25% |
| Upsell completion (IAP) | 60% |
| Add-on pack rate | 15% of base cohort |

---

## 12. Cancellation flow + win-back (the biggest gap most solo devs skip)

Industry data: **cancel flows save 25–45%** of users who click Cancel. This is the highest-ROI retention tool in subscription apps.

### Three-step save funnel (premium tier cancel)

1. **Pause first.** "Take a 30-day break? Your library, preferences, and credits are saved. No charge." Highest acceptance for kids/family/consumer apps.
2. **Plan downgrade.** "Switch to [entry tier]. Keep your favourites and [reduced usage] per month."
3. **Win-back add-on.** "Take a [pack] on us when you come back." Creates a return reason without refund.
4. If all declined → confirm cancel + win-back at D7, D14, D30.

### Entry-tier cancel

1. Pause for 30 days.
2. Add-on pack as last try ("one more month of [usage] for [name]").
3. Win-back at D7, D14, D30.

### Why no discount on the base price

Discounting damages perceived pricing integrity. Once one user gets 50% off, the next one expects it. Pause + downgrade + add-on cover the 25–45% save mechanic without breaking pricing discipline.

### Win-back cadence (post-cancel)

| When | Channel | Content |
|---|---|---|
| D7 | Push (if permission) | "[name] hasn't used [app] in 7 days. We made [personalised thing]: [title]. Tap to play/use." |
| D14 | Push + email | "[name]'s [thing] for tonight is waiting. Pick up where you left off." |
| D30 | Email only | "A new [thing] for [name] — free to use, no subscription. Just one." (one-time generosity push) |

The D30 email is the lever — converts a cancel into a return install without a discount.

---

## 13. Multi-app portfolio strategy (the solo-dev money math)

### One app at a time, not in parallel

- **Do not start app B during app A's launch month.** Day-0 retention work for app A is too valuable to fragment.
- One launch month per quarter is plenty.
- Wait until app A's W4 readout before starting any new build.

### Kill or persist (decision tree)

After 6 months of *genuine launch effort* (not just a Product Hunt post):

| Signal | Decision |
|---|---|
| <$100 MRR, no signs of organic compounding | **Kill.** Document learnings. Move on. |
| <$100 MRR but cohort retention rising month over month | **Persist 3 more months.** Refgrow case: 6 months → first paying user → growing slowly. |
| $100–$500 MRR, slow growth | **Persist.** Keep iterating. Add second app *only* if this app's daily work is <2h. |
| $500+ MRR, growing | **Compound.** This is your moat. Hire help before starting app 2. |

### The 37-launch lesson

A founder shipped 37 products in 5 years. Only 1 went viral. The takeaway: virality is rare and nearly impossible to predict. **Most launches "fail" by being slower than expected. Sticking with one project produces more consistent results than chasing the next hit.**

### Cross-app audience compound

The single asset that compounds across apps: **a founder audience that trusts your judgement.**

| Asset | Reuse value |
|---|---|
| X build-in-public following | Direct → email list → first 100 of next app |
| Email list | First paying users of next app |
| IndieHackers reputation | Credibility surface for any launch |
| Niche community presence | Where you can drop next-app updates without spamming |
| Reusable tech stack | 4 weeks → 1 week build-time savings |
| Reusable copy templates (trial close, cancel flow, push) | Days of design work saved |
| Reusable paywall components | Days of build saved |

### Apps that compound (same audience, different problems)

If you find an audience, build *for them*, not *with them*. Examples:

- **Parents of young kids:** bedtime app → screen-time tracker → meal-planner → educational game.
- **Bootstrapped SaaS founders:** invoicing → metrics dashboard → onboarding flow builder.
- **Personal finance:** budget tracker → subscription manager → debt payoff coach.

Same audience, different problems. Each app cross-sells the next via email + push to your owned base.

### Money math reality check

| Paying users | Monthly revenue @ $5/mo | @ $10/mo | @ $20/mo |
|---|---|---|---|
| 100 | $500 | $1,000 | $2,000 |
| 300 | $1,500 | $3,000 | $6,000 |
| 1,000 | $5,000 | $10,000 | $20,000 |
| 3,000 | $15,000 | $30,000 | $60,000 |

Subtract Apple/Google fees (15–30% depending on tier), infra (~5%), TTS/AI/cloud costs (varies). Net: ~60–70% of gross.

The path to "quit your job money" ($5K/month net):
- 1,000 users at $7/mo, OR
- 500 users at $14/mo, OR
- 250 users at $28/mo.

A single app rarely gets to 1,000 users in 30 days. The 30-day target is **break-even** (~30–100 paying users depending on infra). Compounding to $5K+ MRR takes 6–18 months for most successful apps.

---

## 14. Anti-patterns (do not do these — every time, you will be tempted)

| Anti-pattern | Why it fails |
|---|---|
| Launching across 4 platforms on Day 1 | Algorithm slot fatigue; no single channel learns enough about your product |
| Hiding renewal copy | Damages store rating → suppresses ASO long-term |
| Optimising for cheap installs (CPI race-to-bottom) | Cheap installs without trial-start are a net loss |
| Polished brand ads first | Burn budget before learning what message works |
| Permanent free tier from Day 1 | 8× revenue penalty per RevenueCat 2026 |
| Generic "AI-powered" positioning | Loses to ChatGPT in user's head |
| Influencer paid sponsorships before signal | Spend before product loop is proven |
| Discount wheel on first paywall | Damages parent / consumer trust |
| Starting app B during app A's launch month | Steals attention from Day-0 retention work |
| Building before validating | 10 conversations beat 10 features |
| Spreading across 37 launches | Persistence with 1 product beats chasing the next hit |
| Mocking analytics until after launch | You will never go back and instrument |
| Push notifications on first open | Permission burn → no recovery |
| "Just one more feature before launch" | Every week of delay = lost compound time |
| Ignoring D7 retention | <15% means you have a product problem, not a marketing problem |

---

## 15. Build-in-public rhythm (long-term audience asset)

### On X

- 3–5 posts/week minimum. Consistency > frequency.
- One post/week with a **specific number** (signups, MRR, conversion %). Specific beats vague.
- One post/week with a **lesson learned** (what broke and how you fixed it).
- One post/week with a **public bet** ("I'm going to test X this week — will report").
- Hashtag: `#buildinpublic` (low signal in 2026 but indexes).

### Schedule template

| Day | Post type |
|---|---|
| Mon | Specific number update |
| Wed | Lesson learned |
| Fri | Bet for next week / what's shipping |
| Sun (optional) | Reflection or community engagement |

### On IndieHackers

- 1 long-form thread per launch milestone (W1 done, first paying user, first $100 MRR, first $1K MRR, etc.).
- Engagement on others' posts daily — 10:1 useful : promotional ratio.

### On Reddit

- **Problem-narrative post** at W2 D9. Not a launch announcement. ("I kept doing X manually for years — what I built.") A calendar app got 2,000 installs in 48h from a single Reddit problem post in 2026.
- Subreddit selection: where your 10 first users hang out. Not r/SideProject (that's other founders, not buyers).

### On Product Hunt

- One launch per app, around W2 D14 (after W1 gate is green).
- Schedule the launch for a Tuesday for max visibility.
- Pre-warm: 7 days before, post on X "launching on PH next week, follow for the link."

---

## 16. Tools that matter (and what to skip)

### Worth a paid month

- **Mobbin** ($30/mo) — competitor onboarding/paywall library. One month before each new app.
- **Sensor Tower** or **AppTopia** free tier — competitor revenue check.
- **AppRadar / ApppTweak / ASOMobile** — one month of one tool at launch.

### Worth paying for ongoing

- **RevenueCat** (free up to $10K MTR). Avoid building receipt verification yourself.
- **PostHog** (self-host free, or cloud free tier up to 1M events). Funnel analysis non-negotiable.
- **Resend** ($0–$20/mo). Transactional emails.

### Worth skipping for first 3 apps

- Big-name analytics (Amplitude paid, Mixpanel paid above free tier).
- Big-name ad attribution (Adjust, AppsFlyer) — you don't have the volume.
- Custom infrastructure on AWS/GCP — Vercel + Supabase + Cloudflare scales to thousands of users for ~$0–$50/mo.
- A designer for screenshots — Figma + Canva templates + AI image gen cover it.

---

## 17. The 10-step launch checklist (print this)

For every app:

1. ☐ 10 real customer conversations done (not surveys).
2. ☐ One-sentence positioning written (not "AI-powered" anything).
3. ☐ Stack chosen and wired (Supabase + RevenueCat + OneSignal + Resend + PostHog).
4. ☐ Server-authoritative entitlement E2E.
5. ☐ Day-0 aftercare screen built.
6. ☐ Loading-personalisation screen with social proof.
7. ☐ 10 funnel events instrumented and visible in dashboard.
8. ☐ Privacy policy live; Data Safety form filled.
9. ☐ 8 screenshots in 2 locales; social proof on screenshot 1 if possible.
10. ☐ Subscription products + add-on packs configured in both stores.

After this, launch.

After launch:

11. ☐ Daily organic post for 30 days.
12. ☐ W2 micro-test ($50–100) only after W1 gate.
13. ☐ Push schedule wired (Day 1 / 2 / 3 / weekly).
14. ☐ Upsell trigger wired.
15. ☐ Cancel flow wired (pause → downgrade → add-on → win-back).
16. ☐ W3 scale only after W2 gate.
17. ☐ Founder X build-in-public ≥3 posts/week.
18. ☐ Problem-narrative Reddit post at W2 D9.
19. ☐ Product Hunt launch around W2 D14 (if PH fits audience).
20. ☐ 30-day readout written. Decide: persist, kill, or expand.

---

## 18. Sources

### Primary 2026 reports
- RevenueCat — *State of Subscription Apps 2026*. <https://www.revenuecat.com/state-of-subscription-apps/>
- Adapty — *State of In-App Subscriptions 2026*. <https://adapty.io/state-of-in-app-subscriptions/>
- Business of Apps — *App Subscription Trial Benchmarks 2026*. <https://www.businessofapps.com/data/app-subscription-trial-benchmarks/>
- Airship — *Mobile App Push Notification Benchmarks 2026*. <https://www.airship.com/resources/mobile-app-push-notification-benchmarks-2026/>
- AppLaunchFlow — *ASO Best Practices 2026*. <https://www.applaunchflow.com/blog/aso-2026-guide>
- AppRadar — *What is ASO? Actionable Guide 2026*. <https://appradar.com/academy/what-is-app-store-optimization-aso>
- Retainly — *Cancel Flow Best Practices: Save 25–45% of Cancellations*. <https://getretainly.app/blog/cancel-flow-best-practices>
- Recurly — *Customer winback strategies for subscription success*. <https://recurly.com/blog/customer-winback-strategies-for-subscriptions/>
- NeoAds — *Hard paywalls convert less but earn more*. <https://neoads.substack.com/p/hard-paywalls-convert-less-but-earn>

### Indie playbooks 2026
- OpenHunts — *How to Get Your First 100 Users in 2026*. <https://openhunts.com/blog/how-to-get-your-first-100-users>
- Firsto — *How to Get Your First 1,000 Users After Launch (2026 Playbook)*. <https://firsto.co/blog/first-1000-users-after-launch>
- Prems AI — *Indie Hacker Marketing Playbook 2026*. <https://prems.ai/blog/indie-hacker-marketing-playbook-2026>

### Reddit cases consumed
- r/SideProject — *I've launched 37 products in 5 years and not doing that again* (1,904 score)
- r/IndieHackers — *Building SaaS in 2025? My best advice* (826 score)
- r/IndieHackers — *Built a tiny money app. 2,000 users. $528 revenue.*
- r/IndieHackers — *Sold my math solver for $30k after building it in a week* (841 score)
- r/IndieHackers — *Made my first dollar with an app vibe-coded in 2 days*

### High-signal X threads (via earlier Yawnly research)
- athcanft — *iOS Apps: Lazymaxxers Guide to $10K MRR*
- dams_app — *10-point ASO framework for 150K+ organic monthly downloads*
- rsalimx — *Idea-to-Store in days using Sensor Tower + Mobbin*
- RevenueCat — *55% of 3-day trials cancel on Day 0*
- matteo_spada — *Discount wheels, trust proof, polished onboarding*
- alexcooldev — *One painful feature + slideshow volume + comment-mining*
- simonecanciello — *Pink-niche rebuild + Creator Search Insights*
- jogicodes — *US TikTok from abroad operating setup*
- natiakourdadze — *CSI → trending → Astro → Sensor Tower validation loop*

---

This is a template. Adapt the language and channels per app. The architecture stays.
