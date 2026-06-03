# Agentic OS — Knowledge Base System Requirements

---

## 0. Status & Current Architecture

> This document is the original design spec and is kept for intent and rationale. The system is now packaged as a **Claude Code plugin** (`kb-management`), which changes how several requirements below are realized. Where the sections that follow describe shell scripts or a subagent, read them against this mapping.

**Engine vs. instance ownership.** The engine — the skills and commands — lives *in the plugin* and is shared machine-wide. Only per-instance scaffolding (the schema `CLAUDE.md`, `kb.config.md`, `index.md`, `log.md`, and the content folders) is copied into each KB. KBs no longer carry their own copies of the skills, so they don't drift.

**Two update channels.**
- `/plugin update kb-management` updates the engine (skills + commands) everywhere at once.
- `/kb-setup` (one KB) or `/kb-setup --all` (every registered KB; `--dry-run` to preview) refreshes the plugin-owned scaffolding inside each KB.

**Mapping from this spec to the plugin:**

| Spec concept | Plugin realization |
|---|---|
| `install.sh` (§5) and per-instance copied skills | `/kb-setup` command + `kb-template/` scaffolding; skills come from the plugin, not copied in |
| `update.sh` (§7.1) | `/plugin update` (engine) + `/kb-setup --all` (instances) |
| `remove.sh` (§8) | `/kb-remove` command |
| KB-Lookup **subagent** (§2.3, §3.2) | `kb-lookup` **skill** — so subagents can use it too; read-only by instruction |
| Per-instance marker | `kb.config.md` at the KB root (also carries `kb_version`) |
| Instance `contents/` subfolder | content folders live at the KB root (`ingest/ raw/ sources/ pages/ notes/`) |

Existing KBs made by the pre-plugin installer are migrated in place by `/kb-setup` (removes redundant copied-in skills, adds `kb.config.md`, refreshes the schema), asking before any deletion.

---

## 1. Overview

This document defines the requirements for a modular, multi-instance Knowledge Base (KB) system built on top of Claude Code. The system allows users to install one or more independent knowledge bases that can be queried from any Claude Code project session. Each KB is self-contained and independently versioned. A single global agent handles all KB queries, routing to the correct KB based on a centrally maintained registry.

The system is designed around a full CRUD lifecycle:

- **Create** — Install a new KB instance via the installation script
- **Read** — Query one or more KBs from any active project session
- **Update** — Push updates globally and migrate local KB structures via linting
- **Delete** — Remove a KB instance and clean up global references

---

## 2. Foundational Concepts

### 2.1 The Contract
Every KB instance, regardless of domain or version, must maintain a consistent interface so the global KB-Lookup Agent can traverse it without needing to know its internal structure in detail. The contract is an `index.md` file at the root of the KB contents folder that describes what the KB contains and how it is organized.

### 2.2 Global vs. Local
The system operates across two layers:

- **Global layer** (`~/.claude/`) — shared across all Claude Code sessions. Houses the KB registry, the KB-Lookup Agent, and routing decision rules.
- **Local layer** (per KB instance) — scoped to each installed KB. Houses the KB content, local CLAUDE.md, and local skills for intake, query, and linting.

### 2.3 Skills vs. Agent
The KB-Lookup Agent is defined as a **subagent**, not a skill. This means it runs in isolation, does not pollute the main conversation context with traversal work, and returns only clean results to the calling session. Local KB management capabilities (intake, query, linting) must be defined as **local skills** — not as instructions inside CLAUDE.md — so they are structured, invocable, and independently updatable.

---

## 3. Global Architecture Requirements

### 3.1 Global CLAUDE.md
- A `CLAUDE.md` file must exist at `~/.claude/CLAUDE.md`
- It must contain a **KB Registry** section listing all installed KB instances in the following format:

```
## Knowledge Base Registry

| Domain | Path | Content Description |
|--------|------|-------------|
| client-kb | ~/projects/client-kb | Client-specific notes, history, and engagements |
| frameworks-kb | ~/projects/fw-kb | Consulting frameworks, methods, and playbooks |
| history-kb | ~/projects/hist-kb | Past deliverables and lessons learned |
```

- It must contain **decision instructions** that tell Claude when to query the KB:
  - **Query KB first** when the task involves client-specific information, internal frameworks or methodologies, past project history, or any domain listed in the registry
  - **Use web search** for current events, publicly available information, to supplement information found in the KB, or topics not covered by any installed KB
  - **Ask before querying** if it is unclear which KB is relevant, rather than invoking blindly

### 3.2 KB-Lookup Agent
- A single global KB-Lookup Agent must be defined at `~/.claude/agents/kb-lookup.md`
- Defined as a subagent with YAML frontmatter
- Must accept two inputs: a **KB path** (from the registry) and a **query**
- System prompt must instruct the agent to:
  - Navigate to the provided KB path
  - Locate and read `index.md` as the entry point
  - Traverse KB contents based on the query
  - Return only relevant results in a concise, structured format
- Must handle a missing KB path or missing `index.md` gracefully, returning a clear error rather than failing silently
- Tool access restricted to read-only: `Read`, `Grep`, `LS` — no write or execute tools

---

## 4. KB Instance Requirements

### 4.1 Directory Structure
Each installed KB instance must follow this structure:

