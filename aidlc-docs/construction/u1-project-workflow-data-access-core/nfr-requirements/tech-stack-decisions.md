# Tech Stack Decisions - U1 Project, Workflow, and Data Access Core

## Purpose

This document records technology decisions needed to satisfy U1 NFR requirements. Decisions are scoped to U1 and avoid selecting implementation details owned by U2/U4/U10/U11/U13.

## Decision Summary

| Area | Decision | Status |
|---|---|---|
| Language | TypeScript | Accepted |
| Runtime Boundary | U1 implemented as shared packages consumed by Next.js API and worker where needed | Accepted |
| Database | PostgreSQL | Accepted from project-level design |
| ORM/Data Access | Prisma with U1 repository wrappers requiring `ProjectScope` | Accepted |
| Validation | Zod schemas at API and service/use case boundaries | Accepted for U1 |
| Auth Integration | Provider-neutral Auth adapter producing `Actor`, `WorkspaceScope`, and `ProjectScope` | Accepted; concrete provider deferred |
| PBT Framework | `fast-check` | Accepted |
| Unit Test Runner | Vitest for TypeScript package tests | Accepted for U1 package-level tests |
| Pagination | Cursor pagination, default limit 25, max limit 100 | Accepted |
| Observability | Structured logs plus OpenTelemetry-compatible metrics naming | Accepted; backend/exporter deferred |
| Audit Durability | PostgreSQL audit outbox with auto retry, manual re-run, and intentional close states | Accepted |
| Contract Compatibility | Package-level semantic versioning for `packages/shared-types` plus migration notes | Accepted |

## Package Placement

| Package | Purpose |
|---|---|
| `packages/shared-types/` | U1 shared DTOs, `Result<T>`, error codes, workflow/stage/gate types, dashboard refs |
| `packages/project-workflow/` | Project creation, workflow state, transition decisions, dashboard aggregation |
| `packages/data-access/` | Prisma client boundary, transaction wrapper, project-scoped repository contracts |

## TypeScript and DTO Decisions

| Decision | Rationale |
|---|---|
| Use discriminated union `Result<T>` with `warnings?: UserFacingWarning[]` on success | Prevent invalid states and carry non-blocking warnings such as `AUDIT_DISPATCH_PENDING`. |
| Keep formal `U1ErrorCode` as string literal union | Enables field-level UI errors and stable API contracts. |
| Use package-level semantic versioning for `packages/shared-types` | Avoids adding individual `schemaVersion` to every DTO while still controlling shared contract changes. |
| Avoid provider-specific auth/job/security/audit types in U1 DTOs | Preserves U1 ownership boundary. |

## Validation Stack

| Decision | Rationale |
|---|---|
| Use Zod for API boundary schemas | TypeScript-friendly schema validation with inferred types. |
| Use Zod or equivalent domain schemas at service/use case boundaries | Protects use cases from invalid internal callers and generated code mistakes. |
| Do not repeat full DTO validation inside repositories | Repository safety is enforced by typed `ProjectScope`, transaction context, and Prisma query shape. |

## Data Access Stack

| Decision | Rationale |
|---|---|
| Use Prisma over PostgreSQL | Matches project-level stack and provides typed query construction. |
| Wrap Prisma access in U1 repository interfaces | Enforces `ProjectScope` and prevents direct persistence coupling. |
| Use one PostgreSQL transaction per mutation use case | Required for project/workflow/outbox consistency. |
| Use unique indexes for normalized name and slug | Required for deterministic duplicate handling. |

## Listing Scope Decision

`listProjects(workspaceId)` lists workspace-accessible projects:

- Workspace `owner` and `builder` can list all projects in the workspace.
- Workspace `viewer` can list only projects where the actor has `ProjectMembership`.
- Default listing excludes `archived` unless explicitly filtered.
- Cursor pagination uses `updatedAt DESC, id DESC`.
- Default page size is 25.
- Maximum page size is 100.

## Required Indexes and Constraints

