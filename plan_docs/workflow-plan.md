# Workflow Execution Plan: project-setup

**Generated:** 2026-04-13  
**Workflow:** project-setup (dynamic workflow)  
**Dynamic Workflow File:** `ai-workflow-assignments/dynamic-workflows/project-setup.md`  
**Repository:** `intel-agency/workflow-orchestration-queue-india38-b`  
**Branch:** `dynamic-workflow-project-setup`  

---

## 1. Overview

| Field | Value |
|---|---|
| **Workflow Name** | `project-setup` |
| **Project Name** | workflow-orchestration-queue (OS-APOW) |
| **Project Description** | Headless agentic orchestration platform that transforms GitHub Issues into Execution Orders, dispatching specialized AI agents (Opencode workers) to autonomously fulfill tasks — from application planning to verified Pull Requests — without human intervention. |
| **Total Main Assignments** | 6 |
| **Total Event Assignments** | 3 (create-workflow-plan, validate-assignment-completion, report-progress) |
| **Triggering Event** | Successful completion of `prebuild-devcontainer` workflow_run on `main` |

### High-Level Summary

This workflow initializes the `workflow-orchestration-queue` repository from its template state into a fully configured project with application plan, project scaffolding, and agent-focused documentation. It is a **planning-only workflow** — no application code is written. The workflow:

1. Creates a setup branch and initializes the repository (branch protection, labels, GitHub Project, workspace rename, PR).
2. Creates a comprehensive application plan as a GitHub issue with milestones.
3. Scaffolds the project structure (Python/uv project, Dockerfile, CI/CD, docs).
4. Creates a project-specific `AGENTS.md` for AI coding agents.
5. Produces a debriefing report with learnings and execution trace.
6. Merges the setup PR and cleans up.

### Assignment Execution Order

```
[pre-script-begin] → create-workflow-plan ← YOU ARE HERE
[main-1]           → init-existing-repository
[post-1]           → validate-assignment-completion + report-progress
[main-2]           → create-app-plan
[post-2]           → validate-assignment-completion + report-progress
[main-3]           → create-project-structure
[post-3]           → validate-assignment-completion + report-progress
[main-4]           → create-agents-md-file
[post-4]           → validate-assignment-completion + report-progress
[main-5]           → debrief-and-document
[post-5]           → validate-assignment-completion + report-progress
[main-6]           → pr-approval-and-merge
[post-6]           → validate-assignment-completion + report-progress
[post-script]      → Apply orchestration:plan-approved label to app plan issue
```

---

## 2. Project Context Summary

### Key Facts from `plan_docs/`

| Category | Details |
|---|---|
| **Application Name** | workflow-orchestration-queue (OS-APOW) |
| **Tagline** | Headless Agentic Orchestration — from Interactive AI Coding to Autonomous Background Service |
| **Primary Language** | Python 3.12+ |
| **Package Manager** | uv (Rust-based) |
| **Web Framework** | FastAPI + Uvicorn (for webhook notifier) |
| **Data Validation** | Pydantic |
| **HTTP Client** | httpx (async) |
| **Containerization** | Docker + Docker Compose |
| **Shell Bridge** | `./scripts/devcontainer-opencode.sh` — sole interface between Python orchestrator and worker containers |
| **LLM Agent Runtime** | opencode CLI (GLM-5 model via ZhipuAI) |
| **Primary State Store** | GitHub Issues (labels as state machine: `agent:queued` → `agent:in-progress` → `agent:success`/`agent:error`) |
| **Branch Strategy** | `main` (stable), `develop` (integration) |
| **No global.json** | Python ecosystem — uses `pyproject.toml` + `uv.lock` |

### Architecture (4 Pillars)

1. **The Ear** (Work Event Notifier) — FastAPI webhook receiver for GitHub events
2. **The State** (Work Queue) — GitHub Issues as distributed state (Markdown-as-a-Database)
3. **The Brain** (Sentinel Orchestrator) — Async polling service that claims tasks, manages worker lifecycle
4. **The Hands** (Opencode Worker) — Isolated DevContainer where LLM agent executes tasks

### Development Phases

