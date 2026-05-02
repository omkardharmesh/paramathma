---
title: Supabase Edge Functions
summary: File structure, env secret handling, request/response envelopes, JWT/auth, errors, webhooks, migrations.
type: guideline
status: canonical
owner: human
created: 2026-05-02
updated: 2026-05-02
last_verified: 2026-05-02
applies_to: [global]
---

# Supabase Edge Functions

Reusable rules for Deno-based Supabase Edge Functions. Project-specific function names, business logic, and product flows stay in `projects/<project>/`.

## File Structure

```
supabase/functions/
  _shared/                # shared helpers (cors, error, auth, supabase client)
  <function-name>/
    index.ts              # entry point — handler only
    [helpers].ts          # function-specific helpers
```

- One responsibility per function.
- Helpers shared across functions live in `_shared/`. Project-specific helpers stay inside the function folder.
- Keep `index.ts` small. Push business logic into named helpers so the handler is easy to scan.

## Env / Secret Handling

- Read secrets via `Deno.env.get("SECRET_NAME")`. Fail fast if a required secret is missing.
- Never hardcode secrets in source. Never log secret values.
- Function-side secrets (service role key, IAP API keys, third-party keys) live in the Supabase secrets store, not in the repo.
- Document secret **names** in the project's `repo-map.md` or backend doc. Do not store the secret value in the wiki.

## Request Envelope

- Use `application/json` for request bodies.
- Validate the body before doing anything else. Reject with 4xx + a normalized error envelope when invalid.
- For authenticated endpoints, require a valid JWT. Read the user via the Supabase client constructed with the user's `Authorization: Bearer <jwt>`.

## Response Envelope

Use a single normalized response envelope across functions. Pick one shape per project and stick to it. A typical pattern:

```jsonc
// success
{ "ok": true, "data": { ... } }

// error
{ "ok": false, "error": { "code": "string", "message": "string" } }
```

The Kotlin DTO layer must match exactly. If the function changes the envelope, update the DTOs in the same change.

## JWT / Auth

- A function that touches user data must require a JWT.
- A public function (webhook receiver, healthcheck) that does not require a JWT must verify legitimacy another way (signature, allowlist, shared secret in header).
- A function may use the **service role** internally only after authenticating the caller. Never accept a service-role-issued call from an untrusted client.

## Error Normalization

- Catch all thrown errors at the handler boundary.
- Map known errors to typed codes (e.g. `unauthorized`, `not_found`, `conflict`, `rate_limited`, `bad_request`, `internal`).
- Never leak stack traces or third-party error bodies in responses.
- Log enough server-side context (request id, user id, code) to debug without exposing secrets.

## Webhooks and Idempotency

Webhook receivers (payments, IAP, push providers) require extra care:
- **Verify the signature first.** Reject unsigned or invalid-signature requests with 4xx.
- **Idempotency.** Persist the provider's event id and short-circuit duplicate deliveries. Providers retry.
- **Timeouts.** Acknowledge fast (200) after persisting; defer heavy work to a background path or a follow-up function call.
- **Replay safety.** Treat any side effect (entitlement grant, email send, fulfillment) as something the same event id might trigger twice and design accordingly.

## Migration Alignment

When a function changes the database (insert/update/delete/return shape), the change must land in three places in lock-step:
1. SQL migration file under `supabase/migrations/`.
2. Edge Function code.
3. Kotlin DTOs and `toDomain()` mappers.

Drift between any of the three is a P1 review finding.

## Verification Before Shipping

- Test the function locally (`supabase functions serve`) or against staging before pointing the mobile client at it.
- Run a migration smoke check before shipping: a clean DB applies all migrations cleanly.
- Smoke-test the success path and at least one error path from a real client (not just curl).

## Do/Don't

- **Do** keep `index.ts` small and push logic into named helpers.
- **Do** use a single response envelope per project and match it in DTOs.
- **Do** verify webhook signatures and dedupe by event id.
- **Don't** hardcode secrets, log secrets, or send secrets in responses.
- **Don't** ship a function change without updating the matching migration and DTO.
- **Don't** accept long-running synchronous work in a webhook handler.
