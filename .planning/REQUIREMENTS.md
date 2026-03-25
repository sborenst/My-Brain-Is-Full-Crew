# Requirements: Vault Mapping System

**Defined:** 2026-03-24
**Core Value:** Any Obsidian user can adopt the Crew without restructuring their existing vault

## v1 Requirements

### Vault Mapping

- [x] **MAP-01**: Architect creates `Meta/vault-map.md` with YAML-in-markdown format mapping logical roles to actual folder paths
- [x] **MAP-02**: For existing vaults, Architect scans top-level and nested folder structure to detect current layout
- [x] **MAP-03**: For existing vaults, Architect interviews user on ambiguous or missing mappings (e.g., "Where do you keep people notes?")
- [x] **MAP-04**: For new vaults, Architect asks user folder name preferences and offers current defaults as suggestions
- [x] **MAP-05**: vault-map.md is human-readable and manually editable
- [x] **MAP-06**: Vault mapping runs at the start of Phase 4 (after user interview, before folder creation)

### Agent Path Resolution

- [x] **AGT-01**: All 8 agent .md files have hardcoded paths replaced with role tokens ({inbox}, {projects}, {areas}, {resources}, {archive}, {people}, {meetings}, {daily}, {templates}, {meta}, {moc})
- [x] **AGT-02**: All 8 agents have a path resolution preamble explaining how to read vault-map.md and resolve tokens
- [ ] **AGT-03**: CLAUDE.md updated with role tokens where it references vault paths operationally
- [x] **AGT-04**: If vault-map.md is absent, agents fall back to current default paths (full backward compatibility)

### Documentation

- [ ] **DOC-01**: `docs/vault-mapping.md` created explaining the vault-map pattern, path resolution, customization, and default mappings

### Scripts

- [x] **SCR-01**: `launchme.sh` and `updateme.sh` reviewed and updated if they contain hardcoded vault path references

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
| MAP-01 | Phase 2 | Complete |
| MAP-02 | Phase 2 | Complete |
| MAP-03 | Phase 2 | Complete |
| MAP-04 | Phase 2 | Complete |
| MAP-05 | Phase 2 | Complete |
| MAP-06 | Phase 2 | Complete |
| AGT-01 | Phase 3 | Complete |
| AGT-02 | Phase 3 | Complete |
| AGT-03 | Phase 3 | Pending |
| AGT-04 | Phase 3 | Complete |
| DOC-01 | Phase 4 | Pending |
| SCR-01 | Phase 1 | Complete |

**Coverage:**
- v1 requirements: 12 total
- Mapped to phases: 12
- Unmapped: 0

---
*Requirements defined: 2026-03-24*
*Last updated: 2026-03-24 after roadmap creation*
