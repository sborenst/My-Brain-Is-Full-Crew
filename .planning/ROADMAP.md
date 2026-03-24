# Roadmap: Vault Mapping System for My-Brain-Is-Full-Crew

## Overview

This roadmap transforms the Crew from a system with hardcoded folder paths into one that adapts to any existing Obsidian vault. The work proceeds in four phases: first auditing the scope of hardcoded paths in scripts, then building the Architect's vault-mapping capability, then replacing hardcoded paths across all 8 agents and CLAUDE.md with role tokens, and finally documenting the pattern so users understand and can customize it.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Scripts Audit** - Review and update launchme.sh and updateme.sh for hardcoded vault path references
- [ ] **Phase 2: Vault Mapping** - Enhance Architect agent with vault-mapping onboarding phase that generates vault-map.md
- [ ] **Phase 3: Agent Path Resolution** - Replace all hardcoded paths across 8 agents and CLAUDE.md with role tokens and resolution preamble
- [ ] **Phase 4: Documentation** - Create docs/vault-mapping.md explaining the pattern, customization, and defaults

## Phase Details

### Phase 1: Scripts Audit
**Goal**: Scripts are clean of hardcoded vault paths or have well-understood fallback behavior
**Depends on**: Nothing (first phase)
**Requirements**: SCR-01
**Success Criteria** (what must be TRUE):
  1. `launchme.sh` either contains no hardcoded vault folder references or has been updated to use configurable/dynamic paths
  2. `updateme.sh` either contains no hardcoded vault folder references or has been updated to use configurable/dynamic paths
  3. Any path references in scripts are documented in a comment explaining their purpose
**Plans:** 1 plan

Plans:
- [x] 01-01-PLAN.md — Audit both scripts for hardcoded vault content paths and add minimal documentation if warranted

### Phase 2: Vault Mapping
**Goal**: Architect agent guides users through vault mapping during onboarding and produces a vault-map.md that covers both new and existing vault scenarios
**Depends on**: Phase 1
**Requirements**: MAP-01, MAP-02, MAP-03, MAP-04, MAP-05, MAP-06
**Success Criteria** (what must be TRUE):
  1. Running Architect onboarding on an existing vault produces a `Meta/vault-map.md` that reflects the user's actual folder layout after an interview covering ambiguous mappings
  2. Running Architect onboarding on a new vault produces a `Meta/vault-map.md` with user-chosen or default folder names
  3. The generated vault-map.md is valid YAML-in-markdown that a human can open, read, and manually edit without documentation
  4. Vault mapping runs at the correct onboarding sequence position (Phase 4, after user interview)
**Plans**: TBD

### Phase 3: Agent Path Resolution
**Goal**: All 8 agent files and CLAUDE.md use role tokens for vault paths and can resolve those tokens from vault-map.md, with full backward compatibility when vault-map.md is absent
**Depends on**: Phase 2
**Requirements**: AGT-01, AGT-02, AGT-03, AGT-04
**Success Criteria** (what must be TRUE):
  1. No hardcoded folder paths (e.g., `00-Inbox/`, `05-People/`) remain in any of the 8 agent .md files or CLAUDE.md
  2. Each agent file contains a path resolution preamble that explains how to read vault-map.md and substitute role tokens
  3. When vault-map.md is absent, agents behave identically to their pre-change behavior using built-in default paths
  4. A user with a non-standard vault layout who has a valid vault-map.md gets agent actions directed to their actual folders
**Plans**: TBD

### Phase 4: Documentation
**Goal**: Users and contributors can understand the vault-mapping pattern, customize their vault-map.md, and know the default values from a single reference document
**Depends on**: Phase 3
**Requirements**: DOC-01
**Success Criteria** (what must be TRUE):
  1. `docs/vault-mapping.md` exists and explains what vault-map.md is and why it exists
  2. The document shows all available role tokens, their default paths, and how to override them
  3. The document explains how agents resolve tokens at runtime so contributors understand the pattern
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Scripts Audit | 1/1 | Complete | 2026-03-24 |
| 2. Vault Mapping | 0/TBD | Not started | - |
| 3. Agent Path Resolution | 0/TBD | Not started | - |
| 4. Documentation | 0/TBD | Not started | - |
