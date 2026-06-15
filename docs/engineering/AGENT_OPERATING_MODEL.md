# Agent Operating Model — Planning Platform

**Document ID:** AGENTOPS-T01  
**Status:** Active  
**Applies to:** All AI-assisted implementation sprints for the Planning Platform  
**Related architecture:** D021 (Runtime Planning State Plane), D022 (Planning Knowledge Plane), D023 (Planning Control Plane), D024 (Planning Application Plane)

---

## 1. Purpose

This document defines how AI development agents operate during Planning Platform implementation sprints. It establishes a **deterministic, testable, trackable, and traceable** operating model so that:

- Sprint execution follows a repeatable sequence with explicit entry and exit criteria.
- Implementation is **task-scoped** — no agent works outside an assigned, approved task boundary.
- Agent responsibilities and permissions are unambiguous.
- Every change is backed by an **evidence bundle** and passes **quality gates** before merge.
- Requirements trace forward to code, tests, and pull requests, with **human approval** before merge.

This model applies from Epic 1 onward and evolves only through documented change to this file.

---

## 2. Operating Principles

| Principle | Definition |
|-----------|------------|
| **Deterministic execution** | Each sprint and task follows the workflows in Sections 6–7 without skipping steps. Gate failures block progression. |
| **Task-scoped implementation** | Agents implement only what is specified in an approved task record. Scope creep requires a new task and human approval. |
| **Separation of implement vs. review** | Reviewer agents are read-only by default. Code modification is restricted to implementation agents (Section 4). |
| **Evidence over assertion** | No task is complete without a linked evidence bundle (Section 9). Claims without artifacts are rejected at gates. |
| **Traceability by default** | Every merged change links requirement → task → branch → PR → tests → gate results (Section 11). |
| **Human-in-the-loop merge** | No agent merges to `main`. A human approves every PR after gates pass (Section 13). |
| **Plane-aware design** | Work respects the four architecture planes (D021–D024). Cross-plane changes require Architecture Guardian sign-off. |
| **Tenant-safe by construction** | Security and tenant isolation are verified on every sprint, not deferred. |

---

## 3. Agent Role Types

Agents fall into three role types. Each agent is assigned exactly one primary role type per task invocation.

### 3.1 Orchestration Agents

Coordinate sprint and task lifecycle, assign work, enforce gates, and aggregate evidence. They **do not** modify application source code.

**Examples:** Sprint Orchestrator Agent.

### 3.2 Reviewer Agents (Read-Only)

Analyze architecture, domain models, security, UX, and compliance. They produce findings, recommendations, and gate verdicts. They **must not** modify code, infrastructure-as-code, or tests unless explicitly escalated and reclassified (Section 4).

**Examples:** Architecture Guardian Agent, Domain Modeling Agent, Security & Tenant Isolation Agent, UX Review Agent.

### 3.3 Implementation Agents (Write-Capable)

Execute approved tasks by modifying code, tests, configuration, or infrastructure within task scope. They produce diffs, tests, and evidence artifacts.

**Examples:** Backend Implementation Agent, Frontend Workspace Agent, Metadata Runtime Agent (metadata/config only), Test & Quality Agent, DevOps & Cloud Agent.

> **Note:** Metadata Runtime Agent is write-capable for metadata and runtime configuration artifacts only, not general application logic unless the task explicitly assigns that scope.

---

## 4. Agent Permissions

### 4.1 Default Permission Matrix

| Capability | Orchestration | Reviewer (default) | Implementation |
|------------|:-------------:|:------------------:|:--------------:|
| Read repository | ✓ | ✓ | ✓ |
| Read docs / ADRs / tasks | ✓ | ✓ | ✓ |
| Write findings / gate reports | ✓ | ✓ | ✓ (task evidence only) |
| Create branches | ✓ | ✗ | ✓ (task branch only) |
| Modify application code | ✗ | ✗ | ✓ (authorized agents only) |
| Modify tests | ✗ | ✗ | ✓ (Test & Quality Agent; others per task) |
| Modify CI/CD / IaC | ✗ | ✗ | ✓ (DevOps & Cloud Agent) |
| Open / update PRs | ✓ | ✗ | ✓ |
| Merge to `main` | ✗ | ✗ | ✗ |
| Approve PRs | ✗ | ✗ | ✗ |

