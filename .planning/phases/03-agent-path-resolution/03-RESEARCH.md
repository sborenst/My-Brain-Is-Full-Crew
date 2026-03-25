# Phase 3: Agent Path Resolution - Research

**Researched:** 2026-03-24
**Domain:** Markdown agent prompt editing — token substitution in LLM system prompts
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

- **Phase boundary**: Modify only the 8 agent .md files and CLAUDE.md. No script changes, no new agents, no documentation files (DOC-01 is Phase 4).
- **Preamble placement**: Inline preamble (~5-8 lines) inserted **after the User Profile section** in each agent.
- **Preamble scope**: Each preamble lists only the default values for roles that specific agent actually uses — not all 11 roles.
- **Preamble self-contained**: No shared reference doc — each agent file is independently operable.
- **Token syntax**: Double curly braces — `{{inbox}}`, `{{projects}}`, `{{people}}`, etc. No trailing slash in the token itself.
- **All 11 roles tokenized**: Including `{{meta}}` and `{{templates}}` — full consistency.
- **Exception — Architect Phase 3b (lines 259-388)**: The vault-mapping section showing default folder names to the user stays as literal values (display values, not operational paths). Do NOT tokenize this section.
- **Fallback — vault-map.md absent**: Warn user once ("No vault-map.md found, using default paths") then proceed with defaults listed in the preamble.
- **Fallback — role missing from vault-map.md**: Warn user about the specific missing role and ask what to do.
- **No disk verification**: Trust vault-map.md; normal file operations handle missing folders gracefully.
- **CLAUDE.md scope**: Zero operational hardcoded paths to replace. Add a short vault-map awareness section explaining that vault-map.md exists and agents handle path resolution (satisfies AGT-03). Descriptive routing-table mentions stay as-is.
- **CLAUDE.md propagation**: Deployed via hard overwrite by launchme.sh/updateme.sh — the addition reaches all users on next update.

### Claude's Discretion

- Exact wording of the preamble text (must cover: read vault-map.md, parse YAML frontmatter, resolve tokens, fallback defaults).
- Exact wording of the vault-map awareness section in CLAUDE.md.
- How to handle Architect's Phase 4 references — some are operational, some are display; Claude determines which to tokenize based on context.
- Order of operations if multiple Meta/ files need reading (user-profile.md, vault-map.md, vault-structure.md).

### Deferred Ideas (OUT OF SCOPE)

- Scripts backup existing CLAUDE.md as CLAUDE_ORIGINAL.md before overwriting — future phase.
- Vault health check validating vault-map.md paths still exist on disk (MAP-09, v2).
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| AGT-01 | All 8 agent .md files have hardcoded paths replaced with role tokens (`{{inbox}}`, `{{projects}}`, etc.) | Token inventory below identifies all 123 references across 8 agents; architect Phase 3b exclusion documented |
| AGT-02 | All 8 agents have a path resolution preamble explaining how to read vault-map.md and resolve tokens | Preamble pattern designed; insertion point (post User Profile section) confirmed in all 8 agents |
| AGT-03 | CLAUDE.md updated with role tokens where it references vault paths operationally | CLAUDE.md confirmed to have zero operational paths; vault-map awareness section is the full AGT-03 delivery |
| AGT-04 | If vault-map.md is absent, agents fall back to current default paths (full backward compatibility) | Fallback behavior defined; warn-once pattern; default table in each preamble is the source of truth |
</phase_requirements>

---

## Summary

This phase is a targeted text-editing operation across 9 files (8 agent .md files + CLAUDE.md). The technical domain is LLM system prompt engineering: specifically, how to instruct a language model at runtime to read a configuration file and use its values to substitute placeholder tokens before acting. No code, no scripts, no new files are created.

The core challenge is precision, not complexity. There are 123 hardcoded path references across the 8 agents, distributed unevenly (architect has 51, sorter 24, postman 21; connector has only 2). The architect.md has a protected zone (lines 259-388, Phase 3b) that must not be tokenized — these are display values shown to the user during vault setup, not operational paths the Architect acts on. Every other hardcoded path outside that zone is a tokenization target.

