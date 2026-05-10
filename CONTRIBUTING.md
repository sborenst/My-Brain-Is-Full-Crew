# Contributing to My Brain Is Full - Crew

Thank you for your interest in making the Crew better. This project was born from personal need, and it grows through shared ones.

---

## Ways to contribute

### Improve an existing agent

Found that an agent behaves weirdly, gives poor results, or misses edge cases?

1. Open an issue describing the problem with a concrete example
2. Or submit a PR with the improvement

Agent source files live in `agents/<agent-name>.md`. They use a platform-neutral format with `capabilities:` (not tool names) and `model`: `low`/`mid`/`high` (not platform-specific model names). The build system translates these into each platform's native format. All agents are written in English, and they automatically respond in the user's language.

To test your changes locally, build and install into a test vault:
```bash
bash scripts/build.sh --platform claude-code   # or gemini-cli, opencode, etc.
bash scripts/launchme.sh --platform claude-code --target /tmp/test-vault
```

### Propose a new core crew member

> **Note**: Users can create custom agents directly within their vault by saying "create a new agent". The Architect handles the entire process. The section below is for proposing new *core* agents that ship with the project.

Have an idea for a new core agent? Open an issue with:

- **Name**: both a descriptive English name and a short codename
- **Role**: what problem does it solve?
- **Triggers**: when should it activate? (include phrases in multiple languages)
- **Tool access**: which tools does it need? (Read, Write, Edit, Bash, Glob, Grep)
- **Vault integration**: which folders does it read/write?
- **Inter-agent coordination**: which other agents should it suggest chaining to?
- **Why it matters**: what gap in the current crew does it fill?

### Add usage examples

Real-world examples of how you use the Crew help everyone. Add them to `docs/examples.md` or share them in an issue.

### Report a bug

Open an issue with:
- What you asked the agent to do
- What it actually did
- What you expected
- Your vault structure (roughly) if relevant

---

## Agent file structure

Each agent is a standalone `.md` file with YAML frontmatter in the **source format** (platform-neutral):

```yaml
---
name: <agent-codename>
description: >
  One paragraph description used for auto-triggering.
  Include trigger phrases in multiple languages (English, Italian, French,
  Spanish, German, Portuguese) for maximum discoverability.
capabilities: [read, write, edit]
model: mid
---

# <Display Name> — <Subtitle>

[Agent instructions in English]
```

The build system translates `capabilities` into platform-specific tool lists or permission blocks, and `model` into platform-specific model names (e.g., `mid` → `sonnet` for Claude Code, `gemini-2.5-flash` for Gemini CLI).

### Frontmatter fields (source format)

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Lowercase, hyphens only (e.g., `my-agent`) |
| `description` | Yes | When the platform should auto-invoke this agent. Include multilingual triggers |
| `capabilities` | Yes | List from: `read`, `write`, `edit`, `bash`, `webfetch`, `websearch`, `task`, `todo` |
| `model` | No | `low`, `mid`, or `high` (default: inherits from parent) |
| `exclude` | No | List of platforms to exclude this agent from (e.g., `[opencode]`) |

### Key rules for agent files

