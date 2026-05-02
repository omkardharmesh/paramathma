---
title: Supabase Usage
summary: Supabase client boundaries, RLS, schema verification, secrets, and DTO/migration alignment.
type: guideline
status: canonical
owner: human
created: 2026-05-02
updated: 2026-05-02
last_verified: 2026-05-02
applies_to: [global]
---

# Supabase Usage

Reusable Supabase rules for any project that uses Supabase as its product backend. Project-specific schemas, plan SKUs, and feature flags stay in `projects/<project>/context.md` or `decisions.md`.

## Client / Key Boundary

- The mobile client uses the **anon key** only.
- The **service role key** never reaches a mobile client. Service-role operations live behind Edge Functions or trusted servers.
- Anon-key calls are subject to RLS. Service-role calls bypass RLS — treat that as a privileged path that must be small, audited, and tested.

## When the Mobile Client Calls Supabase Directly

Use direct SDK calls (e.g. `io.github.jan-tennert.supabase`) when:
- The operation is allowed by RLS for the authenticated user.
- The operation does not require trusted server logic (no aggregation across users, no privileged writes, no third-party API key).
- The shape returned to the client is the shape the UI consumes.

## When to Route Through an Edge Function

Use an Edge Function when any of these are true:
- The operation needs the service role key.
- The operation calls a third-party API with a secret (TTS, payments, IAP receipt verification).
- The operation must be idempotent or signed (webhooks, payment verification).
- The operation aggregates across users or runs business rules that should not be replicated on the client.

## Client Wrapper Pattern

Project repositories should extend a Supabase-aware base (e.g. `SupabaseBaseRepository`) that provides:
- Coroutine-friendly call execution (e.g. `executeSupabaseCall`).
- Error normalization at the data boundary.
- Consistent retry/backoff policy where appropriate.

Do not embed retry loops in ApiService or in use cases. Retry/error strategy belongs at the repository boundary.

## RLS

- Every product table in Supabase must have RLS enabled.
- Every policy must be reviewed against the actual access pattern, not assumed.
- A new feature that touches Supabase must update RLS in the same migration as the schema change.

## Schema Verification Before SQL

- Always read the actual table schema before writing SQL: column names, types, constraints, defaults.
- Inspect migration files or `\d table_name`. Never guess column names from memory.
- For remote checks against the linked project, prefer linked queries:
  - `rtk supabase db query --linked "<SQL>"`
  - `rtk supabase db query --linked --output json "<SQL>"` for structured logs.
- Use local-only queries explicitly when intended:
  - `rtk supabase db query --local "<SQL>"`

## Secrets

- The wiki never stores raw secrets.
- Allowed in docs: env var names, Supabase secret names (e.g. `SUPABASE_SERVICE_ROLE_KEY` as a name), `op://` references, Bitwarden item names/IDs.
- The mobile app reads the anon key from a gitignored project-local config (e.g. `appkeys.properties`) accessed via BuildConfig.
- A secret leaked into git history requires immediate rotation.

## Migration / DTO / Function Alignment

When a Supabase table or Edge Function changes:
1. Update the migration file.
2. Update the Edge Function payload (request and response shapes).
3. Update the Kotlin DTOs (`@Serializable` types) and `toDomain()` mappers.
4. Verify all three by running the Edge Function locally or against staging before shipping.

A change that lands in only one of the three layers is a P1 finding.

## Subscriptions / Billing

When billing or entitlement is involved:
- Treat the **server** (Supabase tables + Edge Functions) as the source of truth.
- Do not let the mobile client unilaterally grant or extend entitlement.
- Verify IAP receipts server-side (Edge Function calling Apple/Google) before persisting entitlement.
- The client may cache entitlement for UX, but must reconcile against the server on app start and on entitlement-relevant events.

## Do/Don't

- **Do** keep the service role key on the server side.
- **Do** read the schema before writing SQL.
- **Do** keep migration, function, and DTO in lock-step.
- **Don't** expose the service role key to a mobile client or to a public Edge Function endpoint.
- **Don't** copy SKU names, plan IDs, or pricing into this guideline — those are project-specific.
