---
id: yawnly-20260504-pricing-implementation-spec
project: yawnly
repo_path: /Users/eloelo/Downloads/Yawnly
status: draft
mode: plan-only
risk: medium
created: 2026-05-04
owner: human
executor: hermes
---

# Spec: Yawnly Pricing Implementation (D009)

## Goal

Implement D009-locked pricing (₹33/₹99 two-tier subs, 3-day trial, add-on packs) across the full stack. Reuses existing schema: `subscriptions`, `user_entitlements`, `entitlements`, `product_config`, `daily_usage`, `payment_orders`. Adds migration for new SKUs, tier tracking, credit split, trial tracking. Edge Functions: `verify-iap`, `check-entitlement`, `consume-credit`, `get-usage-stats`. Client: entitlement gating in `feature/paywall/` only.

## Why

D009 defines Yawnly's monetization. Old `yawnly_weekly_29`/`yawnly_monthly_89` SKUs superseded. Without this infra there's no subscription flow, no trial→paid conversion, no audio credit system.

## End Goal

Implementable task packet. Every decision resolved with evidence or flagged as needing a VPS/test/Play Console confirmation.

## Output Format

This document — `plan-only`. Reviewed against existing migrations before writing.

## Recommended Approach

1. SQL migration `005_pricing_skus_v2.sql` — seed new product_config rows, widen subscriptions.plan CHECK, add tier/monthly_credit_balance/addon_credit_balance/trial_started_at columns, create user_daily_usage, update existing RPCs.
2. Google Play Console — create subscription/add-on SKUs in draft, extract base plan IDs.
3. Edge Functions: verify-iap, check-entitlement, consume-credit, get-usage-stats.
4. Update purchase-finalizer.ts — add tier setting on SUBS upsert, credit split on INAPP grant.
5. Client — entitlement gating in `feature/paywall/` package.

## Common Failure Handling

- **Existing legacy subscribers**: keep yawnly_weekly_29/yawnly_monthly_89 active in product_config + CHECK constraint.
- **yawnly_one_time_1 INAPP purchases**: migrate story_credit_balance → addon_credit_balance.
- **Play sandbox vs production mismatch**: test ₹33→₹99 IMMEDIATE_WITH_TIME_PRORATION in closed track first.
- **SKU drift between product_config and Play Console**: validation SQL script.

## Allowed Paths

- `/Users/eloelo/Downloads/Yawnly/supabase/` (migrations + Edge Functions)
- `/Users/eloelo/Downloads/Yawnly/composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature/paywall/`
- `/Users/eloelo/Downloads/Yawnly/composeApp/src/commonMain/kotlin/com/techmo/yawnly/profile/`
- `/Users/eloelo/Downloads/MyWiki/projects/yawnly/task-packets/`

## Required Context

- `projects/yawnly/decisions.md` — D009, D007
- `projects/yawnly/pricing-economics.md` — unit economics
- `inbox/hermes-proposals/yawnly-pricing-tiers-locked-20260504.md` — full proposal
- `projects/yawnly/context.md` — project context
- `Yawnly/supabase/migrations/001_baseline.sql` — existing schema
- `Yawnly/supabase/migrations/003_product_pricing_columns.sql` — display_price/period_label
- `Yawnly/supabase/functions/_shared/purchase-finalizer.ts` — existing finalizer
- `Yawnly/supabase/functions/_shared/story-credits.ts` — existing credit grant (unchanged; RPC writes to new columns internally)
- `Yawnly/supabase/functions/payments-verify-order/index.ts` — existing verify
- `Yawnly/supabase/functions/google-webhook/index.ts` — existing webhook
- `Yawnly/supabase/functions/_shared/trial-config.ts` — trial config
- `Yawnly/supabase/functions/generate-story/service.ts` — story generation credit consumption
- `Yawnly/supabase/functions/get-entitlement/index.ts` — existing entitlement endpoint (to be deprecated)

## Non-Goals

- No Ktor/legacy/core/ changes
- No Ktor/legacy/core/ changes
- No TTS routing changes (TtsConfigClient already has KOKOROS)
- No Lottie/3D buddy
- No Room changes (D003)
- No Family/Premium tier (deferred)
- No multi-currency (deferred per migration 003)

## Stop Conditions

Standard — Allowed Paths, deps, secrets, arch drift, >3 unexpected file edits.

---

## 1. Google Play Console SKU Setup

### New SKUs

| SKU | Type | Price | Sub type | Base Plan ID (example) |
|-----|------|-------|----------|------------------------|
| `yawnly_monthly_33` | SUBS | ₹33/mo | Prepaid monthly | `monthly_33_base` |
| `yawnly_monthly_99` | SUBS | ₹99/mo | Prepaid monthly | `monthly_99_base` |
| `yawnly_addon_10` | INAPP | ₹29 | Managed product | n/a |
| `yawnly_addon_25` | INAPP | ₹59 | Managed product | n/a |
| `yawnly_addon_50` | INAPP | ₹99 | Managed product | n/a |

### Deprecated SKUs (keep active for existing subscribers)

`yawnly_weekly_29`, `yawnly_monthly_89`, `yawnly_one_time_1` — remain in `product_config` and `subscriptions.plan` CHECK constraint.

