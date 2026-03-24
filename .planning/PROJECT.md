# Vault Mapping System for My-Brain-Is-Full-Crew

## What This Is

A vault-mapping system that lets My-Brain-Is-Full-Crew adapt to any existing Obsidian vault structure instead of requiring users to adopt the project's default folder names. The Architect agent generates a `Meta/vault-map.md` during onboarding that maps logical roles (inbox, projects, people, etc.) to actual folder paths, and all 8 agents resolve paths from this mapping instead of using hardcoded defaults.

## Core Value

Any Obsidian user can adopt the Crew without restructuring their existing vault. Zero-friction onboarding for existing vaults while preserving the current experience for new vaults.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Architect agent enhanced with vault-mapping phase during onboarding (Phase 4 start)
- [ ] `Meta/vault-map.md` generated for both new and existing vaults
- [ ] For existing vaults: Architect scans folder structure, interviews user on ambiguous mappings, writes vault-map.md
- [ ] For new vaults: Architect asks user preferences, offers defaults, writes vault-map.md
- [ ] All 8 agent .md files updated: hardcoded paths replaced with role tokens ({inbox}, {people}, {meetings}, etc.)
- [ ] Path resolution preamble added to all 8 agents explaining how to read and use vault-map.md
- [ ] CLAUDE.md (dispatcher) updated to use role tokens where it references operational paths
- [ ] `docs/vault-mapping.md` created explaining the pattern, path resolution, customization, and defaults
- [ ] Backward compatible: if vault-map.md doesn't exist, agents fall back to current default paths
- [ ] vault-map.md format is simple, human-readable, and easy to manually edit
- [ ] `launchme.sh` and `updateme.sh` scripts updated if they reference hardcoded vault paths

### Out of Scope

- New "Cartographer" agent — vault mapping is folded into the Architect's existing responsibilities
- Changes to `references/agent-orchestration.md`, `references/agents-registry.md`, `references/agents.md` — these are descriptive docs, not operational
- Automatic vault migration or restructuring — we map to existing structure, never move user files
- Changes to `.mcp.json` or MCP integration logic
- Changes to `scripts/generate-skills.py` output format (skills are auto-generated from agents)

## Context

- Issue #15 on gnekt/My-Brain-Is-Full-Crew proposes this feature
- The repo owner (gnekt) agreed with the approach and suggested folding it into the Architect rather than creating a new agent
- Currently ~161 hardcoded folder path references across all agent files (e.g., `00-Inbox/`, `05-People/`, `Meta/`)
- The project uses a PARA + Zettelkasten hybrid structure with numbered prefixes (00- through 07-)
- This is a contribution PR to the upstream repo — changes must be generic, not specific to any one vault

## Constraints

- **Backward compatibility**: Existing users who run `updateme.sh` must not break. If vault-map.md is absent, everything works as before.
- **Minimal diff philosophy**: Changes should be surgical. Don't refactor unrelated code. Don't change agent behavior beyond path resolution.
- **Generic solution**: No knowledge of any specific user's vault baked into the code. The mapping is discovered at runtime.
- **Human-readable format**: vault-map.md must be simple YAML-in-markdown that users can hand-edit if needed.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Fold into Architect, not new agent | Architect already owns onboarding and structure. Repo owner preference. | — Pending |
| Role tokens in agent prompts | Prevents LLMs from latching onto hardcoded paths and ignoring vault-map.md | — Pending |
| Phase 4 timing for vault mapping | Needs user interview data (Phase 1-3) before scanning/mapping | — Pending |
| Agents + CLAUDE.md scope only | Reference docs are descriptive, not operational. Minimal diff. | — Pending |
| New docs/vault-mapping.md | Explains the pattern without cluttering existing reference docs | — Pending |

---
*Last updated: 2026-03-24 after initialization*