### 4.2 Authorized Code-Writing Agents

**Only** these agents may modify code, tests, or deployable configuration:

1. **Backend Implementation Agent**
2. **Frontend Workspace Agent**
3. **Test & Quality Agent**
4. **DevOps & Cloud Agent**

All other agents operate read-only unless a human explicitly reclassifies a one-off task (documented in the task record with justification).

### 4.3 Branch and Repository Rules

- **No agent may commit directly to `main`.**
- All implementation occurs on task-scoped branches (Section 12).
- Agents must not push secrets, credentials, or environment-specific values into the repository.

### 4.4 Tool Usage Constraints

- Reviewer agents invoke read, search, lint, and static-analysis tools only.
- Implementation agents may invoke build, test, format, and deploy tools as defined in the task and Section 14.
- Orchestration agents may invoke project-management and CI status tools; they do not run mutating build steps on behalf of implementers except to verify gate status.

---

## 5. Standard Agent Roles

Each role has a defined mandate, inputs, outputs, and gate participation.

### 5.1 Sprint Orchestrator Agent

| Attribute | Detail |
|-----------|--------|
| **Type** | Orchestration |
| **Mandate** | Plan sprint execution, decompose epics into tasks, assign agents, track gate status, block incomplete work. |
| **Inputs** | Sprint goal, epic backlog, architecture constraints (D021–D024), prior sprint evidence. |
| **Outputs** | Sprint plan, task assignments, gate checklist, sprint closure report. |
| **Writes code** | No |

### 5.2 Architecture Guardian Agent

| Attribute | Detail |
|-----------|--------|
| **Type** | Reviewer (read-only) |
| **Mandate** | Verify changes align with D021–D024 plane boundaries, ADRs, and non-functional requirements. |
| **Inputs** | Task specs, diffs, ADRs, plane ownership map. |
| **Outputs** | Architecture gate report (pass / fail / conditional), ADR recommendations. |
| **Writes code** | No |

### 5.3 Domain Modeling Agent

| Attribute | Detail |
|-----------|--------|
| **Type** | Reviewer (read-only) |
| **Mandate** | Validate domain models, metadata schemas, planning entities, and knowledge-plane consistency (D022). |
| **Inputs** | Domain specs, metadata definitions, API contracts. |
| **Outputs** | Domain review report, modeling findings, schema alignment checklist. |
| **Writes code** | No (produces model artifacts in docs/evidence only) |

### 5.4 Backend Implementation Agent

| Attribute | Detail |
|-----------|--------|
| **Type** | Implementation |
| **Mandate** | Implement backend services, APIs, state-plane logic (D021), and control-plane integrations (D023) per task scope. |
| **Inputs** | Approved task, API/domain specs, architecture gate pre-checks. |
| **Outputs** | Code diffs, unit/integration tests, task evidence bundle. |
| **Writes code** | Yes |

### 5.5 Metadata Runtime Agent

| Attribute | Detail |
|-----------|--------|
| **Type** | Implementation (scoped) |
| **Mandate** | Define and evolve planning metadata, runtime configuration, and knowledge-plane artifacts (D022) within task boundaries. |
| **Inputs** | Metadata specs, domain review outputs, runtime plane requirements. |
| **Outputs** | Metadata files, configuration manifests, validation scripts, evidence artifacts. |
| **Writes code** | Yes — metadata, config, and runtime descriptors only unless task expands scope |

### 5.6 Frontend Workspace Agent

