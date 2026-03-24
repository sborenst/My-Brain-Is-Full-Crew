# Architecture

**Analysis Date:** 2026-03-24

## Pattern Overview

**Overall:** Dispatcher-driven multi-agent system orchestrating isolated AI agents through a central routing coordinator.

**Key Characteristics:**
- **Agent autonomy**: Each agent operates independently with its own system prompt, model assignment, and tool restrictions
- **Dispatcher-centric routing**: All agent coordination flows through `CLAUDE.md` — agents never call each other directly
- **Reactive multi-routing**: The dispatcher chains agents based on their output, call history, and depth limits (max 3 agents per request)
- **Language-transparent**: All agents respond in the user's language automatically — no configuration needed
- **Hierarchical capabilities**: Agents are stratified by responsibility level (read-only searchers vs. full-write admins)

## Layers

**Dispatcher Layer:**
- Purpose: Route user requests to agents, read agent output, chain agents reactively, enforce limits
- Location: `CLAUDE.md`
- Contains: Routing priority rules (8 agents in order), multi-agent coordination logic, anti-recursion enforcement
- Depends on: Agent registry at `references/agents-registry.md`, agent descriptions in agent YAML frontmatter
- Used by: All communication originates from user → dispatcher → agent

**Agent Definition Layer:**
- Purpose: Define 8 specialized agents with distinct prompts, tools, models, and trigger phrases
- Location: `agents/` directory (8 `.md` files, one per agent)
- Contains: Each agent has YAML frontmatter (`name`, `description`, `tools`, `model`) + full system prompt body
- Depends on: Shared reference docs at `references/`, user profile at `Meta/user-profile.md`
- Used by: Dispatcher reads agent descriptions automatically; agents read shared references

**Vault Structure Layer:**
- Purpose: Define the PARA+Zettelkasten hybrid structure that agents interact with
- Location: Documented in agent prompts; generated/maintained during initialization
- Contains: `00-Inbox/` (capture), `01-Projects/`, `02-Areas/`, `03-Resources/`, `04-Archive/`, `05-People/`, `06-Meetings/`, `07-Daily/`, `MOC/` (Maps of Content), `Meta/`, `Templates/`
- Depends on: Architect agent for creation and maintenance
- Used by: All agents read/write to this structure; Sorter, Seeker, Connector, Librarian actively maintain it

**Reference Documentation Layer:**
- Purpose: Shared knowledge about orchestration, agent capabilities, and protocols
- Location: `references/` directory
- Contains: `agent-orchestration.md` (coordination protocol), `agents-registry.md` (capability matrix), `agents.md` (detailed descriptions)
- Depends on: Manual updates when agents are added or capabilities change
- Used by: Dispatcher for chaining decisions, agents for suggesting next agents, developers for understanding coordination

**Integration Layer:**
- Purpose: External service connectors for email, calendar, transcription, etc.
- Location: `.mcp.json` (Gmail, Google Calendar), agent prompts (API integrations)
- Contains: MCP server definitions, API credentials via environment variables
- Depends on: User authorization during onboarding
- Used by: Postman (Gmail, Google Calendar), Transcriber (audio APIs), others as extensions

## Data Flow

**Standard User Request Flow:**

1. **User sends message** → Dispatcher reads intent
2. **Dispatcher selects agent** → Uses priority table from CLAUDE.md (postman > transcriber > scribe > seeker > architect > sorter > connector > librarian)
3. **Agent executes** → Reads user profile (`Meta/user-profile.md`), reads vault structure (`Meta/vault-structure.md`), performs task
4. **Agent detects additional work** → Includes `### Suggested next agent` section in output
5. **Dispatcher reads output** → Checks against `agents-registry.md`, validates chain position, decides to chain or return
6. **If chaining** → Invokes next agent with call chain context: `"Call chain so far: [scribe, architect]. You are step 2 of max 3."`
7. **Return to user** → After max depth reached or no more suggestions

**Vault Initialization Flow:**

1. User: *"Initialize my vault"*
2. Dispatcher → Architect
3. Architect: Runs onboarding → asks user profile questions → creates entire folder structure → generates templates, MOCs, tag taxonomy → saves profile
4. Architect (optional): If Postman is enabled, creates `.mcp.json` for Gmail/Calendar
5. User profile saved to `Meta/user-profile.md`

**Capture → File → Link → Maintain Flow:**

1. User: *"Save this: [messy text]"*
2. Dispatcher → Scribe
3. Scribe: Creates note in `00-Inbox/` → checks if area exists via `Meta/vault-structure.md` → if not, suggests Architect
4. If Architect called: Creates missing area structure
5. User: *"Triage my inbox"*
6. Dispatcher → Sorter
7. Sorter: Files notes from `00-Inbox/` to correct locations → suggests Connector if batch is interconnected
8. If Connector called: Adds wikilinks between related notes
9. On schedule: User or cron → Librarian runs health check → detects broken links, duplicates → suggests Architect for structural fixes

**State Management:**

