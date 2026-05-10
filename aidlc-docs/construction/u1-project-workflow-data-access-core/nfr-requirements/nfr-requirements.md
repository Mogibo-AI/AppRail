# NFR Requirements - U1 Project, Workflow, and Data Access Core

## Purpose

This document defines measurable non-functional requirements for U1. U1 is a foundation unit, so the requirements focus on project/workspace scale, workflow transition correctness, project-scoped data access, audit durability, dashboard resilience, shared DTO compatibility, observability, and property-based testing.

## Revision Summary

This revision resolves review findings by making latency budgets, load profile, pagination limits, listing scope, fail-closed behavior, outbox lifecycle, metric contracts, log safety rules, schema versioning, dashboard/next-action behavior, and PBT CI execution measurable.

## Answer Analysis

| Question | Decision |
|---|---|
| Q1 Project scale | Support around 1,000 projects per workspace with indexes and cursor pagination |
| Q2 API latency | Target P95 500ms for U1 major APIs |
| Q3 Dashboard dependency timeout | Revised to exact 200ms dependent section timeout to keep dashboard end-to-end P95 500ms |
| Q4 Availability | No explicit SLO in MVP; prioritize recovery and auditability |
| Q5 Audit outbox durability | Outbox write is mandatory in transaction; dispatch failures are retained until manual re-run or intentional close |
| Q6 Auth integration | Define provider-neutral Auth adapter; Auth.js/Cognito selection deferred to NFR Design |
| Q7 Mutation consistency | Update project/workflow/outbox in a single PostgreSQL transaction |
| Q8 Validation | Validate at API and service/use case boundaries; repository requires `ProjectScope` |
| Q9 PBT framework | Adopt `fast-check` for TypeScript PBT |
| Q10 PBT CI | PR CI uses fixed seed with `numRuns >= 100`; nightly uses logged seed with `numRuns >= 1000` |
| Q11 Observability | Structured logs, request ID, error code, audit outbox status, API latency metric, dashboard section unavailable metric |
| Q12 DTO/schema compatibility | Use package-level semantic versioning and migration notes; avoid breaking changes during MVP |

## MVP Load Profile

All U1 latency and pagination requirements are measured against this MVP load profile.

| Dimension | Target Profile |
|---|---|
| Active workspaces | 100 active workspaces |
| Projects per workspace | Up to 1,000 projects |
| Concurrent authenticated users | 50 active users across the environment |
| Sustained U1 API load | 20 requests/second total |
| Burst U1 API load | 50 requests/second for up to 60 seconds |
| Dashboard read load | 10 requests/second within total U1 API load |
| Project listing load | 5 requests/second within total U1 API load |
| Mutation load | 2 requests/second within total U1 API load |
| Project list default page size | 25 |
| Project list maximum page size | 100 |
| Expected dependent port P95 | 150ms for U4/U10/U11 summary ports |
| Dependent port timeout | 200ms exact timeout per section/check |

## Scalability Requirements

| ID | Requirement | Acceptance Criteria |
|---|---|---|
| U1-NFR-SCALE-001 | U1 must support at least 1,000 projects per workspace in MVP. | Under MVP load profile, indexed project listing and dashboard reads satisfy latency budgets below. |
| U1-NFR-SCALE-002 | Project listing must use cursor pagination. | `ListProjectsInput.limit` defaults to 25, must not exceed 100, and must include an opaque cursor for subsequent pages. |
| U1-NFR-SCALE-003 | Project listing scope is workspace-accessible projects, not only actor-created projects. | `listProjects(workspaceId)` returns projects the actor can access through workspace role or project membership. |
| U1-NFR-SCALE-004 | Project uniqueness checks must remain index-backed. | Unique indexes exist for normalized name and slug according to the index section below. |
| U1-NFR-SCALE-005 | Dashboard aggregation must call U4/U10/U11 summary ports in parallel. | Slow dependent sections do not serially block each other. |

## Listing Scope and Index Requirements

MVP `listProjects` behavior:

- `workspaceId` is required.
- Workspace `owner` and `builder` can list all projects in the workspace.
- Workspace `viewer` can list only projects where the actor has `ProjectMembership`.
- Default listing excludes `archived` unless explicitly filtered.
- Canonical project routes use `projectId`; `slug` is a friendly segment and remains unique within `(workspaceId, createdByActorId)`.

