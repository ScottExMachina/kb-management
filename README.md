# Personal Knowledge Base

A local, AI-powered knowledge base that runs inside [Claude Code](https://claude.ai/code). Drop sources into an inbox folder, run `/kb-ingest`, and Claude builds and maintains a structured wiki of concepts, people, organizations, and ideas — with full wikilink cross-references and an append-only log.

## How it works

- **Global layer** (`~/.claude/`) — a registry of all your KB instances and a read-only lookup agent that can be invoked from any Claude Code session
- **Local layer** (per KB) — the schema, three management skills, and your content

From any Claude Code project, Claude can consult your KB registry and invoke the `kb-lookup` agent to pull relevant context before answering. From within a KB project, you invoke skills directly to ingest, query, and maintain content.

## Prerequisites

- [Claude Code](https://claude.ai/code) installed and authenticated

## Setup

Clone the repo, then open Claude Code in it:

```bash
git clone <repo-url>
cd kb-setup
claude .
```

Then run:

```
/kb-management install
```

Claude will ask for:
1. **Install directory** — where the KB instance lives (e.g. `~/knowledge/personal-kb`)
2. **Domain name** — a short identifier for the registry (e.g. `personal`, `work`, `research`)
3. **Description** — one line describing what this KB contains

When complete, open the install directory in Claude Code:

```bash
claude ~/knowledge/personal-kb
```

## Daily use

### Ingesting a source

Drop any file (PDF, markdown, text) into `ingest/`, then:

```
/kb-ingest
```

Claude moves the file to `raw/`, summarizes it, creates or updates pages in `pages/`, and updates the index and log.

### Querying the KB

From within the KB project:

```
/kb-query
```

From any other Claude Code project, Claude will consult the global registry and invoke the `kb-lookup` agent automatically when your query touches a registered domain.

### Validating the KB

```
/kb-lint
```

Checks for broken wikilinks, orphan pages, missing categories, contradictions, and data gaps. Offers to auto-migrate or guide manual fixes.

## Directory structure

```
<install-dir>/
├── .claude/
│   ├── CLAUDE.md              # KB schema (read-only)
│   └── skills/
│       ├── kb-ingest/SKILL.md
│       ├── kb-query/SKILL.md
│       └── kb-lint/SKILL.md
├── index.md                   # Content catalog
├── log.md                     # Append-only operation record
├── ingest/                    # Drop sources here
├── raw/                       # Archived originals (never modified)
├── sources/                   # Source summary pages
├── pages/                     # Knowledge base pages
└── notes/                     # Your personal notes (Claude reads, never writes)
```

## Multiple knowledge bases

Run `/kb-management install` again from this repo to set up additional KB instances. Each gets its own entry in the global registry. When querying from any project, Claude routes to the correct KB based on the domain.

## Updating

Pull the latest version of this repo, then open Claude Code in it and run:

```
/kb-management update           # Update all registered instances
/kb-management update --dry-run # Preview changes without applying
```

## Removing a KB

Open Claude Code in this repo and run:

```
/kb-management remove
```

Removes the registry entry. Your content files are never deleted — remove them manually if needed.

