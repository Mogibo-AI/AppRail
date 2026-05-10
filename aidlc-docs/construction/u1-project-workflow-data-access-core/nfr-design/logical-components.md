# Logical Components - U1 Project, Workflow, and Data Access Core

## Purpose

This document defines the logical components needed to implement the U1 NFR Design. Components are grouped by package boundary and remain scoped to project metadata, workflow state, stage transition decisions, project-scoped data access, audit outbox durability, and dashboard contracts.

## Package Boundaries

```text
apps/web
  -> packages/auth-boundary
  -> packages/project-workflow
  -> packages/shared-types

packages/project-workflow
  -> packages/data-access
  -> packages/shared-types
  -> U4/U10/U11 ports

packages/data-access
  -> PostgreSQL via Prisma
```

| Package | Ownership |
|---|---|
| `packages/shared-types/` | U1 DTOs, `Result<T>`, `U1ErrorCode`, workflow enums, dashboard contracts, scope contracts |
| `packages/auth-boundary/` | Provider-neutral auth and scope resolution ports |
| `packages/project-workflow/` | U1 use cases, workflow transition decisions, dashboard aggregation, outbox dispatcher logic |
| `packages/data-access/` | Prisma-backed repositories, transaction wrapper, project-scoped query enforcement |
| `apps/web` | API route handlers and UI integration points consuming U1 contracts |

## Component Overview

| Component | Package | Responsibility |
|---|---|---|
| U1 API Adapter | `apps/web` | Validate HTTP shape, attach request ID, pass auth context to use cases |
| Auth Boundary | `packages/auth-boundary` | Resolve `Actor`, `WorkspaceScopeDecision`, and `ProjectScopeDecision` from provider-specific auth |
| Project Use Cases | `packages/project-workflow` | `createProject`, `listProjects`, `getProject`, `archiveProject` |
| Workflow Use Cases | `packages/project-workflow` | `getWorkflowState`, `canAdvanceStage`, `advanceStage`, `blockStage` |
| Dashboard Aggregator | `packages/project-workflow` | Compose `ProjectDashboard`, dependent sections, and `NextAction` |
| Workflow Metadata Cache | `packages/project-workflow` | Cache DB-backed stage and transition metadata for 300 seconds |
| Dependency Port Clients | `packages/project-workflow` | U4 job summary, U10 security summary/check, U11 approval/audit summary/check |
| Circuit Breaker Registry | `packages/project-workflow` | Track per-section dependency circuit state |
| Outbox Dispatcher | `packages/project-workflow` | Lease and deliver audit outbox entries to U11 AuditPort |
| Transaction Manager | `packages/data-access` | Execute one PostgreSQL transaction per mutation use case |
| Project Repositories | `packages/data-access` | Project, membership, workflow, metadata, and outbox persistence |
| Observability Adapter | `packages/project-workflow` and `apps/web` | Structured logs and OpenTelemetry-compatible metrics |
| PBT Generators | `packages/project-workflow/test/generators/` | Domain generators for property-based tests |

## Component Diagram

```text
                  +-----------------------+
                  |        apps/web       |
                  | API routes + U13 use  |
                  +-----------+-----------+
                              |
                              v
                  +-----------------------+
                  |    U1 API Adapter     |
                  | validation + request  |
                  +-----------+-----------+
                              |
                              v
        +---------------------+---------------------+
        |              Project Workflow             |
        | use cases + dashboard + transition logic  |
        +------+-------------+-------------+--------+
               |             |             |
               v             v             v
     +----------------+ +-----------+ +------------------+
     | Auth Boundary  | | U4/U10/U11| |  Data Access     |
     | scope decisions| | ports     | | repositories     |
     +----------------+ +-----------+ +---------+--------+
                                                |
                                                v
                                      +------------------+
                                      | PostgreSQL/Prisma|
                                      +------------------+
```

## Shared Type Contracts

`packages/shared-types` is the stable contract layer for U1.

Required exported contract groups:

| Contract Group | Examples |
|---|---|
| Result and errors | `Result<T>`, `U1ErrorCode`, `UserFacingWarning` |
| Project metadata | `Project`, `ProjectSummary`, `ProjectStatus`, `CreateProjectInput`, `CreateProjectResult` |
| Workspace/project scope | `WorkspaceScope`, `WorkspaceScopeDecision`, `ProjectScope`, `ProjectScopeDecision` |
| Workflow | `WorkflowState`, `WorkflowStageId`, `ApprovalGateType`, `StageTransitionDefinition`, `AdvanceDecision` |
| Dashboard | `ProjectDashboard`, `DashboardSection<T>`, `NextAction` |
| Audit refs | `AuditEventInput`, `AuditOutboxEntryRef`, outbox status refs |

