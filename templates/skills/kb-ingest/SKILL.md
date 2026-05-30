# kb-ingest

Ingest a new source from `ingest/` into the knowledge base.

## When to use

Invoke with `/kb-ingest` when one or more files have been placed in `ingest/` and are ready to be processed.

## Steps

1. Check `ingest/` for files. If a specific file was named, confirm it is there. If ingesting all, process each file in turn.
2. Move the file from `ingest/` to `raw/`. Do not modify the file content.
3. Read the file from `raw/` completely.
4. Summarize key takeaways in 3–5 bullets.
5. Create a source summary page in `sources/` using the Source Summary template from CLAUDE.md. Filename: lowercase, hyphenated, derived from the source title.
6. For each concept, person, organization, named thing, or idea worth its own page:
   - Check whether a matching page already exists in `pages/`
   - If it exists: update it, noting any contradictions with existing claims explicitly rather than silently overwriting
   - If it does not exist: create a new page using the Page template from CLAUDE.md, including the correct `category:` value from the category system
7. Update `index.md`:
   - Increment the page count
   - Update the date
   - Add the new source under **Sources**
   - Add any new pages under **Pages**
8. Append to `log.md`:

```
## [YYYY-MM-DD] ingest | [Source Title]

- [1–2 line summary of what changed]
- Pages created: N, pages updated: N
- Contradictions: [any found, or "none"]
```

9. Report back: source title, pages created, pages updated, any contradictions found.

## Rules

- Stay faithful to what the source says. Do not editorialize.
- Flag contradictions with existing pages explicitly — never silently overwrite a claim.
- Assign exactly one category to every new page using the category table in CLAUDE.md.
- Use 3–6 tags per page: domain tags and association tags. Never restate the category as a tag.
- All filenames: lowercase, hyphenated.
- All wikilinks: `[[filename|Display Text]]` format.