### ⚠️ Base Plan IDs

`monthly_33_base` / `monthly_99_base` are placeholder names. Google Play generates base plan IDs when you create them. The actual IDs must be extracted from Play Console after creation and confirmed before client code uses them in `SubscriptionUpdateParams`.

### Trial Setup

In Google Play Console: create each subscription base plan with a **Free Trial** offer (3 days, no payment method required). Alternatively, handle trial entirely server-side (existing trial logic) and skip Play Console trial offers — simpler, preferred. **Decision: server-side trial** — Google Play Console trial offers add complexity (3-day trial per base plan, billing date complexities). The existing trial logic in `consume_story_access` RPC already works and is more flexible.

---

## 2. Supabase Schema Changes

### 2a. Seed New SKUs in `product_config`

```sql
-- 005_pricing_skus_v2.sql
-- ============================================================

-- Seed new D009 SKUs
insert into public.product_config (sku_id, product_type, active, display_order, badge, credit_amount, display_price, period_label) values
    ('yawnly_monthly_33', 'SUBS', true, 1, null,          0,  '₹33',  'month'),
    ('yawnly_monthly_99', 'SUBS', true, 2, 'BEST VALUE',  0,  '₹99',  'month'),
    ('yawnly_addon_10',   'INAPP', true, 3, '+10',       10,  '₹29',  null),
    ('yawnly_addon_25',   'INAPP', true, 4, '+25',       25,  '₹59',  null),
    ('yawnly_addon_50',   'INAPP', true, 5, '+50',       50,  '₹99',  null)
on conflict (sku_id) do nothing;
```

`display_price` and `period_label` columns were added by migration `003_product_pricing_columns.sql`. Old SKU prices remain as-is.

### 2b. Widen `subscriptions.plan` CHECK

```sql
alter table public.subscriptions
    drop constraint if exists subscriptions_plan_check;

alter table public.subscriptions
    add constraint subscriptions_plan_check
    check (plan in (
        'yawnly_weekly_29',
        'yawnly_monthly_89',
        'yawnly_monthly_33',
        'yawnly_monthly_99'
    ));
```

### 2c. Add `tier` Column to `subscriptions`

```sql
alter table public.subscriptions
    add column if not exists tier text
    check (tier in ('trial', 'basic_33', 'premium_99'));

-- Backfill existing subscriptions so legacy paid users are not mis-reported as trial
update public.subscriptions
set tier = case
    when plan in ('yawnly_weekly_29', 'yawnly_monthly_33') then 'basic_33'
    when plan in ('yawnly_monthly_89', 'yawnly_monthly_99') then 'premium_99'
    else 'trial'
end
where tier is null;
```

Set by `purchase-finalizer` during subscription upsert via SKU→tier map (read from `product_config`, not a static map — see Section 3):

| SKU | Tier |
|-----|------|
| `yawnly_weekly_29` | `basic_33` |
| `yawnly_monthly_89` | `premium_99` |
| `yawnly_monthly_33` | `basic_33` |
| `yawnly_monthly_99` | `premium_99` |

### 2d. Trial Tracking on `user_profiles`

```sql
alter table public.user_profiles
    add column if not exists trial_started_at timestamptz,
    add column if not exists trial_ends_at timestamptz;
```

Set by `handle-trial-start` Edge Function on first call. `trial_ends_at = trial_started_at + interval '3 days'`.

### 2e. Monthly/Addon Credit Split on `user_entitlements`

```sql
alter table public.user_entitlements
    add column if not exists monthly_credit_balance int not null default 0,
    add column if not exists addon_credit_balance int not null default 0;

-- Migrate existing users: existing balance → addon_credit_balance
update public.user_entitlements
set addon_credit_balance = story_credit_balance,
    monthly_credit_balance = 0
where story_credit_balance > 0;

-- story_credit_balance becomes a computed concept (monthly + addon),
-- no longer a stored column. Keep the column for backward compat but
-- do NOT write to it for new grants — use monthly/addon split instead.
```

### 2f. Create `user_daily_usage` (new table — avoids `daily_usage` collision)

The existing `daily_usage` table (001_baseline.sql lines 130-139) has columns `date (PK)`, `request_count`. Cannot be reused for the 3-column guardrail tracking. Creating a new table:

```sql
create table if not exists public.user_daily_usage (
    id uuid primary key default gen_random_uuid(),
    user_id uuid not null references auth.users(id) on delete cascade,
    usage_date date not null default current_date,
    on_demand_count int not null default 0,
    pregen_count int not null default 0,
    text_story_count int not null default 0,
    created_at timestamptz not null default now(),
    updated_at timestamptz not null default now(),
    unique(user_id, usage_date)
);

create index if not exists idx_user_daily_usage_user_date on public.user_daily_usage(user_id, usage_date);

alter table public.user_daily_usage enable row level security;

create policy "Users can read own daily usage"
    on public.user_daily_usage for select
    using (auth.uid() = user_id);

revoke insert, update, delete on public.user_daily_usage from authenticated, anon;
grant select on public.user_daily_usage to authenticated;
grant all on public.user_daily_usage to service_role;
```

