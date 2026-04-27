# Workflow Execution Plan: project-setup

**Dynamic Workflow:** `project-setup`  
**Repository:** `intel-agency/workflow-orchestration-queue-india38-b`  
**Project:** workflow-orchestration-queue (OS-APOW — Sentinel Orchestrator)  
**Plan Created:** 2026-04-27  
**Total Main Assignments:** 6  
**Total Event Assignments:** 3  

---

## 1. Overview

This plan governs the execution of the `project-setup` dynamic workflow for the **workflow-orchestration-queue** project (also known as OS-APOW). The project is a headless agentic orchestration platform that transforms GitHub Issues into autonomous execution orders dispatched to specialized AI worker agents running inside DevContainers.

The workflow performs initial repository setup: it configures GitHub Projects, labels, and branch protection; synthesizes an application plan from the seeded planning documents; scaffolds the actual Python project structure; creates agent-facing documentation; captures lessons learned; and merges everything via a setup PR.

**High-level flow:**

```
pre-script-begin → [6 main assignments × (validate + report)] → post-script-complete
```

**Key directive:** All GitHub Actions workflows created or modified during this workflow MUST pin actions to their specific commit SHA (not version tags).

---

## 2. Project Context Summary

### Project Identity
- **Name:** workflow-orchestration-queue (OS-APOW)
- **Purpose:** Headless agentic orchestration platform — replaces interactive AI coding with autonomous background execution triggered by GitHub Issues and labels
- **Repository:** `intel-agency/workflow-orchestration-queue-india38-b`
- **Self-Bootstrapping:** The system is designed to build itself. Phase 1 is manually seeded; Phases 2–3 are built by the AI using its own orchestration logic.

### Technology Stack
- **Language:** Python 3.12+ (async), PowerShell Core (pwsh), Bash
- **Frameworks:** FastAPI, Uvicorn (ASGI), Pydantic (validation)
- **HTTP Client:** httpx (async)
- **Package Manager:** uv (Rust-based, replaces pip/poetry)
- **Containerization:** Docker, DevContainers, Docker Compose
- **AI Runtime:** opencode CLI (v1.2.24) with ZhipuAI GLM models
- **NOT .NET** — purely Python + Shell ecosystem (no global.json, no .csproj)

### Four Architectural Pillars
| Pillar | Component | Technology |
|--------|-----------|------------|
| Ear | Work Event Notifier | FastAPI webhook receiver, HMAC verification |
| State | Work Queue | GitHub Issues + Labels ("Markdown as a Database") |
| Brain | Sentinel Orchestrator | Async Python polling service, shell-bridge dispatcher |
| Hands | Opencode Worker | DevContainer + LLM agent executing markdown workflows |

### Target Project Structure
```
workflow-orchestration-queue/
├── pyproject.toml
├── uv.lock
├── src/
│   ├── notifier_service.py
│   ├── orchestrator_sentinel.py
│   ├── models/
│   │   └── work_item.py
│   └── queue/
│       └── github_queue.py
├── scripts/
│   ├── devcontainer-opencode.sh
│   ├── gh-auth.ps1
│   └── update-remote-indices.ps1
├── local_ai_instruction_modules/
└── docs/
```

### Key Constraints
- **Action SHA Pinning:** All GitHub Actions must use full commit SHA (not `@v3` tags)
- **3 Required Env Vars Only:** `GITHUB_TOKEN`, `GITHUB_ORG`, `SENTINEL_BOT_LOGIN` (all others hardcoded with sensible defaults per Simplification Report S-3)
- **Environment Reset:** Hardcoded to `"stop"` between tasks (Simplification Report S-4)
- **Single-Repo Polling:** MVP targets one repo only; cross-repo is future phase (S-5)
- **Credential Scrubbing:** Must strip `ghp_*`, `ghs_*`, `gho_*`, `github_pat_*`, `Bearer`, `sk-*`, ZhipuAI keys from all public logs; NO IPv4 scrubbing (S-7)
- **No Encryption Verbiage:** Removed "encrypted" from log descriptions (S-8)
- **Stdout-Only Logging:** No file handlers; use `docker logs` (S-10)
- **No raw_payload Field:** Removed from WorkItem (S-11)