| Phase | Name | Scope |
|---|---|---|
| **Phase 0** | Seeding & Bootstrapping | Manual clone, seed plan docs, project setup workflow (this workflow) |
| **Phase 1** | The Sentinel (MVP) | Persistent polling engine, shell-bridge dispatch, task claiming with distributed locking, heartbeat comments |
| **Phase 2** | The Ear (Webhook Automation) | FastAPI webhook receiver, HMAC verification, intelligent triaging |
| **Phase 3** | Deep Orchestration | Architect sub-agent, hierarchical decomposition, PR feedback loop, self-healing |

### Key Architecture Decisions

- **ADR 07: Shell-Bridge Execution** — Orchestrator interacts with workers exclusively via `devcontainer-opencode.sh` (no Docker SDK)
- **ADR 08: Polling-First Resiliency** — Polling is primary discovery; webhooks are optimization
- **ADR 09: Provider-Agnostic Interface** — `ITaskQueue` ABC enables future provider swapping (Linear, Jira)

### Plan Review Findings (Already Addressed)

The Plan Review (`OS-APOW Plan Review.md`) identified 10 issues (I-1 through I-10) and 9 recommendations (R-1 through R-9). The Simplification Report resolved 8 of 11 items (S-3 through S-11), keeping S-1 (ITaskQueue ABC retained) and S-2 (doc duplication retained for agent reinforcement). Key resolved items:

- Unified data model in `src/models/work_item.py` (I-1/R-3)
- Assign-then-verify locking implemented in `src/queue/github_queue.py` (I-2/R-2)
- Jittered exponential backoff in sentinel (I-3)
- Connection pooling via shared `httpx.AsyncClient` (I-4/R-5)
- Environment variable validation at startup (I-5/R-6)
- Heartbeat coroutine implemented (I-6/R-1)
- Credential scrubbing via `scrub_secrets()` (R-7)
- Subprocess timeout safety net (R-8)
- Graceful shutdown via SIGTERM/SIGINT handling (R-4)
- Reduced to 3 required env vars: `GITHUB_TOKEN`, `GITHUB_ORG`, `GITHUB_REPO` (S-3)
- Hardcoded env reset to "stop" (S-4)
- Single-repo polling only (S-5)
- Consolidated queue in `src/queue/github_queue.py` (S-6)
- Removed IPv4 scrubbing pattern (S-7)
- Removed "encrypted" log verbiage (S-8)
- Moved Phase 3 details to appendix (S-9)
- Stdout-only logging (S-10)
- Removed `raw_payload` field (S-11)

### Reference Implementation Files in `plan_docs/`

| File | Purpose |
|---|---|
| `orchestrator_sentinel.py` | Reference Sentinel implementation (Phase 1 core) |
| `notifier_service.py` | Reference Notifier implementation (Phase 2 webhook receiver) |
| `src/models/work_item.py` | Unified data model (`WorkItem`, `TaskType`, `WorkItemStatus`, `scrub_secrets()`) |
| `src/queue/github_queue.py` | Consolidated GitHub queue (`ITaskQueue` ABC + `GitHubQueue` concrete) |
| `interactive-report.html` | Static presentation dashboard |

### Existing Repository State

The repository is a fresh template clone with:

- `AGENTS.md` — Template version (will be replaced by `create-agents-md-file`)
- `.devcontainer/` — Consumer devcontainer referencing prebuilt GHCR image
- `.github/` — Workflows (orchestrator-agent, publish-docker, prebuild-devcontainer, validate), labels, issue templates
- `.opencode/` — Agent definitions and commands
- `scripts/` — PowerShell and shell scripts (auth, labels, validation, opencode server)
- `local_ai_instruction_modules/` — Development rules, workflows, delegation instructions
- `plan_docs/` — All planning documents (this file included)
- `test/` — Shell-based tests
- `global.json` — .NET SDK versioning (not needed for Python project — to be removed or noted)

---

## 3. Assignment Execution Plan

### Assignment 1: `create-workflow-plan` (pre-script-begin event)

