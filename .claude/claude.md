# PKB — Installer Repo

This repo is the distributable installer for the Personal Knowledge Base system. It contains templates and a management skill only. No personal KB content belongs here.

## What this repo contains

- `templates/` — all files copied or referenced during installation
  - `CLAUDE.md` — the local schema loaded by Claude Code inside each KB instance
  - `agents/kb-lookup.md` — global subagent installed to `~/.claude/agents/`
  - `skills/` — three local skills installed into each KB instance
  - `index.md`, `log.md` — starter content files
- `.claude/skills/kb-management/` — skill for installing, updating, and removing KB instances
- `docs/` — requirements and design reference

## Usage

Open Claude Code in this repo and run:

```
/kb-management install   # Set up a new KB instance
/kb-management update    # Update all installed instances (supports --dry-run)
/kb-management remove    # Remove a KB instance
```

## Working in this repo

When editing templates or docs, Claude Code operates in this repo directory only. KB instance content lives elsewhere and is never touched from here.
