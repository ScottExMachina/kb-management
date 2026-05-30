# kb-query

Query the knowledge base and synthesize an answer from existing pages.

## When to use

Invoke with `/kb-query` when the user asks a question and wants an answer synthesized from this knowledge base specifically — as opposed to the global KB-Lookup Agent, which queries across all registered KBs.

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