The preamble design is the key architectural decision of this phase. It must be short enough to not bloat each agent (5-8 lines per the decision), yet complete enough that an LLM can independently resolve paths with no external documentation. The fallback logic (vault-map.md absent → warn once, use defaults) must be explicit in the preamble itself, because the preamble is the agent's only instruction for path handling.

**Primary recommendation:** Treat this as a two-pass edit per agent file — pass 1 inserts the preamble after User Profile; pass 2 finds and replaces all operational hardcoded paths with tokens.

---

## Standard Stack

This phase has no library dependencies. The work is pure Markdown text editing on `.md` files.

### Tools for Implementation

| Tool | Purpose |
|------|---------|
| `Read` | Read each agent file before editing |
| `Edit` | Make surgical replacements (preferred over Write for existing files) |
| `Grep` | Verify all hardcoded paths are found before editing; verify none remain after |

### No Installation Needed

All files are plain Markdown. No npm, no Python packages, no configuration changes.

---

## Architecture Patterns

### vault-map.md Format (from Phase 2 — confirmed)

```yaml
---
inbox: 00-Inbox
projects: 01-Projects
areas: 02-Areas
resources: 03-Resources
archive: 04-Archive
people: 05-People
meetings: 06-Meetings
daily: 07-Daily
templates: Templates
meta: Meta
moc: MOC
---
```

All 11 roles always present. Values are folder names/paths relative to vault root. No trailing slashes in the values.

### The 11 Role Tokens

| Token | Default folder | Agents that use it |
|-------|---------------|--------------------|
| `{{inbox}}` | `00-Inbox` | sorter, postman, architect, transcriber, scribe, librarian |
| `{{projects}}` | `01-Projects` | sorter, postman, architect, librarian, seeker |
| `{{areas}}` | `02-Areas` | sorter, architect, librarian, scribe |
| `{{resources}}` | `03-Resources` | sorter, architect, librarian |
| `{{archive}}` | `04-Archive` | sorter, architect, librarian |
| `{{people}}` | `05-People` | postman, sorter, architect |
| `{{meetings}}` | `06-Meetings` | postman, transcriber, architect |
| `{{daily}}` | `07-Daily` | transcriber, architect |
| `{{templates}}` | `Templates` | architect, scribe |
| `{{meta}}` | `Meta` | all agents (read Meta/user-profile.md, etc.) |
| `{{moc}}` | `MOC` | architect, connector, librarian, sorter |

Note: `{{meta}}` replaces the `Meta/` prefix in paths like `Meta/user-profile.md` → `{{meta}}/user-profile.md`. However — the User Profile section preamble that reads `Meta/user-profile.md` is already established infrastructure. The decision is whether to tokenize `Meta/` in that specific line. Given that `meta: Meta` is in vault-map.md and all 11 roles must be tokenized per the decision, the answer is yes — but be careful not to tokenize `Meta/` inside the Phase 3b display block.

### Preamble Design Pattern

Insert after `## User Profile` section and before `## Inter-Agent Coordination` (or whatever section follows User Profile in each agent). The preamble is its own subsection.

**Template structure (adapt defaults per agent):**

```markdown
## Vault Path Resolution

Read `Meta/vault-map.md` to resolve folder paths used in this file. The file has YAML frontmatter mapping role tokens to actual folder names. For each `{{token}}` in this prompt, substitute the corresponding value from vault-map.md.

If `Meta/vault-map.md` is absent: warn the user once — "No vault-map.md found, using default paths" — then use these defaults:

| Token | Default |
|-------|---------|
| `{{inbox}}` | `00-Inbox` |
| `{{projects}}` | `01-Projects` |
| ... (only roles this agent uses) |

If vault-map.md is present but a role is missing: warn the user — "vault-map.md does not define [role]. What folder should I use?" — and wait for their answer.
```

**Key design constraints:**
- Keep the table to only roles the specific agent actually uses (minimal cognitive load, keeps preamble short).
- The preamble wording must be clear to an LLM: "read → parse YAML → substitute tokens → if absent use defaults."
- No trailing slash in token or default — callers add slashes contextually (`{{inbox}}/my-note.md`).