| Attribute | Detail |
|-----------|--------|
| **Type** | Implementation |
| **Mandate** | Implement application-plane UI and workspace experiences (D024): views, interactions, client-side integration. |
| **Inputs** | Approved task, UX guidelines, API contracts, design tokens. |
| **Outputs** | Frontend diffs, component tests, accessibility notes, evidence bundle. |
| **Writes code** | Yes |

### 5.7 Security & Tenant Isolation Agent

| Attribute | Detail |
|-----------|--------|
| **Type** | Reviewer (read-only) |
| **Mandate** | Assess authn/authz, tenant isolation, data boundaries, secret handling, and compliance with security ADRs. |
| **Inputs** | Diffs, threat model, tenant isolation requirements, dependency reports. |
| **Outputs** | Security gate report, tenant isolation checklist, findings with severity. |
| **Writes code** | No |

### 5.8 Test & Quality Agent

| Attribute | Detail |
|-----------|--------|
| **Type** | Implementation |
| **Mandate** | Author and maintain tests, quality tooling, coverage thresholds, and test evidence for task and sprint scope. |
| **Inputs** | Task specs, implementation diffs, acceptance criteria. |
| **Outputs** | Test code, quality reports, gate evidence, flake/remediation notes. |
| **Writes code** | Yes (tests and quality config) |

### 5.9 DevOps & Cloud Agent

| Attribute | Detail |
|-----------|--------|
| **Type** | Implementation |
| **Mandate** | CI/CD pipelines, infrastructure-as-code, observability hooks, deployment manifests, environment promotion support. |
| **Inputs** | Sprint DevOps tasks, architecture constraints, security requirements. |
| **Outputs** | Pipeline/config diffs, deploy verification logs, DevOps gate evidence. |
| **Writes code** | Yes (CI/CD, IaC, ops config) |

### 5.10 UX Review Agent

| Attribute | Detail |
|-----------|--------|
| **Type** | Reviewer (read-only) |
| **Mandate** | Review UI against UX standards, accessibility (WCAG target level per project ADR), consistency, and task acceptance criteria. |
| **Inputs** | Frontend diffs, screenshots/recordings, UX spec, component library docs. |
| **Outputs** | UX review report, accessibility findings, conditional pass/fail for UX gate. |
| **Writes code** | No |

---

## 6. Sprint Workflow

