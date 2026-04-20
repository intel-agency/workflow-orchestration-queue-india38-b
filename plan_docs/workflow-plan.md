# Workflow Execution Plan: `project-setup`

**Workflow:** `project-setup` dynamic workflow
**Repository:** `intel-agency/workflow-orchestration-queue-india38-b`
**Project:** workflow-orchestration-queue — Headless Agentic Orchestration Platform
**Branch:** `dynamic-workflow-project-setup`
**Date:** 2026-04-20
**Trigger:** `prebuild-devcontainer` workflow completed successfully on `main`

---

## 1. Overview

This plan governs the execution of the `project-setup` dynamic workflow, which initializes this newly-cloned template repository into a fully-scaffolded, project-ready state. The workflow consists of 6 sequential assignments executed on a dedicated feature branch. Each assignment is delegated to a specialist sub-agent by the Orchestrator.

**Total Estimated Duration:** ~105 minutes

**Constraint Summary:**

- All work targets branch `dynamic-workflow-project-setup` (branched from `main`)
- GitHub Actions workflows **must** pin actions to specific commit SHAs (not version tags)
- This is project scaffolding only — no runtime application code is executed
- All plan docs in `plan_docs/` are the authoritative source for project context

---

## 2. Source Plan Documents

The following documents in `plan_docs/` provide the authoritative context for all assignments:

| Document | Purpose | Primary Consumer |
|----------|---------|-----------------|
| `OS-APOW Development Plan v4.2.md` | Phased roadmap, user stories, risk assessment | Assignments 2, 3 |
| `OS-APOW Architecture Guide v3.2.md` | System architecture, ADRs, data flow, security model | Assignments 2, 3, 4 |
| `OS-APOW Implementation Specification v1.2.md` | Detailed requirements, tech stack, project structure | Assignments 2, 3 |
| `OS-APOW Plan Review.md` | Code review findings, issues (I-1 through I-10), recommendations (R-1 through R-9) | Assignment 3 |
| `OS-APOW Simplification Report v1.md` | Applied simplifications (S-3 through S-11 implemented; S-1, S-2 kept) | Assignment 3 |
| `orchestrator_sentinel.py` | Reference implementation for Phase 1 Sentinel | Assignment 3 |
| `notifier_service.py` | Reference implementation for Phase 2 Notifier | Assignment 3 |
| `src/models/work_item.py` | Unified data model (WorkItem, TaskType, WorkItemStatus, scrub_secrets) | Assignment 3 |
| `src/queue/github_queue.py` | Consolidated GitHub queue (ITaskQueue ABC + GitHubQueue) | Assignment 3 |

---

## 3. Assignment Execution Order

```
 ┌─────────────────────────────────┐
 │  1. init-existing-repository    │
 │  [devops-engineer]  ~15 min     │
 └──────────────┬──────────────────┘
                │
                ▼
 ┌─────────────────────────────────┐
 │  2. create-app-plan             │
 │  [planner]  ~20 min             │
 └──────────────┬──────────────────┘
                │
                ▼
 ┌─────────────────────────────────┐
 │  3. create-project-structure    │
 │  [backend-developer]  ~30 min   │
 └──────────────┬──────────────────┘
                │
                ▼
 ┌─────────────────────────────────┐
 │  4. create-agents-md-file       │
 │  [documentation-expert]  ~15 min│
 └──────────────┬──────────────────┘
                │
                ▼
 ┌─────────────────────────────────┐
 │  5. debrief-and-document        │
 │  [documentation-expert]  ~15 min│
 └──────────────┬──────────────────┘
                │
                ▼
 ┌─────────────────────────────────┐
 │  6. pr-approval-and-merge       │
 │  [github-expert]  ~10 min       │
 └─────────────────────────────────┘
```

---

## 4. Assignment Details

### Assignment 1: `init-existing-repository`

**Responsible Agent:** `devops-engineer`
**Estimated Duration:** ~15 minutes
**Dependencies:** None (first assignment)
**Status:** 🔵 Not Started

#### Description

Bootstrap the repository: create the feature branch, import branch protection rules, create a GitHub Project (if applicable), import labels from `.github/.labels.json`, rename devcontainer/workspace files to match the new project name, and open a setup PR.

#### Tasks