- **Persistent vault state**: Files on disk (`00-Inbox/`, `02-Areas/`, etc.)
- **Metadata state**: `Meta/user-profile.md` (user preferences, language), `Meta/vault-structure.md` (what areas exist), `Meta/agent-log.md` (audit trail)
- **No inter-agent shared state**: No agent-messages.md, no temporary coordination files — dispatcher orchestrates all coordination
- **Language state**: Determined by first user message; agents remember this for entire session

## Key Abstractions

**Agent:**
- Purpose: Autonomous AI specialist with defined trigger phrases, tool set, and model
- Examples: `agents/architect.md`, `agents/scribe.md`, `agents/seeker.md`
- Pattern: YAML frontmatter (metadata) + Markdown system prompt (instructions)

**Suggested Next Agent:**
- Purpose: Signal from one agent to dispatcher that another agent should run
- Examples: Scribe signals Architect when note needs nonexistent area; Connector signals Librarian when broken links found
- Pattern: Markdown section at end of agent output: `### Suggested next agent` with Agent, Reason, Context fields

**Call Chain:**
- Purpose: Track agent invocations in current request to prevent loops and depth overruns
- Examples: `[]` (initial) → `[scribe]` (after Scribe runs) → `[scribe, architect]` (after Architect runs)
- Pattern: Dispatcher maintains chain, passes it to next agent with position info

**User Profile:**
- Purpose: Persistent user context for language, projects, interests, preferences
- Location: `Meta/user-profile.md` (created during onboarding)
- Pattern: YAML metadata + narrative context; used by all agents before processing

**Vault Structure Registry:**
- Purpose: Single source of truth for what folders/areas exist and their purposes
- Location: `Meta/vault-structure.md` (auto-generated and maintained)
- Pattern: Nested folder tree with purpose descriptions; read by Scribe, Sorter, Seeker before placing content

## Entry Points

**User Chat:**
- Location: Claude Code conversation interface (CLI or Desktop)
- Triggers: User types a message in conversation
- Responsibilities: All user messages start here; dispatcher reads intent and routes

**Onboarding:**
- Location: User says "Initialize my vault" or equivalent in any language
- Triggers: First-time setup or explicit onboarding request
- Responsibilities: Architect runs full setup, asks profile questions, creates structure

**Weekly Defragmentation:**
- Location: User says "Weekly review", "defragment the vault", "weekly defrag"
- Triggers: On schedule or explicit request
- Responsibilities: Architect runs full structural audit; Librarian may be chained for link/duplicate cleanup

**Installation:**
- Location: `scripts/launchme.sh` (first-time) or `scripts/updateme.sh` (updates)
- Triggers: User runs bash script from cloned repo
- Responsibilities: Copy agents to `.claude/agents/`, copy references to `.claude/references/`, optionally set up `.mcp.json`

## Error Handling

**Strategy:** Agents complete their task and signal issues; dispatcher chains appropriate agents for remediation. Never block or ask user for clarification.

**Patterns:**

- **Missing structure detected** (Scribe, Sorter, Seeker): Place content in fallback location (usually `00-Inbox/`) + suggest Architect via `### Suggested next agent`
- **Broken links found** (Librarian, Connector): Log issue, suggest Librarian or Architect depending on severity
- **Duplicate notes** (Librarian): Report findings, suggest merge/archive process
- **Malformed frontmatter** (Librarian): Report which notes, suggest Scribe to reformat
- **Max depth reached** (Dispatcher): Return current results, include summary of deferred work (e.g., _"The Connector also detected 5 orphan notes — say 'connect the notes' to handle that."_)
- **Agent invocation fails** (Dispatcher): Log error, return graceful message to user, suggest manual retry

## Cross-Cutting Concerns

**Logging:**
- Approach: Each agent logs significant actions to `Meta/agent-log.md` with timestamp, agent name, action, and context
- Example: "Reactive structure creation triggered by Scribe: created Finance area for [note title]"
- Used for: Audit trail, debugging, vault history

**Validation:**
- Approach: Agents validate their input independently before processing
  - Architect: Checks current structure via `Meta/vault-structure.md` before proposing changes
  - Scribe: Verifies target area exists before placing note (or suggests Architect)
  - Librarian: Confirms frontmatter requirements before logging issues
- Pattern: Read metadata first, validate against vault structure, proceed or suggest remediation

**Authentication & Language:**
- Approach: Language determined by first user message; matched automatically by all agents
- Pattern: Agents detect user language, respond in same language for entire session
- OAuth handled during Postman/Calendar setup (one-time authorization); credentials stored by Claude Code

**Resource Isolation:**
- Approach: Each agent has restricted tool set (e.g., Seeker is read-only, Architect has full Bash)
- Pattern: Defined in YAML frontmatter: `tools: Read, Write, Edit, Bash, Glob, Grep` or restricted subset
- Effect: Seeker cannot accidentally delete notes; Architect can restructure; Scribe can create but not archive

**Multilingual Support:**
- Approach: All agent prompts written in English (for developer clarity), but responses match user language
- Pattern: System prompts include multilingual trigger phrases for all languages; agents detect user language once and respond consistently
- Effect: User can seamlessly mix languages in triggers; responses always match first message language

---

*Architecture analysis: 2026-03-24*