1. **Write in English.** All agent instructions are in English. Agents respond in the user's language automatically.
2. **Multilingual triggers.** The `description` field should include natural trigger phrases in at least English and Italian, ideally more languages.
3. **Read user profile.** Agents should read `{{meta}}/user-profile.md` for personalization. Never hardcode personal data.
4. **Use vault-role tokens for folder paths.** Never hardcode folder names like `00-Inbox/` or `Meta/`. Use tokens (`{{inbox}}`, `{{meta}}`, etc.) so the Crew works with any vault structure. See [Vault path tokens](#vault-path-tokens) below.
5. **Inter-agent coordination.** Every agent must include the coordination section with `### Suggested next agent` output format. See `references/agent-orchestration.md`.
6. **Conservative by default.** Agents never delete, always archive. They ask before making structural decisions.
7. **Minimal tools.** Only grant the tools the agent actually needs. Read-only agents should use `disallowedTools: Write, Edit`.

---

## Vault path tokens

Every agent and skill uses **vault-role tokens** instead of hardcoded folder paths. This allows the Crew to work with any Obsidian vault structure.

### The 11 tokens

| Token | Default | Purpose |
|-------|---------|---------|
| `{{inbox}}` | `00-Inbox` | Capture inbox |
| `{{projects}}` | `01-Projects` | Active projects |
| `{{areas}}` | `02-Areas` | Life areas |
| `{{resources}}` | `03-Resources` | Reference material |
| `{{archive}}` | `04-Archive` | Completed content |
| `{{people}}` | `05-People` | Person notes |
| `{{meetings}}` | `06-Meetings` | Meeting notes |
| `{{daily}}` | `07-Daily` | Daily notes |
| `{{templates}}` | `Templates` | Note templates |
| `{{meta}}` | `Meta` | Crew config |
| `{{moc}}` | `MOC` | Maps of Content |

### Rules for using tokens

1. **Every agent/skill file needs a Vault Path Resolution preamble** after the YAML frontmatter. It tells the LLM to read `Meta/vault-map.md` (always this literal path — never `{{meta}}/vault-map.md`, to avoid circular lookup) and substitute the tokens listed in its table.
2. **Only list tokens the file actually uses** in the preamble table.
3. **Only substitute vault-role tokens** — the 11 listed above. Many files use `{{...}}` for template placeholders (`{{date}}`, `{{Name}}`, `{{YYYY}}`, `{{N}}`). Those must **not** be substituted. The preamble must explicitly say this.
4. **Use tokens everywhere**: in instructions, examples, templates, output formats, and `### Suggested next agent` sections. No hardcoded folder names outside the preamble defaults table.

See [`docs/vault-mapping.md`](docs/vault-mapping.md) for the full specification.

---

## Inter-agent coordination

Agents coordinate through a dispatcher-driven orchestration system. When an agent detects work for another agent, it includes a `### Suggested next agent` section in its output. The dispatcher reads this and chains the next agent automatically. The protocol is documented in `references/agent-orchestration.md` and the agent registry is at `references/agents-registry.md`. If your new or improved agent needs to coordinate with existing ones, follow that protocol.

---

## Custom agents vs. core agents

**Custom agents** are created by users within their own vault using the Architect agent. They live in the user's platform agents directory (e.g., `.claude/agents/`) and are personal to that vault. Custom agents:
- Are created through a conversational flow with the Architect
- Follow the same file structure and conventions as core agents
- Participate in the dispatcher's routing and orchestration system
- Have lower priority than core agents
- Are tracked in `references/agents-registry.md` and `references/agents.md`

**Core agents** ship with the project and are maintained by contributors. To propose a new core agent, open an issue (see above).

If your custom agent solves a problem that many users would benefit from, consider proposing it as a core agent!

---

## Agent directory

| File | Agent name | Role | Tools |
|------|-----------|------|-------|
| `architect.md` | Architect | Vault Structure & Setup | Read, Write, Edit, Bash, Glob, Grep |
| `scribe.md` | Scribe | Text Capture | Read, Write, Edit, Glob, Grep |
| `sorter.md` | Sorter | Inbox Triage | Read, Write, Edit, Glob, Grep, Bash |
| `seeker.md` | Seeker | Search & Retrieval | Read, Glob, Grep |
| `connector.md` | Connector | Knowledge Graph | Read, Edit, Glob, Grep |
| `librarian.md` | Librarian | Vault Maintenance | Read, Write, Edit, Bash, Glob, Grep |
| `transcriber.md` | Transcriber | Audio & Transcription | Read, Write, Glob, Grep |
| `postman.md` | Postman | Email & Calendar | Read, Write, Edit, Glob, Grep |

---

## Hooks

Three hooks ship with the crew, protecting vault integrity across all platforms:

| Hook | Event | What it does |
|------|-------|-------------|
| `protect-system-files` | `before-tool-use` | Blocks edits to core agents, skills, references, and the dispatcher file. Custom agents are allowed through. |
| `validate-frontmatter` | `after-tool-use` | Warns if a written `.md` file has broken YAML frontmatter (missing delimiters, tabs, unquoted colons). |
| `notify` | `on-notification` | Sends a desktop notification (macOS/Linux) when the platform needs attention during long agent chains. |

Hook source files live in `hooks/`. Each hook has a `.hook.yaml` (metadata: name, script, triggers, match-tool filters) and a `.sh` (implementation). Hooks are **platform-agnostic** — they read `platform_dir` and `dispatcher_name` from the neutral JSON input to determine which paths to protect. The adapter layer handles translating platform-native events into the neutral schema before calling the hooks.

If you add a new hook:
1. Create `hooks/<name>.hook.yaml` with `name`, `script`, `triggers` (using the neutral event vocabulary: `before-tool-use`, `after-tool-use`, `on-notification`, `on-session-start`, `on-prompt-submit`)
2. Create `hooks/<name>.sh` reading neutral JSON from stdin
3. Use `$PLATFORM_DIR` and `$DISPATCHER_NAME` (extracted from JSON input) instead of hardcoded paths

---

## Adding a new platform adapter

The build system uses a **source-of-truth + per-platform adapters** architecture. Source files (`agents/`, `skills/`, `references/`, `hooks/`, `DISPATCHER.md`) are platform-neutral. Each adapter translates them into a platform's native format.

### Adapter contract

Every adapter is a single file at `adapters/<platform-name>/adapter.sh` that implements these functions:

| Function | Responsibility |
|----------|---------------|
| `adapter_translate_dispatcher(src, dst)` | Copy `DISPATCHER.md` to the platform's dispatcher filename |
| `adapter_translate_references(src, dst)` | Copy reference `.md` files to the platform's references directory |
| `adapter_translate_skills(src, dst)` | Copy skill `SKILL.md` files to the platform's skills directory |
| `adapter_translate_agents(src, dst)` | Translate agent frontmatter (capabilities → tools/permissions, model → native name) and write to agents directory |
| `adapter_translate_hooks(src, dst)` | Copy hook scripts and generate platform-native hook configuration (settings.json, JS plugin, etc.) |
| `adapter_translate_mcp(src, dst)` | Read `mcp/servers.yaml` and write platform-native MCP config |
| `adapter_finalize(src, dst)` | Any final assembly (e.g., merging multiple config files into one) |

The entry point is `adapter_build(src, dst)` which calls all seven functions in order.

### How to add a new platform

1. **Create the adapter directory**: `mkdir -p adapters/<name>/templates/`

2. **Create `adapters/<name>/adapter.sh`** with:
   - Platform constants (e.g., `MY_PLATFORM="my-platform"`, `MY_FW_DIR="myplatform"`, `MY_DISPATCHER="MY_DISPATCH.md"`)
   - Vocabulary mapping functions (capabilities → native tools, events → native events, model tiers → native model names)
   - All 7 `adapter_translate_*` functions + `adapter_build`
   - Call `rewrite_platform_paths "$file" "$MY_FW_DIR" "$MY_DISPATCHER"` on every output text file

3. **Add the platform to install scripts**: add a case to the `case "$PLATFORM"` block in `scripts/launchme.sh` and `scripts/updateme.sh`, setting `DIST_COMPONENTS_DIR`, `VAULT_COMPONENTS_DIR`, `DISPATCHER_SRC`, `DISPATCHER_DST`, `MCP_SRC`, `MCP_DST`, and `HAS_PLUGINS`.

4. **Write tests**: create `tests/adapters/<name>/adapter.test.sh` with tests for each translation function.

5. **Verify**: `bash scripts/build.sh --platform <name>` should produce a complete `dist/<name>/` tree. Check that no `.platform/` or `DISPATCHER.md` placeholders leak into the output.

The shared library `adapters/lib.sh` provides parsing helpers (`parse_frontmatter`, `parse_capabilities`, `should_include`, `parse_hook_yaml`, `agent_body`, `enumerate_agents`, `enumerate_hooks`) and the `rewrite_platform_paths` function. Your adapter sources this automatically via `scripts/build.sh`.

Look at `adapters/claude-code/adapter.sh` or `adapters/gemini-cli/adapter.sh` as reference implementations.

---

## Testing

The project has two levels of tests:

### Unit tests

Per-adapter unit tests live in `tests/adapters/`:

```
tests/adapters/
├── lib.test.sh                    Shared library tests (18 tests)
├── claude-code/adapter.test.sh    CC adapter tests (10 tests)
├── opencode/adapter.test.sh       OC adapter tests (17 tests)
├── opencode/config-merge.test.sh  OC config merge tests (6 tests)
└── gemini-cli/adapter.test.sh     Gemini adapter tests (13 tests)
```

Run them in isolation (each adapter must be tested in its own shell since they share function names):

```bash
# All lib tests
bash -c 'source adapters/lib.sh; source tests/adapters/lib.test.sh; P=0; F=0; for fn in $(declare -F | awk "{print \$3}" | grep "^test_"); do $fn >/dev/null 2>&1 && P=$((P+1)) || { echo "FAIL: $fn"; F=$((F+1)); }; done; echo "$P pass, $F fail"'

# CC adapter tests
bash -c 'source adapters/lib.sh; source adapters/claude-code/adapter.sh; source tests/adapters/claude-code/adapter.test.sh; P=0; F=0; for fn in $(declare -F | awk "{print \$3}" | grep "^test_"); do $fn >/dev/null 2>&1 && P=$((P+1)) || { echo "FAIL: $fn"; F=$((F+1)); }; done; echo "$P pass, $F fail"'

# Same pattern for opencode (grep "^test_oc_") and gemini-cli (grep "^test_gemini_")
```

### Regression test

`tests/regression/run.sh` builds the Claude Code adapter and compares the output byte-for-byte against a pre-captured snapshot. This catches accidental changes to the CC build output.

```bash
bash tests/regression/run.sh
```

If you change source files or the CC adapter, you may need to update the snapshot:

```bash
bash scripts/build.sh --platform claude-code
cp -r dist/claude-code/.claude/* tests/regression/snapshot/.claude/
cp dist/claude-code/CLAUDE.md tests/regression/snapshot/CLAUDE.md
bash tests/regression/run.sh   # should now pass
```

### When to run tests

- After modifying any adapter: run that adapter's tests
- After modifying `adapters/lib.sh`: run all adapter tests
- After modifying source files (agents, skills, references, hooks, DISPATCHER.md): run the regression test
- Before submitting a PR: run everything

---

## Philosophy

This project is built for people who are already overwhelmed. Contributions should make things **simpler**, not more complex.

When in doubt, ask: *"Does this make life easier for someone who's barely keeping it together?"*

If yes, it belongs here.

---

## Code of conduct

Be kind. Treat contributors and users with the same care you'd want when you're not at your best.
