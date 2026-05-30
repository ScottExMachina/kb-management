# kb-lint

Validate the knowledge base structure, find inconsistencies and gaps, and manage migrations.

## When to use

Invoke with `/kb-lint` to health-check the knowledge base, or after running `update.sh` to check whether a schema update requires structural changes to existing content.

## Steps

1. Read `index.md` for the full page and source inventory.
2. Read all pages and source summaries (or a representative sample if the KB is large).
3. Check for the following issues:

| Check | What to look for |
|---|---|
| Contradictions | Claims on different pages that disagree |
| Stale claims | Information that newer sources have superseded |
| Orphan pages | Pages with no inbound wikilinks from other pages |
| Missing pages | Things mentioned frequently across pages that lack their own page |
| Broken wikilinks | Links using `[[filename]]` format pointing to filenames that do not exist |
| Missing cross-references | Pages that are clearly related but do not link to each other |
| Data gaps | Topics where a new source would meaningfully improve coverage |
| Missing categories | Pages missing a `category:` frontmatter field |
| Invalid categories | Pages using a category value not in the category table in CLAUDE.md |
| Tag rule violations | Tags that restate the category, or Tool pages using non-descriptor tags |
| index.md format | Heading is not `# Knowledge Base Index`; missing or malformed Sources/Pages sections; entries not in `- [Title](path) — summary` format |
| log.md format | Heading is not `# Knowledge Base Log`; entries not in `## [YYYY-MM-DD] operation \| Description` format |

4. Present findings as a prioritized list — highest-impact issues first. For each issue, name the file(s) involved and suggest the fix.
5. For each issue, ask before applying any change.
6. Offer one of three paths for structural issues (e.g. after a schema migration):
   - **Auto-migrate** — apply the fix automatically, preserving all links and file references
   - **Manual fix** — pause and instruct the user on what to change; re-run `/kb-lint` afterward to confirm
   - **Pass** — if structure already matches expectations, validate and exit
7. Append to `log.md`:

```
## [YYYY-MM-DD] lint | [summary]

- Issues found: N, issues fixed: N
- Notable: [any significant contradictions, gaps, or migration actions]
```

## Rules

- Never apply a fix without explicit user confirmation.
- When auto-migrating, preserve all wikilink targets — update the display text if needed but never change the filename half of a link without also renaming the file.
- Report clearly when a re-run is needed after manual fixes.