The old `daily_usage` table is left untouched — it is used by the `generate-story` Edge Function via the `increment_daily_usage` RPC. The new `user_daily_usage` table is for per-user guardrail tracking and does not conflict with the global rate-limiting purpose of `daily_usage`.

### 2g. New RPC: `get_pricing_entitlement`

Returns all D009 entitlement fields. Full PL/pgSQL body:

```sql
create or replace function public.get_pricing_entitlement(
    p_user_id uuid
) returns table (
    has_access boolean,
    tier text,
    subscription_active boolean,
    credit_balance int,
    monthly_credits_remaining int,
    addon_credits_remaining int,
    trial_remaining int,
    trial_has_started boolean,
    daily_on_demand_used int,
    daily_pregen_used int,
    daily_text_used int,
    daily_on_demand_limit int,
    daily_pregen_limit int,
    daily_text_limit int,
    monthly_on_demand_limit int,
    upsell_eligible boolean
) language plpgsql security definer set search_path = public as $$
declare
    v_sub_status text;
    v_sub_tier text;
    v_sub_plan text;
    v_sub_started_at timestamptz;
    v_sub_expires_at timestamptz;
    v_monthly_balance int;
    v_addon_balance int;
    v_trial_start timestamptz;
    v_trial_end timestamptz;
    v_daily_on_demand int default 0;
    v_daily_pregen int default 0;
    v_daily_text int default 0;
    v_days_since_start int;
    v_monthly_limit int;
    v_daily_on_demand_limit int;
    v_daily_pregen_limit int;
    v_daily_text_limit int;
begin
    -- Subscription / trial state
    select s.status, s.tier, s.plan, s.started_at, s.expires_at
    into v_sub_status, v_sub_tier, v_sub_plan, v_sub_started_at, v_sub_expires_at
    from public.subscriptions s
    where s.user_id = p_user_id
    order by s.started_at desc
    limit 1;

    -- User profile trial tracking
    select up.trial_started_at, up.trial_ends_at
    into v_trial_start, v_trial_end
    from public.user_profiles up
    where up.user_id = p_user_id;

    -- Credit balances
    select coalesce(ue.monthly_credit_balance, 0), coalesce(ue.addon_credit_balance, 0)
    into v_monthly_balance, v_addon_balance
    from public.user_entitlements ue
    where ue.user_id = p_user_id;

    -- Daily usage (today)
    select coalesce(ud.on_demand_count, 0), coalesce(ud.pregen_count, 0), coalesce(ud.text_story_count, 0)
    into v_daily_on_demand, v_daily_pregen, v_daily_text
    from public.user_daily_usage ud
    where ud.user_id = p_user_id and ud.usage_date = current_date;

    -- Determine limits by tier
    if v_sub_tier = 'premium_99' then
        v_monthly_limit := 30;
        v_daily_on_demand_limit := 3;
        v_daily_pregen_limit := 60;
        v_daily_text_limit := 10;
    elsif v_sub_tier = 'basic_33' then
        v_monthly_limit := 10;
        v_daily_on_demand_limit := 3;  -- same as premium_99
        v_daily_pregen_limit := 0;     -- no pre-gen in basic
        v_daily_text_limit := 10;
    else
        v_monthly_limit := 0;
        v_daily_on_demand_limit := 0;
        v_daily_pregen_limit := 0;
        v_daily_text_limit := 10;
    end if;

    -- Trial remaining
    if v_trial_start is not null and v_trial_end is not null then
        if v_trial_end > now() then
            trial_remaining := extract(day from (v_trial_end - now()));
        else
            trial_remaining := 0;
        end if;
    else
        trial_remaining := 3;  -- not started yet, show 3 days
    end if;

    -- Upsell eligibility (basic_33 only)
    v_days_since_start := extract(day from (coalesce(now() - v_sub_started_at, '0')));
    upsell_eligible := (
        v_sub_tier = 'basic_33'
        and v_sub_status = 'active'
        and (
            v_monthly_balance <= 2                                      -- 8+ credits consumed (of 10)
            or (v_monthly_balance = 0 and v_days_since_start >= 15)     -- day 15+ with 0 credits
        )
    );

    -- Overall access
    has_access := (
        (v_sub_status = 'active' and (v_sub_expires_at is null or v_sub_expires_at > now()))
        or (v_sub_status = 'trial' and v_trial_end is not null and v_trial_end > now())
    );

    return query select
        has_access,
        coalesce(v_sub_tier, 'trial')::text,
        v_sub_status = 'active',
        v_monthly_balance + v_addon_balance,
        v_monthly_balance,
        v_addon_balance,
        greatest(trial_remaining, 0),
        v_trial_start is not null,
        v_daily_on_demand,
        v_daily_pregen,
        v_daily_text,
        v_daily_on_demand_limit,
        v_daily_pregen_limit,
        v_daily_text_limit,
        case when v_sub_tier = 'premium_99' then 30 else 0 end,
        upsell_eligible;
end;
$$;

revoke all on function public.get_pricing_entitlement(uuid) from public, anon, authenticated;
grant execute on function public.get_pricing_entitlement(uuid) to service_role;
```

### 2h. Add `last_credit_reset_at` to `subscriptions` (idempotency guard)

```sql
alter table public.subscriptions
    add column if not exists last_credit_reset_at timestamptz;
```