| Table / Area | Constraint or Index |
|---|---|
| `Project` | Unique `(workspaceId, createdByActorId, normalizedName)` |
| `Project` | Unique `(workspaceId, createdByActorId, slug)` |
| `Project` | Index `(workspaceId, status, updatedAt DESC, id DESC)` |
| `Project` | Index `(workspaceId, createdByActorId, status, updatedAt DESC, id DESC)` |
| `WorkspaceMembership` | Unique `(workspaceId, actorId)` |
| `ProjectMembership` | Unique `(projectId, actorId)` |
| `ProjectMembership` | Index `(actorId, projectId)` |
| `WorkflowState` | Unique `(projectId)` |
| `AuditOutbox` | Index `(status, nextAttemptAt, updatedAt)` |

## Auth Adapter Decision

U1 defines an auth boundary but does not select the concrete provider in this stage.

```ts
type AuthContextProvider = {
  getActor(): Promise<Result<Actor>>;
  getWorkspaceScope(workspaceId: ID, operation: WorkspaceOperation, actor: Actor): Promise<Result<WorkspaceScopeDecision>>;
  getProjectScope(projectId: ID, operation: ProjectOperation, actor: Actor): Promise<Result<ProjectScopeDecision>>;
};
```

Concrete Auth.js or Cognito integration is deferred to U1 NFR Design or the unit that owns application authentication. U1 must depend only on `Actor`, `WorkspaceScopeDecision`, and `ProjectScopeDecision`.

## PBT Stack Decision

| Decision | Rationale |
|---|---|
| Use `fast-check` | It supports TypeScript, custom generators, shrinking, reproducible seeds, and integration with Vitest. |
| Use Vitest for U1 package tests | Provides fast TypeScript test execution for package-level logic. |
| PR CI runs `fast-check` with `numRuns >= 100` per property | Keeps CI predictable while executing properties. |
| Nightly CI runs `fast-check` with `numRuns >= 1000` per property | Increases edge-case discovery without slowing every PR. |
| Use `U1_PBT_SEED`, default `20260509`, in PR CI | Makes PR failures reproducible. |

## PBT Generator Requirements

| Generator | Purpose |
|---|---|
| `arbProjectName` | Normalization, slug generation, uniqueness |
| `arbWorkspaceId` | Workspace-scoped uniqueness and listing |
| `arbActor` | Scope and role checks |
| `arbWorkspaceScope` | Workspace access properties |
| `arbProjectScope` | Read/mutate/archive access properties |
| `arbWorkflowStageSequence` | Stage transition order properties |
| `arbDashboardSection` | Available/unavailable dashboard section properties |
| `arbAuditOutboxEntry` | Retry/manual re-run state properties |

## Observability Stack Decision

| Area | Decision |
|---|---|
| Logs | Structured JSON-compatible logs with request ID, operation, formal error code, and safe opaque IDs |
| Metrics | OpenTelemetry-compatible metric names and labels; exporter/backend deferred |
| Required Metrics | API latency, API error count, dashboard section unavailable count, audit outbox current count, audit outbox transition count, fail-closed transition count |
| Sensitive Data | Do not log idea text, target-user free text, project name, workspace name, actor display name, secrets, tokens, raw DB errors, or provider payloads |

## Metric Names

| Metric | Type | Required Labels | Cardinality Limit |
|---|---|---|---|
| `u1_api_latency_ms` | Histogram | `operation`, `result` | `operation` <= 12, `result` <= 3 |
| `u1_api_error_total` | Counter | `operation`, `error_code` | `operation` <= 12, `error_code` <= formal `U1ErrorCode` count |
| `u1_dashboard_section_unavailable_total` | Counter | `section`, `reason`, `retryable` | `section` <= 4, `reason` <= 6, `retryable` <= 2 |
| `u1_audit_outbox_current` | Gauge | `status` | `status` <= 6 |
| `u1_audit_outbox_transitions_total` | Counter | `from_status`, `to_status`, `result` | `from_status` <= 6, `to_status` <= 6, `result` <= 3 |
| `u1_workflow_transition_fail_closed_total` | Counter | `reason` | `reason` <= 5 |

