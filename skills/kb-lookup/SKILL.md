---
name: kb-lookup
description: |
  Look up a Personal Knowledge Base by path and return a synthesized, cited answer
  to a query. Use when a question matches a registered KB domain and you want to
  pull relevant context from that KB — including from within a subagent or when you
  are not inside the KB folder. Takes a KB path (from the global registry) plus a
  query. Read-only: never writes, moves, or modifies files.
version: 0.1.0
license: MIT
---

# kb-lookup

Query a single knowledge base instance by path and return relevant results to the caller. This is the cross-KB lookup capability: it works on whatever KB path it is given, so it can be invoked from any session — and, because it is a skill rather than an agent, from inside a subagent too.

## When to use

- The user's question matches a domain in the Knowledge Base Registry (in `~/.claude/CLAUDE.md`) and personal captured knowledge would improve the answer.
- You need KB context from within an agent/subagent context (an agent cannot dispatch another agent, but it can invoke this skill).
- You want to read a KB you are not currently "inside" (you have its path, not its folder open as the project).

To answer from the KB you are *currently inside* instead, use the `kb-query` skill.

## Inputs

- **KB path** — the absolute path to the KB instance root (from the global registry).
- **Query** — the question or topic to look up.

If either is missing, ask for it (or, if a registry is available, resolve the path from the matching domain) before proceeding.

## Steps

1. Verify the KB path exists and contains an `index.md`. If the path is missing, return: `KB not found at [path]`. If `index.md` is missing, return: `index.md not found at [path] — KB may not be initialized`.
2. Read `index.md` at the KB path.
3. Identify which sources and pages in the index are relevant to the query.
4. Read the relevant pages in full.
5. Synthesize a concise, structured answer using `[[filename|Display Text]]` wikilink citations.
6. Note any gaps — aspects of the query the KB does not cover.

## Output format

Return results in this structure:

```
## KB: [KB path]

[Synthesized answer with wikilink citations]

**Gaps:** [brief description, or "none"]
```

Match the format to the query:
- Comparisons → table
- Analyses → headers and bullets
- Narrative → prose with wikilinks

Lead with the answer. Keep the response concise — the caller may synthesize across multiple KBs.

## Read-only

Never write, move, or modify files. Only read. (When a tool-restricted agent runs this skill, that constraint is also enforced by its toolset.)

## Error handling

- Missing KB path → return error, do not attempt to continue
- Missing `index.md` → return error noting the KB may not be initialized
- Empty index → return: `KB is empty — no pages have been ingested yet`
- No relevant pages found → return: `No relevant content found for: [query]`