| Field | Content |
|---|---|
| **Assignment** | `create-workflow-plan`: Create Workflow Execution Plan |
| **Goal** | Produce a comprehensive, project-specific execution plan for the project-setup dynamic workflow before any other assignment begins. |
| **Key Acceptance Criteria** | 1. Dynamic workflow file read and fully understood. 2. Every referenced assignment traced and read. 3. All `plan_docs/` documents read and summarized. 4. Plan lists each assignment in order with goal, criteria, dependencies, and risks. 5. Plan committed as `plan_docs/workflow-plan.md`. |
| **Project-Specific Notes** | This is the current assignment. The primary app spec is `OS-APOW Implementation Specification v1.2.md` (no `ai-new-app-template.md` exists). Rich supporting docs provide extensive context. |
| **Prerequisites** | Repository exists, `plan_docs/` directory populated |
| **Dependencies** | None (first event) |
| **Risks / Challenges** | No formal app template file — the Implementation Specification serves as the equivalent. Agent must treat it as the canonical app spec. |
| **Events** | None |

---

### Assignment 2: `init-existing-repository`

| Field | Content |
|---|---|
| **Assignment** | `init-existing-repository`: Initiate Existing Repository |
| **Goal** | Create the setup branch, import branch protection, create GitHub Project, import labels, rename workspace files, and open the setup PR. |
| **Key Acceptance Criteria** | 1. Branch `dynamic-workflow-project-setup` created (first step). 2. Branch protection ruleset imported from `.github/protected-branches_ruleset.json`. 3. GitHub Project created with Board columns (Not Started, In Progress, In Review, Done). 4. Labels imported from `.github/.labels.json`. 5. Workspace file renamed: `workflow-orchestration-queue-india38-b.code-workspace` → `workflow-orchestration-queue.code-workspace`. 6. Devcontainer name updated to `workflow-orchestration-queue-devcontainer`. 7. PR created from branch to `main`. |
| **Project-Specific Notes** | - **No `protected-branches_ruleset.json`** exists at `.github/protected-branches_ruleset.json` — the branch protection step may need to be skipped or a ruleset created from scratch. Verify existence before attempting import. - **Labels file** at `.github/.labels.json` contains 146 lines of labels including `agent:*`, `orchestration:*`, `priority:*`, etc. Use `scripts/import-labels.ps1`. - The GitHub Project should be named `workflow-orchestration-queue`. - Requires `GH_ORCHESTRATION_AGENT_TOKEN` with `administration:write` scope for branch protection (if ruleset exists). |
| **Prerequisites** | GitHub authentication with `repo`, `project`, `read:project`, `read:user`, `user:email` scopes |
| **Dependencies** | `create-workflow-plan` completed |
| **Outputs** | `$pr_num` — the setup PR number (critical input for `pr-approval-and-merge`) |
| **Risks / Challenges** | **HIGH:** Branch protection import requires PAT with `administration:write` scope. If `GH_ORCHESTRATION_AGENT_TOKEN` is unavailable or lacks scope, this step fails. **MEDIUM:** `.github/protected-branches_ruleset.json` does not exist — step may need to be skipped with documentation. **LOW:** GitHub Project creation may fail if org-level permissions aren't configured. |
| **Events** | `validate-assignment-completion` + `report-progress` (post-assignment-complete) |

---

### Assignment 3: `create-app-plan`

| Field | Content |
|---|---|
| **Assignment** | `create-app-plan`: Create Application Plan |
| **Goal** | Analyze plan_docs and create a comprehensive application plan documented as a GitHub issue, with milestones for each phase. |
| **Key Acceptance Criteria** | 1. Application template analyzed (Implementation Spec v1.2 is the primary spec). 2. `plan_docs/tech-stack.md` created. 3. `plan_docs/architecture.md` created. 4. GitHub Issue created using the application-plan issue template (`.github/ISSUE_TEMPLATE/application-plan.md`). 5. Milestones created for each phase (Phase 0–3). 6. Issue linked to GitHub Project and assigned to "Phase 1: Foundation" milestone. 7. Labels applied: `planning`, `documentation`. 8. No code written — planning only. |
| **Project-Specific Notes** | - **No `ai-new-app-template.md`** — treat `OS-APOW Implementation Specification v1.2.md` as the primary application spec. - Rich supporting docs provide comprehensive context: Architecture Guide (system diagrams, ADRs), Development Plan (phases, stories, acceptance criteria, risk assessment), Plan Review (10 issues, 9 recommendations), Simplification Report (11 items, 8 resolved). - Reference code in `plan_docs/` (notifier_service.py, orchestrator_sentinel.py, src/) provides implementation guidance — these should be referenced but not treated as the final code. - Tech stack: Python 3.12+, FastAPI, Pydantic, httpx, uv, Docker. No .NET. - `global.json` exists in repo root but is irrelevant to this Python project — note this in the plan. - Must follow the application-plan issue template at `.github/ISSUE_TEMPLATE/application-plan.md`. |
| **Prerequisites** | `init-existing-repository` completed (GitHub Project exists, labels imported, branch ready) |
| **Dependencies** | GitHub Project created, labels imported |
| **Outputs** | Application plan issue number (needed for `post-script-complete` event) |
| **Risks / Challenges** | **MEDIUM:** The Implementation Specification is very detailed — the agent must distill it into a structured plan without losing critical requirements. **LOW:** Phase 1 (Sentinel MVP) has the most detail; Phases 2-3 are more speculative — milestones should reflect this granularity difference. |
| **Events** | `pre-assignment-begin`: `gather-context` (if triggered). `on-assignment-failure`: `recover-from-error`. `post-assignment-complete`: `validate-assignment-completion` + `report-progress`. |

