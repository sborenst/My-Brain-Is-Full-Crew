# Codebase Structure

**Analysis Date:** 2026-03-24

## Directory Layout

```
My-Brain-Is-Full-Crew/                    ← cloned inside user's Obsidian vault
├── agents/                               ← 8 agent definitions (auto-copied to .claude/agents/)
│   ├── architect.md                       1285 lines: Vault structure, governance, onboarding
│   ├── scribe.md                          409 lines: Text capture, note creation
│   ├── sorter.md                          276 lines: Inbox triage, note filing
│   ├── seeker.md                          330 lines: Vault search, content retrieval
│   ├── connector.md                       332 lines: Knowledge graph, link analysis
│   ├── librarian.md                       489 lines: Vault health, maintenance, QA
│   ├── transcriber.md                     485 lines: Audio, meeting transcription
│   └── postman.md                         816 lines: Gmail, Google Calendar integration
├── references/                           ← Shared docs read by agents and dispatcher
│   ├── agent-orchestration.md             Coordination protocol, call chains, suggestions
│   ├── agents-registry.md                 Capabilities matrix, single source of truth
│   └── agents.md                          Detailed agent descriptions (for reference)
├── scripts/                              ← Installation and maintenance
│   ├── launchme.sh                        First-time installer (copies agents to .claude/)
│   ├── updateme.sh                        Update script (pulls new versions)
│   └── generate-skills.py                 Converts agents to Cowork-format skills
├── docs/                                 ← User-facing documentation
│   ├── getting-started.md                 Step-by-step setup for non-technical users
│   ├── examples.md                        Real-world usage patterns
│   ├── DISCLAIMERS.md                     Legal and safety notes
│   ├── mobile-access.md                   Remote Control setup
│   └── agents/                            Deep-dive docs per agent (8 files)
│       ├── architect.md
│       ├── scribe.md
│       ├── sorter.md
│       ├── seeker.md
│       ├── connector.md
│       ├── librarian.md
│       ├── transcriber.md
│       └── postman.md
├── .claude-plugin/                       ← Plugin manifest for --plugin-dir loading
│   └── plugin.json                        Name, version, keywords, author metadata
├── .mcp.json                             ← MCP server definitions (Gmail, Google Calendar)
├── CLAUDE.md                             ← Dispatcher routing rules (MANDATORY read)
├── README.md                             ← Overview, quick start, philosophy
├── CONTRIBUTING.md                       ← Contribution guidelines
├── LICENSE                               ← MIT license
├── TERMS_OF_USE.md                       ← Legal terms (must accept during onboarding)
└── .planning/                            ← GSD planning docs (this directory structure)
    └── codebase/                         ← Analysis documents
        ├── ARCHITECTURE.md               ← Pattern, layers, data flow
        └── STRUCTURE.md                  ← This file
```

After running `launchme.sh`, the user's vault looks like:

```
your-obsidian-vault/
├── .claude/                              ← Created by installer
│   ├── agents/                           ← 8 agents auto-loaded by Claude Code CLI
│   │   ├── architect.md
│   │   ├── scribe.md
│   │   ├── sorter.md
│   │   ├── seeker.md
│   │   ├── connector.md
│   │   ├── librarian.md
│   │   ├── transcriber.md
│   │   └── postman.md
│   ├── skills/                          ← Converted agents for Cowork/Desktop format
│   │   └── {agent-name}/SKILL.md        (8 skill directories)
│   └── references/                      ← Shared docs agents read
│       ├── agent-orchestration.md
│       ├── agents-registry.md
│       └── agents.md
├── CLAUDE.md                            ← Dispatcher (symlink or copy from repo)
├── .mcp.json                            ← Gmail + Calendar (if enabled during setup)
├── My-Brain-Is-Full-Crew/               ← The cloned repo (for updates)
│   └── (all contents above)
└── ... user's Obsidian notes and vault structure
```

## Directory Purposes

**agents/:**
- Purpose: Defines the 8 isolated AI agents that run the crew
- Contains: One `.md` file per agent with YAML frontmatter + system prompt
- Key files: `architect.md` (largest, 1285 lines), `postman.md` (825 lines), rest ~300-500 lines
- Installed to: `.claude/agents/` (auto-loaded by Claude Code at session start)

**references/:**
- Purpose: Shared documentation read by agents and the dispatcher
- Contains: Orchestration protocol, agent registry, detailed agent capabilities
- Used by: All agents reference these when suggesting next agents or reading current vault state
- Key files: `agents-registry.md` (capability matrix), `agent-orchestration.md` (coordination rules)