### Insertion Point in Each Agent

All 8 agents follow the same structural pattern:

```
[YAML frontmatter]
# Agent Name — Title
[Language/golden rule]
---
## User Profile
[reads Meta/user-profile.md]
---
← INSERT PREAMBLE HERE →
## Inter-Agent Coordination  (or next section)
```

The `---` horizontal rule after the User Profile section is the insertion target. The preamble goes between the `---` after User Profile and the start of the next section.

### CLAUDE.md Change

CLAUDE.md has zero operational hardcoded paths (confirmed by code context analysis). The only change is adding a vault-map awareness section. Suggested location: after the routing priority table, before the individual agent sections. Content: 3-5 sentences explaining vault-map.md exists, agents resolve paths from it, and no dispatcher action is required.

---

## Path Reference Inventory by Agent

This is critical information for the planner — it determines task scope per agent.

### architect.md (51 references, ~1285 lines)

- **Protected zone**: lines 259-388 (Phase 3b) — display values, DO NOT tokenize
- **Tokenize everywhere else**: Weekly Defragmentation section (lines 72-160), Reactive Structure Detection examples, Phase 4 folder creation steps, Area Scaffolding Procedure, and any other operational references
- Key patterns to replace: `` `00-Inbox/` ``, `` `01-Projects/` ``, `` `02-Areas/` ``, `` `03-Resources/` ``, `` `04-Archive/` ``, `` `05-People/` ``, `` `06-Meetings/` ``, `` `07-Daily/` ``, `` `Templates/` ``, `` `Meta/` ``, `` `MOC/` ``
- Largest single agent — deserves its own plan task

### sorter.md (24 references, ~276 lines)

- Primary inbox triage agent — heavy `00-Inbox/` user
- All path references are operational
- Example at line 22: "Process all notes sitting in `00-Inbox/`" → "Process all notes sitting in `{{inbox}}`"
- Example at line 42: "verify the destination folder exists in `Meta/vault-structure.md`" → uses `{{meta}}`
- The suggested-next-agent output block examples (lines 53-56) contain paths — these are template examples that show the user what gets output. They should be tokenized since the agent uses them as templates for real outputs.

### postman.md (21 references, ~816 lines)

- Heavy path user: `00-Inbox/`, `05-People/`, `06-Meetings/`, `01-Projects/`
- Notable: postman.md already uses `{{...}}` syntax for template variables like `{{Sender Name}}`, `{{YYYY}}`, `{{MM}}`, `{{name}}` — the role tokens use the same syntax but represent folder paths, not user data placeholders
- This means the preamble must be clear that `{{inbox}}` etc. are vault-role tokens (resolved from vault-map.md) while `{{Sender Name}}` etc. are data placeholders (resolved from email content). They coexist without conflict since role tokens match exactly the 11 role names.
- Example at line 317: `` `06-Meetings/{{YYYY}}/{{MM}}/` `` — after tokenizing becomes `` `{{meetings}}/{{YYYY}}/{{MM}}/` `` (the outer `06-Meetings` becomes `{{meetings}}`, inner `{{YYYY}}` stays as data placeholder)

### scribe.md (8 references, ~409 lines)

- Moderate path usage — primarily `00-Inbox/` and `Meta/`
- Straightforward replacements

### transcriber.md (7 references, ~485 lines)

- Uses `06-Meetings/` and `00-Inbox/` primarily
- Straightforward replacements

### librarian.md (6 references, ~489 lines)

- Uses `00-Inbox/`, `01-Projects/`, `02-Areas/`, `03-Resources/`, `04-Archive/`, `MOC/`
- Straightforward replacements

### seeker.md (5 references, ~330 lines)

- Read-only agent — uses paths in search/grep operations
- Straightforward replacements

### connector.md (2 references, ~332 lines)

