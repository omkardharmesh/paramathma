---
title: Hermes Setup Extension Plan
id: hermes-setup-extension-20260502
project: hermes-config
type: implementation-plan
status: draft
created: 2026-05-02
mode: plan-only
---

# Hermes Setup Extension — Implementation Plan

## Summary

Three features to extend the Hermes pilot setup:
1. **`/project_it`** — project registry skill
2. **Multi-profile Telegram bots** — dev, AI research, personal
3. **Encrypted cloud backup** — periodic, safe, automated

All three are achievable with existing Hermes primitives (profiles, skills, memory, cron, backup). No Hermes source code changes required.

---

## Feature 1: `/project_it` Skill

### Recommendation: Skill + JSON registry (no code changes)

Hermes skills are loaded into sessions and can instruct the agent how to respond to specific commands. A `project-it` skill would teach Hermes to manage a project registry.

### Where the registry lives

```
~/.hermes/project-registry.json
```

Single file. Loaded by the skill on every session. Updated by the agent via the `write_file` tool when the user adds/selects projects.

### Registry schema

```json
{
  "version": 1,
  "current": "yawnly",
  "projects": {
    "yawnly": {
      "name": "Yawnly",
      "repo_path": "/Users/eloelo/Downloads/Yawnly",
      "wiki_path": "/Users/eloelo/Downloads/MyWiki/projects/yawnly",
      "wiki_index": "/Users/eloelo/Downloads/MyWiki/index.md",
      "stack": ["cmp", "supabase", "kotlin"],
      "added": "2026-05-02"
    }
  }
}
```

### Behavior

- `/project_it list` — Read registry, show all projects with active marker
- `/project_it current` — Show current project name, path, wiki
- `/project_it select <name>` — Set `current` in registry, announce switch, offer to load wiki context
- `/project_it add <name> --repo <path> --wiki <path>` — Register a new project
- `/project_it remove <name>` — Deregister (does not delete files)
- `/project_it share <profile>` — Write current session context to `inbox/cross-profile/from-<current_profile>/` so another profile bot can read it

### How it works technically

- The skill is loaded with `--skills project-it` or pinned for auto-load
- When the agent sees `/project_it` in a message, the skill instructs it to:
  1. Read `~/.hermes/project-registry.json`
  2. Parse the command
  3. Respond accordingly
  4. For `select`: switch the terminal working directory, load wiki context into the session
- The skill also tells Hermes to save the active project to memory so cross-session context survives

### Why skill over slash-command or custom tool

- **Slash commands** require modifying `hermes_cli/commands.py` — maintenance burden on Hermes upgrades
- **Custom tools** require `tools/project_manager.py` + registry registration — same problem
- **Skill** leverages existing infrastructure: SKILL.md file, loaded per session, uses built-in file/memory tools
- Skills are portable across Hermes installs, profiles, and upgrades

### Files created

```
~/.hermes/skills/project-it/SKILL.md       (the skill)
~/.hermes/project-registry.json            (the data)
```

### Exact skill content (SKILL.md)

The skill would instruct Hermes to:
- Recognize `/project_it` prefixed messages as project commands
- Maintain the registry JSON
- On `select`: read the project's wiki context files, set terminal CWD to the repo path, announce the active project
- On `list`: display a compact table from the registry
- Save the current project to memory (`memory` tool) so future sessions know the active project

Zero Hermes source edits. The skill uses existing `read_file`, `write_file`, `memory`, `terminal` tools.

---

## Feature 2: Multi-Profile Telegram Bots

### Current state

- 1 profile: `default`
- 1 gateway: launchd service `ai.hermes.gateway` running the default profile
- 1 Telegram bot, single-user allowlist

### Target state

4 profiles, each with its own Telegram bot:

- **default** — General / pilot. Scope: MyWiki + Yawnly + scratch. Bot: existing.
- **dev** — Software development. Scope: Yawnly repo + coding repos. Bot: new.
- **research** — AI/ML research. Scope: broader web + research tools. Bot: new.
- **personal** — Personal productivity. Scope: Apple tools, notes, reminders. Bot: new.

### How Hermes profiles and gateways work together