### Known Risks (from Plan Review)
| Risk | Impact | Mitigation |
|------|--------|------------|
| Divergent WorkItem models between sentinel & notifier | High | Unified model in `src/models/work_item.py` (I-1 / R-3) |
| Race condition in task claiming | High | Assign-then-verify pattern (I-2 / R-2) |
| No jittered exponential backoff on poller | Medium | Add backoff with jitter on 403/429 (I-3) |
| httpx.AsyncClient created per-call | Medium | Connection pooling in `__init__()` (I-4 / R-5) |
| Hardcoded secrets in notifier scaffold | Medium | Use `os.environ` with startup validation (I-5 / R-6) |
| No heartbeat implementation | High | Background asyncio coroutine (I-6 / R-1) |
| No cost guardrails | Medium | Deferred; `stalled-budget` label defined but logic deferred (I-7) |
| Bare `except: pass` in claim_task | High | Catch specific exceptions (I-9) |
| No environment reset between tasks | Medium | Add teardown step (I-10) |

### Seeded Reference Code
- `plan_docs/orchestrator_sentinel.py` — reference implementation for Sentinel
- `plan_docs/notifier_service.py` — reference implementation for Notifier
- `plan_docs/src/models/work_item.py` — unified data model
- `plan_docs/src/queue/github_queue.py` — consolidated queue implementation
- `plan_docs/interactive-report.html` — static presentation dashboard

---

## 3. Assignment Execution Plan

### Assignment 1: `init-existing-repository`

**Goal:** Set up the existing repository by configuring GitHub settings, creating an issue-tracking project, importing labels, and opening the initial setup PR.

**Key Acceptance Criteria:**
- Branch `dynamic-workflow-project-setup` created (all work commits here)
- Branch protection ruleset imported from `.github/protected-branches_ruleset.json`
- GitHub Project created and linked to repository (columns: Not Started, In Progress, In Review, Done)
- Labels imported from `.github/.labels.json`
- Workspace and devcontainer files renamed to match project name
- PR opened from branch to `main`

**Project-Specific Notes:**
- The labels file (`.github/.labels.json`) already contains agent-specific labels (`agent:queued`, `agent:in-progress`, `agent:success`, `agent:error`, `agent:infra-failure`, `agent:stalled-budget`) plus standard labels — import all of them.
- Branch protection import requires `GH_ORCHESTRATION_AGENT_TOKEN` with `administration: write` scope (not plain `GITHUB_TOKEN`).
- The repo name `workflow-orchestration-queue-india38-b` should be used when renaming workspace/devcontainer files.
- The devcontainer already references a prebuilt GHCR image — the rename in `devcontainer.json` only affects the `name` field, not the `image` field.

**Prerequisites:**
- GitHub authentication with scopes: `repo`, `project`, `read:project`, `read:user`, `user:email`
- `administration: write` scope on the repository
- `gh` CLI installed and authenticated

**Dependencies:** None (first assignment)

**Risks/Challenges:**
- Branch protection ruleset import may fail if `GH_ORCHESTRATION_AGENT_TOKEN` lacks `administration: write` scope — must report error, not silently skip
- PR creation requires at least one commit — branch must have commits before `gh pr create`
- GitHub Projects API can be inconsistent; may need retry logic

**Events:**
- `post-assignment-complete`: `validate-assignment-completion`, `report-progress`

**Output:** `#initiate-new-repository.init-existing-repository` — includes PR number needed by Assignment 6

---

### Assignment 2: `create-app-plan`

**Goal:** Synthesize the planning documents in `plan_docs/` into a comprehensive application plan, documented as a GitHub Issue with milestones, using the Appendix A template.

**Key Acceptance Criteria:**
- Application template thoroughly analyzed (all plan_docs documents)
- Plan follows the specified Python tech stack and 4-pillar architecture
- Plan documented in a GitHub Issue using the application-plan issue template
- Milestones created for each development phase (Phase 0–3)
- Issue linked to GitHub Project and assigned to "Phase 1: Foundation" milestone
- Labels applied (`planning`, `documentation`)
- `plan_docs/tech-stack.md` and `plan_docs/architecture.md` created
- **NO code written** — planning only

