# Codebase Concerns

**Analysis Date:** 2026-03-24

---

## Critical Dependencies at Risk

**Postman Agent — Gmail + Google Calendar MCP Connectors:**
- Risk: Complete reliance on Anthropic's HTTP-based MCP servers at `https://gmail.mcp.claude.com/mcp` and `https://gcal.mcp.claude.com/mcp`
- Files: `.mcp.json`, `agents/postman.md` (lines 1-29)
- Impact: If MCP servers go down, disabled, or require authentication changes, Postman becomes completely non-functional. No fallback mechanism exists. Users cannot read email or access calendar features.
- Current mitigation: Manual `.mcp.json` setup during installation; users can skip setup if not needed
- Recommendations: (1) Document alternative manual email import workflows; (2) Add status checking for MCP connectivity; (3) Provide clear error messaging when MCP servers are unavailable

**Claude Code Model Availability:**
- Risk: Entire system depends on Claude Code CLI being available with specific model access
- Files: All agent frontmatter (e.g., `agents/architect.md` line 25, `agents/postman.md` line 28)
- Impact: If Claude Code is deprecated, moved to premium-only, or model assignments change, the entire crew stops working
- Current mitigation: Modular agent design allows selective activation
- Recommendations: Document fallback instructions if Claude Code changes; maintain agent prompts as portable documents

---

## Architectural Fragility

**Call Chain Depth Limit (Max 3 Agents):**
- Problem: Hardcoded 3-agent maximum per user request (`CLAUDE.md` line 140)
- Files: `CLAUDE.md` (lines 136-141), `references/agent-orchestration.md` (lines 80-88)
- Risk: Complex tasks requiring 4+ agents will fail silently. Work gets deferred to user with "say X next" messages, fragmenting workflows. Users must make multiple requests to complete interdependent work.
- Example: User asks Postman to process email → suggests Architect for missing structure → suggests Sorter for filing → suggests Connector for linking. Only first 3 execute; Connector deferred.
- Safe modification: Track failed chains; implement explicit "continue workflow" command; increase limit with proper recursion guards

**Dispatcher Auto-Chaining Logic is Implicit:**
- Problem: CLAUDE.md lines 115-142 describe dispatcher behavior but do NOT explicitly guarantee when chaining occurs
- Files: `CLAUDE.md` (Multi-agent routing section)
- Risk: Agent outputs might suggest next agent correctly but dispatcher might ignore it. No protocol ensures dispatcher actually reads/honors suggestions. This can cause silent work gaps.
- Safe modification: Add explicit validation in dispatcher: "Before responding to user, check all returned `### Suggested next agent` sections and list what was deferred"

**Agent-to-Agent Communication Deprecated But Not Removed:**
- Problem: Old message board system (`Meta/agent-messages.md`) is deprecated but migration path is unclear
- Files: `agents/librarian.md` (lines 52, 383), `references/agent-orchestration.md` (lines 93, 100-104)
- Risk: Legacy vaults might still have old message files. Agents are instructed to ignore them, but no active cleanup happens. This creates dead code paths and potential confusion about what's actually being used.
- Current mitigation: Librarian can rename old files to `-DEPRECATED`
- Recommendations: Add automatic cleanup in Librarian's deep clean mode; document migration path clearly in onboarding

---

## Test Coverage Gaps

**No Unit Test Coverage:**
- Problem: Zero test infrastructure for agent prompts or logic
- Files: No `.test.md`, no `jest.config.js`, no test directory
- Risk: Changes to agent behavior cannot be validated before deployment. Prompt modifications might introduce regressions unnoticed. Cross-agent coordination breaks silently.
- Test coverage: None
- Priority: High — prompt engineering is subtle; small changes have large downstream effects

**No Integration Test Workflow:**
- Problem: No documented way to test multi-agent chains before users run them
- Risk: User reports "when I ask X, the agents don't coordinate" — no reproducible test exists to validate fix
- Safe modification: Create test vault with documented scenarios (e.g., "Postman finds email about new project → Architect creates structure → Sorter files notes")

