---
title: MyWiki Project Onboarding
summary: XHIGH-complexity walkthrough for boarding new projects into MyWiki. Covers file templates, workflow phases, error recovery, and hard constraints.
type: guideline
status: draft
owner: human
applies_to: [global]
created: 2026-05-14
updated: 2026-05-14
last_verified: 2026-05-14
supersedes:
---

# Onboard New Projects into MyWiki — XHIGH

## Pre-flight (load these first)

### 1. Load the onboarding skill
`skill_view(name="mywiki-project-onboarding")`
Follow its procedure exactly.

### 2. Read MyWiki index
`read_file(path="/Users/eloelo/Downloads/MyWiki/index.md")`
Note the current Active Projects list format and the overall structure.

### 3. Read the page schema
`read_file(path="/Users/eloelo/Downloads/MyWiki/meta/page-schema.md")`

Key caps to respect:
| File | Soft cap |
|------|----------|
| `context.md` | 150 lines |
| `linked-guidelines.md` | 150 lines (keep short) |
| `current-state.md` | 100 lines |
| `index.md` | 150 lines total |

### 4. Read a reference project for content style
`read_file(path="/Users/eloelo/Downloads/MyWiki/projects/yawnly/context.md")`
Study the tone — declarative, specific about filesystem paths, exact build commands, hard constraints section. Each section is a collection of labeled facts, not prose paragraphs.

### 5. Read the Hermes delegation guideline (for repo-local handoff context)
`read_file(path="/Users/eloelo/Downloads/MyWiki/guidelines/agents/hermes-delegation.md")`
Specifically the Repo-Local Agent Handoffs section — because once onboarded, these projects may need handoff docs for sandboxed workers.

---

## Workflow (Step by Step)

### Phase 1: Present & Collect (INTERACTIVE)

**Step 1.1:** Greet the user, confirm the onboarding goal.

**Step 1.2:** Collect all required info. Use this checklist:

| Info | Required | Question phrasing |
|------|----------|-------------------|
| Project name (slug) | ✅ | "What's the slug? e.g. `fashion-app`, `yawnly-marketing`" |
| One-line summary | ✅ | "Give me a ~15-word description for the index.md entry" |
| Product description | ✅ | "What it does, who for, how it makes money?" |
| Repo path | ✅ | "Full path on disk" |
| Tech stack | ✅ | "Language, framework, DB, etc. If undecided: TBD" |
| Build commands | ⚠️ | "Compile/run/deploy. If undecided: TBD" |
| Hard constraints | ✅ | "Non-negotiables, do-not-touch areas" |
| External services | ⚠️ | "APIs, third-party deps" |
| Global guidelines to link | ⚠️ | "Which MyWiki guidelines apply?" |

**Step 1.3:** For fields the user is unsure about:
- Stack → set as `TBD` in context.md
- Build commands → set as `TBD`
- Summary → draft a placeholder, flag it

### Phase 2: Create Files (EXECUTE)

For **each** app, create these four files.

#### 2.1 `projects/<slug>/context.md`

Template:

```yaml
---
title: <Project Name> Project Context
summary: <15-word description matching index.md entry>
type: project-context
status: draft
updated: <YYYY-MM-DD>
---
```

Required sections IN ORDER:

1. **## Product**
   - One-liner about what the app does
   - Target audience
   - Monetization model (or TBD)
   - Status: ideation / in development / MVP / etc.

2. **## Repo**
   - Path: `/Users/eloelo/Downloads/<RepoName>` (or TBD)
   - Remote: (or TBD)
   - Package root: (or TBD)

3. **## Stack** (or TBD)
   - Language / framework
   - Frontend
   - Backend / DB
   - Auth
   - DI (if relevant)
   - Any other tools

4. **## Build Commands** (or TBD)
   - Dev server / hot reload
   - Production build
   - Deploy

5. **## Hard Constraints** (or TBD)
   - Any non-negotiables the user stated
   - Do-not-touch areas
   - Architecture rules

6. **## What Not To Touch** (or TBD)
   - Usually empty for new projects

7. **## External Services**
   - Planned third-party APIs
   - Payment processors
   - Social platform APIs

8. **## Current Release Status**
   - Link to `current-state.md`