### 2i. Update `reset_monthly_credits` RPC

New RPC that writes to `monthly_credit_balance` instead of `story_credit_balance`. Includes idempotency check via `last_credit_reset_at`:

```sql
create or replace function public.reset_monthly_credits(p_user_id uuid, p_tier text)
returns void language plpgsql security definer set search_path = public as $$
declare
    v_credit_limit int;
    v_last_reset timestamptz;
begin
    -- Idempotency: skip if already reset this calendar month
    select last_credit_reset_at into v_last_reset
    from public.subscriptions
    where user_id = p_user_id;

    if v_last_reset is not null and date_trunc('month', v_last_reset) = date_trunc('month', now()) then
        return;
    end if;

    v_credit_limit := case
        when p_tier = 'premium_99' then 30
        when p_tier = 'basic_33' then 10
        else 0
    end;

    insert into public.user_entitlements (user_id, monthly_credit_balance, updated_at)
    values (p_user_id, v_credit_limit, now())
    on conflict (user_id) do update set
        monthly_credit_balance = v_credit_limit,
        updated_at = now();

    -- Record reset timestamp
    update public.subscriptions
    set last_credit_reset_at = now()
    where user_id = p_user_id;
end;
$$;

revoke all on function public.reset_monthly_credits(uuid, text) from public, anon, authenticated;
grant execute on function public.reset_monthly_credits(uuid, text) to service_role;
```

### 2j. Update existing `grant_story_credits` RPC

The existing RPC writes to `story_credit_balance`. For D009, it must grant to `addon_credit_balance` instead:

```sql
create or replace function public.grant_story_credits(
    p_user_id uuid,
    p_delta int,
    p_reason text,
    p_purchase_token text default null,
    p_sku_id text default null,
    p_order_id text default null
)
returns table(applied boolean, balance int)
language plpgsql security definer set search_path = public
as $$
declare
    v_inserted_id uuid;
    v_new_monthly int;
    v_new_addon int;
begin
    if p_user_id is null then raise exception 'p_user_id is required'; end if;
    if p_delta = 0 then raise exception 'p_delta cannot be 0'; end if;
    if p_reason is null or btrim(p_reason) = '' then raise exception 'p_reason is required'; end if;

    if p_purchase_token is not null and btrim(p_purchase_token) <> '' then
        insert into public.story_credit_ledger (user_id, delta, reason, purchase_token, sku_id, order_id)
        values (p_user_id, p_delta, p_reason, p_purchase_token, p_sku_id, p_order_id)
        on conflict (purchase_token) where purchase_token is not null do nothing
        returning id into v_inserted_id;
    else
        insert into public.story_credit_ledger (user_id, delta, reason, purchase_token, sku_id, order_id)
        values (p_user_id, p_delta, p_reason, null, p_sku_id, p_order_id)
        returning id into v_inserted_id;
    end if;

    if v_inserted_id is null then
        -- Duplicate — return current balance
        select coalesce(monthly_credit_balance, 0), coalesce(addon_credit_balance, 0)
        into v_new_monthly, v_new_addon
        from public.user_entitlements where user_id = p_user_id;
        return query select false, v_new_monthly + v_new_addon;
        return;
    end if;

    -- Grant addon credits (INAPP purchases)
    insert into public.user_entitlements (user_id, addon_credit_balance, updated_at)
    values (p_user_id, p_delta, now())
    on conflict (user_id) do update set
        addon_credit_balance = public.user_entitlements.addon_credit_balance + excluded.addon_credit_balance,
        updated_at = now();

    select coalesce(monthly_credit_balance, 0), coalesce(addon_credit_balance, 0)
    into v_new_monthly, v_new_addon
    from public.user_entitlements where user_id = p_user_id;

    return query select true, v_new_monthly + v_new_addon;
end;
$$;

revoke all on function public.grant_story_credits(uuid, int, text, text, text, text) from public, anon, authenticated;
grant execute on function public.grant_story_credits(uuid, int, text, text, text, text) to service_role;
```

### 2k. Rewrite `consume_story_access` RPC

**⚠️ BREAKING CHANGE:** The existing `generate-story/service.ts` calls this RPC with the old signature `(uuid, boolean, int)` and expects `{ok, source, remaining}`. After this migration, `generate-story/service.ts` **must** be updated to call the new signature `(uuid, text)` and handle the new return shape `{allowed, remaining_credits, upsell_eligible, daily_remaining}`.

The existing RPC deducts from `story_credit_balance`. D009 deducts from monthly first, then addon, with daily guardrails:

```sql
create or replace function public.consume_story_access(
    p_user_id uuid,
    p_credit_type text default 'on_demand'  -- 'on_demand', 'pregen', 'text'
) returns table(
    allowed boolean,
    remaining_credits int,
    upsell_eligible boolean,
    daily_remaining int
) language plpgsql security definer set search_path = public as $$
declare
    v_tier text;
    v_sub_status text;
    v_monthly_balance int;
    v_addon_balance int;
    v_daily_limit int;
    v_daily_used int;
    v_new_monthly int;
    v_new_addon int;
    v_trial_end timestamptz;
    v_days_since_start int;
    v_is_upsell boolean default false;
begin
    -- Get subscription info
    select s.status, s.tier
    into v_sub_status, v_tier
    from public.subscriptions s
    where s.user_id = p_user_id
    order by s.started_at desc
    limit 1;

    -- No sub — auto-create trial (matching current prod behavior)
    if v_sub_status is null then
        insert into public.subscriptions (user_id, status, stories_used)
        values (p_user_id, 'trial', 0)
        on conflict (user_id) do nothing;
        select s.status, s.tier
        into v_sub_status, v_tier
        from public.subscriptions s
        where s.user_id = p_user_id
        order by s.started_at desc
        limit 1;
    end if;

    if v_sub_status not in ('active', 'trial') then
        return query select false, 0, false, 0;
        return;
    end if;

    -- Check trial (time + story count guardrail per D009: 10-15 stories)
    if v_sub_status = 'trial' then
        select up.trial_ends_at into v_trial_end
        from public.user_profiles up where up.user_id = p_user_id;
        if v_trial_end is null or v_trial_end <= now() then
            return query select false, 0, false, 0;
            return;
        end if;
        -- Trial users: enforce story count limit (D009: 10-15 stories)
        -- subscriptions.stories_used is incremented on each story generation
        if (select coalesce(stories_used, 0) from public.subscriptions where user_id = p_user_id) >= 15 then
            return query select false, 0, false, 0;
            return;
        end if;
        -- Increment stories_used for trial users
        update public.subscriptions set stories_used = stories_used + 1 where user_id = p_user_id;
        return query select true, 0, false, 999;
        return;
    end if;

    -- Get credit balances (FOR UPDATE prevents concurrent double-deduction)
    select coalesce(ue.monthly_credit_balance, 0), coalesce(ue.addon_credit_balance, 0)
    into v_monthly_balance, v_addon_balance
    from public.user_entitlements ue
    where ue.user_id = p_user_id
    for update;

    -- Non-on-demand (pregen, text): no credit deduction, just daily guardrail
    if p_credit_type <> 'on_demand' then
        return query select true, v_monthly_balance + v_addon_balance, false, 999;
        return;
    end if;

    -- On-demand: check daily guardrail
    v_daily_limit := case when v_tier = 'premium_99' then 3 else 3 end;

    select coalesce(ud.on_demand_count, 0) into v_daily_used
    from public.user_daily_usage ud
    where ud.user_id = p_user_id and ud.usage_date = current_date;

    if v_daily_used >= v_daily_limit then
        return query select false, v_monthly_balance + v_addon_balance, false, 0;
        return;
    end if;

    -- Deduct from monthly first, then addon
    if v_monthly_balance > 0 then
        v_new_monthly := v_monthly_balance - 1;
        v_new_addon := v_addon_balance;
    elsif v_addon_balance > 0 then
        v_new_monthly := 0;
        v_new_addon := v_addon_balance - 1;
    else
        -- No credits left
        return query select false, 0, false, v_daily_limit - v_daily_used;
        return;
    end if;

    -- Apply deduction
    update public.user_entitlements
    set monthly_credit_balance = v_new_monthly,
        addon_credit_balance = v_new_addon,
        updated_at = now()
    where user_id = p_user_id;

    -- Update daily usage
    insert into public.user_daily_usage (user_id, usage_date, on_demand_count, updated_at)
    values (p_user_id, current_date, 1, now())
    on conflict (user_id, usage_date) do update set
        on_demand_count = public.user_daily_usage.on_demand_count + 1,
        updated_at = now();

    -- Check upsell eligibility
    if v_tier = 'basic_33' then
        v_days_since_start := extract(day from (now() - coalesce(
            (select started_at from public.subscriptions where user_id = p_user_id order by started_at desc limit 1),
            now()
        )));
        v_is_upsell := (v_new_monthly <= 2) or (v_new_monthly = 0 and v_days_since_start >= 15);
    end if;

    return query select true, v_new_monthly + v_new_addon, v_is_upsell, v_daily_limit - (v_daily_used + 1);
end;
$$;

revoke all on function public.consume_story_access(uuid, text) from public, anon, authenticated;
grant execute on function public.consume_story_access(uuid, text) to service_role;
```

---

## 3. Shared TypeScript: `resolveTier` Helper

Instead of a static map (which drifts when SKUs change), derive `tier` from `product_config`:

### 3a. Add `tier` Column to `product_config`

```sql
alter table public.product_config
    add column if not exists tier text
    check (tier in ('trial', 'basic_33', 'premium_99'));

-- Set tiers for all SKUs
update public.product_config set tier = 'basic_33' where sku_id in ('yawnly_weekly_29', 'yawnly_monthly_33');
update public.product_config set tier = 'premium_99' where sku_id in ('yawnly_monthly_89', 'yawnly_monthly_99');
update public.product_config set tier = null where sku_id in ('yawnly_one_time_1', 'yawnly_addon_10', 'yawnly_addon_25', 'yawnly_addon_50');
```

### 3b. `resolve-tier.ts` — DB Lookup

```typescript
export async function resolveTier(supabase: SupabaseClient, skuId: string): Promise<string | null> {
  const { data, error } = await supabase
    .from("product_config")
    .select("tier")
    .eq("sku_id", skuId)
    .single();
  if (error || !data) return null;
  return data.tier;
}
```