Each Hermes profile has isolated:
- `~/.hermes/profiles/<name>/config.yaml` — independent model, tools, personality
- `~/.hermes/profiles/<name>/.env` — independent API keys (including Telegram bot token)
- `~/.hermes/profiles/<name>/skills/` — independent skill set
- `~/.hermes/profiles/<name>/sessions/` — independent session store
- `~/.hermes/profiles/<name>/SOUL.md` — independent persona

The gateway reads the profile's config on startup. Running multiple gateways means multiple launchd services, each pointed at a different profile.

### Implementation steps

**Step 1: Create Telegram bots**
- Create 3 new bots via @BotFather on Telegram
- Receive 3 bot tokens
- Store tokens securely (Bitwarden / 1Password)

**Step 2: Create profiles**

```bash
# Dev profile — clone default config, override tools/scope
hermes profile create dev --clone
hermes --profile dev config set model.default deepseek-v4-pro
hermes --profile dev config set model.provider deepseek
hermes --profile dev config set display.personality technical
# Add telegram token to ~/.hermes/profiles/dev/.env
# TELEGRAM_BOT_TOKEN=<dev-bot-token>

# Research profile — broader web/search
hermes profile create research --clone
hermes --profile research config set display.personality technical
# TELEGRAM_BOT_TOKEN=<research-bot-token>

# Personal profile — productivity-focused
hermes profile create personal --clone
hermes --profile personal config set display.personality helpful
# TELEGRAM_BOT_TOKEN=<personal-bot-token>
```

**Step 3: Install separate gateway services**

Each profile needs its own launchd plist:

```bash
# Stop the existing gateway first (if it uses default)
hermes gateway stop

# Install per-profile gateways
HERMES_PROFILE=dev hermes gateway install --name ai.hermes.gateway.dev
HERMES_PROFILE=research hermes gateway install --name ai.hermes.gateway.research
HERMES_PROFILE=personal hermes gateway install --name ai.hermes.gateway.personal
HERMES_PROFILE=default hermes gateway install --name ai.hermes.gateway.default
```

**Note:** `--name` may not be a real flag. If Hermes doesn't support named gateway services, the alternative is:
- Manually create launchd plists in `~/Library/LaunchAgents/`
- Each plist sets `HERMES_PROFILE=<name>` environment variable and runs `hermes gateway run --replace`
- Use `launchctl load/unload` to manage

**Step 4: Isolate tools and paths**

- **dev**: terminal, file, github, delegation. Paths: Yawnly, coding repos.
- **research**: web, browser, terminal, file, vision, search. Paths: MyWiki, research dirs.
- **personal**: apple-notes, apple-reminders, imessage, maps, spotify. Paths: ~/Desktop, ~/Documents.
- **default**: all (current). Paths: MyWiki, Yawnly, scratch.

Tool isolation: use `hermes --profile <name> tools` to enable/disable toolsets per profile.
Path isolation: enforced via memory + task packet conventions — not a Hermes config primitive, but the wiki/skills can encode it.

### Allowed-paths enforcement strategy

Hermes doesn't have a built-in "path allowlist" per profile. Instead:
- Encode allowed paths in each profile's SOUL.md or a skill loaded at startup
- The agent respects the constraint (as it does with task packet Allowed Paths)
- For stronger enforcement: use a cron job that audits session logs for out-of-scope writes

### Cross-profile knowledge sharing

MyWiki is shared across all profiles. This is intentional — it acts as the bridge between siloed bots.

**How it works:**
- Every profile can read and write MyWiki
- Each profile has its own memory, sessions, persona, and Telegram bot
- Research bot writes findings to MyWiki inbox
- Dev bot reads them via First Read Protocol and uses them

**Handoff convention:**
Create a folder per source profile under MyWiki:

```
MyWiki/inbox/cross-profile/
  from-research/
  from-dev/
  from-personal/
```

The `/project_it share <profile>` command writes the current session summary to `inbox/cross-profile/from-<current_profile>/<timestamp>-<topic>.md`. The target profile bot discovers it during its next session startup or via an explicit `/read_inbox` command.

**Example flow:**
1. Research bot finishes a deep dive on vector DBs
2. User says: `/project_it share dev`
3. Research bot writes `inbox/cross-profile/from-research/20260502-vector-db-comparison.md`
4. User switches to dev bot, says: "read latest from research"
5. Dev bot reads the file, incorporates findings into Yawnly architecture decisions