1. Create branch `dynamic-workflow-project-setup` from `main`
2. Import labels from `.github/.labels.json` using `scripts/import-labels.ps1`
   - Labels include: `agent:queued`, `agent:in-progress`, `agent:success`, `agent:error`, `agent:infra-failure`, `agent:stalled-budget`, `epic`, `story`, `implementation:complete`, etc.
3. Create GitHub Project board for the repository (if applicable per org settings)
4. Verify branch protection rules on `main` (require PR reviews, status checks)
5. Rename workspace file if needed (e.g., `workflow-orchestration-queue-india38-b.code-workspace`)
6. Open a draft PR: `dynamic-workflow-project-setup` → `main` titled "Project Setup: Scaffold workflow-orchestration-queue"

#### Acceptance Criteria

- [ ] Branch `dynamic-workflow-project-setup` exists and is checked out
- [ ] All labels from `.github/.labels.json` are imported to the repository
- [ ] Draft PR is open with title "Project Setup: Scaffold workflow-orchestration-queue"
- [ ] `gh label list` confirms all `agent:*` labels exist
- [ ] No changes to `main` branch (all work on feature branch)

---

### Assignment 2: `create-app-plan`

**Responsible Agent:** `planner`
**Estimated Duration:** ~20 minutes
**Dependencies:** Assignment 1 (requires feature branch to exist)
**Status:** 🔵 Not Started

#### Description

Create a comprehensive application plan as a GitHub Issue with milestones. The plan must synthesize all plan docs into a single actionable issue that serves as the master reference for the project.

#### Tasks

1. Read all plan documents in `plan_docs/`:
   - Development Plan v4.2 (phased roadmap, user stories)
   - Architecture Guide v3.2 (system design, ADRs)
   - Implementation Specification v1.2 (requirements, tech stack, project structure)
   - Plan Review (issues I-1 through I-10, recommendations R-1 through R-9)
   - Simplification Report (applied simplifications S-3 through S-11)
2. Create GitHub milestones aligned with the Development Plan phases:
   - **Phase 0:** Seeding & Bootstrapping
   - **Phase 1:** The Sentinel (MVP)
   - **Phase 2:** The Ear (Webhook Automation)
   - **Phase 3:** Deep Orchestration & Self-Healing
3. Create a master GitHub Issue titled `[Application Plan] workflow-orchestration-queue` containing:
   - Executive summary from Architecture Guide
   - Phase breakdown with user story references
   - Tech stack summary (Python 3.12+, FastAPI, httpx, Pydantic, uv, Docker)
   - Key architectural decisions (ADR 07: Shell-Bridge, ADR 08: Polling-First, ADR 09: Provider-Agnostic)
   - Applied simplifications from the Simplification Report
   - Known issues from Plan Review that must be addressed during implementation
   - Cross-references to plan doc filenames for agent context
4. Link milestones to the issue
5. Use `scripts/create-milestones.ps1` if available

#### Acceptance Criteria

- [ ] GitHub Issue `[Application Plan] workflow-orchestration-queue` exists
- [ ] Issue body contains: executive summary, 4 phases with stories, tech stack, ADRs
- [ ] Milestones exist: Phase 0 (Seeding), Phase 1 (Sentinel), Phase 2 (Ear), Phase 3 (Deep Orchestration)
- [ ] Issue references all 5 plan doc filenames
- [ ] Plan Review issues (I-1 through I-10) and applied simplifications (S-3 through S-11) are noted
- [ ] Issue is committed to the `dynamic-workflow-project-setup` branch (any metadata files)

---

### Assignment 3: `create-project-structure`

**Responsible Agent:** `backend-developer`
**Estimated Duration:** ~30 minutes
**Dependencies:** Assignment 2 (uses application plan issue for context)
**Status:** 🔵 Not Started

#### Description

Create the full project scaffolding for a Python application with FastAPI. This is the largest assignment — it produces the directory structure, configuration files, Docker setup, CI workflows, and test infrastructure.

#### Target Project Structure