---

### Assignment 4: `create-project-structure`

| Field | Content |
|---|---|
| **Assignment** | `create-project-structure`: Create Project Structure |
| **Goal** | Create the actual project scaffolding (directories, project files, Dockerfile, CI/CD, docs) based on the application plan. |
| **Key Acceptance Criteria** | 1. Python/uv project structure created (`pyproject.toml`, `uv.lock`, `src/` layout). 2. Dockerfile for each service created. 3. `docker-compose.yml` for local development. 4. Basic CI/CD pipeline established. 5. Documentation structure (README.md, docs/). 6. Development environment validated. 7. `.ai-repository-summary.md` created at repo root. 8. All GitHub Actions pinned to commit SHA. 9. Repository summary linked from README.md. 10. Initial commit on setup branch. |
| **Project-Specific Notes** | - Target structure per Implementation Spec:  
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
  │   ├── create-app-plan.md
  │   ├── perform-task.md
  │   └── analyze-bug.md
  └── docs/
  ```
  - Reference implementation code in `plan_docs/src/` should guide the actual `src/` creation. The `plan_docs/` versions are validated against Plan Review recommendations.  
  - **Healthcheck:** Must use Python stdlib (not curl) for Docker healthcheck commands.  
  - **Dockerfile:** When using `uv pip install -e .`, ensure `COPY src/ ./src/` appears before the install command.  
  - **CI/CD:** Must pin all GitHub Actions to SHA (directive from dynamic workflow).  
  - **Existing scripts:** The repo already has `scripts/` — coordinate with existing scripts, don't overwrite.  
  - **Existing docs:** The repo has `docs/` directory — extend, don't replace.  
  - `global.json` (.NET SDK versioning) should be removed or explicitly noted as irrelevant. |
| **Prerequisites** | Application plan exists as GitHub issue, `create-app-plan` completed |
| **Dependencies** | Application plan (for tech stack, structure decisions), `init-existing-repository` (branch, PR) |
| **Outputs** | Complete project scaffolding on setup branch |
| **Risks / Challenges** | **MEDIUM:** Existing `scripts/` and `docs/` directories in template may conflict with new project structure — must coordinate carefully. **MEDIUM:** `global.json` (.NET) exists but project is Python-only — decision needed on removal. **LOW:** CI/CD SHA pinning requires looking up current SHAs for all actions used. |
| **Events** | `post-assignment-complete`: `validate-assignment-completion` + `report-progress`. |

---

### Assignment 5: `create-agents-md-file`

| Field | Content |
|---|---|
| **Assignment** | `create-agents-md-file`: Create AGENTS.md File |
| **Goal** | Create a comprehensive `AGENTS.md` at repo root providing AI coding agents with project-specific context, build commands, conventions, and testing instructions. |
| **Key Acceptance Criteria** | 1. `AGENTS.md` exists at repo root. 2. Contains project overview (purpose, tech stack). 3. Contains verified setup/build/test commands. 4. Contains code style and conventions. 5. Contains project structure / directory layout. 6. Contains testing instructions. 7. Contains PR/commit guidelines. 8. Commands validated by running them. 9. File committed to setup branch. |
| **Project-Specific Notes** | - An `AGENTS.md` already exists (template version) — this will be **replaced** with project-specific content.  
  - Tech stack to document: Python 3.12+, uv, FastAPI, Pydantic, httpx, Docker, opencode CLI.  
  - Build commands: `uv sync`, `uv run python -m src.orchestrator_sentinel`, etc.  
  - Test commands: `pytest` (to be established).  
  - Lint: `ruff check`, `ruff format`.  
  - Key conventions from Implementation Spec: docstrings in Sphinx/Google format, structured logging, env var validation at startup.  
  - Must cross-reference with `README.md`, `.ai-repository-summary.md`, and `plan_docs/`.  
  - The existing `AGENTS.md` contains template-specific instructions (opencode, devcontainer, GHCR) — preserve relevant portions, add project-specific sections. |
| **Prerequisites** | Project structure exists (`create-project-structure` completed), build/test tooling in place |
| **Dependencies** | `create-project-structure` (actual project files must exist for command validation) |
| **Outputs** | Updated `AGENTS.md` at repo root |
| **Risks / Challenges** | **LOW:** Build/test commands must be verified to work — if scaffolding has issues, this assignment will surface them. **LOW:** Balancing template-specific instructions (devcontainer, CI) with project-specific content (Python, FastAPI). |
| **Events** | `post-assignment-complete`: `validate-assignment-completion` + `report-progress`. |

---

### Assignment 6: `debrief-and-document`

| Field | Content |
|---|---|
| **Assignment** | `debrief-and-document`: Debrief and Document Learnings |
| **Goal** | Produce a comprehensive debriefing report capturing learnings, deviations, and improvement recommendations from the entire project-setup workflow execution. |
| **Key Acceptance Criteria** | 1. Report follows the 12-section structured template. 2. All deviations from assignments documented. 3. Execution trace saved as `debrief-and-document/trace.md`. 4. Report reviewed and approved. 5. Report committed and pushed to setup branch. 6. Plan-impacting findings flagged as ACTION ITEMS with issue filing recommendations. |
| **Project-Specific Notes** | - Must capture deviations specific to this project (e.g., missing `protected-branches_ruleset.json`, no `ai-new-app-template.md`).  
  - Must include execution trace with all commands run, files created/modified.  
  - Should recommend whether `global.json` should be removed.  
  - Should flag any missing features from plan_docs reference code vs. what was scaffolded. |
| **Prerequisites** | All main assignments completed |
| **Dependencies** | All prior main assignments completed |
| **Outputs** | `debrief-and-document/report.md`, `debrief-and-document/trace.md` |
| **Risks / Challenges** | **LOW:** Thoroughness of the trace depends on logging discipline throughout the workflow. |
| **Events** | `post-assignment-complete`: `validate-assignment-completion` + `report-progress`. |

---

### Assignment 7: `pr-approval-and-merge`

| Field | Content |
|---|---|
| **Assignment** | `pr-approval-and-merge`: Pull Request Approval and Merge |
| **Goal** | Complete the full PR approval and merge process: resolve PR comments, obtain approval, merge the setup PR, delete the setup branch, close setup issues. |
| **Key Acceptance Criteria** | 1. CI verification: All required status checks pass. 2. CI remediation loop executed (up to 3 attempts) if checks fail. 3. Code review delegated to `code-reviewer` subagent (not self-review). 4. Auto-reviewer comments waited for and addressed. 5. `ai-pr-comment-protocol.md` workflow executed. 6. All review threads resolved (GraphQL verification). 7. Stakeholder/Delegating Agent approval obtained. 8. PR merged successfully. 9. Source branch deleted. 10. Related issues closed. |
| **Project-Specific Notes** | - **Self-approval is acceptable** per the dynamic workflow spec: "This is an automated setup PR — self-approval by the orchestrator is acceptable."  
  - **$pr_num** comes from `init-existing-repository` output.  
  - CI remediation loop MUST still execute even with self-approval.  
  - The PR will contain all changes from all assignments (init, app plan, project structure, AGENTS.md, debrief).  
  - Must follow `ai-pr-comment-protocol.md` and `pr-review-comments` assignment.  
  - Post-merge: delete `dynamic-workflow-project-setup` branch, close any related setup issues. |
| **Prerequisites** | All main assignments completed, setup PR exists with all changes |
| **Dependencies** | `$pr_num` from `init-existing-repository` |
| **Outputs** | `result`: `"merged"` | `"pending"` | `"failed"` |
| **Risks / Challenges** | **MEDIUM:** CI checks may fail due to the template's existing CI validating against Python project files. The `validate` workflow runs lint/scan/test — may need updates to handle the new Python project structure. **LOW:** Merge conflicts unlikely since all work is on the setup branch. |
| **Events** | `post-assignment-complete`: `validate-assignment-completion` + `report-progress`. |

---

### Event Assignments (Executed Repeatedly)

#### `validate-assignment-completion`

| Field | Content |
|---|---|
| **Goal** | Validate that the just-completed assignment met all acceptance criteria. |
| **Execution** | After each main assignment (6 times total). Delegated to independent `qa-test-engineer` agent. |
| **Key Actions** | Check file outputs, run verification commands, create validation report at `docs/validation/`. |
| **Failure Handling** | Block progression, notify orchestrator, provide remediation steps. |

#### `report-progress`

| Field | Content |
|---|---|
| **Goal** | Generate structured progress report, capture outputs, checkpoint state. |
| **Execution** | After each main assignment (6 times total). |
| **Key Actions** | Progress report with step/duration/status, capture outputs to workflow context, validate acceptance criteria, checkpoint state, file GitHub issues for action items. |

---

### Post-Script-Complete Event

| Field | Content |
|---|---|
| **Action** | Apply `orchestration:plan-approved` label to the application plan issue |
| **Source** | Application plan issue created during `create-app-plan` |
| **Purpose** | Signal that the plan is ready for epic creation, triggering the next orchestration pipeline phase |

---

## 4. Sequencing Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    project-setup Dynamic Workflow                        │
│                                                                          │
│  ┌─────────────────────── pre-script-begin ──────────────────────┐      │
│  │  [1] create-workflow-plan  ──→  plan_docs/workflow-plan.md    │      │
│  └───────────────────────────────────────────────────────────────┘      │
│                          │                                               │
│                          ▼                                               │
│  ┌───────────── Main Script: initiate-new-repository ─────────────┐     │
│  │                                                                 │     │
│  │  [2] init-existing-repository                                   │     │
│  │      ├─ Create branch: dynamic-workflow-project-setup           │     │
│  │      ├─ Import branch protection (if ruleset exists)            │     │
│  │      ├─ Create GitHub Project: workflow-orchestration-queue     │     │
│  │      ├─ Import labels from .github/.labels.json                 │     │
│  │      ├─ Rename workspace file + update devcontainer name        │     │
│  │      └─ Create PR (output: $pr_num)                             │     │
│  │           │                                                     │     │
│  │           ├── validate-assignment-completion                    │     │
│  │           └── report-progress                                   │     │
│  │                                                                 │     │
│  │  [3] create-app-plan                                            │     │
│  │      ├─ Analyze OS-APOW Implementation Spec + supporting docs   │     │
│  │      ├─ Create plan_docs/tech-stack.md                          │     │
│  │      ├─ Create plan_docs/architecture.md                        │     │
│  │      ├─ Create GitHub Issue (application plan)                  │     │
│  │      ├─ Create milestones: Phase 0, 1, 2, 3                    │     │
│  │      └─ Link issue to Project + milestone                       │     │
│  │           │                                                     │     │
│  │           ├── validate-assignment-completion                    │     │
│  │           └── report-progress                                   │     │
│  │                                                                 │     │
│  │  [4] create-project-structure                                   │     │
│  │      ├─ Create pyproject.toml, src/ layout                      │     │
│  │      ├─ Create Dockerfile, docker-compose.yml                   │     │
│  │      ├─ Create CI/CD workflows (SHA-pinned)                     │     │
│  │      ├─ Create README.md, docs/ structure                       │     │
│  │      ├─ Create .ai-repository-summary.md                        │     │
│  │      └─ Validate build/test/lint                                │     │
│  │           │                                                     │     │
│  │           ├── validate-assignment-completion                    │     │
│  │           └── report-progress                                   │     │
│  │                                                                 │     │
│  │  [5] create-agents-md-file                                      │     │
│  │      ├─ Gather project context from existing docs               │     │
│  │      ├─ Validate build/test commands                            │     │
│  │      ├─ Draft AGENTS.md (replace template version)              │     │
│  │      └─ Commit to setup branch                                  │     │
│  │           │                                                     │     │
│  │           ├── validate-assignment-completion                    │     │
│  │           └── report-progress                                   │     │
│  │                                                                 │     │
│  │  [6] debrief-and-document                                       │     │
│  │      ├─ Create 12-section debrief report                        │     │
│  │      ├─ Save execution trace                                    │     │
│  │      └─ Commit to setup branch                                  │     │
│  │           │                                                     │     │
│  │           ├── validate-assignment-completion                    │     │
│  │           └── report-progress                                   │     │
│  │                                                                 │     │
│  │  [7] pr-approval-and-merge (input: $pr_num)                     │     │
│  │      ├─ CI verification & remediation loop (≤3 attempts)        │     │
│  │      ├─ Code review by code-reviewer subagent                   │     │
│  │      ├─ Resolve review comments                                 │     │
│  │      ├─ Obtain approval (self-approval OK)                      │     │
│  │      ├─ Merge PR                                                │     │
│  │      ├─ Delete setup branch                                     │     │
│  │      └─ Close setup issues                                      │     │
│  │           │                                                     │     │
│  │           ├── validate-assignment-completion                    │     │
│  │           └── report-progress                                   │     │
│  └─────────────────────────────────────────────────────────────────┘     │
│                          │                                               │
│                          ▼                                               │
│  ┌─────────────────────── post-script-complete ───────────────────┐      │
│  │  [8] Apply `orchestration:plan-approved` label to plan issue   │      │
│  └───────────────────────────────────────────────────────────────┘      │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘

Dependency Chain:
  create-workflow-plan
    └→ init-existing-repository (outputs: branch, $pr_num, project, labels)
       └→ create-app-plan (inputs: plan_docs/, project; outputs: plan issue, milestones)
          └→ create-project-structure (inputs: plan issue, branch; outputs: scaffolding)
             └→ create-agents-md-file (inputs: scaffolding; outputs: AGENTS.md)
                └→ debrief-and-document (inputs: all outputs; outputs: report, trace)
                   └→ pr-approval-and-merge (inputs: $pr_num, all changes; outputs: merged)
                      └→ Apply orchestration:plan-approved label
```

