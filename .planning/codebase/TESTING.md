# Testing Patterns

**Analysis Date:** 2026-03-24

## Overview

This codebase does not use a traditional test framework (no Jest, Vitest, or unit test suites). Instead, testing is **manual and behavioral** — agents are validated through:

1. **Trigger phrase testing** — Does the agent activate for its documented triggers?
2. **Output format validation** — Does the agent produce correctly-formatted suggestions?
3. **Vault interaction testing** — Can agents correctly read/write vault files?
4. **Orchestration testing** — Do agents coordinate correctly via the dispatcher?
5. **Language testing** — Do agents respond in the user's language?

This is appropriate for a crew of AI subagents where behavior is deterministic given system prompts, and validation is primarily about format correctness and coordination integrity.

---

## Testing Philosophy

**Why no unit tests:**
- Agents are system prompts (natural language), not code functions
- Claude's behavior is deterministic for well-written prompts
- Primary risk is prompt clarity/instruction ordering, not logic errors
- Testing happens via conversation, not automated test suites

**What IS tested:**
- Prompt structure (valid YAML frontmatter, required sections)
- Trigger phrase coverage (all required languages present)
- Output format compliance (orchestration suggestions match template)
- Vault interaction safety (read/write operations don't corrupt structure)
- Agent coordination (no circular chains, max depth enforcement)

**Testing is the user's responsibility:**
- From CONTRIBUTING.md line 18: `claude --plugin-dir ./` loads agents locally for manual testing
- Encourage real-world usage before merging changes
- Issues reported with concrete examples (CONTRIBUTING.md line 39–44)

---

## Manual Testing Setup

**Prerequisites:**
- Claude Code (Pro, Max, or Team subscription)
- Obsidian vault with `.claude/agents/` directory
- A test vault (not production) for validation

**Local Testing (Development):**

```bash
cd /path/to/My-Brain-Is-Full-Crew
claude --plugin-dir ./
```

This loads agents from the repo's `agents/` directory instead of `.claude/agents/` in the vault.

**Live Testing in Vault:**

```bash
cd /path/to/your-vault
# After cloning repo into vault:
bash My-Brain-Is-Full-Crew/scripts/launchme.sh
# Agents are now loaded from .claude/agents/
```

**Hot Reload:**

Within Claude Code session:
```
/reload-plugins
```

Picks up changes to agent `.md` files without restarting.

---

## Trigger Phrase Testing

**What to test:**
- Agent activates for each documented trigger phrase
- Agent activates for multilingual variants (IT, FR, ES, DE, PT, JA)
- Agent does NOT activate for unrelated keywords

**Example test cases for architect:**

| Trigger | Language | Expected | Test |
|---------|----------|----------|------|
| "initialize the vault" | English | Architect | Say phrase → Architect should activate |
| "create a new area" | English | Architect | Say phrase → Architect should activate |
| "inizializza il vault" | Italian | Architect | Say phrase → Architect should activate |
| "initialiser le vault" | French | Architect | Architect should activate |
| "search the vault" | English | Seeker NOT Architect | Should route to Seeker |

**Testing approach:**

1. In a test vault session, say each trigger phrase
2. Verify the correct agent activates (check dispatcher priority in CLAUDE.md lines 34–43)
3. If multiple agents could match, verify highest priority wins
4. Log failures: "Trigger `[phrase]` did not activate `[agent]`"

**From CONTRIBUTING.md:**

Propose new agents with "Triggers: when should it activate? (include phrases in multiple languages)" — this documents the test contract.

---

## Output Format Validation

**Standard Output Template for Suggestions:**

All agents must use this exact format when suggesting the next agent (see `agent-orchestration.md` lines 24–31):

```markdown
### Suggested next agent
- **Agent**: {name from agents-registry.md}
- **Reason**: {what needs to be done and why}
- **Context**: {relevant details — note titles, folder paths, specific issues}
```

**What to validate:**

1. ✅ Section header is exactly `### Suggested next agent`
2. ✅ All three fields present: Agent, Reason, Context
3. ✅ Agent name matches a valid agent in `references/agents-registry.md`
4. ✅ Reason is a 1-2 sentence summary
5. ✅ Context includes specific file paths or note titles (actionable)

**Example from scribe.md lines 51–56:**

```markdown
### Suggested next agent
- **Agent**: architect
- **Reason**: No area exists for "Personal Finance" — note placed in Inbox as fallback
- **Context**: Created "Monthly Budget.md" in 00-Inbox/. Suggest creating 02-Areas/Personal Finance/ with sub-folders, _index.md, MOC, and templates.
```

✅ Valid — all three fields, specific paths, actionable.

**Test cases:**

| Agent | Scenario | Expected Output |
|-------|----------|-----------------|
| scribe | Creates note but no destination area | Suggests architect with missing area path |
| sorter | Finds 3 related notes not in same project | Suggests connector with note list |
| librarian | Detects broken wikilinks | Suggests seeker for content verification |
| architect | Creates MOC but area doesn't exist | Should have created area first (bug) |

---

## Vault Interaction Testing

**Safe Read Operations:**

All agents verify vault files exist before using them:

```
Read `Meta/user-profile.md` to understand the user's context...
```

**Test:**
1. Create a test vault without `Meta/user-profile.md`
2. Invoke agent (e.g., scribe with "save this")
3. ✅ Agent should handle missing file gracefully (note: log error, continue with defaults)
4. ❌ Agent should NOT crash or produce malformed output

**Safe Write Operations:**

Agents write to vault only to specific directories:
- `00-Inbox/` — Scribe, Transcriber (new notes)
- `01-Projects/`, `02-Areas/`, etc. — Sorter (filing)
- `Meta/` — Architect, Librarian (configuration)
- `MOC/` — Architect (structure)
- `Templates/` — Architect (templates)

**Test:**
1. Create test note in `00-Inbox/test-note.md`
2. Invoke sorter to file it
3. ✅ Note moved to correct area folder
4. ✅ Original location is empty
5. ❌ Note should not be duplicated

**Archival Pattern (Never Delete):**

From CONTRIBUTING.md line 85: "Agents never delete, always archive."

**Test:**
1. Invoke librarian on duplicate notes
2. ✅ Duplicate moved to `04-Archive/`
3. ❌ Duplicate should not be deleted

---

## Orchestration Testing

**Call Chain Enforcement:**

From `agent-orchestration.md` lines 69–81:
- No agent invoked twice in one chain: `[scribe, architect, sorter]` valid; `[scribe, architect, scribe]` invalid
- Max depth 3: If 4th agent needed, return results + defer
- No circular references: If A suggests B and B is already in chain, skip

**Test scenarios:**

| Scenario | Chain | Expected | Test |
|----------|-------|----------|------|
| User says "save this" | [] | [scribe] | Scribe handles, suggests next |
| Scribe suggests architect | [scribe] | [scribe, architect] | Architect runs, suggests next |
| Architect suggests sorter | [scribe, architect] | [scribe, architect, sorter] | Sorter runs, depth limit hit |
| Sorter suggests architect | [scribe, architect, sorter] | Return results | Max depth 3 — don't invoke architect |
| User says "save and organize" | [] | [scribe] first | Only one agent per user request start |

**Test approach:**

1. Instrument dispatcher to log call chain at each step
2. Verify chain has no duplicates using `len([scribe, architect]) == 2`
3. Verify chain never exceeds 3
4. Log all deferral messages for user visibility

---

## Language Support Testing

**Requirement:**

From CONTRIBUTING.md line 81: "Agents should read `Meta/user-profile.md` for personalization."

From all agents: "Always respond to the user in their language. Match the language the user writes in."

**Test cases:**

| User Input | Language | Expected Response |
|-----------|----------|-------------------|
| "salvami questo" | Italian | Scribe responds in Italian |
| "cherche dans le vault" | French | Seeker responds in French |
| "busca en el vault" | Spanish | Seeker responds in Spanish |
| "такси" | Russian | Agent should respond in Russian (or gracefully defer) |

**Testing approach:**

1. Write test message in target language
2. Invoke agent
3. ✅ Response contains text in target language
4. ❌ Response should not contain English system prompts (e.g., "Golden Rule:" shouldn't appear)

---

## Prompt Structure Validation

**YAML Frontmatter Check:**

All agent files MUST have valid YAML frontmatter (from CONTRIBUTING.md lines 53–67):

```yaml
---
name: <agent-codename>
description: >
  One paragraph description used for auto-triggering.
  Include trigger phrases in multiple languages.
tools: Read, Write, Edit, Glob, Grep
model: sonnet
---
```

**Validation script approach** (pseudo-code):

```python
def validate_agent(filepath):
    with open(filepath) as f:
        text = f.read()

    # Check structure
    assert text.startswith('---'), "Missing frontmatter start"
    parts = text.split('---\n', 2)
    assert len(parts) == 3, "Malformed frontmatter"

    # Parse YAML
    fm = parse_yaml(parts[1])

    # Validate fields
    assert 'name' in fm, "Missing 'name' field"
    assert 'description' in fm, "Missing 'description' field"
    assert 'tools' in fm, "Missing 'tools' field"
    assert fm['name'].islower(), "name must be lowercase"
    assert '-' in fm['name'] or '_' not in fm['name'], "Use hyphens, not underscores"

    # Check trigger languages
    desc = fm['description']
    for lang in ['EN:', 'IT:', 'FR:', 'ES:']:
        assert lang in desc, f"Missing {lang} triggers"

    # Check body structure
    body = parts[2]
    assert '## Inter-Agent Coordination' in body, "Missing coordination section"
    assert '### Suggested next agent' in body, "Missing suggestion format template"

    return True
```

**Key validations:**

- ✅ Frontmatter is valid YAML
- ✅ Required fields: name, description, tools
- ✅ Name is lowercase, hyphens only
- ✅ Description includes triggers in EN, IT, FR, ES, DE, PT, JA
- ✅ Body includes `## Inter-Agent Coordination`
- ✅ Body includes `### Suggested next agent` section
- ✅ All file references use backticks (`` `Meta/...` ``)

**From `generate-skills.py` lines 22–58:**

The existing skill generator parses agents using regex:

```python
m = re.match(r"^---\n(.*?\n)---\n(.*)", text, re.DOTALL)
```

This validates structure; similar approach works for validation.

---

## Inter-Agent Coordination Testing

**Scenario: Scribe suggests Architect**

**Setup:**
1. Create test note about "Personal Finance" (new topic)
2. Scribe reads `Meta/vault-structure.md` (missing Finance area)
3. Scribe creates note in `00-Inbox/` as fallback

**Expected output:**

```markdown
### Suggested next agent
- **Agent**: architect
- **Reason**: No area exists for "Personal Finance"
- **Context**: Created "Monthly Budget.md" in 00-Inbox/. Suggest creating 02-Areas/Personal Finance/ with structure.
```

**Test validation:**
- ✅ Dispatcher sees this suggestion
- ✅ Consults `references/agents-registry.md` → architect is active
- ✅ Architect is not in chain `[scribe]` → eligible
- ✅ Depth is 1, max is 3 → eligible
- ✅ Dispatcher invokes architect with message: "Call chain so far: [scribe]. You are step 2 of max 3."

**Assertion:**
- Architect receives full context
- Architect creates structure
- Cycle completes

---

## Real-World Testing: Edge Cases

**Test Case 1: Missing Vault Structure**

```
Setup:
  - Delete 02-Areas/ folder
  - Invoke sorter to file notes

Expected:
  - Sorter reads Meta/vault-structure.md → detects missing areas
  - Notes stay in 00-Inbox/
  - Suggests architect: "Cannot file notes — vault structure incomplete"

Assertion:
  - ✅ No notes moved to non-existent folders
  - ✅ Architect suggestion included
```

**Test Case 2: Circular Orchestration**

```
Setup:
  - Scribe creates note, suggests architect
  - Architect suggests sorter
  - Sorter suggests connector
  - Connector suggests architect (circular!)

Expected:
  - Dispatcher sees [scribe, architect, sorter, connector]
  - Depth = 4, max = 3 → STOP
  - Return results to user: "The Connector also detected 5 orphan notes..."
  - Connector's architect suggestion is deferred (not acted on)

Assertion:
  - ✅ No circular execution
  - ✅ Max depth enforced
  - ✅ User informed of deferred work
```

**Test Case 3: Language Fallback**

```
Setup:
  - User writes in Klingon (unsupported language)
  - Agent triggered

Expected:
  - Agent cannot identify user language
  - Gracefully falls back to English
  - Or: Agent asks for clarification

Assertion:
  - ✅ No crash
  - ✅ Usable response
```

---

## Testing Checklist

**Before merging an agent change:**

- [ ] YAML frontmatter is valid (parse test)
- [ ] All required fields present (name, description, tools)
- [ ] Description includes trigger phrases in EN, IT, FR, ES, DE, PT, JA
- [ ] Agent name is lowercase, hyphens only
- [ ] Body includes `## Inter-Agent Coordination` section
- [ ] Body includes `### Suggested next agent` template with all 3 fields
- [ ] All file paths use backticks (`` `Meta/...` ``)
- [ ] No `[[wikilinks]]` in agent prompts (those are vault content, not prompt)
- [ ] Language: "Always respond to the user in their language" present
- [ ] Conservative behavior: references to `Meta/user-profile.md` for personalization
- [ ] Manual test: `claude --plugin-dir ./` and trigger agent locally
- [ ] Verify output format on real user message
- [ ] Check for circular coordination suggestions
- [ ] Verify vault interaction (read/write safety)

**From CONTRIBUTING.md line 48–51:**

"Found that an agent behaves weirdly, gives poor results, or misses edge cases? Open an issue describing the problem with a concrete example."

Standard issue format:
- What you asked the agent to do
- What it actually did
- What you expected
- Your vault structure (roughly) if relevant

---

## Continuous Validation

**Automated checks (recommended, not yet implemented):**

1. **Frontmatter validation:** Run `scripts/generate-skills.py` as validation (currently one-way conversion)
2. **Trigger coverage:** Parse description field → extract triggers in each language → log count
3. **Coordination format:** Grep for `### Suggested next agent` → validate all 3 fields present
4. **File path syntax:** Grep for backticks vs `[[wikilinks]]` in prompt body
5. **Language check:** Grep for "Always respond to the user in their language" in agent body

**Human review checklist (CONTRIBUTING.md):**

Before approving a PR:
1. Check that agent file follows structure
2. Test trigger phrases locally
3. Review orchestration suggestions (will this create circular chains?)
4. Verify language support
5. Check for conservative defaults (never delete, always ask)

---

*Testing analysis: 2026-03-24*