Used by `purchase-finalizer.ts` when upserting a subscription row. This ensures SKU→tier mapping is single-source-of-truth in the database.

---

## 4. IAP Verification Flow

### Changes to `purchase-finalizer.ts`

Three changes only:

1. **SUBS branch**: after upserting subscription, set `tier`:
```typescript
// In the SUBS upsert, add:
tier: resolveTier(input.skuId),
```

2. **SUBS branch — initial credit grant**: after the upsert succeeds, call `reset_monthly_credits` so first-month subscribers get their credit balance immediately:
```typescript
// After subscription upsert succeeds
const tier = resolveTier(input.skuId);
if (tier) {
  await supabase.rpc("reset_monthly_credits", {
    p_user_id: userId,
    p_tier: tier,
  });
}
```

3. **INAPP branch**: grant credits via `grantCreditsForPurchase` — no change needed. The `grantCreditsForPurchase` already reads `credit_amount` from `product_config` and calls `grant_story_credits` RPC. The RPC now writes to `addon_credit_balance`.

4. **`_shared/story-credits.ts`**: no changes needed. The `grant_story_credits` RPC signature is unchanged — it now writes to `addon_credit_balance` internally. Existing callers (`grantCreditsForPurchase` in `purchase-finalizer.ts`) work without modification.

### Why Not Extend `payments-verify-order` Instead of Creating `verify-iap`

`payments-verify-order` requires a `payment_orders` row (pre-order flow). The subscription purchase path (Google Play Billing direct IAP) doesn't create a `payment_orders` row — it goes directly from `BillingClient.launchBillingFlow()` → purchase callback → client calls verify endpoint. Creating a `payment_orders` row before every subscription launch adds complexity without value.

`verify-iap` is a thin wrapper (calls `finalizePurchase`), not a third verification path. `finalizePurchase` is the shared core; `verify-iap`, `verify-order`, and `restore-purchase` all call it. This is by design per the existing architecture.

### Client Migration Path for `verify-iap`

The existing `InAppBillingHandler` uses `payments-verify-order` (pre-order flow). For new subscription SKUs (`yawnly_monthly_33`, `yawnly_monthly_99`), the client must either:
- **Option A**: Continue using the existing pre-order flow (`CreateOrderUseCase` → `payments-verify-order`). The `verify-iap` Edge Function is then not needed for subscriptions.
- **Option B**: Migrate subscriptions to direct IAP (`BillingClient.launchBillingFlow` → `verify-iap`). INAPP add-ons can still use the pre-order flow or also migrate.

**Decision needed before implementation.** The spec assumes Option B for subscriptions but does not explicitly migrate the client billing handler.

### Google Webhook

No changes needed. `google-webhook/index.ts` updates `subscriptions` by `store_subscription_id` regardless of SKU. It doesn't touch `tier` column — that's set once by `finalizePurchase`. If RTDN fires `RENEWED`, the existing webhook sets `status: "active"` correctly.

### Subscription Renewal Credit Reset

On `RENEWED` notification in `google-webhook.ts`, add a call to `reset_monthly_credits`:

```typescript
// After the existing .update() block
const { data: sub } = await supabase
  .from("subscriptions")
  .select("user_id, tier")
  .eq("store_subscription_id", purchaseToken)
  .single();

if (sub?.tier && newStatus === "active") {
  await supabase.rpc("reset_monthly_credits", {
    p_user_id: sub.user_id,
    p_tier: sub.tier,
  });
}
```

---

## 5. Edge Functions

### EF1: `verify-iap`

| | |
|---|---|
| POST | `/verify-iap` |
| Auth | Bearer |
| Body | `{ purchaseToken, skuId, productType: "SUBS"\|"INAPP" }` |
| Response | `{ ok: true, data: { tier?, creditBalance?, subscriptionActive? } }` |

Implementation: 60 lines. Calls `finalizePurchase`, reads resulting state from DB. No `payment_orders` interaction.

### EF2: `check-entitlement`

| | |
|---|---|
| GET | `/check-entitlement` |
| Auth | Bearer |
| Response | Full D009 entitlement object from `get_pricing_entitlement` RPC |

Implementation: thin wrapper around `get_pricing_entitlement` RPC. Mirrors `get-entitlement/index.ts` shape but richer response.

Replaces `get-entitlement`. Migration path:
1. Deploy `check-entitlement` alongside existing `get-entitlement`.
2. Update client code (`SupabaseAuthRepositoryImpl`, `TonightsStoryViewModel`, `ProfileViewModel`) to call `check-entitlement` instead of `get-entitlement`.
3. After all client versions in production have migrated, remove `get-entitlement` Edge Function.

**Note:** `get-entitlement` calls the old `get_story_entitlement` RPC, which is **not** being rewritten in this spec. The two systems can coexist temporarily, but `get_story_entitlement` and `consume_story_access` will diverge in their trial/credit logic. Migrate quickly.

### EF3: `consume-credit`

| | |
|---|---|
| POST | `/consume-credit` |
| Auth | Bearer |
| Body | `{ creditType: "on_demand"\|"pregen"\|"text", storyId?: string }` |
| Response | `{ ok: true, data: { remainingCredits, dailyRemaining, upsellEligible } }` |