---

## 5. Open Questions

The following ambiguities require stakeholder/orchestrator input before or during execution:

| # | Question | Impact | Suggested Resolution |
|---|---|---|---|
| 1 | **No `ai-new-app-template.md` exists in `plan_docs/`.** The `create-app-plan` assignment expects this file. Should `OS-APOW Implementation Specification v1.2.md` be treated as the primary app spec? | Assignment 3 (create-app-plan) | Yes — the Implementation Specification is functionally equivalent and more comprehensive. Agent should treat it as the canonical app spec. |
| 2 | **No `.github/protected-branches_ruleset.json` exists.** The `init-existing-repository` assignment expects this file for branch protection import. | Assignment 2 (init-existing-repository) | Skip the branch protection import step if the file doesn't exist. Document the skip in the debrief. The ruleset can be created manually or in a follow-up issue. |
| 3 | **`global.json` (.NET SDK versioning) exists in repo root but the project is Python-only.** Should it be removed? | Assignment 4 (create-project-structure) | Remove it during project structure creation. It's a leftover from the .NET template and irrelevant to the Python stack. Document in debrief. |
| 4 | **Is `GH_ORCHESTRATION_AGENT_TOKEN` available with `administration:write` scope?** Required for branch protection ruleset import via GitHub API. | Assignment 2 (init-existing-repository) | If not available, the branch protection step is already moot (Q2 above). Verify at runtime and document outcome. |
| 5 | **Should the reference code in `plan_docs/src/` be copied directly into the project `src/` directory?** Or should it serve as guidance for fresh implementation? | Assignment 4 (create-project-structure) | The reference code has been validated against Plan Review recommendations and reflects the intended implementation. Copy it as the initial scaffolding and note it as reference code to be refined during Phase 1 implementation. |
| 6 | **How should existing template scripts in `scripts/` interact with new project scripts?** The template has PowerShell scripts (validate, auth, labels, etc.) and the project needs `devcontainer-opencode.sh`. | Assignment 4 (create-project-structure) | Keep existing template scripts. Add project-specific scripts alongside. The `devcontainer-opencode.sh` may already exist or need to be created. Coordinate, don't overwrite. |

---

## Approval

- [ ] Orchestrator/Stakeholder approval obtained
- [ ] Plan committed as `plan_docs/workflow-plan.md`