**Project-Specific Notes:**
- There is **no single `ai-new-app-template.md` file** — the application spec is distributed across multiple documents in `plan_docs/`:
  - OS-APOW Implementation Specification v1.2.md (primary app spec)
  - OS-APOW Architecture Guide v3.2.md (system design)
  - OS-APOW Development Plan v4.2.md (phased roadmap)
  - OS-APOW Plan Review.md (gap analysis and recommendations)
  - OS-APOW Simplification Report v1.md (accepted simplifications)
- The agent must synthesize from ALL of these documents to produce a unified plan.
- The issue template is at `.github/ISSUE_TEMPLATE/application-plan.md` — verify it exists.
- The plan should reflect all accepted simplifications from the Simplification Report (S-3 through S-11 are IMPLEMENTED; S-1 and S-2 are KEPT).
- The plan should incorporate the Plan Review recommendations (R-1 through R-8) as implementation requirements.
- Tech stack is Python (async), NOT .NET — adjust all references accordingly.
- **Do NOT apply `orchestration:plan-approved` label** — that is applied by the workflow's `post-script-complete` event after all assignments finish.

**Prerequisites:**
- `init-existing-repository` completed (labels, project, branch exist)
- `gather-context` pre-assignment event completed

**Dependencies:**
- GitHub Project from Assignment 1 (for linking the plan issue)
- Labels from Assignment 1 (for applying `planning`, `documentation`)

**Risks/Challenges:**
- No single app template file — agent must synthesize from multiple documents
- Issue template may reference `.md` appendix format that differs from actual plan_docs structure
- Large amount of content to distill — risk of information overload in the issue
- Phase 3 features already moved to appendix per Simplification Report S-9 — plan must clearly scope MVP to Phases 0–2

**Events:**
- `pre-assignment-begin`: `gather-context`
- `on-assignment-failure`: `recover-from-error`
- `post-assignment-complete`: `validate-assignment-completion`, `report-progress`

**Output:** `#initiate-new-repository.create-app-plan` — includes plan issue number needed for `post-script-complete` event

---

### Assignment 3: `create-project-structure`

**Goal:** Create the actual Python project scaffolding — directory structure, configuration files, Docker setup, CI/CD pipelines, documentation, and the repository summary — based on the approved application plan.

**Key Acceptance Criteria:**
- Python project structure with `pyproject.toml`, `uv.lock`, `src/` layout
- Dockerfile for each service (Sentinel, Notifier) with `COPY src/ ./src/` before `uv pip install -e .`
- `docker-compose.yml` for local development (use Python stdlib for healthchecks, NOT curl)
- Initial CI/CD workflow with all actions pinned to commit SHAs
- Documentation structure (README.md, docs/)
- Repository summary at `.ai-repository-summary.md`
- All builds/tests pass
- Stakeholder approval obtained

**Project-Specific Notes:**
- Tech stack is Python — use `pyproject.toml` + `uv`, NOT `.sln`/`.csproj`
- Reference code in `plan_docs/src/` provides starting implementations:
  - `plan_docs/src/models/work_item.py` → copy to `src/models/work_item.py`
  - `plan_docs/src/queue/github_queue.py` → copy to `src/queue/github_queue.py`
  - `plan_docs/orchestrator_sentinel.py` → copy to `src/orchestrator_sentinel.py`
  - `plan_docs/notifier_service.py` → copy to `src/notifier_service.py`
- Existing scripts/ directory in the template repo already has shell bridge scripts — preserve them
- `.editorconfig` or `ruff.toml` should be used for Python linting/formatting configuration
- CI workflow must run: `uv sync`, `ruff check`, `ruff format --check`, `pytest`
- Dockerfile should use `uv` for dependency installation
- Docker Compose should NOT use `curl` in healthchecks (use Python stdlib instead)
- All GitHub Actions in CI/CD must be pinned to commit SHAs

**Prerequisites:**
- `init-existing-repository` completed (branch exists)
- `create-app-plan` completed (plan issue and milestones exist)

**Dependencies:**
- Plan from Assignment 2 (defines exact structure, components, phases)
- Branch from Assignment 1 (all commits go to `dynamic-workflow-project-setup`)