```
workflow-orchestration-queue/
├── pyproject.toml                    # uv-managed dependencies and metadata
├── uv.lock                           # Deterministic lockfile (generated by uv)
├── src/
│   ├── __init__.py
│   ├── notifier_service.py           # FastAPI webhook receiver (Phase 2)
│   ├── orchestrator_sentinel.py      # Background polling orchestrator (Phase 1)
│   ├── models/
│   │   ├── __init__.py
│   │   ├── work_item.py              # Unified WorkItem, TaskType, WorkItemStatus, scrub_secrets()
│   │   └── github_events.py          # Schemas for GitHub webhook payloads
│   └── queue/
│       ├── __init__.py
│       └── github_queue.py           # ITaskQueue ABC + GitHubQueue implementation
├── tests/
│   ├── __init__.py
│   ├── conftest.py                   # Shared fixtures (mock httpx, mock subprocess)
│   ├── test_work_item.py             # Unit tests for WorkItem model and scrub_secrets()
│   ├── test_github_queue.py          # Unit tests for GitHubQueue (claim, fetch, update)
│   ├── test_sentinel.py              # Unit tests for Sentinel orchestration logic
│   └── test_notifier.py             # Unit tests for FastAPI webhook handler
├── scripts/
│   ├── devcontainer-opencode.sh      # Shell bridge (already exists in template)
│   ├── gh-auth.ps1                   # Already exists
│   └── update-remote-indices.ps1     # Already exists
├── local_ai_instruction_modules/     # Already exists — keep as-is
│   ├── create-app-plan.md
│   ├── perform-task.md
│   └── analyze-bug.md
├── docs/                             # Project documentation
│   └── architecture.md               # Derived from Architecture Guide v3.2
├── Dockerfile                        # Production container for Sentinel + Notifier
├── docker-compose.yml                # Sentinel + Notifier service orchestration
├── .env.example                      # Template with all required env vars (no secrets)
└── README.md                         # Project overview, setup instructions
```

#### Tasks

1. **Create `pyproject.toml`** with:
   - Project metadata: `workflow-orchestration-queue`
   - Python requirement: `>=3.12`
   - Dependencies: `fastapi`, `uvicorn`, `pydantic>=2.0`, `httpx`, `hmac` (stdlib — no install needed)
   - Dev dependencies: `pytest`, `pytest-asyncio`, `pytest-httpx`, `respx`
   - uv-managed configuration

2. **Create `src/` application code** using reference implementations from `plan_docs/`:
   - `src/models/work_item.py` — Copy from `plan_docs/src/models/work_item.py` (already unified per R-3)
   - `src/models/github_events.py` — New Pydantic schemas for GitHub webhook payloads
   - `src/queue/github_queue.py` — Copy from `plan_docs/src/queue/github_queue.py` (already consolidated per S-6)
   - `src/orchestrator_sentinel.py` — Adapt from `plan_docs/orchestrator_sentinel.py` (already implements R-1 through R-8)
   - `src/notifier_service.py` — Adapt from `plan_docs/notifier_service.py` (already validates env vars per R-6)
   - Ensure all imports use `src.` prefix consistently

3. **Create `tests/` structure** with:
   - `conftest.py` with shared fixtures (mock httpx clients, mock WorkItems)
   - `test_work_item.py` — Test WorkItem creation, TaskType enum, scrub_secrets() patterns
   - `test_github_queue.py` — Test fetch_queued_tasks, claim_task (assign-then-verify), update_status
   - `test_sentinel.py` — Test polling loop, heartbeat coroutine, signal handling, subprocess timeout
   - `test_notifier.py` — Test HMAC signature validation, webhook routing, env var validation

4. **Create `Dockerfile`** for production:
   - Base: `python:3.12-slim`
   - Install `uv` for dependency management
   - Copy `pyproject.toml` + `uv.lock`, run `uv sync`
   - Copy `src/` code
   - Health check endpoint
   - Non-root user

5. **Create `docker-compose.yml`**:
   - `sentinel` service (runs `orchestrator_sentinel.py`)
   - `notifier` service (runs `notifier_service.py` via uvicorn)
   - Shared environment from `.env`
   - Network isolation between services
   - Resource constraints (2 CPUs, 4GB RAM per container)

6. **Create `.env.example`** with required variables:
   ```
   # Required
   GITHUB_TOKEN=
   GITHUB_ORG=intel-agency
   GITHUB_REPO=workflow-orchestration-queue
   SENTINEL_BOT_LOGIN=
   WEBHOOK_SECRET=
   ```