```
<user-specified-path>/
├── .claude/
│   ├── CLAUDE.md          # Local KB management instructions
│   └── skills/
│       ├── kb-intake/
│       │   └── SKILL.md   # Intake skill
│       ├── kb-query/
│       │   └── SKILL.md   # Query skill
│       └── kb-lint/
│           └── SKILL.md   # Linting skill
└── contents/              # Real folder or symlink to external location
    └── index.md           # The contract — required entry point
    └── log.md             # A record of changes performed by the kb sills
    └── ingest/            # Folder where new sources are ingested from
    └── raw/               # Where immutable copies of ingested sources stored
    └── summaries/         # Summaries of raw sources
    └── pages/             # Knoweldge base pages
    └── notes/             # User managed notes that may be read but not updated by Claude Code





```

### 4.2 The Contract (index.md)
Every KB's `contents/` folder must contain an `index.md` at its root that includes:
- A list of the summaries and pages within the KB

### 4.3 Local CLAUDE.md
- The local `CLAUDE.md` inside each KB instance governs how that KB is managed
- It must reference the three local skills for intake, query, and linting
- It must **not** contain inline instructions for intake, query, or linting logic — those must live in the local skills

### 4.4 Local Skills
Each KB instance must include three local skills defined as proper skill packages (not CLAUDE.md instructions) so they are invocable, versioned, and updatable independently:

- **kb-intake** — handles ingestion of new content into the KB, enforcing structure and naming conventions
- **kb-query** — handles querying the KB contents locally (distinct from the global agent query)
- **kb-lint** — validates the KB structure against the expected schema and manages migrations (see Section 6.2)

---

## 5. Installation

### 5.1 Installation Script
Each KB repo must include an `install.sh` script that runs interactively. The script must:

1. Prompt the user for a **target installation directory** where the KB instance will be created
2. Create the KB directory structure at that location
3. Prompt the user: **"Do you want to store KB contents at a different location?"**
   - If yes: prompt for the contents path, create the `contents/` folder at that location, and create a symlink at `<install-dir>/contents/` pointing to it. Note to the user that this is useful for third-party tools like Obsidian that can read and sync from that directory
   - If no: create the `contents/` folder directly inside the install directory
4. Check whether `~/.claude/CLAUDE.md` exists:
   - If not: bootstrap the global layer — create `CLAUDE.md` with a KB Registry section and install the KB-Lookup Agent to `~/.claude/agents/`
   - If yes: check whether the KB Registry section exists and add it if missing
5. Check whether the KB-Lookup Agent already exists at `~/.claude/agents/kb-lookup.md`:
   - If not: install it from the repo's bundled agent definition
   - If yes: skip without overwriting
6. Append the new KB's domain name, path, and description as a new entry in the KB Registry
7. Check for duplicate entries before appending
8. Output a summary of what was installed, what was skipped, and where the contents folder is located

---

## 6. Daily Use

### 6.1 Querying a KB
From any active Claude Code project session, Claude consults the global CLAUDE.md registry to determine which KB is relevant. It then invokes the KB-Lookup Agent, passing the KB path and the query. The agent traverses the KB via `index.md`, retrieves relevant content, and returns clean results to the main conversation.

### 6.2 Querying Multiple KBs
If a query spans multiple domains, Claude may invoke the KB-Lookup Agent more than once, passing a different KB path each time. Results are returned separately and synthesized in the main conversation.

### 6.3 Local KB Management
From within a KB project folder, the three local skills can be invoked directly:

- `/kb-intake` — ingest new content into the KB
- `/kb-query` — query the KB contents locally
- `/kb-lint` — validate and maintain the KB structure (see Section 7.2)

---

## 7. Maintenance & Updates

### 7.1 Global Update Script
A global update script (`update.sh`) must be included in the KB repo. When run, it:

- Supports a **dry-run mode** (`--dry-run`) that previews all changes without applying them
- Updates the global KB-Lookup Agent at `~/.claude/agents/kb-lookup.md` if a newer version is bundled in the repo
- Updates the KB Registry format in `~/.claude/CLAUDE.md` if the format has changed
- Reads the KB Registry and iterates over all installed KB instances
- For each installed KB, pushes updated versions of:
  - The local `CLAUDE.md`
  - All three local skills (`kb-intake`, `kb-query`, `kb-lint`)
- Reports which instances were updated and which were skipped

### 7.2 Local Linting (kb-lint skill)
The `kb-lint` skill runs at the local KB level and validates the content structure against what the current skill version expects. After a global update pushes a new version of the linting skill, users should run `kb-lint` to check for any structural differences. When run, it must offer one of three paths:

1. **Auto-migrate** — the skill automatically updates the content structure to match the expected schema, preserving all links and file references
2. **Manual fix** — the skill pauses and instructs the user on what needs to change. The user makes edits themselves, then re-runs `kb-lint`. On re-run, the skill checks the structure and either confirms it is correct or points out remaining differences
3. **Pass** — if the structure already matches expectations, the skill validates and exits without prompting

---

## 8. Removal

### 8.1 Remove Script
Each KB repo must include a `remove.sh` script. When run, it:

1. Lists all installed KB instances from the global registry and prompts the user to select which one to remove
2. Asks for **confirmation** before proceeding
3. Removes the KB's entry from the global registry in `~/.claude/CLAUDE.md`
4. Checks whether the registry is now empty:
   - If yes: removes the entire KB Registry section from `~/.claude/CLAUDE.md` and deletes the KB-Lookup Agent at `~/.claude/agents/kb-lookup.md`
   - If no: leaves the global layer intact
5. Outputs a summary of what was removed and what was left in place
