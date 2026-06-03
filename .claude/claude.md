# PKB — Plugin Repo

This repo is the distributable **Claude Code plugin** for the Personal Knowledge Base system. It contains the engine (skills + commands) and the per-instance scaffolding template only. No personal KB content belongs here.

## What this repo contains

- `.claude-plugin/` — plugin manifest (`plugin.json`) and marketplace entry (`marketplace.json`)
- `commands/` — slash commands (the engine)
  - `kb-setup.md` — create/sync/migrate a KB; `--all` syncs every registered KB; `--dry-run` previews
  - `kb-remove.md` — unregister a KB (content preserved)
- `skills/` — the engine skills, installed machine-wide with the plugin
  - `kb-ingest/`, `kb-query/`, `kb-lint/` — in-KB operations (guard on `kb.config.md` in the cwd)
  - `kb-lookup/` — read-only cross-KB lookup by path; callable from any session or subagent
- `kb-template/` — source of truth for per-instance scaffolding (`CLAUDE.md` schema, `kb.config.md`, `index.md`, `log.md`, and the content dirs). `/kb-setup` copies/refreshes from here.
- `docs/` — requirements and design reference

## Architecture

The engine lives in the plugin and is shared machine-wide; only content scaffolding is copied into each KB. Two update channels:

- `/plugin update kb-management` — updates skills and commands everywhere at once
- `/kb-setup` (or `/kb-setup --all`) — refreshes the plugin-owned schema and config inside each KB

The global registry table in `~/.claude/CLAUDE.md` lists the user's KBs and their paths; the `kb-lookup` skill reads a KB by path for cross-KB queries.

## Working in this repo

When editing skills, commands, the template, or docs, Claude Code operates in this repo directory only. KB instance content lives elsewhere and is never touched from here.

To try changes locally: `/plugin marketplace add /Users/scott/Code/kb-management` then `/plugin install kb-management`.