Compatibility is tracked in `packages/shared-types/CHANGELOG.md` using package-level semantic versioning.

## Auth Boundary Component

`packages/auth-boundary` provides provider-neutral scope resolution.

```ts
type AuthBoundary = {
  getActor(context: AuthRequestContext): Promise<Result<Actor>>;
  getWorkspaceScope(input: ResolveWorkspaceScopeInput): Promise<Result<WorkspaceScopeDecision>>;
  getProjectScope(input: ResolveProjectScopeInput): Promise<Result<ProjectScopeDecision>>;
};
```

Design rules:

- Auth.js, Cognito, or any future provider objects do not enter `packages/project-workflow`.
- Use cases call Auth Boundary at entry.
- `WorkspaceScopeDecision` is required for project creation and listing.
- `ProjectScopeDecision` is required for project read, dashboard, workflow mutation, and archive operations.
- Denied project access must map to `PROJECT_NOT_FOUND_OR_DENIED` unless a more specific in-scope error is safe.

## Project Use Case Components

| Use Case | Scope Requirement | Transaction | Audit Outbox |
|---|---|---|---|
| `createProject` | `WorkspaceScope` with create permission | Yes | Required |
| `listProjects` | `WorkspaceScope` with list permission | No | Not required |
| `getProject` | `ProjectScope` with `operation = "read"` | No | Not required |
| `archiveProject` | `ProjectScope` with `operation = "archive"` | Yes | Required |

`createProject` logical flow:

1. Resolve `Actor`.
2. Resolve `WorkspaceScope`.
3. Validate and normalize input.
4. Allocate normalized name and slug within `(workspaceId, createdByActorId)`.
5. In one transaction, create `Project`, create initial `WorkflowState`, create initial membership, and write audit outbox.
6. Return `CreateProjectResult` with `AUDIT_DISPATCH_PENDING` warning when the outbox entry is pending.

`listProjects` logical flow:

1. Resolve `Actor`.
2. Resolve `WorkspaceScope`.
3. Validate limit and signed cursor.
4. Choose owner/builder query or viewer join query.
5. Return ordered page and next signed cursor.

## Workflow Use Case Components

| Use Case | Scope Requirement | Dependency Checks | Transaction |
|---|---|---|---|
| `getWorkflowState` | `ProjectScope` read | None | No |
| `canAdvanceStage` | `ProjectScope` mutate | Approval/security as needed | No |
| `advanceStage` | `ProjectScope` mutate | Approval/security as needed | Yes, after checks pass |
| `blockStage` | `ProjectScope` mutate | Optional security refs | Yes |

`canAdvanceStage` and `advanceStage` share the same decision engine.

Decision engine responsibilities:

- Read current `WorkflowState`.
- Read transition definition from Workflow Metadata Cache.
- Verify `expectedCurrentStage`.
- Resolve required approval gate from transition metadata.
- Evaluate U11 approval status when a gate is required.
- Evaluate U10 security blockers for the transition context.
- Return `AdvanceDecision` and `NextAction`.
- Fail closed on timeout, unavailable dependency, or missing metadata.

`advanceStage` mutates only when the decision is allowed.

## Workflow Metadata Cache Component

```ts
type WorkflowMetadataCache = {
  getStage(stageId: WorkflowStageId): Promise<Result<WorkflowStageDefinition>>;
  getTransition(fromStage: WorkflowStageId, toStage?: WorkflowStageId): Promise<Result<StageTransitionDefinition>>;
  refresh(): Promise<Result<void>>;
};
```

Design rules:

- Cache TTL is 300 seconds.
- Metadata comes from DB tables seeded from the MVP workflow definitions.
- Fixed TypeScript enums remain the public contract.
- Transition decisions require fresh cache or successful DB reload.
- Missing or unavailable metadata returns `WORKFLOW_METADATA_MISSING`.

## Dashboard Aggregator Component

The Dashboard Aggregator composes the U1-owned dashboard response.

Inputs:

- `ProjectScope` with `operation = "read"`.
- `ProjectSummary`.
- `WorkflowState`.
- Workflow metadata.
- U4/U10/U11 summary ports.

Outputs:

- `ProjectDashboard`.
- Section-level unavailable states.
- `NextAction` with route target and disabled state.
- Dashboard unavailable metrics.

Execution pattern:

1. Read project and workflow core state.
2. Read workflow metadata.
3. Start dependent section calls in parallel.
4. Apply exact 200ms timeout per dependent call.
5. Consult Circuit Breaker Registry before calling each dependency.
6. Map each dependency result to `DashboardSection<T>`.
7. Calculate `NextAction`.
8. Return dashboard if core U1 reads succeeded.

The dashboard response fails only for U1-owned core read/scope errors. U4/U10/U11 summary failures degrade only their sections.

## Circuit Breaker Registry Component

```ts
type CircuitBreakerKey =
  | "dashboard.recentJobs"
  | "dashboard.approval"
  | "dashboard.recentAuditEvents"
  | "dashboard.security";

type CircuitBreakerState = {
  key: CircuitBreakerKey;
  state: "closed" | "open" | "half_open";
  consecutiveFailures: number;
  openedAt?: ISODateTime;
  nextProbeAt?: ISODateTime;
};
```

Design rules:

- State is process-local in MVP.
- Failure threshold is 5 consecutive timeout/error results.
- Open duration is 30 seconds.
- One half-open probe determines close or reopen.
- Circuit state never controls workflow stage advancement; it only protects dashboard summary calls.

## Dependency Port Components

U1 depends on ports and refs only.

| Port | Owner Unit | Used By | Timeout |
|---|---|---|---|
| `JobSummaryPort` | U4 | Dashboard recent jobs, job-dependent next action | 200ms |
| `SecurityPolicyPort` | U10 | Dashboard security summary, transition blocker check | 200ms |
| `ApprovalGatePort` | U11 | Dashboard approval summary, transition approval check | 200ms |
| `AuditSummaryPort` | U11 | Dashboard recent audit events | 200ms |
| `AuditPort` | U11 | Outbox dispatch delivery | Dispatcher controlled, not request path |

Request-path dependency calls must return mapped `Result<T>` values and must not expose provider-specific errors.

## Data Access Components

`packages/data-access` owns Prisma-backed persistence boundaries.

Repositories:

| Repository | Scope Requirement |
|---|---|
| `WorkspaceMembershipRepository` | Workspace lookup by actor/workspace |
| `ProjectRepository` | `ProjectScope` for project-owned read/mutate; `WorkspaceScope` for create/list |
| `ProjectMembershipRepository` | `ProjectScope` or `WorkspaceScope` depending on operation |
| `WorkflowStateRepository` | `ProjectScope` |
| `WorkflowDefinitionRepository` | No project scope for global metadata reads; use typed enum keys |
| `AuditOutboxRepository` | `ProjectScope` for project entries; dispatcher lease uses operations context |

Required index alignment:

| Table | Index / Constraint |
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

## Signed Cursor Component

```ts
type CursorCodec = {
  encodeProjectListCursor(payload: ProjectListCursorPayload): Result<string>;
  decodeProjectListCursor(token: string, expected: ProjectListCursorExpectedContext): Result<ProjectListCursorPayload>;
};
```

Rules:

- Cursor token is base64url encoded.
- Payload is HMAC signed.
- Payload includes `workspaceId`, `actorId`, `filtersHash`, sort key, `lastUpdatedAt`, and `lastId`.
- Signature verification occurs before query construction.
- Invalid cursor does not expose internal details.

## Transaction Manager Component

```ts
type TransactionManager = {
  run<T>(input: TransactionInput<T>): Promise<Result<T>>;
};
```

Rules:

- Mutation use cases own transaction boundaries.
- Repository methods receive `TransactionContext`.
- Transaction failure maps to formal U1 data access errors.
- No mutation commits without required audit outbox entry.

## Audit Outbox Dispatcher Component

The U1 Outbox Dispatcher is a logical component in `packages/project-workflow`.

Responsibilities:

- Select eligible `pending` and due `retry_scheduled` entries.
- Lease entries by moving them to `dispatching`.
- Deliver events to U11 through `AuditPort`.
- Mark successful entries `dispatched`.
- Schedule retryable failures with exponential backoff capped at 1 hour.
- Move exhausted or non-retryable entries to `manual_replay_required`.
- Emit outbox status metrics and transition counters.

