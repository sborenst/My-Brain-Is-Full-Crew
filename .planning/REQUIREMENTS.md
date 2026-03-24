# Requirements: Vault Mapping System

**Defined:** 2026-03-24
**Core Value:** Any Obsidian user can adopt the Crew without restructuring their existing vault

## v1 Requirements

### Vault Mapping

- [ ] **MAP-01**: Architect creates `Meta/vault-map.md` with YAML-in-markdown format mapping logical roles to actual folder paths
- [ ] **MAP-02**: For existing vaults, Architect scans top-level and nested folder structure to detect current layout
- [ ] **MAP-03**: For existing vaults, Architect interviews user on ambiguous or missing mappings (e.g., "Where do you keep people notes?")
- [ ] **MAP-04**: For new vaults, Architect asks user folder name preferences and offers current defaults as suggestions
- [ ] **MAP-05**: vault-map.md is human-readable and manually editable
- [ ] **MAP-06**: Vault mapping runs at the start of Phase 4 (after user interview, before folder creation)

### Agent Path Resolution

- [ ] **AGT-01**: All 8 agent .md files have hardcoded paths replaced with role tokens ({inbox}, {projects}, {areas}, {resources}, {archive}, {people}, {meetings}, {daily}, {templates}, {meta}, {moc})
- [ ] **AGT-02**: All 8 agents have a path resolution preamble explaining how to read vault-map.md and resolve tokens
- [ ] **AGT-03**: CLAUDE.md updated with role tokens where it references vault paths operationally
- [ ] **AGT-04**: If vault-map.md is absent, agents fall back to current default paths (full backward compatibility)

### Documentation

- [ ] **DOC-01**: `docs/vault-mapping.md` created explaining the vault-map pattern, path resolution, customization, and default mappings

### Scripts

- [ ] **SCR-01**: `launchme.sh` and `updateme.sh` reviewed and updated if they contain hardcoded vault path references

## v2 Requirements

### Extended Mapping

- **MAP-07**: Vault-map.md supports nested sub-mappings (e.g., people.work, people.personal)
- **MAP-08**: Architect can re-run vault mapping to update vault-map.md after vault restructuring
- **MAP-09**: Vault health check validates vault-map.md paths still exist

## Out of Scope

| Feature | Reason |
|---------|--------|
| Separate Cartographer agent | Folded into Architect per repo owner preference |
| Automatic vault migration/restructuring | We map to existing structure, never move user files |
| Reference doc updates (agents-registry.md, agents.md, agent-orchestration.md) | Descriptive docs, not operational — minimal diff philosophy |
| MCP integration changes | Unrelated to path resolution |
| generate-skills.py changes | Skills are auto-generated from agent files; they'll pick up changes automatically |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| MAP-01 | — | Pending |
| MAP-02 | — | Pending |
| MAP-03 | — | Pending |
| MAP-04 | — | Pending |
| MAP-05 | — | Pending |
| MAP-06 | — | Pending |
| AGT-01 | — | Pending |
| AGT-02 | — | Pending |
| AGT-03 | — | Pending |
| AGT-04 | — | Pending |
| DOC-01 | — | Pending |
| SCR-01 | — | Pending |

**Coverage:**
- v1 requirements: 12 total
- Mapped to phases: 0
- Unmapped: 12 ⚠️

---
*Requirements defined: 2026-03-24*
*Last updated: 2026-03-24 after initial definition*