Logic:
- Check `user_daily_usage` for guardrail limits
- For on-demand: deduct from monthly_credit_balance first, then addon_credit_balance
- For pregen: increment pregen_count (no credit deduction)
- For text: increment text_story_count (no credit deduction)
- If guardrail hit: `{ ok: false, error: "DAILY_LIMIT_REACHED" }`

### EF4: `get-usage-stats`

| | |
|---|---|
| GET | `/get-usage-stats` |
| Auth | Bearer |
| Response | `{ ok: true, data: { daily: user_daily_usage rows (30d), monthly: { total_on_demand, total_pregen } } }` |

### EF5: `handle-trial-start`

| | |
|---|---|
| POST | `/handle-trial-start` |
| Auth | Bearer |
| Response | `{ ok: true, data: { trialStartedAt, trialEndsAt } }` |

Idempotent: only sets `trial_started_at` if null. `trial_ends_at = now() + 3 days`.

---

## 6. Client: Entitlement Gating

Files in `composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature/paywall/`:

| File | Purpose |
|------|---------|
| `PricingEntitlementRepository.kt` | Fetch `check-entitlement`, cache 5 min |
| `PricingEntitlementUseCase.kt` | canGenerateAudio, canGenerateText, upsellEligible |
| `PricingViewModel.kt` | Expose state to UI, upsell triggers |
| `UpsellPromptManager.kt` | Anti-annoyance: 48h cooldown, 3/month limit |
| `AddonPurchaseFlow.kt` | Google Play Billing INAPP for add-on packs |

### DI & Navigation Registration

New client files must be registered in:
- `composeApp/src/commonMain/kotlin/com/techmo/yawnly/di/KoinInit.android.kt`
- `composeApp/src/commonMain/kotlin/com/techmo/yawnly/di/KoinInit.ios.kt`
- `composeApp/src/commonMain/kotlin/com/techmo/yawnly/app/Route.kt` (new routes if any)
- `composeApp/src/commonMain/kotlin/com/techmo/yawnly/app/App.kt` (NavHost wiring)

**Per AGENTS.md:** Missing either KoinInit platform causes runtime DI failures.

### Credit Check Flow

```
User taps "Play Audio" → PricingEntitlementUseCase.canGenerateOnDemandAudio()
  → check-entitlement (cached 5 min)
  → if monthly_credits_remaining > 0 OR addon_credits_remaining > 0:
      → consume-credit(on_demand) → proceed
  → else:
      → show buy add-ons / upgrade upsell
```

Pre-gen audio: no credit cost, just check `tier == 'premium_99'`.

Daily guardrail: `check-entitlement` returns `daily_on_demand_used` and `daily_on_demand_limit`. Client enforces and shows "3/day limit reached" message.

---

## 7. Upsell Trigger

### Formula

```
upsell_eligible = (tier == 'basic_33')
  AND (monthly_credits_remaining <= 2 OR (monthly_credits_remaining == 0 AND days_since_billing_start >= 15))
```

`monthly_credits_remaining <= 2` covers the 8th and 9th consumed. `== 0 AND day >= 15` covers full exhaustion after mid-cycle. The OR means: at 2 remaining (8th consumed), trigger fires regardless of day number. This is intentional — credit scarcity alone should prompt upgrade.

### Anti-Annoyance

```kotlin
class UpsellPromptManager(private val prefs: Preferences) {
    fun shouldShow(upsellEligible: Boolean): Boolean {
        if (!upsellEligible) return false
        val lastDismissed = prefs.getString("upsell_last_dismissed_at")
        val dismissalsThisMonth = prefs.getInt("upsell_dismissals_this_month")
        if (lastDismissed != null && hoursSince(lastDismissed) < 48) return false
        if (dismissalsThisMonth >= 3) return false
        return true
    }
    fun onDismissed() { /* set lastDismissed, increment dismissal count */ }
}
```

### Upsell UI Decision

Bottom sheet (not full-screen interstitial, not notification). This aligns with the existing `BaseBottomSheet` pattern in the Yawnly codebase. Confirmation needed from product.

---

## 8. Verification

### Migration Test

```bash
# Apply migration to local Supabase
cd /Users/eloelo/Downloads/Yawnly
supabase migration up

# Verify new SKUs seeded
supabase db query "select sku_id, display_price, period_label from product_config where active = true"

# Verify daily_usage unchanged (not dropped)
supabase db query "select column_name, data_type from information_schema.columns where table_name = 'daily_usage'"

# Verify new table exists
supabase db query "select column_name from information_schema.columns where table_name = 'user_daily_usage'"
```

### Edge Function Tests

```bash
# Deploy functions
supabase functions deploy verify-iap
supabase functions deploy check-entitlement
supabase functions deploy consume-credit
supabase functions deploy get-usage-stats

# Test check-entitlement
curl -H "Authorization: Bearer <test-token>" "https://<project>.supabase.co/functions/v1/check-entitlement"

# Expected shape:
# { "ok": true, "data": { "hasAccess": true/false, "tier": "trial"|"basic_33"|"premium_99", ... } }
```

### Client Compile Check