- Lightest path user
- Quick edit

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead |
|---------|-------------|-------------|
| Parsing vault-map.md YAML | Custom YAML parser in agent prompt | Instruct the LLM to read the file and parse the frontmatter naturally — LLMs handle YAML frontmatter natively |
| Shared preamble as a separate reference file | A new `references/vault-map-resolution.md` file each agent includes | Inline preamble per agent — self-contained, no new files, survives agent file isolation |
| Complex fallback logic | Multi-step conditional in preamble | Simple "if absent warn and use table" — one sentence each case |
| Token registry | A separate file enumerating all 11 tokens | The preamble table itself is the registry for each agent |

**Key insight:** LLMs do not need procedural code to parse YAML — they understand natural language instructions like "read the YAML frontmatter and substitute." The preamble is prose instructions to the LLM, not code.

---

## Common Pitfalls

### Pitfall 1: Tokenizing the Architect's Phase 3b Display Block

**What goes wrong:** Replacing `00-Inbox` with `{{inbox}}` inside the vault-mapping conversation block (architect.md lines 259-388), which shows these literal names to the user.
**Why it happens:** The grep pattern for hardcoded paths catches these lines just like operational lines.
**How to avoid:** Treat lines 259-388 in architect.md as a no-touch zone. Grep for matches in that range, then manually exclude them. The vault-map.md format block at lines 344-380 in particular must stay as literal values.
**Warning signs:** If the Architect shows users "`{{inbox}}`" in the default folder list dialog instead of "`00-Inbox`", Phase 3b was accidentally tokenized.

### Pitfall 2: Breaking postman.md's Existing Template Variables

**What goes wrong:** The role token `{{meetings}}` and an existing data placeholder `{{Sender Name}}` coexist in postman.md. Confusion arises if the preamble doesn't clearly distinguish them.
**Why it happens:** Both use `{{...}}` syntax. A poorly written preamble might cause the LLM to try to resolve `{{Sender Name}}` from vault-map.md.
**How to avoid:** Preamble wording must be explicit: "The following role tokens are resolved from vault-map.md: `{{inbox}}`, `{{projects}}`, `{{areas}}`, `{{resources}}`, `{{archive}}`, `{{people}}`, `{{meetings}}`, `{{daily}}`, `{{templates}}`, `{{meta}}`, `{{moc}}`. Other `{{...}}` placeholders in this file are data variables (e.g., `{{Sender Name}}`) — do not resolve them from vault-map.md."
**Warning signs:** Postman tries to look up `{{Sender Name}}` in vault-map.md and either errors or uses a wrong path.

### Pitfall 3: Including All 11 Roles in Every Agent's Preamble

**What goes wrong:** Sorter's preamble includes `{{daily}}` and `{{templates}}` even though sorter never uses them, bloating the preamble unnecessarily.
**Why it happens:** Copy-pasting a master preamble without trimming to agent-specific roles.
**How to avoid:** Per the locked decision, each preamble includes only roles that agent actually uses. Use the token inventory table above (per-agent column) to determine which tokens to include.
**Warning signs:** Preambles exceeding ~10 lines; tokens appearing in the table but never in the agent's operational text.

### Pitfall 4: Adding a Trailing Slash to the Token

**What goes wrong:** Writing `{{inbox}}/` as a replacement, or inserting `{{inbox}}/` in paths, then having double slashes like `{{inbox}}//my-note.md`.
**Why it happens:** Original paths are `` `00-Inbox/` `` with trailing slash. Simple regex replace of `00-Inbox/` produces `{{inbox}}/` — the original slash is consumed, which is actually fine — but inconsistency arises if some replacements are `{{inbox}}` and others are `{{inbox}}/`.
**How to avoid:** Strip the trailing slash from the original before replacing. Replace `00-Inbox/` with `{{inbox}}/` is acceptable (the slash belongs to the path separator, not the token). The token itself never has a trailing slash by decision. Be consistent.

### Pitfall 5: Missing the Meta/ prefix in User Profile Lines

**What goes wrong:** Leaving `Meta/user-profile.md` un-tokenized while tokenizing all other `Meta/` occurrences, creating inconsistency.
**Why it happens:** The User Profile section is "established infrastructure" — editors may unconsciously skip it.
**How to avoid:** `Meta/` in any operational instruction line gets tokenized. The User Profile line becomes `{{meta}}/user-profile.md`. Exception: only if explicitly scoped out by the CONTEXT.md (it is not — all 11 roles including `{{meta}}` are in scope).