## Audit Outbox Decision

| Decision | Rationale |
|---|---|
| Outbox write is inside mutation transaction | A state-changing API must not succeed without durable audit intent. |
| Dispatch is fully asynchronous after API response | Keeps mutation latency predictable and avoids external delivery in the request path. |
| New committed outbox entries start as `pending` | Dispatcher owns delivery after commit. |
| Successful mutating APIs return `AUDIT_DISPATCH_PENDING` in `Result<T>.warnings` when the created outbox entry remains pending at response time | Uses the existing functional `Result<T>.warnings` contract; no separate metadata envelope. |
| Automatic retry attempts are capped at 10 with exponential backoff capped at 1 hour | Provides bounded automatic recovery. |
| Entries requiring operator action move to `manual_replay_required` | Matches selected manual re-run durability policy. |
| Authorized operators may move unresolved entries to `intentionally_closed` with reason | Provides explicit terminal state without deleting audit evidence. |

## Audit Outbox Statuses

| Status | Terminal | Notes |
|---|---|---|
| `pending` | No | Waiting for dispatcher |
| `dispatching` | No | Active delivery attempt |
| `retry_scheduled` | No | Automatic retry scheduled with `nextAttemptAt` |
| `manual_replay_required` | No | Automatic retry exhausted or non-retryable |
| `dispatched` | Yes | U11 accepted event |
| `intentionally_closed` | Yes | Authorized close with reason |

## Performance Decision

| API Area | Target |
|---|---|
| `createProject` | End-to-end API P95 <= 500ms under MVP load |
| `listProjects` | End-to-end API P95 <= 500ms with cursor page size <= 100 |
| `getProjectDashboard` | End-to-end API P95 <= 500ms when dependent ports meet P95 <= 150ms or time out at 200ms |
| `getProjectDashboard` local work | U1 local processing P95 <= 250ms |
| Dashboard dependent sections | Exact 200ms timeout per section, run in parallel |
| `canAdvanceStage` / `advanceStage` | End-to-end API P95 <= 500ms; ApprovalGatePort and SecurityPolicyPort timeout at exact 200ms and fail closed |

## Compatibility Decision

| Contract | Compatibility Requirement |
|---|---|
| `packages/shared-types` | Package-level semantic version is the authoritative contract version |
| `Result<T>` | Breaking changes require package major version bump and migration note |
| `U1ErrorCode` | Removing or renaming codes is a breaking change |
| `ProjectDashboard` | Section shape must remain compatible; new sections are additive |
| `NextAction` | Existing target fields must not be removed during MVP |
| Workflow stage/gate names | Renames require explicit migration plan |
| `ProjectScope` and `WorkspaceScope` | Contract changes require migration note and dependent unit review |

## Migration Note Format

```markdown
## [packages/shared-types version] - [date]

- Change type: additive | breaking | removal | behavior
- Affected contracts:
- Summary:
- Required migration:
- Dependent units to review:
- Backward compatibility:
- Rollback notes:
```

## Deferred Decisions

| Decision | Deferred To | Reason |
|---|---|---|
| Auth.js vs Cognito | U1 NFR Design or Auth-owning unit design | U1 only needs provider-neutral scope contracts at NFR Requirements stage |
| Metrics backend/exporter | Infrastructure Design | Requires deployment/runtime choice |
| Alert rules and SLO dashboard | NFR Design or Infrastructure Design | User selected recovery/auditability over explicit MVP SLO |
| Exact CI workflow file layout | Code Generation / Build and Test | Depends on repository scaffold |

## Extension Compliance Summary

| Extension | Status | Notes |
|---|---|---|
| Security Baseline | Compliant | Validation, scope enforcement, safe errors, audit outbox, log safety, and security separation are reflected in tech stack choices. |
| Property-Based Testing | Compliant | `fast-check` is selected and concrete CI seed/reproducibility expectations are defined. |