**scripts/:**
- Purpose: Installation and maintenance automation
- Contains: Bash installers for first-time setup and updates
- Key files: `launchme.sh` (copies to `.claude/`, optionally sets up `.mcp.json`), `generate-skills.py` (converts agents to Cowork format)

**docs/:**
- Purpose: User-facing documentation and examples
- Contains: Setup guides, usage examples, agent deep-dives, disclaimers
- Used by: Users during onboarding and for reference; not loaded by agents
- Key files: `getting-started.md` (non-technical setup), `examples.md` (real-world usage)

**.claude-plugin/:**
- Purpose: Plugin manifest for alternative loading mode
- Contains: Only `plugin.json` with name, version, keywords
- Use case: If user loads via `claude --plugin-dir ./` instead of cloning into vault

## Key File Locations

**Entry Points:**

- `CLAUDE.md`: Dispatcher routing rules and multi-agent coordination logic — **MUST READ FIRST**
  - Defines priority ordering of 8 agents (postman highest, librarian lowest)
  - Contains explicit trigger phrases for each agent in multiple languages
  - Defines call chain tracking, anti-recursion rules (max depth 3), and decision flow
  - The "constitutional authority" of the crew system

**Configuration:**

- `.mcp.json`: MCP server definitions (Gmail, Google Calendar)
  - Type: HTTP MCP servers
  - URLs: `gmail.mcp.claude.com/mcp`, `gcal.mcp.claude.com/mcp`
  - Set up during onboarding if user enables Postman

- `.claude-plugin/plugin.json`: Plugin metadata
  - Used only when loading via `--plugin-dir` flag
  - Contains name, version, keywords, author

**Core Logic (Agents):**

- `agents/architect.md`: Vault structure, onboarding, weekly defragmentation
  - Tools: Read, Write, Edit, Bash, Glob, Grep (full access)
  - Model: Opus (largest model)
  - Key responsibility: "The user will NEVER touch the vault structure — only you"

- `agents/scribe.md`: Text capture, note creation from raw input
  - Tools: Read, Write, Edit, Glob, Grep (no Bash)
  - Model: Sonnet
  - Key responsibility: Transform messy input into clean frontmatter + tags + links

- `agents/sorter.md`: Inbox triage, filing notes to correct locations
  - Tools: Read, Write, Edit, Glob, Grep, Bash
  - Model: Sonnet
  - Key responsibility: Empty inbox daily, route notes to correct areas, update MOCs

- `agents/seeker.md`: Search, retrieve, analyze vault content
  - Tools: Read, Glob, Grep (read-only — cannot modify)
  - Model: Sonnet
  - Key responsibility: Find information, synthesize answers with citations, detect structural gaps

- `agents/connector.md`: Knowledge graph analysis, wikilink suggestions
  - Tools: Read, Edit, Glob, Grep (no Write, no Bash — can edit existing links only)
  - Model: Sonnet
  - Key responsibility: Discover missing connections, analyze link density, suggest MOCs

- `agents/librarian.md`: Vault health, duplicates, broken links, audits
  - Tools: Read, Write, Edit, Bash, Glob, Grep (full access)
  - Model: Opus (complex analysis)
  - Key responsibility: Detect duplicates, fix broken links, ensure structural integrity

- `agents/transcriber.md`: Audio transcription, meeting notes
  - Tools: Read, Write, Edit, Glob, Grep (no Bash)
  - Model: Sonnet
  - Key responsibility: Convert audio/transcriptions to structured meeting notes with action items

- `agents/postman.md`: Gmail, Google Calendar integration
  - Tools: Read, Write, Edit, Glob, Grep + MCP (Gmail, Google Calendar)
  - Model: Sonnet
  - Key responsibility: Read email, create calendar events, track deadlines, draft replies

**Testing:**

- No test files present in codebase (agents are tested via Claude Code interaction, not unit tests)

## Naming Conventions

**Files:**

- Agent files: lowercase with `.md` extension (e.g., `architect.md`, `scribe.md`)
- Reference docs: lowercase with hyphens (e.g., `agent-orchestration.md`, `agents-registry.md`)
- User documentation: PascalCase for main docs (e.g., `README.md`, `CLAUDE.md`, `CONTRIBUTING.md`)
- Scripts: lowercase with `.sh` or `.py` extension (e.g., `launchme.sh`, `updateme.sh`)

**Directories:**