Sprint execution is a fixed sequence. Steps are not skipped; failures loop back to the indicated step.

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. SPRINT INIT          Sprint Orchestrator                     │
│    Define goal, select tasks, assign agents, open sprint record │
└───────────────────────────────┬─────────────────────────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. ARCHITECTURE PRE-FLIGHT Architecture Guardian (+ Domain)     │
│    Confirm task breakdown respects D021–D024 boundaries         │
└───────────────────────────────┬─────────────────────────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. TASK EXECUTION       Implementation agents (parallel tasks)  │
│    Task-scoped branches, evidence bundles per task              │
└───────────────────────────────┬─────────────────────────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. REVIEW GATES         Reviewer agents (read-only)             │
│    Architecture, Security, UX, Domain as applicable             │
└───────────────────────────────┬─────────────────────────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. QUALITY GATES        Test & Quality + CI                     │
│    Tests pass, coverage thresholds, lint/static analysis        │
└───────────────────────────────┬─────────────────────────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ 6. DEVOPS GATE          DevOps & Cloud Agent                    │
│    Pipeline green, deploy smoke verified in target environment  │
└───────────────────────────────┬─────────────────────────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ 7. TRACEABILITY AUDIT   Sprint Orchestrator                     │
│    Requirement → task → PR → test mapping complete              │
└───────────────────────────────┬─────────────────────────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ 8. HUMAN APPROVAL       Human reviewer                          │
│    Approve and merge each PR; no agent merge to main            │
└───────────────────────────────┬─────────────────────────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ 9. SPRINT CLOSURE       Sprint Orchestrator                     │
│    Sprint artifacts archived; retrospective evidence recorded   │
└─────────────────────────────────────────────────────────────────┘
```

### Sprint Entry Criteria

- Sprint goal documented and linked to epic/requirement IDs.
- Task list approved by human owner.
- Architecture pre-flight scheduled.

### Sprint Exit Criteria (All Required)

- Every in-scope task has a **complete evidence bundle**.
- **Architecture gate:** pass (or conditional pass with documented waivers approved by human).
- **Security & tenant isolation gate:** pass.
- **Test & quality gate:** pass.
- **DevOps gate:** pass.
- **Traceability audit:** complete with no orphan PRs or untested requirements.
- **Human approval:** all sprint PRs merged or explicitly deferred with reason.

**No sprint is complete without architecture, security, test, traceability, and DevOps gates.**

---

## 7. Task Workflow

Each implementation task follows this workflow independently within a sprint.

| Step | Actor | Action |
|------|-------|--------|
| **T1 — Task intake** | Sprint Orchestrator | Create task record: ID, scope, acceptance criteria, plane(s), assigned agent, requirement links. |
| **T2 — Pre-implementation review** | Architecture Guardian / Domain Modeling (if applicable) | Confirm task scope aligns with architecture and domain model. Read-only. |
| **T3 — Branch creation** | Assigned implementation agent | Create task branch from latest sprint integration branch (Section 12). |
| **T4 — Implementation** | Assigned implementation agent | Implement within scope; commit atomically with task ID in messages. |
| **T5 — Self-check** | Implementation agent | Run local lint, unit tests, relevant integration tests; capture logs. |
| **T6 — Test augmentation** | Test & Quality Agent (when separate from implementer) | Add/update tests for acceptance criteria; produce test evidence. |
| **T7 — Evidence bundle** | Implementation agent (+ Test & Quality) | Assemble bundle (Section 9); attach to task record. |
| **T8 — PR creation** | Implementation agent | Open PR with template fields: task ID, requirement IDs, gate checklist, evidence links. |
| **T9 — Reviewer gates** | Reviewer agents | Architecture, Security, UX, Domain reviews as triggered by change type. Read-only. |
| **T10 — CI / quality gate** | Test & Quality Agent + CI | Pipeline must pass; coverage and static analysis thresholds met. |
| **T11 — Human approval** | Human reviewer | Review PR, evidence, gate reports; approve or request changes. |
| **T12 — Merge** | Human reviewer | Merge to integration branch; never by agent. |
| **T13 — Task closure** | Sprint Orchestrator | Mark task complete; link merged commit SHA to traceability matrix. |

### Task Rules

- **No implementation task is complete without evidence.**
- Tasks must not exceed approved scope; new scope requires a new task ID.
- Blocked tasks are escalated to Sprint Orchestrator and human owner within one business day.

---

## 8. Required Sprint Artifacts

The following artifacts must exist for every sprint and be stored under `docs/sprints/<sprint-id>/` (or equivalent sprint registry).

| Artifact | Owner | Description |
|----------|-------|-------------|
| **Sprint charter** | Sprint Orchestrator | Goal, dates, epic links, in/out of scope. |
| **Task registry** | Sprint Orchestrator | All task IDs, assignments, status, requirement links. |
| **Sprint architecture pre-flight report** | Architecture Guardian | Plane alignment confirmation before implementation. |
| **Gate summary** | Sprint Orchestrator | Aggregated pass/fail for all gates (Section 10). |
| **Traceability matrix** | Sprint Orchestrator | Requirement → task → PR → test → commit mapping. |
| **Sprint closure report** | Sprint Orchestrator | Outcomes, deferred items, waivers, retrospective notes. |
| **Human approval log** | Human reviewer | Record of PR approvals with reviewer identity and timestamp. |

---

## 9. Required Evidence Bundle

Each task must produce an **evidence bundle** before PR review. Bundles live at `docs/evidence/<sprint-id>/<task-id>/` or are linked from the PR description.

### Mandatory Bundle Contents

| # | Artifact | Description |
|---|----------|-------------|
| E1 | **Task summary** | What was implemented, scope boundaries, known limitations. |
| E2 | **Requirement linkage** | IDs of requirements/user stories satisfied. |
| E3 | **Change manifest** | Files changed, with brief rationale per significant file. |
| E4 | **Diff reference** | Link to PR or commit range. |
| E5 | **Test evidence** | Test commands run, pass/fail output summary, coverage delta if applicable. |
| E6 | **Manual verification** | Steps performed and results (screenshots, logs, or recordings for UI/API). |
| E7 | **Reviewer gate results** | Architecture / Security / UX / Domain reports when triggered. |
| E8 | **Risk and rollback notes** | Deployment impact, feature flags, rollback procedure. |

### Bundle Quality Rules

- Evidence must be **reproducible** — another engineer can re-run commands and obtain equivalent results.
- Placeholder or "TBD" evidence fails the task gate.
- Sensitive data in logs must be redacted before storage.

---

## 10. Quality Gates

Quality gates are binary pass/fail checkpoints. Conditional pass requires documented waiver approved by a human owner.

### 10.1 Gate Catalog

| Gate | Owner Agent | Pass Criteria |
|------|-------------|---------------|
| **G1 — Architecture** | Architecture Guardian | Changes respect D021–D024 boundaries; no unauthorized cross-plane coupling; ADRs updated if decisions changed. |
| **G2 — Domain model** | Domain Modeling Agent | Entities, metadata, and knowledge-plane artifacts consistent with approved domain spec (when task touches domain). |
| **G3 — Security & tenant isolation** | Security & Tenant Isolation Agent | No critical/high findings open; tenant boundaries enforced; secrets not committed. |
| **G4 — Test & quality** | Test & Quality Agent | All required tests pass; coverage meets sprint threshold; no new lint/static-analysis regressions. |
| **G5 — UX** | UX Review Agent | UI tasks meet UX spec and accessibility checklist (when task touches UI). |
| **G6 — DevOps** | DevOps & Cloud Agent | CI pipeline green; deploy/smoke successful in designated environment; observability hooks present if required. |
| **G7 — Traceability** | Sprint Orchestrator | Every requirement in task scope maps to merged code and at least one test or verification record. |
| **G8 — Evidence completeness** | Sprint Orchestrator | Bundle sections E1–E8 present and valid. |

### 10.2 Gate Execution Order

For each PR:

1. Evidence completeness (G8) — pre-check before reviewer time is spent.
2. Architecture (G1) and Domain (G2) — as applicable.
3. Implementation CI (G4 partial — automated checks).
4. Security (G3).
5. UX (G5) — if UI changed.
6. Test & quality final (G4).
7. DevOps (G6) — before merge to integration branch.
8. Traceability (G7) — at task closure and sprint closure.

Failed gates block merge. Fix forward on the task branch or close PR and reopen after remediation.

---

## 11. Traceability Model

Traceability ensures every merged change is auditable from business intent to production-ready code.

### 11.1 Traceability Chain

```
Requirement (REQ-xxx)
    └── Task (TASK-xxx)
            └── Branch (feature/TASK-xxx-short-desc)
                    └── Pull Request (PR-xxx)
                            ├── Evidence bundle (E1–E8)
                            ├── Gate reports (G1–G8)
                            ├── Tests (TEST-xxx or test file paths)
                            └── Merge commit (SHA)