| Table / Area | Required Index or Constraint | Purpose |
|---|---|---|
| `Project` | Unique `(workspaceId, createdByActorId, normalizedName)` | Creator-scoped duplicate name prevention |
| `Project` | Unique `(workspaceId, createdByActorId, slug)` | Creator-scoped friendly slug generation |
| `Project` | Index `(workspaceId, status, updatedAt DESC, id DESC)` | Workspace owner/builder listing |
| `Project` | Index `(workspaceId, createdByActorId, status, updatedAt DESC, id DESC)` | Creator-filtered listing |
| `WorkspaceMembership` | Unique `(workspaceId, actorId)` | Workspace scope lookup |
| `ProjectMembership` | Unique `(projectId, actorId)` | Project role lookup |
| `ProjectMembership` | Index `(actorId, projectId)` | Viewer/member accessible project listing |
| `WorkflowState` | Unique `(projectId)` | One active workflow state per project |
| `AuditOutbox` | Index `(status, nextAttemptAt, updatedAt)` | Retry/manual re-run operations |

## Latency Budget Requirements

| ID | Requirement | Acceptance Criteria |
|---|---|---|
| U1-NFR-PERF-001 | U1 core APIs must meet P95 500ms under MVP load profile. | Applies to `createProject`, `listProjects`, `getProject`, `getWorkflowState`, `canAdvanceStage`, and `advanceStage`, measured end-to-end inside the API boundary excluding client network time. |
| U1-NFR-PERF-002 | `getProjectDashboard` must meet P95 500ms end-to-end under MVP load profile. | Applies when dependent U4/U10/U11 summary ports satisfy expected P95 <=150ms or time out at 200ms. |
| U1-NFR-PERF-003 | `getProjectDashboard` U1 local processing budget is P95 250ms. | Includes project/workflow reads, next action calculation, section composition, and serialization; excludes waiting for dependent port responses. |
| U1-NFR-PERF-004 | Dashboard dependent section timeout is exactly 200ms per section. | U4 jobs, U10 security, and U11 approval/audit sections run in parallel; timed-out sections become `DashboardSection.status = "unavailable"`. |
| U1-NFR-PERF-005 | Workflow transition dependent checks have exact 200ms timeout per port. | ApprovalGatePort and SecurityPolicyPort checks run in parallel where both are needed. |
| U1-NFR-PERF-006 | Workflow transition decisions must be bounded. | `canAdvanceStage` reads current workflow state, transition metadata, gate status, and blocker summary only. |

## Availability and Resilience Requirements

| ID | Requirement | Acceptance Criteria |
|---|---|---|
| U1-NFR-AVAIL-001 | MVP does not declare a formal uptime SLO for U1. | NFR Design may add operational objectives, but this stage prioritizes recovery and auditability. |
| U1-NFR-AVAIL-002 | Dashboard must degrade by section, not fail as a whole, when U4/U10/U11 summary ports are unavailable. | U1 returns `ok: true` with unavailable `DashboardSection<T>` entries when project/workflow core read succeeds. |
| U1-NFR-AVAIL-003 | Stage transition checks must fail closed when approval/security ports are unavailable. | U1 must not advance a stage unless required approval and security checks are successfully evaluated. |
| U1-NFR-AVAIL-004 | Mutations must be transactionally safe. | Project/workflow/outbox changes commit together or fail together. |
| U1-NFR-AVAIL-005 | Audit dispatch failures must be recoverable. | Outbox entries remain available until auto retry succeeds, manual re-run succeeds, or an authorized actor intentionally closes them. |

## Fail-Closed Transition Requirements

| Scenario | U1 Behavior | Public Error Code | Observability Reason |
|---|---|---|---|
| ApprovalGatePort timeout | Do not advance stage | `WORKFLOW_APPROVAL_REQUIRED` | `approval-check-timeout` |
| ApprovalGatePort unavailable/error | Do not advance stage | `WORKFLOW_APPROVAL_REQUIRED` | `approval-check-unavailable` |
| SecurityPolicyPort timeout | Do not advance stage | `WORKFLOW_BLOCKED_BY_SECURITY` | `security-check-timeout` |
| SecurityPolicyPort unavailable/error | Do not advance stage | `WORKFLOW_BLOCKED_BY_SECURITY` | `security-check-unavailable` |
| Transition metadata unavailable | Do not advance stage | `WORKFLOW_METADATA_MISSING` | `workflow-metadata-unavailable` |

