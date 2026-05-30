<install-dir>/
├── .claude/
│   ├── CLAUDE.md              # KB schema (read-only)
│   └── skills/
│       ├── kb-ingest/
│       │   └── SKILL.md
│       ├── kb-query/
│       │   └── SKILL.md
│       └── kb-lint/
│           └── SKILL.md
├── index.md                   # Content catalog — updated on every ingest
├── log.md                     # Append-only operation record
├── ingest/                    # Staging area — files queued for ingestion (transient)
├── raw/                       # Permanent archive of ingested sources (never modified)
├── sources/                   # Source summary pages
├── pages/                     # All named things: concepts, people, orgs, products
└── notes/                     # Personal notes — Claude reads but does not write here