```

### 11.2 Identifier Conventions

| Entity | Pattern | Example |
|--------|---------|---------|
| Requirement | `REQ-<epic>.<story>` | `REQ-1.3` |
| Task | `TASK-<sprint>-<seq>` | `TASK-S01-004` |
| Pull request | Platform PR number + task link | PR #42 → `TASK-S01-004` |
| Test | Reference by path or `TEST-<task>-<seq>` | `TEST-S01-004-01` |
| Evidence | `docs/evidence/<sprint>/<task>/` | `docs/evidence/S01/TASK-S01-004/` |

### 11.3 Traceability Matrix (Minimum Columns)

| REQ ID | Task ID | PR | Branch | Merge SHA | Test refs | Gates passed | Human approver |
|--------|---------|-----|--------|-----------|-----------|--------------|----------------|

The Sprint Orchestrator maintains this matrix and resolves gaps before sprint closure.

---

## 12. Branch and Pull Request Rules

### 12.1 Branch Hierarchy

```
main                          ← production-ready; human merge only
  └── sprint/<sprint-id>      ← sprint integration branch
        └── task/<task-id>-<slug>   ← task-scoped implementation
```

- **`main`:** Protected. No agent commits or merges.
- **`sprint/<sprint-id>`:** Integration branch for the active sprint. Human or approved automation merges task PRs here after gates.
- **`task/<task-id>-<slug>`:** Single-task scope. Implementation agents create and push here only.

### 12.2 Pull Request Requirements

Every PR must include:

- [ ] Task ID and linked requirement IDs
- [ ] Summary of changes and explicit out-of-scope declaration
- [ ] Evidence bundle link
- [ ] Gate checklist with status (pending / pass / fail)
- [ ] Rollback / feature-flag notes if applicable
- [ ] Plane(s) affected (D021 / D022 / D023 / D024)

### 12.3 PR Lifecycle Rules

- One primary task per PR; drive-by changes are forbidden.
- Draft PRs may be opened for early feedback; gate execution begins when marked ready for review.
- PRs failing any mandatory gate remain open until fixed or closed with documented reason.
- Squash or merge strategy follows repository convention; merge commit on `main` must reference task ID.

---

## 13. Human Approval Rules

AI agents **propose**; humans **approve**. Merge authority is never delegated to agents.

### 13.1 Required Human Actions

| Action | When | Who |
|--------|------|-----|
| Approve sprint scope and task list | Sprint init | Product / engineering owner |
| Waive gate failures | Conditional pass | Engineering lead + domain owner |
| Approve pull request | After all mandatory gates pass | Assigned human reviewer (CODEOWNERS) |
| Merge to `main` or sprint integration | Post-approval | Human reviewer only |
| Close sprint | All exit criteria met | Sprint owner |

### 13.2 Approval Preconditions

Human PR approval requires:

1. All applicable gates passed (Section 10) or approved waivers on record.
2. Evidence bundle complete (Section 9).
3. Traceability matrix row complete for the task.
4. No unresolved critical findings from Security or Architecture reviewers.

### 13.3 Escalation

- **Critical security finding:** Stop merge; Security & Tenant Isolation Agent and human security contact notified.
- **Architecture drift:** Architecture Guardian fails gate; human architect resolves before merge.
- **Scope dispute:** Sprint Orchestrator pauses task; human owner clarifies scope in task record.

---

## 14. Tool Usage Model

Tools are categorized by agent permission. Agents use only tools allowed for their role type.

### 14.1 Tool Categories

| Category | Examples | Orchestration | Reviewer | Implementation |
|----------|----------|:-------------:|:--------:|:--------------:|
| **Read / search** | File read, repo search, doc fetch | ✓ | ✓ | ✓ |
| **Static analysis** | Lint, type-check, SAST (read-only mode) | ✓ | ✓ | ✓ |
| **Build & test** | Unit/integration test runners, coverage | Observe | Observe | ✓ |
| **Format / codegen** | Formatters, scaffolding | ✗ | ✗ | ✓ |
| **Write / patch** | File edit, branch push | ✗ | ✗ | ✓ (authorized agents) |
| **CI/CD** | Pipeline trigger, deploy smoke | Status only | ✗ | ✓ (DevOps agent) |
| **PM / tracking** | Task status, sprint registry updates | ✓ | Findings only | Task updates |

### 14.2 Tool Usage Rules

- Match tools to task scope — do not run full-suite E2E for doc-only changes unless gate policy requires it.
- Capture command outputs in evidence bundles (redacted as needed).
- Reviewer agents must not invoke write, patch, or deploy tools.
- Implementation agents run tests before requesting reviewer time.
- DevOps & Cloud Agent alone applies infrastructure changes outside application source trees unless task assigns otherwise.

### 14.3 Evidence Capture from Tools

Minimum captures for implementation tasks:

- Test runner summary (pass count, fail count, duration)
- Linter/static-analysis summary
- CI pipeline URL and final status
- Deploy smoke output (when DevOps gate applies)

---

## 15. Epic 1 Application

Epic 1 establishes the foundational Planning Platform capabilities across all four architecture planes. The operating model applies as follows.

### 15.1 Epic 1 Sprint Mapping (Illustrative)

| Sprint focus | Primary implementation agents | Primary reviewer agents |
|--------------|------------------------------|-------------------------|
| **S01 — Foundation & repo scaffolding** | DevOps & Cloud, Backend Implementation | Architecture Guardian, Security & Tenant Isolation |
| **S02 — Domain & metadata baseline (D022)** | Metadata Runtime, Backend Implementation | Domain Modeling, Architecture Guardian |
| **S03 — State plane core (D021)** | Backend Implementation, Test & Quality | Architecture Guardian, Security & Tenant Isolation |
| **S04 — Control plane baseline (D023)** | Backend Implementation, DevOps & Cloud | Architecture Guardian, Security & Tenant Isolation |
| **S05 — Application workspace shell (D024)** | Frontend Workspace, Test & Quality | UX Review, Architecture Guardian |
| **S06 — Integration & hardening** | All implementation agents as needed | All reviewer agents; full gate suite |

Exact sprint IDs and dates are defined in the Epic 1 sprint charter, not in this document.

### 15.2 Epic 1 Agent Assignment Principles

- **Sprint Orchestrator** runs every Epic 1 sprint from init through closure.
- **Architecture Guardian** participates in every sprint pre-flight and every PR touching plane boundaries.
- **Security & Tenant Isolation Agent** gates every sprint before closure — tenant isolation is non-negotiable from S01 onward.
- **Domain Modeling Agent** leads reviews for S02 and any task modifying planning entities or metadata.
- **Test & Quality Agent** pairs with each implementation task from S03 onward at minimum.
- **UX Review Agent** joins all Frontend Workspace tasks from S05 onward.

### 15.3 Epic 1 Definition of Done (Task Level)

A task in Epic 1 is done when:

- [ ] Acceptance criteria met and evidenced (Section 9)
- [ ] Applicable gates passed (Section 10)
- [ ] PR merged to sprint integration branch by human approver
- [ ] Traceability matrix updated (Section 11)

### 15.4 Epic 1 Definition of Done (Sprint Level)

An Epic 1 sprint is done when:

- [ ] All sprint exit criteria met (Section 6)
- [ ] Sprint artifacts archived (Section 8)
- [ ] No open critical security or architecture findings
- [ ] Human sprint sign-off recorded in sprint closure report

### 15.5 Epic 1 First Task Recommendation

The first executable task under this model should be:

> **TASK-S01-001:** Initialize sprint registry, evidence directory structure, PR template, and gate checklist templates — owned by **DevOps & Cloud Agent** and **Sprint Orchestrator**, with **Architecture Guardian** pre-flight.

This bootstraps the artifact paths referenced throughout this document without implementing Planning Platform business features.

---

## Document Control

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-06-15 | AGENTOPS-T01 | Initial operating model |

**Review cadence:** End of Epic 1 and after any major change to agent tooling or architecture planes (D021–D024).
