---
name: kb-query
description: |
  Query the Personal Knowledge Base you are currently inside and synthesize an
  answer from its existing pages, with wikilink citations. Use when the user asks
  a question to be answered from this specific KB. For looking up a KB by path
  from outside it (or from another agent), use the kb-lookup skill instead. Only
  acts inside a KB (a folder with a kb.config.md at its root).
version: 0.1.0
license: MIT
---

# kb-query

Query the knowledge base and synthesize an answer from existing pages.

## When to use

Invoke with `/kb-query` when the user asks a question and wants an answer synthesized from this knowledge base specifically — as opposed to the global `kb-lookup` skill, which queries any KB by path and is callable from outside the KB or from another agent.

## Step 0 — KB guard (do this first)

Check for `kb.config.md` in the current working directory (the KB root).

- **If it is missing:** this folder is not a knowledge base. Do nothing. Say: "No `kb.config.md` here — this doesn't look like a KB. `cd` into a KB and try again, or use the `kb-lookup` skill with a KB path." Then stop.
- **If it is present:** continue.

## Steps

1. Read `index.md` to identify which sources and pages are relevant to the query.
2. Read the relevant pages in full.
3. Synthesize an answer using `[[filename|Display Text]]` wikilink citations throughout.
4. Note any gaps — aspects the query touches that the knowledge base does not cover well.
5. Offer to file the answer back as a new page in `pages/`. If the user agrees:
   - Create the page using the Page template from CLAUDE.md
   - Update `index.md`
   - Append to `log.md`:

```
## [YYYY-MM-DD] query | [Query summary]

- Query: [one-line description]
- Pages read: N
- Answer filed as new page: [filename or "no"]
- Gaps noted: [brief description or "none"]
```

## Output format

Match the format to the query type:
- **Comparisons** → table
- **Analyses** → headers and bullets
- **Narrative** → prose with wikilinks

Lead with the answer. Context and citations follow.
