---
description: Create or sync a Personal Knowledge Base (idempotent — install, update, and migrate in one command). With no arguments it asks whether to start a new KB (prompting for the path) or sync an existing one; pass a <path> or --all to skip the menu. Supports --dry-run. Never overwrites your content.
---

You are setting up or syncing a **Personal Knowledge Base (KB)**. This command is idempotent: run it on an empty folder to create a new KB, on an existing KB to sync plugin-owned files, or on a KB made by the old (pre-plugin) installer to migrate it. It must never destroy user content.

## Arguments

- (none) — run the **Interactive entry routine** below. Do **not** assume the current working directory is the target.
- `<path>` — operate directly on the KB at that path (no menu).
- `--all` — operate on **every KB in the global registry** (see "Bulk mode"); no menu.
- `--dry-run` — report everything that would change, create, refresh, or delete, and write nothing. Honor this in every mode, including the interactive routine.

## Source of truth

The bundled KB template lives at `${CLAUDE_PLUGIN_ROOT}/kb-template/`. It mirrors exactly what an installed KB's plugin-owned files should look like. Read from it; copy into the target.

If `${CLAUDE_PLUGIN_ROOT}` is not set, locate the `kb-management` plugin's `kb-template/` directory under the Claude plugins cache and use that. Tell the user which source path you resolved.

The plugin version is the `version` field in `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`. Read it once; you'll stamp it into each KB's `kb.config.md` as `kb_version`.

## Classify the target

Inspect the target folder and decide which case it is:

- **New** — no `kb.config.md` and no `.claude/CLAUDE.md` (empty or non-existent folder).
- **Current** — has `kb.config.md` at the root (already a plugin-era KB).
- **Legacy** — has `.claude/CLAUDE.md` and/or copied-in `.claude/skills/kb-ingest|kb-query|kb-lint`, but **no** `kb.config.md` (made by the old installer).

## Interactive entry routine (no arguments)

When invoked with **no path and no `--all`**, don't assume the cwd is the target. First decide *what to do* with the user:

1. **Gather state.** Read the KB Registry table in `~/.claude/CLAUDE.md` (domain/path/description for each registered KB). Classify the cwd with the "Classify the target" rules above so you know whether it's already a KB (Current/Legacy).
2. **Show the menu.** Always present both options, numbered (in the style of `/kb-remove`):
   1. **Start a new KB** — create a new KB at a path you choose.
   2. **Sync an existing KB** — refresh or migrate a KB that's already set up.

   If the cwd is itself a KB, say so and note it's the recommended sync target.
