# KB instance directory structure

A KB instance created by `/kb-setup` looks like this. The engine skills are not stored here — they come from the `kb-management` plugin.

```
<kb-root>/
├── .claude/
│   └── CLAUDE.md              # KB schema (plugin-owned, refreshed by /kb-setup; read-only)
├── kb.config.md               # KB marker + kb_version + domain/description — do not delete
├── index.md                   # Content catalog — updated on every ingest
├── log.md                     # Append-only operation record
├── ingest/                    # Staging area — files queued for ingestion (transient)
├── raw/                       # Permanent archive of ingested sources (never modified)
├── sources/                   # Source summary pages
├── pages/                     # All named things: concepts, people, orgs, products
└── notes/                     # Personal notes — Claude reads but does not write here
```

`kb.config.md` at the root is the marker that identifies the folder as a KB; the in-KB skills (`kb-ingest`, `kb-query`, `kb-lint`) check for it before acting.
