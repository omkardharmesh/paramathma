---
title: Yawnly Backlog
summary: Tiny project-local priority view. Not a full product backlog.
type: project-context
status: canonical
updated: 2026-05-02
---

# Yawnly — Backlog

Soft cap: 150 lines. If it grows past that, move items into task packets or to GitHub/Linear.

## release-critical

- Server-authoritative entitlement verification path is exercised end-to-end before any subscription change ships (D007).

## next

- Edge Function ↔ Kotlin DTO ↔ migration alignment audit for `generate-story`, `generate-audio`, `payments-verify-order`, `get-entitlement`.
- Story Library scaffolding spec (D008).

## later

- 3D / Lottie buddy work (gated by D004 revisit triggers).
- Offline reading via narrow cache layer (only if D003 revisit triggers; not via Room blanket).
- Recommendations / personalization for the Library (only after Library lands).

## parked

- Migration to a single network stack (delete legacy Ktor scaffold). Parked until either (a) Ktor scaffold is unused, or (b) a third-party REST need genuinely requires Ktor.
- Vector / RAG-based story generation. Parked behind D005 revisit.
- Multi-user / family-account model. Parked until single-user retention is validated.

## done (2026-05-02)

- **Stale-doc reference cleanup.** Research-only → plan-only → implemented. 6 stale refs pruned, bridge files thinned to point at MyWiki. PR: [#15](https://github.com/omkardharmesh/YawnlyCMP/pull/15).