Outbox dispatch does not run inside the user request path.

```ts
type AuditOutboxDispatcher = {
  dispatchDueBatch(input: DispatchDueOutboxBatchInput): Promise<Result<DispatchDueOutboxBatchResult>>;
  rerun(input: ManualOutboxRerunInput): Promise<Result<AuditOutboxEntryRef>>;
  intentionallyClose(input: IntentionallyCloseOutboxInput): Promise<Result<AuditOutboxEntryRef>>;
};
```

Manual recovery contracts:

| Input | Required Fields |
|---|---|
| `ManualOutboxRerunInput` | `outboxEntryId`, `actor`, `reason` |
| `IntentionallyCloseOutboxInput` | `outboxEntryId`, `actor`, `reason`, `closedAt` |

Reason is required but must be treated as sensitive operational text. It must not be used as a metric label.

## Observability Components

| Component | Metrics | Logs |
|---|---|---|
| API middleware | `u1_api_latency_ms`, `u1_api_error_total` | request ID, operation, result, safe IDs |
| Use case wrapper | operation result and fail-closed counters | formal error code, workflow stage, gate type |
| Dashboard aggregator | `u1_dashboard_section_unavailable_total` | section, unavailable reason, retryable |
| Outbox dispatcher | `u1_audit_outbox_current`, `u1_audit_outbox_transitions_total` | outbox status, safe outbox ID |

Safe logging rules:

- Plaintext allowed: `requestId`, operation, formal error/warning code, workflow stage ID, gate type, outbox status.
- Opaque internal UUID/ULID IDs may be logged.
- Non-internal IDs must be hashed.
- Do not log project idea text, target users, project name, workspace name, actor display name, email, tokens, secrets, raw DB errors, or provider payloads.

## PBT Components

U1 domain generators live under `packages/project-workflow/test/generators/`.

| File | Exports |
|---|---|
| `project-generators.ts` | `arbProjectName`, `arbWorkspaceId` |
| `actor-scope-generators.ts` | `arbActor`, `arbWorkspaceScope`, `arbProjectScope` |
| `workflow-generators.ts` | `arbWorkflowStageSequence`, transition command generators |
| `dashboard-generators.ts` | `arbDashboardSection`, unavailable reason generators |
| `audit-outbox-generators.ts` | `arbAuditOutboxEntry`, lifecycle command generators |

Property tests consume these generators from package-level tests. Generators must avoid creating invalid states unless the property explicitly tests validation failure.

## Primary Flows

### Dashboard Flow

```text
API Adapter
  -> Auth Boundary resolves ProjectScope(read)
  -> Dashboard Aggregator reads Project + WorkflowState
  -> Workflow Metadata Cache loads transition/action metadata
  -> Circuit Breaker Registry checks dependency availability
  -> U4/U10/U11 summary ports run in parallel with 200ms timeout
  -> DashboardSection values are composed
  -> NextAction is calculated
  -> ProjectDashboard is returned
```

### Stage Advancement Flow

```text
API Adapter
  -> Auth Boundary resolves ProjectScope(mutate)
  -> Workflow Decision Engine reads WorkflowState
  -> Workflow Metadata Cache resolves transition
  -> ApprovalGatePort/SecurityPolicyPort checks run with 200ms timeout
  -> fail closed if required condition cannot be confirmed
  -> Transaction Manager updates WorkflowState and Project status
  -> AuditOutboxRepository writes outbox entry in same transaction
  -> Result includes AUDIT_DISPATCH_PENDING warning
```

### Outbox Dispatch Flow

```text
Outbox Dispatcher tick
  -> AuditOutboxRepository leases due entries
  -> AuditPort delivers event to U11
  -> AuditOutboxRepository records dispatched/retry/manual status
  -> Observability Adapter emits status and transition metrics
```

## Deferred to Later Stages

| Decision | Deferred Stage |
|---|---|
| Physical Prisma schema field names | Infrastructure Design / Code Generation |
| Cursor signing key storage | Infrastructure Design |
| Metrics exporter/backend | Infrastructure Design |
| Exact Next.js route handler layout | Code Generation |
| CI workflow file names | Code Generation / Build and Test |
| Auth.js versus Cognito concrete implementation | Auth-owning unit or Infrastructure Design |