7. **Create CI workflow** `.github/workflows/ci.yml`:
   - Trigger on push to `main` and `develop`, PRs to `main`
   - Jobs: lint (ruff), test (pytest), security scan
   - **Pin all actions to commit SHAs** (e.g., `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`)
   - Use `uv` for dependency installation

8. **Create `docs/architecture.md`** — Condensed architecture guide

9. **Create `README.md`** — Project overview with setup instructions

#### Key Implementation Notes (from Plan Review)

These issues from the Plan Review must be addressed in the scaffolded code:

| Issue | Description | Status in Reference Code |
|-------|-------------|-------------------------|
| I-1 / R-3 | Unified WorkItem model in shared `src/models/work_item.py` | ✅ Already implemented |
| I-2 / R-2 | Assign-then-verify distributed locking in `claim_task()` | ✅ Already implemented |
| I-3 | Jittered exponential backoff on 403/429 | ✅ Already implemented |
| I-4 / R-5 | Single `httpx.AsyncClient` with connection pooling | ✅ Already implemented |
| I-5 / R-6 | Environment variable validation at startup | ✅ Already implemented |
| I-6 / R-1 | Heartbeat coroutine for long-running tasks | ✅ Already implemented |
| R-4 | Graceful shutdown via SIGTERM/SIGINT signal handlers | ✅ Already implemented |
| R-7 | Credential scrubber (`scrub_secrets()`) | ✅ Already implemented |
| R-8 | Subprocess timeout safety net (`asyncio.wait_for`) | ✅ Already implemented |
| S-3 | Only 3 required env vars (others hardcoded) | ✅ Already implemented |
| S-4 | Hardcoded "stop" environment reset mode | ✅ Already implemented |
| S-6 | Consolidated queue in `src/queue/github_queue.py` | ✅ Already implemented |
| S-7 | Removed IPv4 scrubbing pattern | ✅ Already implemented |
| S-10 | stdout-only logging (no FileHandler) | ✅ Already implemented |
| S-11 | Removed `raw_payload` field from WorkItem | ✅ Already implemented |

#### Acceptance Criteria

- [ ] `pyproject.toml` exists with correct dependencies and Python >=3.12
- [ ] `src/` directory contains all 5 source files (sentinel, notifier, work_item, github_events, github_queue)
- [ ] `tests/` directory contains all 5 test files with meaningful test stubs
- [ ] `Dockerfile` builds successfully (`docker build .`)
- [ ] `docker-compose.yml` defines both `sentinel` and `notifier` services
- [ ] `.env.example` lists all 5 required env vars with no hardcoded secrets
- [ ] `.github/workflows/ci.yml` exists with all actions pinned to commit SHAs
- [ ] `docs/architecture.md` exists
- [ ] `README.md` exists with setup instructions
- [ ] All imports use `src.` prefix consistently
- [ ] No placeholder secrets in any file (use `os.environ` or `os.getenv` only)
- [ ] All files committed to `dynamic-workflow-project-setup` branch

---

### Assignment 4: `create-agents-md-file`

**Responsible Agent:** `documentation-expert`
**Estimated Duration:** ~15 minutes
**Dependencies:** Assignment 1 (requires feature branch), Assignment 3 (benefits from knowing project structure)
**Status:** 🔵 Not Started

#### Description

Create/update `AGENTS.md` to reflect the project-specific context for `workflow-orchestration-queue`. This file is the primary instruction source for AI agents operating on this repository. It must replace the template-generic content with project-specific guidance.

#### Tasks

1. Read the existing `AGENTS.md` (template version)
2. Update the following sections to reflect the new project:
   - **Purpose:** Headless agentic orchestration platform (not a generic template)
   - **Tech Stack:** Python 3.12+, FastAPI, httpx, Pydantic, uv, Docker/DevContainers
   - **Repository Map:** Add `src/`, `tests/`, `docs/`, `Dockerfile`, `docker-compose.yml`
   - **Available Tools:** Replace .NET SDK references with Python tooling (uv, pytest, ruff)
   - **Coding Conventions:** Add Python-specific rules (type hints, docstrings, async patterns)
   - **Testing Commands:** Replace shell tests with `uv run pytest`, `uv run ruff check`
   - **Environment Setup:** Update secrets list (WEBHOOK_SECRET, SENTINEL_BOT_LOGIN)
   - **Verification Commands:** Map to Python/uv equivalents
