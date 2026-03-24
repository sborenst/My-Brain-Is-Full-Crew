# Coding Conventions

**Analysis Date:** 2026-03-24

## Overview

This codebase consists of 8 AI agent files (Markdown with YAML frontmatter) that function as Claude Code subagents. Agents are system prompts designed to handle specific vault management tasks in an Obsidian note-taking system. The codebase enforces a strict architectural pattern with standardized formatting, naming, and agent coordination protocols.

---

## File Naming Conventions

**Agent Files:**
- Pattern: `agents/{codename}.md` (lowercase, hyphens only)
- Examples: `architect.md`, `scribe.md`, `connector.md`, `librarian.md`
- No spaces, no underscores in filenames

**Reference/Documentation:**
- Pattern: `references/{topic}.md` (kebab-case)
- Examples: `agent-orchestration.md`, `agents-registry.md`
- Pattern: `docs/{guide}.md`
- Examples: CLAUDE.md, CONTRIBUTING.md, README.md

**Script Files:**
- Pattern: `scripts/{purpose}.sh` or `.py` (kebab-case)
- Examples: `launchme.sh`, `updateme.sh`, `generate-skills.py`

---

## Frontmatter Format

**Required YAML fields for agent files:**

```yaml
---
name: <lowercase-codename>
description: >
  One paragraph description for auto-triggering.
  Include trigger phrases in multiple languages.
tools: Read, Write, Edit, Glob, Grep, Bash
model: opus
---
```

**Field Specifications:**

- `name`: Lowercase, hyphens only (e.g., `my-agent`). Must match filename.
- `description`: Multi-line string (`>`). Must include English trigger phrases AND translations in Italian, French, Spanish, German, Portuguese, and Japanese. Auto-triggers based on content.
- `tools`: Comma-separated list. Valid: `Read`, `Write`, `Edit`, `Bash`, `Glob`, `Grep`. No spaces around commas.
- `disallowedTools`: Optional. Read-only agents use `disallowedTools: Write, Edit` (e.g., `seeker.md`).
- `model`: Optional. Values: `sonnet`, `opus`, `haiku`. Default: inherits from parent. Opus for heavy structural work (`architect.md`, `librarian.md`).

**Examples from codebase:**

`architect.md`:
```yaml
---
name: architect
description: >
  Design and evolve the Obsidian vault structure, templates, naming conventions, and
  tag taxonomy. Trigger phrases (multilingual):
  EN: "initialize the vault", "create a new area", "new project", "add template",
  ...
tools: Read, Write, Edit, Bash, Glob, Grep
model: opus
---
```

`seeker.md`:
```yaml
---
name: seeker
description: >
  Search and retrieve information from the Obsidian vault...
tools: Read, Glob, Grep
model: sonnet
---
```

---

## Code Style

**Language:**
- All agent system prompts written in English (universal)
- Agents automatically respond in user's language (no hardcoding needed)
- Documentation in English

**Markdown Format:**
- H1 titles: `# Agent Name — Subtitle`
- H2 sections: `## Section Title`
- H3 subsections: `### Subsection Title`
- Code blocks: Use triple backticks with language tags (markdown, bash, typescript, python, yaml)
- Lists: Use `- ` for unordered, `1. ` for ordered (with empty line before if following text)
- Links: Markdown `[text](URL)` or wikilinks `[[Note Title]]` in vault context

**Line Length:**
- No hard limit enforced, but aim for ~100 characters in descriptions and lists for readability
- Code examples can exceed this for clarity

**Formatting:**
- No auto-formatter (prettier/eslint)
- Manual consistency via code review standards (see CONTRIBUTING.md)
- Bold for emphasis: `**importance**`
- Italics for references: `*note this*`

---

## Naming Patterns

**Agent Names & Codenames:**
- Full name: Descriptive, title-cased (e.g., "Architect", "Scribe", "Librarian")
- Codename: Lowercase, single word, no hyphens (e.g., `architect`, `scribe`)
- Pattern in files: `name: architect`

**Section Headers in Agent Prompts:**
- `## Golden Rule:` — foundational principle
- `## Core Philosophy` — guiding approach
- `## User Profile` — read `Meta/user-profile.md`
- `## Inter-Agent Coordination` — orchestration patterns
- `## [Mode Name]` — operational modes (e.g., `### Mode 1: Standard Capture`)
- `### When to suggest another agent` — dispatcher coordination rules
- `### Output format for suggestions` — structured feedback template

**Variable/Placeholder Patterns:**
- Template variables: `{{variable}}` (double braces, e.g., `{{date}}`, `{{N}}`)
- User inputs: `[context]` (square brackets, e.g., `[note title]`)
- File paths: Backticks with relative paths (e.g., `` `Meta/user-profile.md` ``, `` `02-Areas/Work/` ``)

**Vault Folder Structure (Standard Names):**
```
00-Inbox/           # Capture zone (temporary)
01-Projects/        # Active projects
02-Areas/           # Life areas (Work, Health, Finance, etc.)
03-Resources/       # Reference material
04-Archive/         # Historical/completed
MOC/                # Maps of Content (topic indexes)
Meta/               # Vault metadata & config
Templates/          # Note templates
```

---

## Import Organization

N/A — This codebase has no imports (Markdown/YAML system prompts, not code).

References to shared documentation:

**Order of reference:**
1. Agent's own prompt section headers
2. `.claude/references/agent-orchestration.md` — how to coordinate
3. `.claude/references/agents-registry.md` — agent directory
4. `.claude/references/agents.md` — detailed agent descriptions

**Example from all agents:**
```markdown
For the full orchestration protocol, see `.claude/references/agent-orchestration.md`.
For the agent registry, see `.claude/references/agents-registry.md`.
```