`WORKFLOW_APPROVAL_REQUIRED` and `WORKFLOW_BLOCKED_BY_SECURITY` are intentionally reused from the functional contract. User-facing text must explain that the system could not safely confirm the required condition, not that a human necessarily rejected it.

## Dashboard and NextAction Requirements

| Condition | Dashboard Section | NextAction Behavior |
|---|---|---|
| U11 approval summary unavailable and current transition requires approval | `approval.status = "unavailable"` | `NextAction.actionType = "wait"`, `disabled = true`, `disabledReason = "DEPENDENCY_UNAVAILABLE"` |
| U10 security summary unavailable and current transition needs blocker check | `security.status = "unavailable"` | `NextAction.actionType = "wait"`, `disabled = true`, `disabledReason = "DEPENDENCY_UNAVAILABLE"` |
| U4 recent jobs unavailable but next action does not require job state | `recentJobs.status = "unavailable"` | NextAction remains enabled if approval/security conditions are known |
| U4 recent jobs unavailable and next action requires job status | `recentJobs.status = "unavailable"` | `NextAction.actionType = "wait"`, `disabled = true`, `disabledReason = "DEPENDENCY_UNAVAILABLE"` |
| Dependent sections available and no blockers/missing approvals | Relevant sections `available` | NextAction points to the target unit and route from U1 contract |

U13 must render the disabled/wait state from U1 rather than recomputing workflow rules.

## Consistency and Durability Requirements

| ID | Requirement | Acceptance Criteria |
|---|---|---|
| U1-NFR-CONS-001 | `createProject`, `advanceStage`, `blockStage`, and `archiveProject` must use a single PostgreSQL transaction for project/workflow/outbox updates. | No mutation succeeds without a durable audit outbox entry. |
| U1-NFR-CONS-002 | Audit outbox write failure is a blocking mutation failure. | API returns `AUDIT_OUTBOX_WRITE_FAILED` and does not commit the project/workflow mutation. |
| U1-NFR-CONS-003 | Audit dispatch is fully asynchronous after commit. | U1 does not attempt outbox dispatch before sending the API response. |
| U1-NFR-CONS-004 | Successful mutating APIs return `AUDIT_DISPATCH_PENDING` in `Result<T>.warnings` when the created outbox entry remains `pending` at response time. | Warning is carried in the existing `Result<T>.warnings` field, not a separate metadata envelope. |
| U1-NFR-CONS-005 | Workflow transition state must be deterministic. | Transition table is the sole source of allowed stage movement in U1. |

## Audit Outbox Lifecycle

| Status | Meaning | Allowed Next Status |
|---|---|---|
| `pending` | Outbox entry was committed and is waiting for dispatch | `dispatching`, `intentionally_closed` |
| `dispatching` | Dispatcher is attempting delivery to U11 | `dispatched`, `retry_scheduled`, `manual_replay_required` |
| `retry_scheduled` | Dispatch failed and automatic retry is scheduled | `dispatching`, `manual_replay_required`, `intentionally_closed` |
| `manual_replay_required` | Automatic retry budget is exhausted or failure is non-retryable | `pending`, `dispatching`, `intentionally_closed` |
| `dispatched` | U11 accepted the audit event | Final |
| `intentionally_closed` | Authorized actor closed the entry with a reason | Final |

| Lifecycle Attribute | Requirement |
|---|---|
| Automatic retry attempts | Maximum 10 attempts |
| Retry schedule | Exponential backoff capped at 1 hour |
| `nextAttemptAt` | Required for `retry_scheduled`; null for `pending`, `manual_replay_required`, `dispatched`, and `intentionally_closed` |
| Manual re-run | Allowed for `manual_replay_required` and `retry_scheduled`; resets status to `pending` or immediately `dispatching` |
| Intentional close | Requires actor, timestamp, and close reason; does not delete the outbox entry |
| Retention | Outbox entries are retained through MVP unless explicitly archived by an operations process defined later |

## Security Requirements