**Risks/Challenges:**
- Reference code in plan_docs may have known issues (from Plan Review) — should be copied as-is with TODO comments for known gaps
- CI/CD workflow actions must be pinned to SHAs — requires looking up current SHA for each action
- Docker builds must work from the start — healthcheck command must use Python stdlib, not curl
- Large number of files to create — methodical approach needed to avoid missing components

**Events:**
- `post-assignment-complete`: `validate-assignment-completion`, `report-progress`

---

### Assignment 4: `create-agents-md-file`

**Goal:** Create a comprehensive `AGENTS.md` file at the repository root that provides AI coding agents with precise, actionable context about the project.

**Key Acceptance Criteria:**
- `AGENTS.md` exists at repository root
- Contains: project overview, setup/build/test commands (verified), code style, project structure, testing instructions, PR/commit guidelines
- All listed commands have been validated by running them
- File committed and pushed to working branch
- Stakeholder approval obtained

**Project-Specific Notes:**
- Commands should reflect the Python/uv stack: `uv sync`, `uv run pytest`, `uv run ruff check`, `uv run ruff format`
- The file complements (not duplicates) README.md and `.ai-repository-summary.md`
- Must include the agent-specific label conventions (`agent:queued`, `agent:in-progress`, etc.)
- Should document the "Markdown as a Database" pattern and assign-then-verify locking protocol
- Should reference the plan_docs/ directory as the source of truth for architecture decisions
- Must note that all GitHub Actions must be pinned to SHAs

**Prerequisites:**
- `create-project-structure` completed (project files exist, commands can be validated)
- `init-existing-repository` completed (labels and conventions are defined)

**Dependencies:**
- Actual project structure from Assignment 3 (to document accurate directory layout)
- Working build/test commands from Assignment 3 (to verify and document)

**Risks/Challenges:**
- Commands must actually work — if the project structure from Assignment 3 has issues, this assignment will surface them
- Balancing completeness with conciseness — agents perform best with focused content

**Events:**
- `post-assignment-complete`: `validate-assignment-completion`, `report-progress`

---

### Assignment 5: `debrief-and-document`

**Goal:** Produce a comprehensive debriefing report capturing learnings, deviations, errors, and improvement recommendations from the entire workflow execution.

**Key Acceptance Criteria:**
- All 12 sections of the report template completed
- All deviations from assignments documented
- Execution trace saved at `debrief-and-document/trace.md`
- Report committed and pushed
- Stakeholder approval obtained
- Plan-impacting findings flagged as ACTION ITEMS with recommended follow-up

**Project-Specific Notes:**
- The Plan Review document (`OS-APOW Plan Review.md`) identified 10 issues (I-1 through I-10) and 9 recommendations (R-1 through R-9) — these should be tracked as actionable items if not addressed during project structure creation.
- The Simplification Report documented which simplifications were IMPLEMENTED vs KEPT — the debrief should confirm these are reflected in the actual codebase.
- Trace should capture: all `gh` API calls, all file operations, timing data, and any CI failures/fixes.

**Prerequisites:**
- All prior assignments completed (Assignments 1–4)

**Dependencies:**
- Complete execution history of Assignments 1–4
- Validation reports from `validate-assignment-completion` events

**Risks/Challenges:**
- Requires accurate recall of all prior assignment execution details
- May surface issues that require going back to fix (would break the linear flow)

**Events:**
- `post-assignment-complete`: `validate-assignment-completion`, `report-progress`

---

### Assignment 6: `pr-approval-and-merge`

**Goal:** Complete the full PR approval and merge process for the setup PR opened in Assignment 1, including CI remediation, code review, comment resolution, and post-merge cleanup.

**Key Acceptance Criteria:**
- All CI checks pass (with up to 3 remediation attempts)
- Code review delegated to `code-reviewer` subagent (not self-review)
- All review comments resolved following `ai-pr-comment-protocol.md`
- `pr-unresolved-threads.json` is empty (GraphQL verification)
- Merge succeeds (`result = "merged"`)
- Source branch deleted
- Related setup issues closed