```bash
cd /Users/eloelo/Downloads/Yawnly
./gradlew :composeApp:compileKotlinAndroid
```

### End-to-End Flow

1. Trial start → `handle-trial-start` → `user_profiles.trial_started_at` set
2. 3 days pass → `check-entitlement` returns `hasAccess: false`
3. Purchase ₹33 → `verify-iap` → subscription upserted with `tier: basic_33`, `monthly_credit_balance: 10`
4. Consume credit → `consume-credit` → balance 9, upsell not yet eligible
5. Consume to 2 remaining → `check-entitlement` returns `upsellEligible: true`
6. Tap upgrade → Google IAP → `verify-iap` → tier changes to `premium_99`
7. Monthly renewal → Google RTDN → webhook fires → `reset_monthly_credits` sets 30 credits

## 9. Rollback Plan

| Step | Rollback SQL |
|------|-------------|
| product_config seed | `delete from product_config where sku_id in ('yawnly_monthly_33', 'yawnly_monthly_99', 'yawnly_addon_10', 'yawnly_addon_25', 'yawnly_addon_50');` |
| subscriptions.plan CHECK widen | `alter table subscriptions drop constraint subscriptions_plan_check; alter table subscriptions add constraint subscriptions_plan_check check (plan in ('yawnly_weekly_29', 'yawnly_monthly_89'));` |
| tier column | `alter table subscriptions drop column if exists tier;` |
| trial columns on user_profiles | `alter table user_profiles drop column if exists trial_started_at, drop column if exists trial_ends_at;` |
| credit split columns | `alter table user_entitlements drop column if exists monthly_credit_balance, drop column if exists addon_credit_balance;` |
| user_daily_usage | `drop table if exists user_daily_usage;` |
| new RPCs | `drop function if exists get_pricing_entitlement; drop function if exists reset_monthly_credits;` |
| updated grant_story_credits | Revert to previous version from 001_baseline.sql |
| last_credit_reset_at column | `alter table subscriptions drop column if exists last_credit_reset_at;` |

---

## Open Questions

1. **Deprecated SKU migration**: `yawnly_one_time_1` already granted credits to existing users. Migrated to `addon_credit_balance` via Section 2e SQL. Confirm no other references.
2. **Base plan IDs**: confirm after creating subscriptions in Play Console. Placeholder IDs in this spec.
3. **Upsell UI**: bottom sheet (recommended) vs full-screen interstitial — confirm with product.
4. **On-demand queue priority**: FIFO for all vs ₹99 priority — decision deferred, not in spec scope.
5. **Trial start trigger**: first sign-in (per Section 2d) — confirmed by D009 "3-day trial, no CC." `trial_started_at IS NULL` check handles it. Question resolved.
6. **Pre-gen audio for basic_33**: no pre-gen access (0/day per RPC limits). Only teaser audio from monthly credits.
7. **Trial model alignment**: D009 says "10-15 personalised stories" for trial. The `consume_story_access` RPC now enforces both time (`trial_ends_at`) and story count (`subscriptions.stories_used >= 15`). Confirm if 15 is the correct hard limit or if it should be 10 with soft allowance.

---

## Summary of Changes

| File | Type | Change |
|------|------|--------|
| `supabase/migrations/005_pricing_skus_v2.sql` | Migration | New SKUs, schema changes, RPCs |
| `supabase/functions/_shared/resolve-tier.ts` | TS (new) | SKU→tier mapping |
| `supabase/functions/_shared/purchase-finalizer.ts` | TS (edit) | Add tier on SUBS upsert |
| `supabase/functions/google-webhook/index.ts` | TS (edit) | Add `reset_monthly_credits` on RENEWED |
| `supabase/functions/verify-iap/index.ts` | Edge Function (new) | IAP verification |
| `supabase/functions/check-entitlement/index.ts` | Edge Function (new) | Full entitlement check |
| `supabase/functions/consume-credit/index.ts` | Edge Function (new) | Atomic credit consumption |
| `supabase/functions/get-usage-stats/index.ts` | Edge Function (new) | Usage stats |
| `supabase/functions/handle-trial-start/index.ts` | Edge Function (new) | Trial init |
| `supabase/functions/generate-story/service.ts` | TS (edit) | Update `consumeStoryAccess` call to new RPC signature |

| `composeApp/.../feature/paywall/PricingEntitlementRepository.kt` | Client (new) | Server entitlement data |
| `composeApp/.../feature/paywall/PricingEntitlementUseCase.kt` | Client (new) | Business logic |
| `composeApp/.../feature/paywall/PricingViewModel.kt` | Client (new) | UI state + triggers |
| `composeApp/.../feature/paywall/UpsellPromptManager.kt` | Client (new) | Anti-annoyance |
| `composeApp/.../feature/paywall/AddonPurchaseFlow.kt` | Client (new) | Add-on IAP |
| `composeApp/.../di/KoinInit.android.kt` | Client (edit) | Register new paywall modules |
| `composeApp/.../di/KoinInit.ios.kt` | Client (edit) | Register new paywall modules |
| `composeApp/.../app/Route.kt` | Client (edit) | Add new pricing routes |
| `composeApp/.../app/App.kt` | Client (edit) | Wire new routes in NavHost |


