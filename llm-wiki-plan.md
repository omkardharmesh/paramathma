# Personal Engineering Wiki — Plan for Review

**Status:** Draft for review
**Date:** 2026-05-02

---

## 1. Context

I am one developer with a growing portfolio of projects:

- 3-4 Compose Multiplatform (CMP) apps
- 1 Supabase project
- 1 new CMP project with Python BE
- More coming: web frontends, FastAPI services, Spring Java/Kotlin BEs, DevOps stuff, self-hosted servers

Stack standards repeat across projects:
- CMP architecture/conventions are the same across all CMP apps
- Supabase patterns are the same wherever Supabase is used
- FastAPI patterns are the same wherever FastAPI is used

I work with multiple AI agents (Claude Code now, possibly Hermes Agent later). Each project has its own AGENTS.md, spec files, local memory context. **Knowledge is fragmented across project repos** — patterns I solved in App A don't transfer to App B without copy-paste. I want a "proper wiki": a single source of truth that all projects (and agents) pull from.

Additional concerns layered on top:
- Subscription tracking (renewals, costs)
- Free-tier quotas across multiple cloud services (must not cross limits)
- Self-hosted server inventory
- Project-level research (a project may have BE+FE or only FE)

---

## 2. Problems we are trying to solve

| Problem | Why it hurts |
|---|---|
| Knowledge fragmented across N project repos | Copy-paste between projects, drift, re-deriving solved problems |
| Stack guides duplicated per project | Update one, forget the others; no single canonical CMP/Supabase guide |
| AI agents have no cross-project context | Each project starts cold; agent re-derives same patterns |
| Free-tier quotas easy to blow through | No central tracking → surprise bills |
| Subscription/vendor lock-in invisible | No single view of what I depend on, exit cost |
| Specs/decisions/post-mortems scattered | Can't learn from past failures across projects |
| Wikis tend to bloat and die | Bad/stale data is worse than no data |

---

## 3. Research summary (May 2026)

What the dev community is actually converging on:

- **Format consensus:** AGENTS.md is the de-facto standard, recognized by Claude Code, Codex CLI, Cursor, Gemini CLI, Copilot CLI ([deployhq guide](https://www.deployhq.com/blog/ai-coding-config-files-guide))
- **Cross-project pattern:** [Karpathy's LLM-Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) (April 2026, viral) — central markdown wiki, agent-maintained, served via MCP. Real implementations: [SamurAIGPT/llm-wiki-agent](https://github.com/SamurAIGPT/llm-wiki-agent), [Ar9av/obsidian-wiki](https://github.com/Ar9av/obsidian-wiki), [HackerNoon writeup of dev running it across 6 projects](https://hackernoon.com/how-i-built-a-self-maintaining-knowledge-base-for-6-projects-using-claude-code-and-karpathys-llm-wiki)
- **Skill sharing:** git submodule or [runkids/skillshare](https://github.com/runkids/skillshare)
- **Search:** ripgrep+frontmatter for <500 pages, then [qmd](https://github.com/tobi/qmd) (Karpathy-endorsed BM25+vector+rerank with built-in MCP)
- **Project init:** copier (NOT cookiecutter) — supports template updates
- **Linting:** lychee (linkrot) + markdownlint in CI
- **Multi-agent:** domain ownership pattern (each folder has one writer, all agents read everything)

What's NOT working in the community:
- Notion/proprietary DB as backend — universal regret on HN
- Symlinking `.claude/skills/` — known broken (claude-code #36659, #20755)
- Heavy wiki hosts (BookStack/Outline/Wiki.js) — kills agent ergonomics
- Auto-generated docs only — drift in weeks
- Per-project wikis with no central layer — duplication
- Cross-vendor quota MCP — doesn't exist in 2026

---

## 4. Proposed structure

```
~/dev/llm-wiki/
├── README.md                          # entry; what this is, how agents should use it
├── index.md                           # Karpathy pattern: agent's first read, links everything
├── log.md                             # append-only chronological activity feed
│
├── stacks/                            # WRITE-ONCE canonical guides per stack
│   ├── frontend/
│   │   ├── compose-multiplatform.md
│   │   ├── web.md
│   │   └── _migrations/               # vN→vN+1 upgrade docs
│   ├── backend/{supabase,fastapi,spring-kotlin}.md
│   ├── devops/{docker,ci-cd,self-hosted}.md
│   └── _archive/                      # superseded versions
│
├── projects/
│   ├── _template/                     # copier scaffold for new projects
│   ├── <name>/
│   │   ├── README.md                  # status, links, owner
│   │   ├── stack.md                   # uses: + deviations from canonical
│   │   ├── decisions/                 # project-local ADRs
│   │   ├── research/                  # project-specific exploration
│   │   ├── postmortems/               # incidents, what broke
│   │   └── AGENTS.md                  # symlinked into project repo via copier
│   └── README.md                      # project index w/ status grid
│
├── patterns/                          # cross-cutting architecture recipes
├── snippets/                          # copy-pasteable code (distinct from patterns)
├── prompts/                           # saved prompt library w/ frontmatter
├── runbooks/                          # ops procedures (rotate keys, deploy, restore)
├── postmortems/                       # cross-project incidents (project-local ones live in projects/)
├── decisions/                         # CROSS-PROJECT ADRs (stack picks, tooling)
├── evaluate/                          # "things to try" backlog: queued/trialed/adopted/rejected
├── vendors/                           # per-vendor: SDK pin, lock-in surface, exit cost
│
├── infra/
│   ├── servers/{mac-local,<vps>}.md
│   ├── subscriptions/<vendor>.md      # what I pay
│   └── quotas/
│       ├── <vendor>.md                # limits + thresholds, frontmatter has last_pulled
│       └── _latest.md                 # auto-generated daily by quota_sweep.py
│
├── glossary.md                        # private vocabulary (eloelo, internal terms)
├── journal/
│   └── weekly/YYYY-WNN.md             # weekly notes, dev journal
├── inbox/
│   └── YYYY-MM-DD.md                  # frictionless capture, processed nightly
│
├── agents/                            # onboarding pages FOR the agents themselves
│   ├── claude-code.md                 # what it owns, what it should/shouldn't touch
│   └── hermes.md
│
├── memory/{wins,failures}.md          # cross-project learnings
└── TASKS.md                           # shared task list, lingua franca for multi-agent
```

### Folder rationale

| Folder | Purpose | Reuse scope |
|---|---|---|
| `stacks/` | Canonical guide per technology, write once | Cross-project |
| `projects/<name>/` | Everything specific to one project | Single project |
| `patterns/` | Architecture recipes used by 2+ projects | Cross-project |
| `snippets/` | Copy-paste code (distinct from patterns) | Cross-project |
| `prompts/` | Saved prompt library | Cross-project |
| `runbooks/` | Ops procedures | Cross-project |
| `postmortems/` | Cross-project incidents | Cross-project |
| `decisions/` | Cross-project ADRs (stack picks, tooling) | Cross-project |
| `evaluate/` | Backlog of things to try | Cross-project |
| `vendors/` | Per-vendor SDK pins, lock-in, exit cost | Cross-project |
| `infra/` | Servers, subscriptions, quotas | Cross-project |
| `glossary.md` | Private vocabulary | Cross-project |
| `journal/weekly/` | Dev journal | Personal |
| `inbox/` | Frictionless capture buffer | Temporary |
| `agents/` | Per-agent onboarding profiles | Cross-project |
| `memory/` | Cross-project wins/failures | Cross-project |
| `TASKS.md` | Multi-agent shared task list | Cross-project |

---

## 5. Frontmatter schema (every page)

```yaml
---
title:
tags: [stack/cmp, project/foo, status/canonical]
created: 2026-05-02
updated: 2026-05-02
last_verified: 2026-05-02       # human or agent confirmed accurate
last_referenced: 2026-05-02     # any agent cited this page
status: canonical | draft | stale | archived
owner: human | claude | hermes
applies_to: [project-a, project-b]    # for stack guides
supersedes: stacks/frontend/_archive/compose-mp-2025-11.md
---
```

Triggers:
- `last_verified > 90 days` → auto-flagged stale
- `last_referenced > 180 days` → archive candidate
- `applies_to: []` after 30 days on a stack guide → archive candidate
- `status: stale` for >60 days → auto-archive
- `status: draft` for >30 days → promote or delete

---

## 6. Tooling stack

| Concern | Tool | Why |
|---|---|---|
| Capture | `wiki add <type> "..."` shell function → `inbox/YYYY-MM-DD.md` + auto-commit | 3-second rule |
| Mobile capture | Drafts (iOS) / Markor (Android) → daily git sync | Read on the go, capture without friction |
| Mobile read | Obsidian + Working Copy (iOS) / GitJournal (Android) | Vault is the same git repo |
| Search (now, <500 pages) | ripgrep + frontmatter filter | Karpathy's index.md-first beats vector at this scale |
| Search (later, >500 pages) | [qmd](https://github.com/tobi/qmd) | BM25+vector+rerank, MCP server built-in, Karpathy-endorsed |
| Project init | [copier](https://copier.readthedocs.io/) (NOT cookiecutter) | Supports updates as templates evolve |
| Linkrot | [lychee-action](https://github.com/lycheeverse/lychee-action) in CI | Catch broken refs |
| Markdown lint | markdownlint in CI | Structural sanity |
| Drift detection | Custom CI script: parse fenced ` ```package.json ` / ` ```pyproject ` blocks vs actual project lockfiles, flag mismatches | Single highest-leverage check |
| Agent-driven lint | Scheduled Claude Code job: pick 3 random `canonical` pages, verify against current code | Catches semantic drift no static tool can |
| Quota sweep | Python cron: hit Supabase/Cloudflare/Vercel/OpenRouter usage APIs → write `infra/quotas/_latest.md` | Git history = trend graph; commit-on-change |
| Quota alerts | Cron → ntfy.sh or Telegram bot at >80% of free tier | Beats checking dashboards |
| Secrets | 1Password CLI (`op://`) or Bitwarden refs in wiki | Wiki documents *which* secret, never the value |
| Encrypted files | `git-crypt` or `age` for `infra/subscriptions/*` if needed | Don't encrypt the whole wiki |
| Backup | GitHub private + second remote (Codeberg or self-hosted Gitea) | 3-2-1 cheaply |
| Size cap enforcement | Custom CI: line-count per folder vs cap table | Mechanical bloat prevention |
| Freshness flagging | CI script: parse frontmatter `last_verified`/`last_referenced`/`applies_to`, emit candidates | Stale detection |
| Reference tracker | Agent stop hook: bump `last_referenced` on any page cited | Distinguish live from dead pages |

---

## 7. Bloat control system (layered defense)

Bad or too much data is worse than no data. Three failure modes to prevent:
1. **Volume bloat** — too many pages, agent context overflows, search useless
2. **Stale bloat** — pages exist but no longer true, agent acts on wrong info
3. **Duplication bloat** — same fact in multiple places, drift between them

### Layer 1 — Entry gate (prevent at write time)

The single promotion question, before any new page exists:

> *"Will I or an agent reference this on a different occasion than today?"*

- **No** → goes in `inbox/`, dies there if not promoted
- **Yes, this project only** → `projects/<name>/{decisions,research,postmortems}/`
- **Yes, across projects** → `patterns/`, `stacks/`, `decisions/`, `snippets/`, etc.

If the answer is "maybe" → inbox. Inbox is the buffer that keeps junk from becoming canonical.

**Structure freeze:** no new top-level folders after week 1. If urge arises to add `experiments/` or `learnings/`, it goes in an existing folder or doesn't exist.

### Layer 2 — Size budgets (mechanical caps)

| File location | Soft cap (warn) | Hard cap (split/trim) |
|---|---|---|
| `stacks/*.md` | 500 lines | 1000 lines |
| `patterns/*.md` | 150 lines | 300 lines |
| `snippets/*.md` | 100 lines | 200 lines |
| `decisions/*.md` | 50 lines | 150 lines |
| `index.md` | 150 lines | 250 lines |
| `inbox/YYYY-MM-DD.md` | uncapped, processed in ≤7 days | — |
| Total wiki page count | 200 pages | 500 pages |

CI script emits warnings at soft cap, fails build at hard cap. Forces split/archive/trim.

### Layer 3 — Freshness tracking

Frontmatter fields enforce mechanically (see section 5). Lint script reports candidates monthly. Human acts in quarterly sweep.

Agent's job: increment `last_referenced` whenever it cites a page. One-line stop hook.

### Layer 4 — Single-source-of-truth rule

Each fact lives in exactly ONE place. If the same fact appears in two places → **merge immediately**. The duplicate is the bug.

**Link-don't-copy from external sources.** Never duplicate Supabase/Compose/Karpathy docs into the wiki. Link out. Wiki holds your opinionated take + deviations + gotchas only.

### Layer 5 — Index discipline

`index.md` is the agent's first read. Hard cap 250 lines. When it busts:
- Either too many pages (prune)
- Or hierarchy is wrong (refactor, not grow)

`index.md` busting is the single best early warning of bloat.

### Layer 6 — Agent write restrictions

Agents append to two places only: `inbox/YYYY-MM-DD.md` and `log.md`. Never auto-write to `patterns/`, `decisions/`, `stacks/`, `runbooks/`. Human curation is the moat.

Encoded in `agents/claude-code.md` and `agents/hermes.md`.

### Layer 7 — Quarterly ruthless prune

The only ritual that matters. 2 hours, every quarter. Calendar event from day one.

Checklist:
- [ ] All `last_verified > 90d`: re-verify or mark `stale`
- [ ] All `last_referenced > 180d`: archive
- [ ] All `applies_to: []` stack guides >30d: archive
- [ ] All `status: stale > 60d`: archive
- [ ] All `status: draft > 30d`: promote or delete
- [ ] Files over hard cap: split or trim
- [ ] Duplicates flagged by drift script: merge
- [ ] Broken links from lychee: fix or remove
- [ ] `inbox/` files >7 days old: process or delete
- [ ] `index.md` over cap: refactor

Skip once: recoverable. Skip twice: wiki dies.

### Layer 8 — Archive, don't delete

`_archive/` per top-level folder. Git keeps full history. Point of `_archive/` is **out of agent's context** (encoded in `agents/*.md`). Lowers psychological barrier to pruning.

### Layer 9 — Token budget for agent loads

Agent's first read is `index.md` only. Then `wiki_read` specific pages by path. Never recursive load. If `index.md` + 5 likely-relevant pages exceeds ~20k tokens, pages are too long → trim.

### Layer 10 — Default to "no entry"

When in doubt, don't add. A small sharp wiki the agent loads beats a sprawling wiki it ignores. Entries should feel earned.

---

## 8. Core practices

**Daily**
- Capture to `inbox/` freely
- Agent appends one-line entry to `log.md` after each session (Claude Code stop hook)
- Any new page must answer the promotion question

**Weekly (15 min)**
- Process `inbox/` into proper homes — empty it or delete
- Write `journal/weekly/YYYY-WNN.md`
- Glance at `TASKS.md` and `evaluate/`

**Monthly**
- Run lint pass (lychee + markdownlint + drift script + size + freshness)
- Review quota trends
- Promote/demote `evaluate/` items
- Read lint report, don't act yet — note candidates for quarterly sweep

**Quarterly (2 hrs, calendar event)**
- Run the 10-item prune checklist (Layer 7 above)
- Sweep `last_verified` on all canonical pages
- Archive superseded stack guides to `_archive/`
- Write migration doc if a stack guide bumped major version

---

## 9. Multi-agent: domain ownership (no locks)

| Domain | Owner | Reads |
|---|---|---|
| `projects/`, `decisions/`, `patterns/` | Claude Code | all |
| `infra/quotas/`, `inbox/` processing | Hermes (autonomous) | all |
| `journal/`, `glossary.md` | human | all |
| `TASKS.md` | shared via `[ ]/[~]/[x]/[!blocked]` markers | all |

Per-agent context files: `.hermes.md` for Hermes, `CLAUDE.md` for Claude Code, both `@import` shared `AGENTS.md` base. Conflicts at merge = ownership rule violated; fix the rule, not the merge.

---

## 10. Migration plan

1. **Tonight (1-2 hr)**: skeleton + `_template` (copier) + ONE stack guide (CMP — biggest reuse value) + frontmatter schema documented + `index.md` + `README.md` + git init + GitHub remote
2. **Week 1**: migrate ONE project end-to-end via `_template`. Validate copier scaffolds, AGENTS.md symlink works, agent reads wiki. Add `wiki add` shell function. Calendar quarterly prune event.
3. **Week 2**: add lychee/markdownlint CI, write `quota_sweep.py` for the 2-3 vendors that matter most, set up Telegram quota alerts
4. **Week 3-4**: migrate remaining projects incrementally as you touch them. Don't bulk-migrate — produces stale junk
5. **Month 2**: install qmd if grep slows you down. Set up agent-driven lint job
6. **Month 3**: first quarterly sweep — prove the maintenance loop works

---

## 11. Anti-patterns (don't do)

1. Notion / proprietary DB as backend — lockin, no plaintext
2. Auto-generated docs only — drift in weeks; always require human curation layer
3. Deep folder hierarchies + parallel tag taxonomy — pick folders as primary, tags as cross-cutting axes only (`stack/*`, `project/*`, `status/*`)
4. Tag inflation — keep <30 tags total or you stop using them
5. Storing raw secrets in the wiki — references only
6. Building a custom vector DB or "AI second brain" tool — qmd already wraps the right stack
7. Per-project wikis with no central layer
8. Symlinking `.claude/skills/` — known broken in worktrees (claude-code #36659)
9. Adding new top-level folders post-launch (structure freeze after week 1)
10. Letting agents auto-write to curated folders
11. Duplicating external docs into the wiki (link, don't copy)
12. Letting `inbox/` accumulate >7 days unprocessed
13. Skipping the quarterly prune

---

## 12. Honest gaps in 2026 (don't try to fix)

- **No cross-vendor quota MCP exists** — custom Python script is the answer
- **Semantic drift detection** is research-grade (RIVA, etc.) — agent-driven lint is the workaround
- **Multi-agent coordination beyond domain-ownership** is unsolved for hobbyists — don't build a lock daemon
- **Mobile capture → wiki** is still meh — live with Drafts → daily sync
- **MCP knowledge-base capability class** isn't standardized in the 2026 roadmap — pick qmd, accept some lock-in
- **agentskills.io as a true open standard** — mostly Hermes-driven, no confirmed Anthropic/OpenAI endorsement

---

## 13. What this is NOT

- Not a team wiki — single-writer, your private brain
- Not a docs site for your apps (those are separate, in their repos)
- Not a replacement for project READMEs (those still exist in repos)
- Not Notion-grade for non-tech notes (use a separate vault for personal life)
- Not a replacement for claude-mem or per-project agent memory (those handle session continuity; this handles cross-project knowledge)

---

## 14. Open questions for review

For the reviewer (you / GPT) to challenge:

1. Is the bloat control system sufficient, or do we need automated archival (vs human-decision archival)?
2. Should `agents/` profiles be in this wiki at all, or live in dotfiles?
3. Is the size cap table aggressive enough? Too aggressive?
4. Should `journal/weekly/` be in this wiki or a separate personal vault?
5. Do we need a `learnings/` or `weekly-review/` folder, or is `journal/` + `memory/` enough?
6. Is the structure freeze rule (no new top-level folders) too strict?
7. Is qmd the right bet for search at scale, or should we wait for an MCP standard?
8. Should `vendors/` and `infra/subscriptions/` merge?
9. Is domain-ownership clear enough between Claude Code and Hermes, or do we need explicit per-folder OWNER files?
10. Should we encrypt the whole `infra/` directory, or only `subscriptions/`?
11. What's missing from this plan that mature personal wikis include but I haven't thought of?

---

## 15. References

- [Karpathy LLM-Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [DAIR.AI breakdown of Karpathy pattern](https://academy.dair.ai/blog/llm-knowledge-bases-karpathy)
- [SamurAIGPT/llm-wiki-agent](https://github.com/SamurAIGPT/llm-wiki-agent)
- [Ar9av/obsidian-wiki](https://github.com/Ar9av/obsidian-wiki)
- [HackerNoon: 6-projects llm-wiki writeup](https://hackernoon.com/how-i-built-a-self-maintaining-knowledge-base-for-6-projects-using-claude-code-and-karpathys-llm-wiki)
- [tobi/qmd search tool](https://github.com/tobi/qmd)
- [copier templates](https://copier.readthedocs.io/)
- [lychee-action](https://github.com/lycheeverse/lychee-action)
- [HumanLayer: writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
- [InfoQ: AGENTS.md value reassessment](https://www.infoq.com/news/2026/03/agents-context-file-value-review/)
- [deployhq AI coding config files guide](https://www.deployhq.com/blog/ai-coding-config-files-guide)
- [runkids/skillshare](https://github.com/runkids/skillshare)
- [Aayush Ostwal: global Claude skills via submodules](https://medium.com/data-science-collective/how-to-add-global-claude-skills-across-multiple-repositories-in-your-organization-fbb0ced551fd)
- [James Donnelly: Obsidian + Claude Code workflow](https://jamesdonnelly.dev/blog/obsidian-claude-code-workflow/)
- [Stack Overflow: second brains for developers](https://stackoverflow.blog/2022/10/03/two-heads-are-better-than-one-what-second-brains-say-about-how-developers-work/)
- [Forte Labs: tagging for PKM](https://fortelabs.com/blog/a-complete-guide-to-tagging-for-personal-knowledge-management/)
- [eleanorkonik: folders vs tags](https://www.eleanorkonik.com/p/yet-another-hot-take-on-folders-versus-tags)