| ID | Requirement | Security Rule Alignment | Acceptance Criteria |
|---|---|---|---|
| U1-NFR-SEC-001 | Workspace access must be enforced before project creation/listing. | SECURITY-08 | `WorkspaceScope` is resolved before `createProject` and `listProjects`. |
| U1-NFR-SEC-002 | Project scope must be enforced before project read/mutate/archive operations. | SECURITY-08 | `ProjectScopeDecision` is required before repository calls. |
| U1-NFR-SEC-003 | Repository calls for project-owned data must require `ProjectScope`. | SECURITY-08, SECURITY-11 | Raw `projectId` is insufficient for project-owned repositories. |
| U1-NFR-SEC-004 | API and service/use case boundaries must validate inputs. | SECURITY-05 | DTO schema validation rejects invalid types, lengths, enum values, and missing required fields. |
| U1-NFR-SEC-005 | U1 must not expose raw database or provider error details to UI. | SECURITY-09 | Public APIs return formal `U1ErrorCode` values and user-facing messages. |
| U1-NFR-SEC-006 | Auth provider must remain abstracted from U1 domain logic. | SECURITY-11 | U1 depends on `Actor`, `WorkspaceScope`, and `ProjectScope` contracts, not Auth.js or Cognito objects. |

## Validation Requirements

| ID | Requirement | Acceptance Criteria |
|---|---|---|
| U1-NFR-VAL-001 | API boundary validation is required for all U1 public endpoints. | `CreateProjectInput`, `ListProjectsInput`, `AdvanceStageInput`, and `BlockStageInput` have schemas. |
| U1-NFR-VAL-002 | Service/use case boundary validation is required before transaction execution. | Use cases validate normalized DTOs and scope decisions before writing. |
| U1-NFR-VAL-003 | Repository layer must require `ProjectScope` but does not repeat full DTO schema validation. | Repository contracts are typed and project-scoped; validation remains at API/use case boundaries. |

## Observability Requirements

| ID | Requirement | Acceptance Criteria |
|---|---|---|
| U1-NFR-OBS-001 | Structured logs are required for U1 API/use case entry points. | Logs include timestamp, request ID, operation name, result status, formal error code, latency bucket, and safe identifiers defined below. |
| U1-NFR-OBS-002 | Sensitive values must not be logged. | Project idea text, target user free text, project name, workspace name, actor display name, tokens, secrets, raw DB errors, and raw provider errors are excluded. |
| U1-NFR-OBS-003 | API latency metrics are required. | Metrics are emitted for `createProject`, `listProjects`, `getProjectDashboard`, and workflow operations. |
| U1-NFR-OBS-004 | Dashboard section unavailable metrics are required. | U1 emits section, reason, and retryable status when a section becomes unavailable. |
| U1-NFR-OBS-005 | Audit outbox current status and status transitions must be observable. | Gauge tracks current count by status; counter tracks transitions. |

## Structured Logging Data Classification

| Data | Logging Rule |
|---|---|
| `requestId` | Log in plaintext |
| `operation` | Log in plaintext |
| `U1ErrorCode` / warning code | Log in plaintext |
| `workflowStageId`, `approvalGateType`, outbox status | Log in plaintext |
| `actorId`, `workspaceId`, `projectId`, `outboxEntryId` | Log as opaque IDs only if they are internal UUID/ULID values; otherwise hash before logging |
| `actor.displayName`, email, project name, workspace name | Do not log; hash only if needed for debugging with explicit justification |
| `initialIdeaText`, `targetUsers`, generated artifacts, raw logs | Do not log |
| Tokens, API keys, cookies, secrets | Do not log |
| Raw DB/provider error payloads | Do not log; map to formal error code and technical details ref |

## Metric Contract

| Metric | Type | Labels | Cardinality Limit |
|---|---|---|---|
| `u1_api_latency_ms` | Histogram | `operation`, `result` | `operation` <= 12, `result` <= 3 |
| `u1_api_error_total` | Counter | `operation`, `error_code` | `operation` <= 12, `error_code` <= formal `U1ErrorCode` count |
| `u1_dashboard_section_unavailable_total` | Counter | `section`, `reason`, `retryable` | `section` <= 4, `reason` <= 6, `retryable` <= 2 |
| `u1_audit_outbox_current` | Gauge | `status` | `status` <= 6 |
| `u1_audit_outbox_transitions_total` | Counter | `from_status`, `to_status`, `result` | `from_status` <= 6, `to_status` <= 6, `result` <= 3 |
| `u1_workflow_transition_fail_closed_total` | Counter | `reason` | `reason` <= 5 |