### Pitfall 6: Forgetting to Update MOC/ References

**What goes wrong:** Only replacing numbered-prefix folders (00-Inbox, 01-Projects, etc.) and missing `MOC/` and `Templates/` and `Meta/` which lack numeric prefixes and are less visually salient.
**Why it happens:** Grep patterns targeting `0N-` prefixes miss non-prefixed folders.
**How to avoid:** Use separate grep passes — one for numeric prefixes, one for `MOC/`, one for `Templates/`, one for `Meta/`.

---

## Code Examples

### Example Preamble — sorter.md

```markdown
## Vault Path Resolution

Read `Meta/vault-map.md` to resolve folder paths used in this file. Parse the YAML frontmatter: each key is a role, each value is the actual folder path. Substitute every `{{token}}` in this prompt with the corresponding value before acting.

If `Meta/vault-map.md` is absent: warn the user once — "No vault-map.md found, using default paths" — then use these defaults:

| Token | Default |
|-------|---------|
| `{{inbox}}` | `00-Inbox` |
| `{{projects}}` | `01-Projects` |
| `{{areas}}` | `02-Areas` |
| `{{resources}}` | `03-Resources` |
| `{{archive}}` | `04-Archive` |
| `{{people}}` | `05-People` |
| `{{meetings}}` | `06-Meetings` |
| `{{moc}}` | `MOC` |
| `{{meta}}` | `Meta` |

If vault-map.md is present but a role is missing: warn the user — "vault-map.md does not define [role]. What folder should I use?" — and wait for their answer before proceeding.
```

### Example Path Replacement — sorter.md line 22

Before:
```
Process all notes sitting in `00-Inbox/`, classify them, move them to the correct vault location, create wikilinks, and update relevant MOC files.
```

After:
```
Process all notes sitting in `{{inbox}}/`, classify them, move them to the correct vault location, create wikilinks, and update relevant `{{moc}}` files.
```

### Example Path Replacement — postman.md with coexisting template variables

Before (line 317):
```
5. **Note creation**: for each relevant event, create a note in `06-Meetings/{{YYYY}}/{{MM}}/` or `00-Inbox/` if it's a future event to plan.
```

After:
```
5. **Note creation**: for each relevant event, create a note in `{{meetings}}/{{YYYY}}/{{MM}}/` or `{{inbox}}/` if it's a future event to plan.
```

Note: `{{YYYY}}` and `{{MM}}` are data placeholders — left unchanged. `06-Meetings` and `00-Inbox` become role tokens.

### Example CLAUDE.md Vault-Map Awareness Section

```markdown
## Vault Path Resolution

This crew uses a vault map (`Meta/vault-map.md`) to adapt to any Obsidian vault structure. When agents reference folder paths, they resolve them from vault-map.md at runtime. If vault-map.md is absent, each agent falls back to its built-in default paths — existing users are unaffected. The Architect generates vault-map.md during onboarding (Phase 3b). No dispatcher action is required for path resolution.
```

---

## State of the Art

| Old Approach | Current Approach | Impact |
|--------------|------------------|--------|
| Hardcoded paths in agent prompts (`00-Inbox/`) | Role tokens resolved from vault-map.md at runtime | Agents work with any vault layout |
| 0 vault configuration files | `Meta/vault-map.md` (from Phase 2) | Single source of truth for folder paths |
| Phase 3b display values (literal) | Phase 3b display values remain literal (unchanged) | Users see real folder names during vault setup |

---

## Open Questions

1. **Architect Phase 4 operational references — which to tokenize**
   - What we know: Phase 4 (lines 390+) contains folder creation commands that reference actual paths. Some of these reference vault-map.md values directly (already partially adapted in Phase 2). Others may still be hardcoded.
   - What's unclear: How many Phase 4 lines outside Phase 3b still have raw hardcoded paths vs. references to vault-map.md values.
   - Recommendation: Read architect.md lines 390-450 during plan execution, apply the rule: if it's an instruction to the LLM to create/move to a specific folder → tokenize. If it's showing the user what folder names look like → leave as-is.

