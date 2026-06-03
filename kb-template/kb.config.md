---
kb_version: 0.1.0
domain: <domain>
description: <description>
folders:
  ingest: ingest
  raw: raw
  sources: sources
  pages: pages
  notes: notes
---

# KB Config

This file marks the folder as a Personal Knowledge Base and records which version of the `kb-management` plugin set it up. The KB skills (`kb-ingest`, `kb-query`, `kb-lint`) check for this file before acting, so **do not delete it**.

## Fields

- **kb_version** — the plugin version that last set up or synced this KB. Updated by `/kb-setup`; informational, and used to tell whether the KB is behind the installed plugin.
- **domain** — the short identifier for this KB in the global registry (`~/.claude/CLAUDE.md`). Used to route cross-KB lookups to the right KB.
- **description** — one line describing what this KB contains. Mirrors the registry entry.
- **folders** — the content folders. Change only if you want different folder names; the skills read these.

## How it's used

1. Drop a source into `ingest/`, then run the **kb-ingest** skill — it files the source into `raw/`, writes a summary in `sources/`, creates or updates wikilinked pages in `pages/`, and updates `index.md` and `log.md`.
2. Run **kb-query** to synthesize an answer from this KB, or **kb-lint** to health-check it.
3. From any other session, the **kb-lookup** skill can read this KB by path (resolved from the registry) without opening the folder.

The schema lives in `.claude/CLAUDE.md` and is plugin-owned (refreshed on `/kb-setup`). This config and your content are yours.