**Security note:**
Cross-profile writes stay inside MyWiki. No external network calls. No profile A talking directly to profile B's gateway. The human is the router.

---

### Files changed

None. MyWiki is already accessible to all profiles.

---

### New files created

- `~/.hermes/skills/project-it/SKILL.md` — `/project_it` skill
- `~/.hermes/project-registry.json` — Project registry data
- `~/.hermes/profiles/dev/**` — Dev profile (full tree)
- `~/.hermes/profiles/research/**` — Research profile (full tree)
- `~/.hermes/profiles/personal/**` — Personal profile (full tree)
- `~/Library/LaunchAgents/ai.hermes.gateway.dev.plist` — Dev gateway service
- `~/Library/LaunchAgents/ai.hermes.gateway.research.plist` — Research gateway service
- `~/Library/LaunchAgents/ai.hermes.gateway.personal.plist` — Personal gateway service
- `~/Library/LaunchAgents/ai.hermes.gateway.default.plist` — Default gateway service (renamed)
- `~/.hermes/scripts/backup-encrypt-upload.sh` — Backup script
- `~/.hermes/backup-age.pub` — Age public key
- `~/Library/Mobile Documents/com~apple~CloudDocs/HermesBackups/` — Cloud backup dir

### Existing files modified

- `~/.hermes/project-registry.json` — Ongoing, updated when projects added/switched

### No Hermes source code edits

All features use existing primitives: profiles, skills, memory, cron, backup, gateway.

### What Hermes already provides

```bash
hermes backup              # Full zip: config, skills, sessions, data
hermes backup --quick      # Critical state: config.yaml, state.db, .env, auth, cron
hermes import <zip>        # Restore
```

Curator also has built-in backup: `curator.backup.enabled: true` with `keep: 5` backups.

### What's missing

- **Encryption** — `hermes backup` produces unencrypted zips. The `.env` file contains API keys.
- **Cloud upload** — backups stay local.
- **Automation** — no periodic schedule (Curator backups are tied to skill maintenance cycles).

### Recommended approach: Cron job + age encryption + iCloud

**Why age instead of GPG:**
- Single-purpose, no keyring complexity
- Asymmetric: public key on machine, private key stored off-machine
- `age` CLI is a single Go binary (`brew install age`)

**Why iCloud Drive:**
- Already available on macOS (`~/Library/Mobile Documents/com~apple~CloudDocs/`)
- No additional setup, no third-party service
- End-to-end encrypted by Apple (adds a second layer beyond age)

**Backup flow (cron job script):**

```bash
#!/bin/bash
# ~/.hermes/scripts/backup-encrypt-upload.sh

set -e
TIMESTAMP=$(date -u +%Y%m%d-%H%M%S)
BACKUP_DIR=~/Library/Mobile\ Documents/com~apple~CloudDocs/HermesBackups
AGE_PUBKEY=~/.hermes/backup-age.pub

# 1. Create quick backup
hermes backup --quick --label "${TIMESTAMP}" -o /tmp/hermes-backup-${TIMESTAMP}.zip

# 2. Encrypt with age
age -r "$(cat $AGE_PUBKEY)" -o "${BACKUP_DIR}/hermes-backup-${TIMESTAMP}.zip.age" /tmp/hermes-backup-${TIMESTAMP}.zip

# 3. Clean up temp
rm /tmp/hermes-backup-${TIMESTAMP}.zip

# 4. Prune old backups (keep last 30)
ls -t "${BACKUP_DIR}"/hermes-backup-*.zip.age | tail -n +31 | xargs rm -f
```

**Cron schedule:** Daily at 4:00 AM IST (`30 22 * * *` UTC = 4:00 AM IST next day)

### Key management

1. Generate age keypair on a secure machine: `age-keygen -o ~/hermes-backup-key.txt`
2. Store public key at `~/.hermes/backup-age.pub` on the Hermes machine
3. Store private key **off this machine** — Bitwarden, 1Password, or encrypted USB
4. To restore: `age -d -i ~/hermes-backup-key.txt hermes-backup-XXX.zip.age > backup.zip && hermes import backup.zip`

### Security considerations

- `.env` contains API keys — must be encrypted before ANY cloud upload
- Age encryption happens locally, before the file leaves the machine
- iCloud adds a second encryption layer at rest
- Private key never touches the Hermes machine
- Backup script runs with `hermes` permissions — no root needed