2. **Meta/ in User Profile read instruction**
   - What we know: Every agent has "read `Meta/user-profile.md`" in its User Profile section. Per the locked decision, `{{meta}}` is in scope.
   - What's unclear: The user discussion did not explicitly address whether this line gets tokenized.
   - Recommendation: Tokenize it (`{{meta}}/user-profile.md`) for consistency with the "all 11 roles" decision. The value in vault-map.md defaults to `Meta` so behavior is identical for current users.

---

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | None (Markdown agent files — no unit test framework) |
| Config file | None |
| Quick run command | Manual: `grep -r '00-Inbox\|01-Projects\|02-Areas\|03-Resources\|04-Archive\|05-People\|06-Meetings\|07-Daily\|Templates/\|MOC/' agents/ CLAUDE.md` (should return 0 results outside architect.md lines 259-388) |
| Full suite command | Same grep + visual inspection of preamble insertion in all 9 files |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| AGT-01 | No hardcoded paths remain in operational sections | automated grep | `grep -rn '00-Inbox\|01-Projects\|02-Areas\|03-Resources\|04-Archive\|05-People\|06-Meetings\|07-Daily' agents/ CLAUDE.md` returns 0 results outside architect Phase 3b zone | ❌ Wave 0 (write grep command as verification step) |
| AGT-01 | Non-prefixed folders (Meta/, MOC/, Templates/) also tokenized | automated grep | `grep -rn '` `` `Meta/\|` ``MOC/\|Templates/' agents/ CLAUDE.md` returns 0 operational results | ❌ Wave 0 |
| AGT-02 | All 8 agents have preamble after User Profile section | automated grep | `grep -c 'Vault Path Resolution' agents/*.md` returns 8 | ❌ Wave 0 |
| AGT-03 | CLAUDE.md has vault-map awareness section | automated grep | `grep -c 'vault-map' CLAUDE.md` returns >= 1 | ❌ Wave 0 |
| AGT-04 | Fallback language present in all preambles | automated grep | `grep -c 'vault-map.md is absent' agents/*.md` returns 8 | ❌ Wave 0 |

### Sampling Rate

- **Per task commit:** Run the AGT-01 grep for the specific agent just edited (quick, < 5 seconds)
- **Per wave merge:** Run all grep checks across all agents
- **Phase gate:** All 4 grep checks green before `/gsd:verify-work`

### Wave 0 Gaps

- [ ] No test files to create — verification is grep-based, embedded as task acceptance criteria
- [ ] The grep commands above serve as the test suite — document them in each task's verification step

---

## Sources

### Primary (HIGH confidence)

- `agents/architect.md` (lines 259-388) — Phase 3b vault-map.md format, protected zone boundary, role token list, default values
- `.planning/phases/03-agent-path-resolution/03-CONTEXT.md` — All locked decisions for this phase
- `.planning/phases/02-vault-mapping/02-CONTEXT.md` — vault-map.md format confirmation, all 11 roles always present decision
- `.planning/codebase/CONVENTIONS.md` — `{{variable}}` syntax as existing convention, User Profile section pattern
- `agents/postman.md` — Confirmed coexistence of role tokens and data template variables

### Secondary (MEDIUM confidence)

- `.planning/REQUIREMENTS.md` — AGT-01 through AGT-04 requirement text
- `.planning/codebase/STRUCTURE.md` — Path reference counts per agent (123 total, per-agent distribution)

### Tertiary (LOW confidence)

- None — all findings are grounded in the project's own files

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — no external libraries; pure Markdown editing
- Architecture: HIGH — token format, vault-map.md format, and preamble placement are all locked decisions
- Pitfalls: HIGH — all identified from direct code inspection (postman.md template variable coexistence verified, Phase 3b boundary confirmed at lines 259-388)
- Path inventory: HIGH — reference counts from CONTEXT.md codebase analysis; individual agent file spot-checks confirm patterns

**Research date:** 2026-03-24
**Valid until:** Stable — no external dependencies. Valid until agent files are structurally changed.