**Project-Specific Notes:**
- `$pr_num` comes from `#initiate-new-repository.init-existing-repository` output
- This is an **automated setup PR** — self-approval by the orchestrator is acceptable (no human stakeholder approval required per dynamic workflow)
- CI remediation loop (Phase 0.5) MUST still execute — up to 3 fix cycles
- Must follow the `ai-pr-comment-protocol.md` protocol exactly
- Must log `✓ Read ai-pr-comment-protocol.md` before starting
- ALL commits must be pushed before merge attempt

**Prerequisites:**
- All prior assignments completed and committed (Assignments 1–5)
- PR number available from Assignment 1

**Dependencies:**
- PR number from Assignment 1
- All commits from Assignments 1–5 must be on the branch

**Risks/Challenges:**
- CI may fail on first run — need diagnostic and fix cycle
- Auto-reviewers (Copilot, CodeQL) may add comments that need resolution
- Must ensure no uncommitted local changes before merge
- Branch deletion after merge means no going back

**Events:**
- `post-assignment-complete`: `validate-assignment-completion`, `report-progress`

**Output:** `result` = `"merged"` (or `"pending"`/`"failed"`)

---

### Event Assignment: `create-workflow-plan` (this assignment)

**Goal:** Create this comprehensive workflow execution plan before any other assignment begins.

**Timing:** `pre-script-begin` event (runs before the main assignment loop)

**Output:** `#events.pre-script-begin.create-workflow-plan` — this document at `plan_docs/workflow-plan.md`

---

### Event Assignment: `validate-assignment-completion`

**Goal:** Validate that each completed assignment has successfully met all its acceptance criteria via independent QA verification.

**Timing:** `post-assignment-complete` event (runs after EACH main assignment)

**Key Notes:**
- Must be delegated to an independent `qa-test-engineer` agent (not the implementer)
- For GitHub operations (issues, PRs, projects), must query live repository state
- Validation report saved to `docs/validation/VALIDATION_REPORT_<assignment-name>_<timestamp>.md`
- If FAILED: block progression and notify stakeholder

---

### Event Assignment: `report-progress`

**Goal:** Provide structured progress reporting, output capture, and checkpointing after each workflow step.

**Timing:** `post-assignment-complete` event (runs after EACH main assignment, after validation)

**Key Notes:**
- Must include **Deviations & Findings** and **Plan-Impacting Discoveries** sections
- All action items MUST be filed as GitHub issues with `priority:low` and `needs-triage` labels
- If no action items, explicitly state `Action Items Filed: none`

---