### Alternative: Syncthing + age

If iCloud isn't preferred, Syncthing to a NAS with the same age encryption step. The flow is identical — only the copy destination changes.

### Files created

```
~/.hermes/scripts/backup-encrypt-upload.sh   (the script)
~/.hermes/backup-age.pub                     (age public key)
~/Library/Mobile Documents/com~apple~CloudDocs/HermesBackups/  (cloud target dir)
```

### Cron job

Create via the `cronjob` tool:
- **Schedule:** `30 22 * * *` (4:00 AM IST daily)
- **Prompt:** Run the backup script at `~/.hermes/scripts/backup-encrypt-upload.sh`
- **Tools:** terminal only
- **Delivery:** local (report to `~/.hermes/cron/output/`)

---

## Exact Files Changed (if approved)

### New files created

- `~/.hermes/skills/project-it/SKILL.md` — `/project_it` skill
- `~/.hermes/project-registry.json` — Project registry data
- `~/.hermes/profiles/dev/**` — Dev profile (full tree)
- `~/.hermes/profiles/research/**` — Research profile (full tree)
- `~/.hermes/profiles/personal/**` — Personal profile (full tree)
- `~/Library/LaunchAgents/ai.hermes.gateway.dev.plist` — Dev gateway service
- `~/Library/LaunchAgents/ai.hermes.gateway.research.plist` — Research gateway service
- `~/Library/LaunchAgents/ai.hermes.gateway.personal.plist` — Personal gateway service
- `~/Library/LaunchAgents/ai.hermes.gateway.default.plist` — Default gateway service (renamed)
- `~/.hermes/scripts/backup-encrypt-upload.sh` — Backup script
- `~/.hermes/backup-age.pub` — Age public key
- `~/Library/Mobile Documents/com~apple~CloudDocs/HermesBackups/` — Cloud backup dir

### Existing files modified

- `~/.hermes/project-registry.json` — Ongoing, updated when projects added/switched

### No Hermes source code edits

All features use existing primitives: profiles, skills, memory, cron, backup, gateway.

---

## Risks and Security Considerations

### `/project_it` skill
- **Risk:** JSON corruption if multiple sessions write concurrently. **Mitigation:** single-user system, sequential access.
- **Risk:** Skill bloat if registry grows large. **Mitigation:** keep registry under 20 projects.

### Multi-profile gateways
- **Risk:** 4 gateway processes = 4x resource usage (CPU, memory, API costs). **Mitigation:** idle gateways use minimal resources; API costs only on active conversations.
- **Risk:** Launchd service naming — Hermes may not support `--name` for gateway install. **Mitigation:** fall back to manual launchd plist creation.
- **Risk:** Profile config drift over time. **Mitigation:** master config template in MyWiki; periodic `hermes config check` per profile.
- **Risk:** Telegram allowlist is single-user. All 4 bots talk to the same user. No multi-user isolation concern.
- **Security:** Each bot token is a separate secret. Store in per-profile `.env` files, never in config.yaml or MyWiki.

### Encrypted backup
- **Risk:** Age private key loss = irrecoverable backups. **Mitigation:** store private key in 2+ places (Bitwarden + encrypted USB).
- **Risk:** Script failure goes unnoticed. **Mitigation:** cron job delivers failure reports; weekly maintenance cron inspects backup freshness.
- **Risk:** iCloud sync lag causes partial uploads. **Mitigation:** script writes to temp first, then moves atomically.
- **Security:** `.env` in backup contains live API keys. Age encryption + iCloud encryption = defense in depth. Private key never on Hermes machine.

---

## Implementation Order (Recommended)

1. **First: `/project_it` skill** — lowest risk, immediate value, no external dependencies
2. **Second: Encrypted backup** — safety net before making bigger changes
3. **Third: Multi-profile gateways** — highest complexity, test one profile at a time (dev first, then research, then personal)

---

## Open Questions

1. Does `hermes gateway install` support a `--name` or `--profile` flag for multiple services? If not, the manual launchd plist approach is the fallback.
2. Should the `default` profile retain the existing Telegram bot, or should all bot traffic move to dedicated profiles?
3. For the research profile: broader web access means higher API costs — is there a budget cap preference?
4. Should the `project-it` skill auto-load on every session (pinned skill), or load on-demand (`/skill project-it`)?