Metrics must not include actor, workspace, project, slug, name, email, or free-text labels.

## Maintainability and Compatibility Requirements

| ID | Requirement | Acceptance Criteria |
|---|---|---|
| U1-NFR-MAINT-001 | Shared DTO/schema compatibility is managed by package-level semantic versioning for `packages/shared-types`. | Individual DTOs do not carry `schemaVersion`; the package version governs the shared contract. |
| U1-NFR-MAINT-002 | Breaking changes are avoided during MVP unless explicitly approved. | Breaking changes require version bump, migration note, and dependent unit review. |
| U1-NFR-MAINT-003 | Migration notes are required for contract changes. | Any change to `Result<T>`, `ProjectDashboard`, `NextAction`, `ProjectScope`, `WorkspaceScope`, or workflow stage/gate names includes a migration note. |
| U1-NFR-MAINT-004 | U1 contracts must keep U2/U4/U10/U11/U13 details behind ports/refs/summaries. | No U1 type imports provider-specific auth, job, security, approval, or UI implementation objects. |

## Migration Note Format

```markdown
## [shared-types version] - [date]

- Change type: additive | breaking | removal | behavior
- Affected contracts:
- Summary:
- Required migration:
- Dependent units to review:
- Backward compatibility:
- Rollback notes:
```

## PBT Requirements

| ID | Requirement | PBT Rule Alignment | Acceptance Criteria |
|---|---|---|---|
| U1-NFR-PBT-001 | U1 uses `fast-check` for TypeScript property-based testing. | PBT-09 | `fast-check` is included as a dev dependency during code generation. |
| U1-NFR-PBT-002 | PR CI runs fixed-seed PBT with at least 100 `numRuns` per property. | PBT-08 | CI logs seed, `numRuns`, and failing shrunk case. |
| U1-NFR-PBT-003 | Nightly CI runs broader PBT with at least 1,000 `numRuns` per property. | PBT-08 | Nightly seed is logged and can be replayed locally. |
| U1-NFR-PBT-004 | PBT uses domain-specific generators. | PBT-07 | Generators exist for project names, workspace IDs, actors, scopes, stage sequences, dashboard sections, and outbox entries. |
| U1-NFR-PBT-005 | PBT complements example-based tests. | PBT-10 advisory for later code generation | Critical project creation and transition examples remain covered by explicit unit tests. |

## PBT CI Profile

| CI Profile | Minimum `numRuns` | Seed Policy | Required Property Areas |
|---|---:|---|---|
| PR CI | 100 per property | Fixed seed from `U1_PBT_SEED`, default `20260509`; seed logged in output | normalization, slug allocation, uniqueness key, scope decisions, workflow transitions, dashboard section mapping |
| Nightly CI | 1,000 per property | Date-based or generated seed; seed logged and stored in CI output | all PR properties plus audit outbox lifecycle and longer workflow command sequences |

**Reproduction requirement**: A failing PBT run must print the seed, `numRuns`, property name, and shrunk counterexample. Local replay must be possible by setting `U1_PBT_SEED` and the property name.

## Non-Goals

- U1 does not select final Auth.js vs Cognito implementation in this stage.
- U1 does not define infrastructure SLO dashboards or alert rules in this stage.
- U1 does not design U4 job execution, U10 security classification, U11 approval storage, or U13 UI internals.
- U1 does not implement full-text search for project listing in MVP.

## Extension Compliance Summary

| Extension | Status | Notes |
|---|---|---|
| Security Baseline | Compliant | SECURITY-05, SECURITY-08, SECURITY-09, SECURITY-11, and SECURITY-13 are directly addressed for U1. Infrastructure-specific rules are deferred to later stages. |
| Property-Based Testing | Compliant | PBT-09 is satisfied by selecting `fast-check`; PBT-07 and PBT-08 are specified with concrete generator, seed, and CI requirements. |

## Blocking Findings

No blocking Security Baseline or PBT findings are present in revised U1 NFR Requirements.
