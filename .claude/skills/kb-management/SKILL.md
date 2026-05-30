# kb-management

Manage Personal Knowledge Base (PKB) instances: install a new KB, update all registered KBs from the latest templates, or remove a registered KB from the registry.

## Prerequisites

This skill runs from within the kb-setup installer repository. The `./templates/` directory must be present at the repo root — verify this before proceeding. If it's missing, stop and tell the user.

## Operations

Invoke with `/kb-management [install|update|remove]`. If no operation is specified, ask the user which one they want.

---

### install

Set up a new KB instance.

**Gather inputs** — ask the user for each of the following in turn:

1. **Target path** — Ask the user where to create the new KB directory. Do not suggest options or present a list unless `~/.claude/CLAUDE.md` has existing KB entries. Then offer their paths so the user can choose a consistent location or provide a different location.

2. **Domain** — Ask the user for a short name for this KB (lowercase, no spaces). Do not suggest options or offer examples in a list. Accept whatever the user types.

3. **Description** — Once the user provides the domain, suggest a brief list of topics based on the domain name. Present it as a suggestion they can accept or replace with their own wording.

**Validate** before doing anything:
- If the target path already exists and is non-empty, tell the user and ask whether to continue.
- If a KB with the same domain is already registered in `~/.claude/CLAUDE.md`, tell the user and ask whether to continue.

**Create the KB directory structure** at the target path. The required layout is:

```
<install-path>/
├── .claude/
│   ├── CLAUDE.md
│   └── skills/
│       ├── kb-ingest/
│       │   └── SKILL.md
│       ├── kb-query/
│       │   └── SKILL.md
│       └── kb-lint/
│           └── SKILL.md
├── index.md
├── log.md
├── ingest/
├── raw/
├── sources/
├── pages/
└── notes/
```

Copy each file from `./templates/` to its corresponding location in the new KB. The directory mapping is:
- `./templates/CLAUDE.md` → `<install-path>/.claude/CLAUDE.md`
- `./templates/skills/kb-ingest/SKILL.md` → `<install-path>/.claude/skills/kb-ingest/SKILL.md`
- `./templates/skills/kb-query/SKILL.md` → `<install-path>/.claude/skills/kb-query/SKILL.md`
- `./templates/skills/kb-lint/SKILL.md` → `<install-path>/.claude/skills/kb-lint/SKILL.md`
- `./templates/index.md` → `<install-path>/index.md`
- `./templates/log.md` → `<install-path>/log.md`

Copy file contents exactly — do not modify them during copying.

Create `ingest/`, `raw/`, `sources/`, `pages/`, and `notes/` as empty directories.

**Update `<install-path>/log.md`** after copying: replace `YYYY-MM-DD` with today's date in ISO 8601 format, and `[path]` with the actual installation path.

**Bootstrap the global layer** at `~/.claude/CLAUDE.md`:
- If the file does not exist, create it containing a KB Registry section in this format:

```markdown
## Knowledge Base Registry

Query the relevant KB using the kb-lookup subagent when a question is likely answered by
personally captured knowledge — sources ingested, concepts built up, or notes written over
time. The KB complements Claude's training data; it does not replace it.

**Query when:** the user asks what they know or have captured on a topic; the domain matches
AND personal context would improve the answer beyond general knowledge; the question
references something likely ingested (a paper, article, tool, decision).

**Skip when:** the question is general knowledge Claude can answer confidently from training
data (syntax, public APIs, common facts, general concepts); the question is about current
events (use web search instead); no KB domain is a plausible match.

When unsure which KB is relevant, ask before querying.

| Domain | Path | Description |
|--------|------|-------------|
```

- If the file exists but has no `## Knowledge Base Registry` section, append the section above to the end of the file.
- If the file exists and already has a KB Registry section, leave it as-is.

**Register the new KB**: Add a row to the KB Registry table in `~/.claude/CLAUDE.md`:
```
| <domain> | <install-path> | <description> |
```
Be careful to insert inside the existing table (after the header and divider rows), not after the table's closing blank line.

**Install the global KB-Lookup agent**: Copy `./templates/agents/kb-lookup.md` to `~/.claude/agents/kb-lookup.md`. Create `~/.claude/agents/` if it does not exist. If `kb-lookup.md` already exists there, skip — do not overwrite.

**Report** what was installed:
- KB path, domain, and description
- Whether the global agent was installed or already existed
- Next steps: open Claude Code from the KB directory and run `/kb-ingest`, `/kb-query`, or `/kb-lint`

---

### update

Push updated templates to all registered KB instances.

**Check for dry-run mode**: If the user included `--dry-run`, describe what would change without making any modifications. Apply this consistently throughout.

**Update the global KB-Lookup agent**: Compare `./templates/agents/kb-lookup.md` with `~/.claude/agents/kb-lookup.md`. If they differ (or the destination is missing), update it. In dry-run, report what would change.

**Find all registered KB paths**: Read `~/.claude/CLAUDE.md` and extract all data rows from the KB Registry table (skip the header row and the `|---|---|---|` divider row). The path is in the second column. Expand `~` to the user's home directory.

**For each registered KB path**:
1. Check that the path exists. If it does not, report it as missing and skip to the next.
2. Compare `./templates/CLAUDE.md` with `<kb-path>/.claude/CLAUDE.md`. If they differ, update the KB's copy.
3. For each skill in `./templates/skills/` (kb-ingest, kb-query, kb-lint): compare the template skill with `<kb-path>/.claude/skills/<skill>/SKILL.md`. If they differ or the file is missing, update the KB's copy.
4. Check `<kb-path>/.claude/skills/` for any skill directories not present in `./templates/skills/`. If any are found, report them and ask the user before removing.

**Report** a summary:
- KBs updated (list paths and which files changed)
- KBs already up-to-date (skipped)
- KBs with missing paths
- In dry-run mode, list every file that would change and why

---

### remove

Remove a registered KB from the global registry.

**List registered KBs**: Read `~/.claude/CLAUDE.md` and display all data rows from the KB Registry table, numbered. Example:
```
1. consulting — ~/projects/consulting-kb — Client notes and engagements
2. personal — ~/personal-kb — Personal reading and research
```
If no KBs are registered, tell the user and stop.

**Prompt** the user to select a KB by number, or type 'q' to cancel. Confirm before proceeding.

**Tell the user** what will happen:
- The registry entry for `<domain>` will be removed from `~/.claude/CLAUDE.md`
- The KB directory at `<path>` will NOT be deleted — content is preserved
- Ask for explicit confirmation

**Remove the registry entry**: Delete the matching row from the KB Registry table in `~/.claude/CLAUDE.md`. Remove only the one matching row; leave the rest of the file intact.

**If the registry table is now empty** (no data rows remain):
- Ask the user whether to remove the `## Knowledge Base Registry` section from `~/.claude/CLAUDE.md`
- Ask the user whether to remove `~/.claude/agents/kb-lookup.md`
- Apply only the removals the user confirms

**Report** what was removed and remind the user that the KB directory still exists.
