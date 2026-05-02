---
title: Yawnly Linked Guidelines
summary: Exact global guidelines that apply to Yawnly.
type: project-context
status: canonical
updated: 2026-05-02
---

# Yawnly — Linked Guidelines

## Read for any Yawnly work

- `guidelines/agents/hermes-delegation.md`

## Read for CMP / shared / Android work

- `guidelines/frontend/compose-multiplatform.md`
- `guidelines/frontend/compose-ui.md`

## Read for backend / Supabase work

- `guidelines/backend/supabase.md`
- `guidelines/backend/supabase-edge-functions.md`

## Golden examples

`golden-examples/cmp/` and `golden-examples/supabase/` are launch placeholders. No Yawnly examples have been promoted yet. When a real Yawnly file becomes a clear pattern (e.g. a clean `*ScreenRoot`/`*Screen` pair, a clean `SupabaseBaseRepository` extension, a clean Edge Function), promote it via a wiki proposal.

## Notes

- Project-specific tokens (SDP/SSP, `UniversalColor`, `BaseActionButton`, `noRippleSafeClick`, `HeightSpacer`/`WidthSpacer`) override generic naming in the global Compose UI guideline. See `projects/yawnly/context.md` for the full list.
- The global Supabase guideline assumes one network stack; Yawnly has Supabase **and** a legacy Ktor skeleton. Use Supabase for all new product work.