- Agent directory: `agents/` (lowercase, plural)
- Reference directory: `references/` (lowercase, plural)
- Script directory: `scripts/` (lowercase, plural)
- Documentation directory: `docs/` (lowercase, plural)
- Meta/config directory: `.planning/`, `.claude/`, `.github/` (lowercase, dot-prefixed)
- Obsidian vault areas: Numbered prefix + hyphens (e.g., `00-Inbox/`, `01-Projects/`, `02-Areas/`)

**Vault Structure (created by Architect):**

- Areas: `NN-AreaName/` (e.g., `00-Inbox/`, `02-Areas/Personal Finance/`)
- Index files: `_index.md` (underscore-prefixed for sorting to top)
- Maps of Content: `MOC/AreaName.md` (in `MOC/` folder, matches area name)
- Templates: `Templates/NoteType.md` (template files in `Templates/` folder)
- Metadata: `Meta/filename.md` (all vault-wide metadata in `Meta/` folder)
  - `Meta/user-profile.md`: User context (language, projects, preferences)
  - `Meta/vault-structure.md`: What areas/folders exist and their purposes
  - `Meta/agent-log.md`: Audit trail of agent actions
  - `Meta/tag-taxonomy.md`: Defined tags and hierarchy

## Where to Add New Code

**New Agent:**
1. Create `agents/new-agent-name.md` following the template format (YAML frontmatter + system prompt)
2. Register in `references/agents-registry.md` (add row to capability matrix)
3. Add trigger phrases to `CLAUDE.md` routing rules
4. Add documentation to `docs/agents/new-agent-name.md`
5. Update `references/agents.md` with detailed description
6. No code changes needed — dispatcher auto-discovers agents from `.claude/agents/` description field

**New Integration (e.g., external API):**
1. If MCP server available: Add to `.mcp.json` in `mcpServers` object
2. If custom API: Add credentials/config to agent prompt, read from environment variables (never hardcode)
3. Document in agent's system prompt how to call the API
4. Update `references/agents-registry.md` to note new capability

**New Reference Documentation:**
1. Create file in `references/new-doc.md`
2. Link from `CLAUDE.md` or appropriate agent prompt via `.claude/references/new-doc.md` path
3. Follow existing markdown format and convention

**Updating Dispatcher Logic:**
1. Edit `CLAUDE.md` only for routing changes
2. Update routing priority table if agent precedence changes
3. Update trigger phrases if user-visible language changes
4. Update `## Multi-agent routing` section with decision logic
5. Never edit agent files from dispatcher — only agents edit themselves via their prompts

## Special Directories

**`.planning/codebase/`:**
- Purpose: GSD analysis documents (ARCHITECTURE.md, STRUCTURE.md, etc.)
- Generated: Yes (by GSD mapping commands)
- Committed: Yes (to git for reference)
- Used by: GSD planner and executor commands to understand codebase

**`.claude/`:**
- Purpose: Runtime environment for Claude Code (agents, skills, references)
- Generated: Yes (by `launchme.sh`, auto-copied from repo)
- Committed: No (to user's vault, not to this repo)
- Used by: Claude Code CLI auto-loads agents from `.claude/agents/` at session start

**`Meta/`** (inside user's vault after setup):
- Purpose: Vault-wide metadata and configuration
- Generated: Yes (by Architect during onboarding)
- Committed: Yes (user's vault)
- Contains: User profile, structure registry, agent logs, tag taxonomy
- Used by: All agents read from here; Architect/Librarian maintain

**`.github/`:**
- Purpose: GitHub issue templates and workflows
- Generated: No (static)
- Committed: Yes (to repo)
- Used by: Community contributors

## Inter-Agent Dependencies

- **All agents depend on**: `Meta/user-profile.md` (read before processing), `Meta/vault-structure.md` (check before placing/filing content)
- **Scribe depends on**: Architect (for new area structure creation)
- **Sorter depends on**: Architect (for filing destinations), Connector (for cross-linking batches)
- **Seeker depends on**: Nothing (read-only, independent)
- **Connector depends on**: Architect (for MOC creation), Librarian (for broken link fixing)
- **Librarian depends on**: Architect (for structural repairs), Seeker (for content reconciliation)
- **Transcriber depends on**: Architect (for meeting structure), Scribe (for note formatting)
- **Postman depends on**: Sorter (for inbox filing), Architect (for email/deadline structure)
- **Dispatcher depends on**: All agents' `### Suggested next agent` sections for chaining decisions

---

*Structure analysis: 2026-03-24*