3. Ensure all template placeholder references are updated:
   - `workflow-orchestration-queue-india38-b` → `workflow-orchestration-queue`
   - Template-specific instructions → project-specific instructions
4. Keep the dynamic workflow and instruction module references intact

#### Acceptance Criteria

- [ ] `AGENTS.md` reflects `workflow-orchestration-queue` (not the template repo)
- [ ] Tech stack section lists: Python 3.12+, FastAPI, httpx, Pydantic, uv, Docker
- [ ] Repository map includes `src/`, `tests/`, `docs/`, `Dockerfile`, `docker-compose.yml`
- [ ] Testing section references `uv run pytest` and `uv run ruff check`
- [ ] No references to .NET SDK, Avalonia, or Bun remain (these are template artifacts)
- [ ] Environment setup lists: `GITHUB_TOKEN`, `WEBHOOK_SECRET`, `SENTINEL_BOT_LOGIN`
- [ ] Verification commands use Python tooling
- [ ] Template placeholder strings (`workflow-orchestration-queue-india38-b`) are replaced

---

### Assignment 5: `debrief-and-document`

**Responsible Agent:** `documentation-expert`
**Estimated Duration:** ~15 minutes
**Dependencies:** Assignments 1–4 (summarizes all prior work)
**Status:** 🔵 Not Started

#### Description

Create a comprehensive debriefing report documenting the project setup process, lessons learned, and recommendations for future workflows.

#### Tasks

1. Create `plan_docs/debrief-report.md` containing:
   - **Setup Summary:** What was accomplished across all 5 prior assignments
   - **Repository State:** Final state of the repository after scaffolding
   - **Plan Document Assessment:**
     - Quality of reference implementations (most Plan Review issues already resolved)
     - Effectiveness of the unified data model (I-1/R-3)
     - Completeness of applied simplifications (S-3 through S-11)
   - **Lessons Learned:**
     - What worked well in the plan docs (clear ADRs, phased rollout)
     - What could be improved (some doc duplication per S-2 — kept intentionally)
     - Template → project transition friction points
   - **Risk Register:**
     - Carried-forward risks from Development Plan §7 (API rate limits, LLM looping, concurrency)
     - New risks identified during setup (if any)
   - **Recommendations:**
     - For Phase 1 implementation priority
     - For CI/CD pipeline refinement
     - For monitoring and observability
   - **Appendix:** Link to the setup PR, list of files created/modified

#### Acceptance Criteria

- [ ] `plan_docs/debrief-report.md` exists
- [ ] Report covers all 5 assignments with completion status
- [ ] Lessons learned section has at least 3 actionable items
- [ ] Risk register includes the 5 risks from Development Plan §7
- [ ] Recommendations section prioritizes Phase 1 (Sentinel MVP) work
- [ ] Report is committed to `dynamic-workflow-project-setup` branch

---

### Assignment 6: `pr-approval-and-merge`

**Responsible Agent:** `github-expert`
**Estimated Duration:** ~10 minutes
**Dependencies:** Assignments 1–5 (all work must be complete)
**Status:** 🔵 Not Started

#### Description

Finalize the setup PR: mark as ready for review, merge into `main`, and perform cleanup of temporary branches and issues.

#### Tasks

1. Convert the draft PR to "Ready for Review"
2. Verify all acceptance criteria from Assignments 1–5 are met:
   - Branch `dynamic-workflow-project-setup` has all scaffolded files
   - No lint errors (`uv run ruff check src/`)
   - No placeholder secrets in any committed file
   - CI workflow is valid YAML with SHA-pinned actions
3. Merge the PR into `main` using squash merge
4. Delete the `dynamic-workflow-project-setup` branch
5. Close any temporary issues created during setup (if applicable)
6. Verify `main` branch has the final project structure
7. Verify any CI workflows triggered by the merge pass

#### Acceptance Criteria

- [ ] PR is merged into `main` (squash merge)
- [ ] Feature branch `dynamic-workflow-project-setup` is deleted
- [ ] `main` branch contains all scaffolded files from Assignments 1–5
- [ ] CI workflow (`.github/workflows/ci.yml`) runs on `main` and completes
- [ ] No orphaned branches or stale setup issues remain
- [ ] `AGENTS.md` on `main` reflects the final project state

---

## 5. Dependency Graph