## 4. Sequencing Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     project-setup Dynamic Workflow                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─── pre-script-begin ─────────────────────────────────────────────────┐  │
│  │  create-workflow-plan  →  plan_docs/workflow-plan.md                │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─── Assignment 1 ────────────────────────────────────────────────────┐   │
│  │  init-existing-repository                                           │   │
│  │  ├── Create branch: dynamic-workflow-project-setup                  │   │
│  │  ├── Import branch protection ruleset                               │   │
│  │  ├── Create GitHub Project + columns                                │   │
│  │  ├── Import labels from .labels.json                                │   │
│  │  ├── Rename workspace/devcontainer files                            │   │
│  │  └── Create PR → outputs: $pr_num                                  │   │
│  │  ─── post-assignment-complete: ───                                  │   │
│  │      validate-assignment-completion → report-progress                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─── Assignment 2 ────────────────────────────────────────────────────┐   │
│  │  create-app-plan                                                    │   │
│  │  ├── pre-assignment-begin: gather-context                           │   │
│  │  ├── Analyze plan_docs/ (all spec documents)                        │   │
│  │  ├── Create plan_docs/tech-stack.md                                 │   │
│  │  ├── Create plan_docs/architecture.md                               │   │
│  │  ├── Create plan issue from application-plan template               │   │
│  │  ├── Create milestones (Phase 0–3)                                  │   │
│  │  └── Link to Project + assign labels                                │   │
│  │  ─── post-assignment-complete: ───                                  │   │
│  │      validate-assignment-completion → report-progress                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─── Assignment 3 ────────────────────────────────────────────────────┐   │
│  │  create-project-structure                                           │   │
│  │  ├── Python project scaffolding (pyproject.toml, src/, tests/)      │   │
│  │  ├── Copy reference code from plan_docs/src/ → src/                 │   │
│  │  ├── Dockerfile + docker-compose.yml                                │   │
│  │  ├── CI/CD workflow (actions pinned to SHAs)                        │   │
│  │  ├── Documentation structure (README, docs/)                        │   │
│  │  └── .ai-repository-summary.md                                      │   │
│  │  ─── post-assignment-complete: ───                                  │   │
│  │      validate-assignment-completion → report-progress                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─── Assignment 4 ────────────────────────────────────────────────────┐   │
│  │  create-agents-md-file                                              │   │
│  │  ├── Gather project context from all docs                           │   │
│  │  ├── Validate build/test commands                                   │   │
│  │  └── Create AGENTS.md at repo root                                  │   │
│  │  ─── post-assignment-complete: ───                                  │   │
│  │      validate-assignment-completion → report-progress                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─── Assignment 5 ────────────────────────────────────────────────────┐   │
│  │  debrief-and-document                                               │   │
│  │  ├── Create debrief report (12 sections)                            │   │
│  │  ├── Save execution trace → debrief-and-document/trace.md          │   │
│  │  └── Flag plan-impacting findings as ACTION ITEMS                   │   │
│  │  ─── post-assignment-complete: ───                                  │   │
│  │      validate-assignment-completion → report-progress                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─── Assignment 6 ────────────────────────────────────────────────────┐   │
│  │  pr-approval-and-merge (input: $pr_num from Assignment 1)          │   │
│  │  ├── CI verification + remediation loop (max 3 attempts)           │   │
│  │  ├── Code review delegation → code-reviewer subagent                │   │
│  │  ├── Resolve all review comments (ai-pr-comment-protocol.md)       │   │
│  │  ├── Merge PR → result = "merged"                                  │   │
│  │  ├── Delete source branch                                          │   │
│  │  └── Close setup issues                                            │   │
│  │  ─── post-assignment-complete: ───                                  │   │
│  │      validate-assignment-completion → report-progress                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─── post-script-complete ────────────────────────────────────────────┐   │
│  │  Apply `orchestration:plan-approved` label to plan issue            │   │
│  │  (from Assignment 2 output)                                         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Open Questions

The following ambiguities should be resolved before or during execution:

| # | Question | Context | Suggested Resolution |
|---|----------|---------|---------------------|
| OQ-1 | **No `ai-new-app-template.md` file in plan_docs/** | The `create-app-plan` assignment expects a single app template file, but the project spec is distributed across 4+ documents | The agent should treat the Implementation Specification v1.2 as the primary app spec and synthesize from all plan_docs documents |
| OQ-2 | **Application-plan issue template availability** | `create-app-plan` references `.github/ISSUE_TEMPLATE/application-plan.md` — verify this exists in the template repo | If missing, the agent should create the plan issue manually following the Appendix A structure from the assignment |
| OQ-3 | **`GH_ORCHESTRATION_AGENT_TOKEN` availability** | Branch protection import (Assignment 1, Step 2) requires this token with `administration: write` scope | If unavailable, skip with explicit error report; branch protection can be configured manually later |
| OQ-4 | **Reference code: scaffold vs. final** | `plan_docs/src/` contains reference implementations — should these be copied as-is (with known issues) or rewritten? | Copy as-is with TODO comments for known issues (I-1 through I-10 from Plan Review); fix in subsequent epics |
| OQ-5 | **Scope of Assignment 3 (project structure)** | The plan describes Phases 0–3 but the MVP scope is Phases 1–2 (Sentinel + Notifier). How much Phase 3 scaffolding should be created? | Create only Phase 1 + Phase 2 components. Phase 3 features stay in the "Future Work" appendix per Simplification Report S-9 |
| OQ-6 | **Existing template files vs. new project files** | The template repo already has scripts/, .devcontainer/, .github/workflows/ — which files are template infrastructure vs. project-specific? | Template infrastructure stays. New project files go in `src/`, `tests/`, `docs/`. Existing scripts in `scripts/` are the shell bridge and should be preserved as-is |

---

*This workflow execution plan was produced by the `create-workflow-plan` assignment as part of the `project-setup` dynamic workflow's `pre-script-begin` event.*
