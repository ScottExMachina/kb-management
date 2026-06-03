---
description: Unregister a Personal Knowledge Base from the global registry. Removes the registry entry only — your KB content is never deleted.
---

You are unregistering a **Personal Knowledge Base** from the global registry in `~/.claude/CLAUDE.md`. This only edits the registry; it never deletes KB content.

## Steps

1. **List registered KBs.** Read `~/.claude/CLAUDE.md` and display every data row from the `## Knowledge Base Registry` table, numbered. Example:
   ```
   1. consulting — ~/projects/consulting-kb — Client notes and engagements
   2. tech — ~/knowledge/tech-kb — Technology and AI notes
   ```
   If there is no registry or no data rows, tell the user there's nothing to remove and stop.

2. **Prompt** the user to pick a KB by number, or type `q` to cancel.

3. **Confirm before acting.** State clearly:
   - The registry entry for `<domain>` will be removed from `~/.claude/CLAUDE.md`.
   - The KB directory at `<path>` will **not** be deleted — content is preserved (remove it manually if you want it gone).
   - Ask for explicit confirmation.

4. **Remove the registry row.** Delete only the one matching row from the registry table. Leave the header, divider, instructions, and every other row intact.

5. **If the table is now empty** (no data rows remain):
   - Ask whether to remove the entire `## Knowledge Base Registry` section from `~/.claude/CLAUDE.md`.
   - If `~/.claude/agents/kb-lookup.md` still exists (the old hand-installed agent), ask whether to remove it too — it's redundant now that the plugin provides the `kb-lookup` skill.
   - Apply only the removals the user confirms. Note: the plugin's `kb-lookup` skill is unaffected by this command; to uninstall the engine entirely, run `/plugin uninstall kb-management`.

6. **Report** what was removed and remind the user the KB directory still exists on disk.

## Hard rules

- Never delete KB content or directories — only edit `~/.claude/CLAUDE.md` and (if confirmed) the old global agent file.
- Remove exactly the one selected row; never touch other entries.
- Ask before any deletion beyond the single registry row.