---

## Error Handling & Validation Patterns

**Agent Execution Patterns:**

1. **Read user profile first** — Every agent begins with: "Before processing any note, read `Meta/user-profile.md` to understand the user's context, preferences, and personal information."
   - `architect.md` line 32
   - `scribe.md` line 29
   - `librarian.md` line 31
   - All agents follow this pattern

2. **Check vault structure** — Agents verify that destination folders exist before filing:
   ```
   "Before filing ANY note, verify the destination folder exists in `Meta/vault-structure.md`."
   ```
   - `sorter.md` line 42
   - Mandatory check prevents silent failures

3. **Signal gaps via Suggested next agent** — When structure is missing:
   ```markdown
   ### Suggested next agent
   - **Agent**: architect
   - **Reason**: [What needs to be done]
   - **Context**: [Specific details]
   ```
   - Standardized output format in all agents
   - Examples in `scribe.md` lines 51–56

4. **Never delete, always archive** — Conservative modification:
   - "Agents never delete, always archive." (CONTRIBUTING.md line 85)
   - Applies to all agents with write access

5. **Conservative by default** — From CONTRIBUTING.md:
   - "Agents never delete, always archive. They ask before making structural decisions."
   - When in doubt, signal the dispatcher instead of acting unilaterally

---

## Comments & Documentation

**When to Comment:**
- Agent prompts are self-documenting via section headers
- Inline comments minimal (used only for complex rules or conditionals)
- Example: `architect.md` line 50 uses `**This is a critical capability.**` for emphasis, not comments

**JSDoc/Documentation Style:**
- No formal documentation generation (not code)
- Structured sections with markdown headers explain each agent's behavior
- Mode descriptions formatted as:
  ```markdown
  ### Mode 1: Standard Capture (default)

  [Description]

  **Process**:
  1. [Step]
  2. [Step]
  ```
- Example: `scribe.md` lines 69–100

**Trigger Phrase Documentation:**
- Multilingual triggers grouped by language in description field
- Pattern: `EN: "trigger1", "trigger2"; IT: "trigger1", "trigger2"`
- Example: `architect.md` lines 5–20

---

## Module Design & Exports

**Agent as a Module:**
- Each agent file is a self-contained system prompt (`.md` file)
- Auto-discovered by Claude Code at session start
- No explicit "exports" — agents are loaded via `description` field matching

**Agent Registration:**
- Single source of truth: `references/agents-registry.md`
- Lists all 8 agents with active/inactive status
- Dispatcher consults this before routing

**Vault Metadata Exports:**
- Agents read and write vault configuration files (not code exports):
  - `Meta/user-profile.md`
  - `Meta/vault-structure.md`
  - `Meta/tag-taxonomy.md`
  - `Meta/agent-log.md`

---

## Dispatcher & Coordination Protocol

**Routing Logic** (defined in `CLAUDE.md`):

```markdown
USER MESSAGE → pick agent by priority table → INVOKE
           ↓
     READ OUTPUT → check agents-registry.md
           ↓
  Does output match another agent's capabilities?
     YES + not in chain + depth < 3 → INVOKE next
     NO or limit reached → RESPOND to user
```

**Call Chain Tracking:**
- Start with empty `[]`
- After each agent: append name to chain
- Pass chain position: `"Call chain so far: [scribe, architect]. You are step 3 of max 3."`
- Rules:
  - No duplicates: never invoke same agent twice
  - No circular patterns: if Agent A suggests B and B is in chain, skip
  - Max depth: 3 agents per request
  - On overflow: return results + note deferral

**Example from `agent-orchestration.md` lines 69–81**

---

## Testing & Quality Assurance Patterns

**Pre-commit Verification:**

No formal tests, but agents include:

1. **Language verification** — All English system prompts should automatically respond in user's language
   - Test: Send message in Italian, agent responds in Italian
   - Test: Send message in Japanese, agent responds in Japanese

2. **Trigger detection** — Agent description field tested via keyword matching
   - Test: User says "initialize the vault" → architect activates
   - Test: Typo in trigger → seeker is fallback for vault search

3. **Inter-agent orchestration** — `### Suggested next agent` format validated
   - Test: Agent output includes all three fields (Agent, Reason, Context)
   - Test: Dispatcher chains agent correctly

4. **Vault structure validation** — Agents check for required files
   - Test: `Meta/user-profile.md` readable → agent personalizes
   - Test: `Meta/vault-structure.md` missing → agent logs error

**Manual Testing Pattern:**

From CONTRIBUTING.md lines 18–21:
```bash
claude --plugin-dir ./
```

Use `--plugin-dir` to load agents locally and test:
- Trigger phrase detection
- Output formatting
- Vault interaction (read/write)
- Inter-agent coordination

Reload with `/reload-plugins` to pick up changes without restart.

---

## Special Conventions

**Language Support:**
- Agent prompts written in English (universal/stable)
- Trigger phrases in 7+ languages (EN, IT, FR, ES, DE, PT, JA)
- Agents auto-detect user language and respond accordingly
- No configuration needed for language switching

**Golden Rules (Mandatory):**

1. **From `architect.md` line 38:** "The Human Never Touches the Vault"
   - Only agents reorganize files
   - Users interact via conversation

2. **From `scribe.md` line 21:** "Always respond to the user in their language. Match the language the user writes in."
   - Applied to all agents

3. **From `agent-orchestration.md` line 96:** "Do NOT call other agents"
   - Only dispatcher invokes agents
   - Agents suggest via `### Suggested next agent`

4. **From CONTRIBUTING.md line 85:** "Conservative by default"
   - Never delete → always archive
   - Ask before major changes

---

*Convention analysis: 2026-03-24*