3. **Branch on the choice:**
   - **Start a new KB →** Ask for the **target path**. Offer defaults to choose from: the current folder, and the parent directories of any registered KBs (so new KBs land in a consistent location). Accept the path as the user types it — don't assume POSIX; Windows backslash paths and `~`/environment expansion are all fine. Then:
     - If the chosen path already contains a `kb.config.md` or a legacy layout, tell the user "that folder is already a KB — syncing it instead" and run the **Single-KB routine** on it (it will classify as Current/Legacy). Never overwrite.
     - Otherwise run the **Single-KB routine** as **New** at that path.
   - **Sync an existing KB →** List the registered KBs numbered, plus the cwd if it's a KB that isn't in the registry yet; flag the cwd KB as the recommended choice when applicable. Have the user **pick one**, then run the **Single-KB routine** on it. If there are no registered KBs **and** the cwd isn't a KB, there's nothing to sync — say so and switch to **Start a new KB**. (Syncing every KB at once is the `--all` flag; don't offer an "all" choice here.)
4. **Dry-run.** With `--dry-run`, still run the menu and prompts, but once the target is chosen, only report what *would* change and write nothing.

## Single-KB routine

### 1. Determine domain + description
- **New:** ask the user for a **domain** (short, lowercase, no spaces) and a one-line **description**. The target path is already settled — by the `<path>` argument, or by the interactive routine's path prompt — so don't re-ask for a location here.
- **Current:** read `domain` and `description` from the existing `kb.config.md`. Don't ask.
- **Legacy:** if a registry row in `~/.claude/CLAUDE.md` matches this path, take `domain` and `description` from it. Otherwise ask.

### 2. Create-if-absent (never overwrite)
Create only what's missing; never touch existing copies of these:
- Content folders `ingest/ raw/ sources/ pages/ notes/` (with a `.gitkeep` in each if empty).
- `index.md` — copy from the template only if absent.
- `log.md` — copy from the template only if absent.
- `kb.config.md` — copy from the template only if absent, then fill `domain` and `description` (step 1) and set `kb_version` to the plugin version.

On a **New** KB only: in the freshly created `log.md`, replace `YYYY-MM-DD` with today's date (ISO 8601) and `[path]` with the target path; set the date in `index.md` to today.

### 3. Sync plugin-owned files (create or refresh)
- `.claude/CLAUDE.md` — the schema. Copy from `${CLAUDE_PLUGIN_ROOT}/kb-template/CLAUDE.md`, overwriting any existing copy (it is plugin-owned, not user content). Report it as refreshed if it changed.
- `kb.config.md` already exists (Current/Legacy): leave `domain`, `description`, and `folders` as the user has them; only update `kb_version` to the plugin version.

### 4. Migrate a Legacy KB
If the target was classified **Legacy**:
- The schema refresh in step 3 already updates `.claude/CLAUDE.md`.
- The copied-in skills `.claude/skills/{kb-ingest,kb-query,kb-lint}` are now redundant (the plugin provides them machine-wide). **List them and ask the user before deleting**; on confirmation, remove those three skill directories. Never remove any other skill the user added.
- Add `kb.config.md` per step 2 (domain/description from the registry row or the user).
- Leave all content untouched (`raw/ sources/ pages/ notes/ index.md log.md`).

### 5. Register in the global registry
In `~/.claude/CLAUDE.md`:
- If there is no `## Knowledge Base Registry` section, append this one:

```markdown
## Knowledge Base Registry

Query the relevant KB using the kb-lookup skill (pass it the KB's path and your query) when a
question is likely answered by personally captured knowledge — sources ingested, concepts built
up, or notes written over time. The KB complements Claude's training data; it does not replace it.

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

- If the section exists but its instructions still say "kb-lookup **subagent**", update that wording to "kb-lookup **skill** (pass it the KB's path and your query)". This is the migration of the registry text.
- Add a row for this KB if its domain isn't already in the table: `| <domain> | <path> | <description> |`. Insert it inside the table (after the header and divider rows). If the domain is already present with this path, leave it.

### 6. Retire the hand-installed global agent (one-time)
If `~/.claude/agents/kb-lookup.md` exists, it's the old hand-copied agent, now superseded by the plugin's `kb-lookup` skill. Tell the user it's redundant and **ask before deleting** it. Apply only if confirmed.

### 7. Report
Summarize for this KB: path, domain, whether it was New/Current/Legacy, and lists of **Created**, **Refreshed**, **Skipped (already present)**, and **Removed (migration)**. For a New KB, end with next steps: drop a file in `ingest/` and run `/kb-ingest`; query with `/kb-query`; health-check with `/kb-lint`.

## Bulk mode (`--all`)

Read the KB Registry table in `~/.claude/CLAUDE.md`, extract each data row (skip the header and `|---|` divider), take the path from the second column, and expand `~` to the home directory. For each path:
- If the path doesn't exist, report it as **missing** and skip.
- Otherwise run the single-KB routine above (it will classify and migrate as needed). Pull domain/description from the registry row, so you don't re-prompt per KB.

Then do step 6 (global agent retirement) once, not per KB. Print a per-KB summary plus totals: synced, migrated, already up-to-date, missing.

## Hard rules

- Never overwrite `index.md`, `log.md`, or anything under `raw/ sources/ pages/ notes/` once it exists. Only `.claude/CLAUDE.md` (schema) is force-refreshed.
- Never change a user's `domain`, `description`, or `folders` in an existing `kb.config.md` — only bump `kb_version`.
- Ask before deleting anything (legacy skill dirs, the old global agent).
- With `--dry-run`, write nothing — only report.
- If anything is ambiguous, ask before writing.
