---
title: Maintenance Checklist
summary: Weekly, monthly, and quarterly maintenance routines for the wiki.
type: meta
status: canonical
updated: 2026-05-02
---

# Maintenance Checklist

The wiki only stays useful if it stays small. Maintenance is manual at launch.

## Weekly (15 minutes)

- Review `inbox/quick-capture/`. Promote, file, or delete.
- Review `inbox/hermes-proposals/`. Promote into canonical pages, file, or delete.
- Review `inbox/claude-proposals/`. Same treatment.
- Promote, delete, or archive any proposal older than 14 days.
- Update the active project's `current-state.md` if reality changed.

## Monthly (30 minutes)

- Check broken links across `index.md` and active project folders.
- Check stale references in project `repo-map.md` files.
- Check page sizes against soft caps in `meta/page-schema.md`.
- Review task packets stuck in `review` or `running`.

## Quarterly (1-2 hours)

- Calendar this from day one.
- Prune stale docs (`status: stale` or `status: archived`).
- Verify canonical guidelines still match how projects build today.
- Review project decisions; mark superseded decisions and link successors.
- Archive dead research from `research/inbox/` to `research/rejected/`.
- Decide if `qmd` / search upgrade is needed (only if `rg` and `index.md` stop retrieving useful pages).

## Hard Rule

If quarterly prune is skipped twice in a row, pause expansion and clean the wiki before adding projects, automations, or new tools.

## Out of Scope at Launch

- No size-check script.
- No CI.
- No auto-archive.
- No auto-promotion.
- No qmd or vector index.

These are added later only when manual maintenance fails.