**Onboarding Not Tested:**
- Problem: Architect's complex onboarding flow (section "Vault Initialization & Onboarding") has no test harness
- Files: `agents/architect.md` (lines 145-237)
- Risk: New users fail during onboarding silently. No way to catch regressions in profile creation or terms acceptance flow
- Safe modification: Create "onboarding checklist" agents can validate against

---

## Security Considerations

**Personal Data in Vault Without Encryption:**
- Risk: Postman saves email content + Gmail metadata to plaintext Obsidian notes in `00-Inbox/`
- Files: `agents/postman.md` (lines 112-232 — template structure)
- Impact: Sensitive information (financial data, health records, passwords) ends up unencrypted on local filesystem. If device is stolen/compromised, all email data is exposed.
- Current mitigation: Strong disclaimer in `docs/DISCLAIMERS.md` (lines 77-94); project explicitly targets personal use only
- Recommendations: (1) Document encryption setup options (encrypted vault plugin); (2) Provide email filtering rules to exclude sensitive senders; (3) Add "sanitize" mode to strip PII before saving

**GDPR/Privacy Compliance Burden on User:**
- Risk: Postman processes third-party personal data (emails contain other people's info) without built-in safeguards
- Files: `docs/DISCLAIMERS.md` (lines 77-94)
- Impact: User is liable for GDPR violations if using Postman on emails containing employee/customer/client data without consent
- Current mitigation: Explicit legal disclaimer requiring user responsibility
- Recommendations: (1) Add pre-filtering warnings; (2) Implement automatic PII masking in email summaries; (3) Provide GDPR-compliant workflow documentation

**No Authentication Audit Trail:**
- Problem: MCP connections to Gmail/Calendar are authenticated at setup but no ongoing validation exists
- Files: `.mcp.json` (lines 3-10)
- Risk: If MCP credentials are compromised or revoked, agents silently fail with no indication of auth failure
- Safe modification: Add health check to dispatcher to verify MCP connectivity on startup

---

## Known Architectural Limitations

**Vault Structure Assumptions Hard-Coded:**
- Problem: All agents assume strict vault structure: `00-Inbox/`, `01-Projects/`, `02-Areas/`, `03-Resources/`, `04-Archive/`, `05-People/`, `06-Meetings/`, `07-Daily/`, `MOC/`, `Templates/`, `Meta/`
- Files: `agents/architect.md` (lines 81-87, 189-195), `agents/sorter.md` (entire filing logic), `agents/seeker.md` (search scoping)
- Risk: Users with different structures will experience agent failures. No graceful fallback if folders are missing or renamed.
- Safe modification: Read structure from `Meta/vault-structure.md` as source of truth rather than assuming defaults; add schema validation

**Multi-Language Support is Prompt-Based Only:**
- Problem: All agent instructions are English; language matching relies on LLM's ability to read user's language and respond accordingly
- Files: All agent files (example: `agents/architect.md` lines 32-34)
- Risk: Language matching is probabilistic — model can fail, especially for non-Latin scripts or less-represented languages. No structured language detection.
- Current mitigation: Agents explicitly instructed to match user language
- Recommendations: (1) Add explicit language detection in dispatcher; (2) Maintain translations of critical instructions; (3) Test with non-English users

**Reactive Structure Detection Can Create Infinite Recursion:**
- Problem: Architect is instructed to "reactively detect missing structure" whenever invoked (section "Reactive Structure Detection")
- Files: `agents/architect.md` (lines 49-68)
- Risk: If Architect detects gap and creates structure, might suggest itself again. No guard against recursive invocation. Combined with dispatcher's 3-agent limit, this could cause work to be partially done.
- Safe modification: Add "already in chain" check; mark structures as "created in this session" to avoid re-detection

---

## Performance Bottlenecks

**Full Vault Scans on Every Defrag:**
- Problem: Architect's weekly defragmentation performs 5 sequential phases with full filesystem scans (lines 77-139)
- Files: `agents/architect.md` (lines 71-139)
- Impact: Large vaults (10k+ notes) will timeout during Bash operations. No pagination, no incremental scanning.
- Safe modification: Implement incremental defrag; scan dates (only files changed since last defrag); add progress reporting

**Librarian's Deep Clean Can Block:**
- Problem: Librarian's "Mode 3: Deep Clean" scans entire vault for duplicates, broken links, frontmatter issues, templates
- Files: `agents/librarian.md` (lines 97-160)
- Risk: Large vaults (5k+ notes) will hit token limits or timeout during link audit phase
- Safe modification: (1) Add parallel processing for link audits; (2) Implement sampling for >3000 notes; (3) Break into sub-phases with progress reports

**Grep/Glob Operations Unoptimized:**
- Problem: Agents use `Grep` and `Glob` with loose patterns that could match thousands of files
- Files: Example in `agents/seeker.md` (search operations), `agents/librarian.md` (vault scans)
- Risk: Queries like `pattern: "\\[\\["` to find wikilinks will scan entire vault including node_modules, git history, backups
- Safe modification: Whitelist directories in all Glob/Grep calls; exclude `.obsidian/`, `.git/`, `node_modules/`

---

## Fragile Areas

**Scribe's Text Refinement Rules Can Over-Correct:**
- Problem: Scribe applies automatic fixes: typo correction, abbreviation expansion, grammar fixes (agents/scribe.md lines 366-369)
- Files: `agents/scribe.md` (lines 366-369)
- Risk: User loses original voice/intent. Code examples get auto-corrected. Non-English abbreviations treated as typos. "xké" → "perché" might not be user's intent.
- Safe modification: (1) Always show diffs before applying fixes; (2) Add "preserve original" mode for code/quotes; (3) Require user approval for non-obvious fixes

**Connector's Relationship Analysis Could Hallucinate Links:**
- Problem: Connector discovers "unexpected overlaps" and "hidden connections" — relies on LLM similarity matching
- Files: `agents/connector.md` (lines 130-165)
- Risk: LLM might suggest spurious connections (e.g., linking notes because they both mention "strategy" even though contexts are unrelated). User trusts connections and builds on false premises.
- Safe modification: (1) Present confidence scores for suggestions; (2) Require user approval before adding links; (3) Maintain reason justifications in added links

**Postman's Email Classification Logic Uses Heuristics:**
- Problem: Priority scoring uses hardcoded weights (+3 for VIP, +2 for action required, etc.) — completely customizable to user
- Files: `agents/postman.md` (lines 100-105)
- Risk: Weights don't reflect user's actual priorities. Important emails marked as low-priority. Spam marked as high-priority.
- Safe modification: Read weights from `Meta/user-profile.md`; allow user to adjust during onboarding

**Transcriber Preserves Transcription Errors By Design:**
- Problem: Transcriber is instructed "never invent content that wasn't said" but "correct obvious transcription errors"
- Files: `agents/transcriber.md` (lines 96-97)
- Risk: What's "obvious" is subjective. Meeting notes might contain garbled action items (e.g., "fix the burgers" vs "fix the bugs"). No way to flag ambiguous corrections.
- Safe modification: (1) Add correction suggestions section before finalizing note; (2) Require user approval for ambiguous fixes; (3) Mark corrected sections with `[TRANSCRIBER NOTE]`

---

## Missing Critical Features

**No Conflict Resolution for Simultaneous Changes:**
- Problem: If two agents modify the same note in parallel (e.g., Sorter files note while Connector links it), no locking or merge strategy exists
- Files: All agents that write (Architect, Scribe, Sorter, Librarian, Connector, Postman, Transcriber)
- Risk: Lost writes; data corruption; conflicting edits
- Workaround: Sequential agent execution (3-agent limit partially mitigates this)
- Recommendations: (1) Implement file-level locking; (2) Add conflict detection with merge strategies; (3) Require file timestamps to validate freshness before writing

**No Undo/Rollback Mechanism:**
- Problem: Agents perform destructive operations (moving files, renaming, merging duplicates) with no undo
- Files: All write agents
- Risk: User asks "merge these duplicates" and notes are combined; user later realizes they shouldn't have merged
- Current mitigation: "Conservative by default" philosophy — agents archive rather than delete
- Recommendations: (1) Maintain operation log; (2) Implement git-like commit/rollback for vault changes; (3) Always keep backups before major operations

**No Rate Limiting on Agent Invocation:**
- Problem: User can spam requests; dispatcher will invoke agents repeatedly with no throttling
- Files: Dispatcher logic in `CLAUDE.md` (no rate limit implementation)
- Risk: User exhausts API quota; agents fail mysteriously
- Safe modification: (1) Track agent invocation frequency; (2) Warn user if >10 agents invoked in 1 hour; (3) Implement exponential backoff for repeated agent requests

**No Scheduled/Automated Maintenance:**
- Problem: Librarian and Architect run only on explicit user request ("weekly review", "defragment")
- Files: `agents/architect.md`, `agents/librarian.md`
- Risk: Vaults degrade over time. Broken links accumulate. Duplicates multiply. MOCs become stale. Unless user remembers to run weekly tasks, vault becomes chaotic.
- Recommendations: Document cron/automation scripts for external scheduling; add integration with Obsidian's built-in periodic note features

---

## Scaling Limits

**Postman Email Processing Limit:**
- Current capacity: Handles up to ~50 unread emails per invocation (agents/postman.md line 152)
- Limit: Breaks at 50+ unread emails; asks user to filter by time window
- Scaling path: (1) Implement pagination/batch processing; (2) Add background processing for large inboxes; (3) Cache processed emails to avoid re-processing

**Vault Size Scaling:**
- Current capacity: Tested/designed for vaults with <1000 notes
- Limit: Seeker and Librarian use full-vault grep/glob patterns; will timeout at 5000+ notes
- Scaling path: (1) Implement incremental indexing; (2) Add note metadata cache; (3) Use Obsidian's built-in search API instead of grep

**Language Model Token Budgets:**
- Current capacity: Single agent invocation must fit in token window
- Limit: Large vault context (>50 files in view) will exceed context window
- Risk: Architect creating structure for large vault might hit token limits mid-task
- Scaling path: (1) Implement context windowing; (2) Add explicit "too large, split into sub-tasks" handling; (3) Use summarization for large file collections

---

## Dependency & Version Management Issues

**No Version Pinning:**
- Problem: `launchme.sh` has no versioning; agents are copied as-is without checking compatibility
- Files: `scripts/launchme.sh`, `scripts/updateme.sh`
- Risk: User has old agents; pulls new dispatcher; incompatibilities silently cause agent failures
- Safe modification: (1) Add version metadata to agent files; (2) Validate compatibility during install; (3) Maintain changelog for breaking changes

**Python Script Dependency Not Enforced:**
- Problem: `scripts/generate-skills.py` requires Python 3 but script just warns if missing (launchme.sh lines 83-103)
- Files: `scripts/launchme.sh` (lines 83-103)
- Risk: Skills for Claude Code Desktop don't get generated; user doesn't realize Cowork won't work until they try
- Safe modification: Make Python 3 a hard requirement for installation; fail loudly if not found

**Git-Based Updates Without Conflict Detection:**
- Problem: `updateme.sh` copies files from repo but doesn't handle user customizations to agent files
- Files: `scripts/updateme.sh`
- Risk: User customized an agent; pulls update; customization is lost
- Safe modification: (1) Warn if agent files have been modified; (2) Implement 3-way merge; (3) Preserve user customizations

---

## Documentation Gaps

**No Clear Emergency Recovery Procedure:**
- Problem: If vault becomes corrupted or agents get stuck, no documented recovery path exists
- Risk: User has no idea how to reset agents, clear message queues, restart dispatcher
- Recommendations: Create "Emergency Recovery" guide documenting: (1) How to reset `Meta/` files; (2) How to restart dispatcher; (3) How to manually restore from backup

**Onboarding Terms Acceptance Has No Enforcement:**
- Problem: Architect asks user to accept terms (agents/architect.md line 237) but no verification that they actually did
- Risk: User might skip terms and proceed; no record of consent
- Safe modification: (1) Require explicit `yes` response; (2) Save acceptance timestamp to `Meta/`; (3) Re-prompt if profile is older than 6 months

**Vault Structure Documentation Not Generated:**
- Problem: `Meta/vault-structure.md` is created by Architect but no schema/template exists for what it should contain
- Risk: Different vaults have inconsistent structure docs; agents can't rely on format
- Safe modification: Create strict schema for `Meta/vault-structure.md`; validate against it in Librarian audits

**MCP Connector Setup is Manual and Fragile:**
- Problem: Users must manually authorize Gmail/Calendar MCP; no clear instructions if authorization fails
- Files: `launchme.sh` (lines 111-130), `README.md` (lines 290-298)
- Risk: Setup fails silently; user doesn't realize Postman isn't connected until they try to use it
- Safe modification: (1) Add validation step to check MCP connectivity; (2) Document auth flow with screenshots; (3) Provide troubleshooting guide

---

## AI Model Behavior Risks

**Hallucinated File Paths:**
- Problem: Agents use Bash/Grep to find files but might suggest paths that don't exist
- Files: All agents using file operations
- Risk: User follows agent suggestion to edit non-existent file; note is lost or created in wrong place
- Safe modification: Always validate file existence before suggesting edits; use `test -f` checks

**Inconsistent Output Formatting:**
- Problem: Agent outputs should follow templates but LLM might deviate (e.g., different frontmatter format, missing sections)
- Risk: Downstream agents expect specific format; deviation causes parsing failures or missed data
- Safe modification: (1) Validate output format; (2) Re-prompt if format is wrong; (3) Add explicit format examples to all templates

**Stochastic Behavior Creates Unreliability:**
- Problem: Same query to same agent might get different results across sessions
- Files: All agents (fundamental LLM behavior)
- Risk: User asks "find duplicates"; one day it finds 3, another day finds 7. Makes it hard to rely on agent output
- Current mitigation: Explicit disclaimer in `docs/DISCLAIMERS.md` (lines 35-53)
- Safe modification: (1) Document variance expectations; (2) Use deterministic seeds where possible; (3) Implement deduplication of agent outputs

---

## Operational Concerns

**No Audit Trail of Agent Actions:**
- Problem: Agents modify vault but changes are not tracked in a central log
- Files: `Meta/agent-log.md` is mentioned but not required
- Risk: User doesn't know what agents changed; can't roll back to previous state; audit compliance fails
- Safe modification: (1) Mandate `Meta/agent-log.md` with every change; (2) Include agent name, action, timestamp, affected files; (3) Implement 30-day retention log

**Dispatcher Chain Failures Are Silent:**
- Problem: If agent in middle of chain fails, remaining agents are skipped with no indication
- Files: Dispatcher logic in `CLAUDE.md`
- Risk: User expects 3 agents to run; 1st completes, 2nd fails, 3rd never invoked; user doesn't realize work is incomplete
- Safe modification: (1) Always report success/failure of each agent; (2) List what was deferred if chain broke; (3) Offer "resume chain" option

**No Resource Usage Warnings:**
- Problem: Large operations (full vault defrag, email processing >50 emails) have no upfront warnings
- Files: All scan-heavy agents
- Risk: User initiates operation that will timeout or use excessive API quota with no warning
- Safe modification: Add preliminary checks: "This will scan 5000 notes — may take 2-3 minutes. Continue? y/n"

---

*Concerns audit: 2026-03-24*