**Style rules:**
- Bullet lists of labeled facts (like Yawnly's style)
- No prose paragraphs
- Be specific about filesystem paths (e.g., `/Users/eloelo/Downloads/FashionApp`)
- Mark any guessed/missing info with `(TBD — ask user to confirm)`
- Do NOT store secrets, API keys, or raw credentials
- Reference env var names instead (e.g., `STRIPE_SECRET_KEY from .env.local`)

#### 2.2 `projects/<slug>/linked-guidelines.md`

```yaml
---
title: <Project Name> Linked Guidelines
summary: Which global guidelines apply to this project.
type: project-context
status: draft
updated: <YYYY-MM-DD>
---
```

For new apps that don't use CMP or Supabase:
- Most MyWiki guidelines won't apply
- Create a minimal file stating that

Content:

```markdown
# <Project Name> — Linked Guidelines

## Read for architectural decisions

- No global MyWiki guidelines apply yet.
- As stack decisions are made, link relevant guidelines here.

## Notes

- <Project Name> does not use Compose Multiplatform, Supabase, or the Yawnly tech stack.
- Global guidelines in `guidelines/frontend/`, `guidelines/backend/`, and `guidelines/agents/` may become relevant once the stack is decided.
```

#### 2.3 `projects/<slug>/current-state.md`

```yaml
---
title: <Project Name> Current State
summary: Ideation phase — no code written yet.
type: current-state
status: draft
updated: <YYYY-MM-DD>
---
```

Content:

```markdown
# <Project Name> — Current State

## Working

- Nothing yet. The project is in ideation.

## In Progress

- Onboarding into MyWiki (this file).

## Blocked

- Awaiting stack decision, repo creation, and product direction finalization.

## Risks

- Undefined tech stack — risk of over-engineering during discovery.
- Undefined monetization model — affects architecture decisions.
- No repo exists yet — cannot validate build commands or deployment.

## Next Steps

1. Finalize app name and product direction.
2. Choose tech stack.
3. Create repo.
4. Build MVP.
```

#### 2.4 Update `projects/<slug>/index.md`

Add to the **Active Projects** section, keeping alphabetical order:

```markdown
- [<Project Name>](projects/<slug>/context.md) — <one-line summary from context.md>
```

### Phase 3: Validate (CHECK)

After creating all files, verify:

1. **Frontmatter correct** — No typos in `type`, `status`. Dates are today.
2. **All 4 files exist per project** — `context.md`, `linked-guidelines.md`, `current-state.md`, plus the index entry.
3. **No schema violations** — frontmatter matches `page-schema.md` minimal schema.
4. **No secrets exposed** — no API keys, tokens, or raw credentials in any file.
5. **Size caps respected** — `context.md` under 150 lines, `current-state.md` under 100 lines.
6. **No deviant file mutations** — only created files in `projects/<slug>/` and one edit to `index.md`. No modifications to `guidelines/`, `meta/`, `inbox/`, or other project files.

### Phase 4: Present Summary (REPORT)

Present to the user in this format:

```
## ✅ Onboarded: <App Name>
📁 projects/<slug>/
├── context.md         (N lines)
├── linked-guidelines.md (N lines)
└── current-state.md   (N lines)
📝 index.md — added entry

## ⚠️ TBD / Draft fields
- <App>: [list any TBDs]

## 🔮 What happens next
1. When you decide on a stack/build commands, update context.md
2. When you create a repo, update the repo path
3. Task packets go in `projects/<slug>/task-packets/`
4. Hermes will know about these projects in future sessions via MyWiki
```

---

## Error Recovery

### If the user doesn't know the answer to a required field

- Mark as `TBD` in the file
- List it in the summary's "TBD / Draft fields" section
- Do NOT block on it — create the files with placeholders

### If the user changes their mind mid-onboarding

- Delete any files already created (outside index.md — just don't add the index entry yet)
- Start over

### If the user provides contradictory info

- Flag it: "You said X earlier, but now you're saying Y. Which is correct?"
- Do NOT silently accept both

### If the slug conflicts with an existing project

- Scan `projects/` directory first
- Propose a different slug

### If the user doesn't respond

- Create the files with what you have, mark everything as `draft`, present the TBD list
- The files are usable as-is — future sessions will see the drafts and know the project exists

---

## Do NOT

- ❌ Do NOT create task packets during onboarding
- ❌ Do NOT modify global guidelines or meta files
- ❌ Do NOT store secrets in any file
- ❌ Do NOT create repo directories unless explicitly asked
- ❌ Do NOT write code, scaffolds, or config files
- ❌ Do NOT commit to git (the index.md change stays local until user approves)
- ❌ Do NOT assume a tech stack — if user says "not sure yet", mark TBD
- ❌ Do NOT add write lane entries to index.md — that's for proposals only
- ❌ Do NOT bulk-load wikiignore pages (see meta/wikiignore.md)

## See Also

- The full xhigh reference with extended examples is at the `mywiki-project-onboarding` skill's `references/onboarding-prompt-xhigh.md`
- `skill_view(name="mywiki-project-onboarding")` for the canonical Hermes onboarding procedure