```
Assignment 1: init-existing-repository
    │
    ├──► Assignment 2: create-app-plan
    │       │
    │       └──► Assignment 3: create-project-structure
    │               │
    │               └──► Assignment 4: create-agents-md-file
    │                       │
    │                       └──► Assignment 5: debrief-and-document
    │                               │
    │                               └──► Assignment 6: pr-approval-and-merge
    │
    └── (Assignment 4 also depends on Assignment 3 for project structure context)
```

**Critical Path:** 1 → 2 → 3 → 4 → 5 → 6 (all sequential, no parallelization)

**Rationale for Sequential Execution:**
- Assignment 1 creates the branch that all subsequent work targets
- Assignment 2 produces the master plan issue that guides Assignment 3's scaffolding decisions
- Assignment 3 creates the project structure that Assignment 4 must document in `AGENTS.md`
- Assignment 5 summarizes all prior work and cannot precede it
- Assignment 6 is the final gate — all work must be complete before merge

---

## 6. Risk / Assumption Register

| # | Risk | Impact | Likelihood | Mitigation |
|---|------|--------|------------|------------|
| R-1 | GitHub API rate limits during label/milestone creation | Medium | Low | Use `GITHUB_TOKEN` (5,000 req/hr); batch operations |
| R-2 | Reference implementations have diverged from plan docs | High | Low | Both already implement Plan Review recommendations (R-1 through R-9) |
| R-3 | Docker build fails due to missing base image | Medium | Medium | `Dockerfile` uses `python:3.12-slim` (public Docker Hub); no GHCR dependency |
| R-4 | CI workflow SHA-pinning uses outdated action versions | Low | Medium | Pin to latest stable commit SHAs; document the pinned versions |
| R-5 | `uv` not available in CI runner | Low | Low | CI workflow installs `uv` via official action |
| R-6 | Merge conflicts if `main` receives other commits during setup | High | Low | Feature branch isolates all work; rebase before merge |
| R-7 | Test scaffolding has import path issues | Medium | Medium | Use `src.` prefix consistently; verify with `uv run pytest` before commit |

**Assumptions:**

1. The `prebuild-devcontainer` workflow completed successfully (confirmed trigger condition)
2. `GITHUB_TOKEN` has sufficient permissions (issues, labels, PRs, contents)
3. No other workflow is concurrently modifying the repository
4. Plan docs in `plan_docs/` are the authoritative source and have not been modified since seeding
5. The Orchestrator agent has access to all specialist sub-agent types listed above

---

## 7. Verification Checklist (Post-Merge)

After Assignment 6 completes, verify the following on `main`:

```bash
# 1. Project structure exists
ls src/orchestrator_sentinel.py src/notifier_service.py src/models/work_item.py src/queue/github_queue.py

# 2. Dependencies resolve
uv sync
uv run python -c "from src.models.work_item import WorkItem, scrub_secrets; print('OK')"

# 3. Tests can be discovered
uv run pytest --collect-only

# 4. Lint passes
uv run ruff check src/ tests/

# 5. CI workflow is valid
grep -c "uses:.*@" .github/workflows/ci.yml  # Should show SHA-pinned actions

# 6. No placeholder secrets
grep -r "YOUR_GITHUB_TOKEN\|your_webhook_secret" src/ tests/  # Should return nothing

# 7. Labels exist
gh label list | grep "agent:"
```

---

## 8. Capacity & Timeline Summary

| Assignment | Agent | Duration | Cumulative |
|------------|-------|----------|------------|
| 1. init-existing-repository | devops-engineer | ~15 min | ~15 min |
| 2. create-app-plan | planner | ~20 min | ~35 min |
| 3. create-project-structure | backend-developer | ~30 min | ~65 min |
| 4. create-agents-md-file | documentation-expert | ~15 min | ~80 min |
| 5. debrief-and-document | documentation-expert | ~15 min | ~95 min |
| 6. pr-approval-and-merge | github-expert | ~10 min | ~105 min |

**Critical Path Duration:** ~105 minutes (all sequential)

**Replanning Checkpoint:** After Assignment 3 (the largest task). If scaffolding takes significantly longer than estimated, re-evaluate durations for Assignments 4–6.

---

*Plan created: 2026-04-20 | Workflow: project-setup | Branch: dynamic-workflow-project-setup*
