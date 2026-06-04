# Personal Knowledge Base

A Claude Code plugin that turns any folder into a local, AI-powered knowledge base. Drop sources into an `ingest/` folder, run `/kb-ingest`, and Claude builds and maintains a structured wiki of concepts, people, organizations, and ideas — with wikilink cross-references and an append-only log. Query any KB from any session.

## How it works

The plugin separates the **engine** from your **content**:

- **Engine (the plugin)** — the skills (`kb-ingest`, `kb-query`, `kb-lint`, `kb-lookup`) and commands (`kb-setup`, `kb-remove`) live in the plugin and are shared machine-wide. Update them once with `/plugin update`; every KB benefits.
- **Instance (each KB)** — a folder you own, holding your sources and pages plus a `kb.config.md` marker, a refreshed copy of the schema, `index.md`, and `log.md`. The plugin never stores your content.
- **Global registry** (`~/.claude/CLAUDE.md`) — a table of your KBs and their paths. From any project, Claude consults it and uses the `kb-lookup` skill to pull relevant context before answering.

Because the engine is in the plugin and not copied into each KB, your knowledge bases never drift, and updating logic is a single `/plugin update`.

## Prerequisites

- [Claude Code](https://claude.ai/code) installed and authenticated

## Install

```
/plugin marketplace add https://github.com/ScottExMachina/kb-management
/plugin install kb-management
```

(For local development, point `marketplace add` at the repo path instead: `/plugin marketplace add /Users/scott/Code/kb-management`.)

## Create a knowledge base

Run:

```
/kb-setup
```

With no arguments, `/kb-setup` asks what you want to do — **start a new KB** (it then prompts you for the path) or **sync an existing one**. For a new KB it also asks for a **domain** (a short identifier like `tech` or `consulting`) and a one-line **description**, scaffolds the structure, writes `kb.config.md`, and registers the KB in `~/.claude/CLAUDE.md`. `/kb-setup` is idempotent — run it again any time to sync the latest plugin-owned files; it never overwrites your content.

To skip the menu, target a folder directly — `/kb-setup ~/knowledge/tech-kb` — or sync every registered KB with `/kb-setup --all`.

## Daily use

**Ingest a source.** Drop any file (PDF, markdown, text) into `ingest/`, then:

```
/kb-ingest
```

Claude moves the file to `raw/`, summarizes it, creates or updates pages in `pages/`, and updates the index and log.

**Query a KB.** From inside the KB:

```
/kb-query
```

From any other project, Claude consults the registry and uses the `kb-lookup` skill automatically when your question touches a registered domain. Because `kb-lookup` is a skill (not an agent), subagents can use it too.

**Validate a KB.**

```
/kb-lint
```

Checks for broken wikilinks, orphan pages, missing categories, contradictions, and gaps. Offers to auto-migrate or guide manual fixes.

## Multiple knowledge bases

Run `/kb-setup` in another folder to add another KB. Each gets its own registry entry; cross-project queries route to the right KB by domain.

## Updating

Two independent channels:

| What | How |
|------|-----|
| Engine (skills + commands) | `/plugin update kb-management` — machine-wide, automatic |
| Per-KB scaffolding (schema, config, new dirs) | `/kb-setup` in one KB, or `/kb-setup --all` for every registered KB |

`/kb-setup --all --dry-run` previews every change across all KBs without writing.

## Migrating KBs from the old installer

Earlier KBs were created by a script that **copied** the skills into each KB's `.claude/skills/` and a `kb-lookup` agent into `~/.claude/agents/`. Those copies are now redundant. Running `/kb-setup` (or `/kb-setup --all`) on such a KB detects the old layout and offers to clean it up: it removes the copied-in skills, adds `kb.config.md`, refreshes the schema, and updates the registry wording — asking before any deletion, and never touching your content.

## Removing a KB

```
/kb-remove
```

Removes the registry entry only. Your content files are never deleted. To uninstall the engine entirely, run `/plugin uninstall kb-management`.

## KB directory structure

```
<kb>/
├── .claude/
│   └── CLAUDE.md              # schema (plugin-owned, refreshed by /kb-setup)
├── kb.config.md               # KB marker + version + domain — do not delete
├── index.md                   # content catalog
├── log.md                     # append-only operation record
├── ingest/                    # drop sources here
├── raw/                       # archived originals (never modified)
├── sources/                   # source summary pages
├── pages/                     # knowledge base pages
└── notes/                     # your personal notes (Claude reads, never writes)
```
