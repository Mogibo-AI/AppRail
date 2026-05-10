# NFR Design Patterns - U1 Project, Workflow, and Data Access Core

## Purpose

This document converts the approved U1 NFR Requirements into concrete non-functional design patterns. U1 remains scoped to project metadata, workflow state, stage transition decisions, project-scoped data access, audit outbox durability, and dashboard contracts.

## Source Decisions

| Area | Design Decision |
|---|---|
| Dashboard dependent section resilience | Timeout plus circuit breaker |
| Workflow transition dependency failure | No request retry; exact 200ms timeout and fail closed |
| Audit outbox dispatcher | U1-owned Outbox Dispatcher Component delivering to U11 AuditPort |
| Audit manual recovery boundary | U1 defines operation contract; UI/execution are delegated to U11/U13 |
| Viewer project listing | `ProjectMembership` join query |
| Cursor token | Opaque base64url JSON payload with HMAC signature |
| Workflow metadata | DB metadata table with process-local TTL cache |
| Scope enforcement | Use case entry resolves `WorkspaceScope` / `ProjectScope` |
| Auth boundary | Dedicated `packages/auth-boundary` provider-neutral package |
| Observability | API middleware plus use case wrapper |
| DTO compatibility | `packages/shared-types/CHANGELOG.md` migration notes |
| PBT organization | `packages/project-workflow/test/generators/` for U1 generators |

## NFR Traceability

| NFR Area | Pattern |
|---|---|
| Dashboard P95 500ms | Parallel fan-out, exact 200ms dependent port timeout, U1 local P95 250ms budget |
| Dashboard availability | Section-level degradation using `DashboardSection<T>` |
| Stage transition safety | Fail-closed dependency checks before mutation transaction starts |
| Audit durability | Transactional outbox write plus asynchronous dispatcher |
| Listing scalability | Cursor pagination with index-backed queries and signed cursor token |
| Scope enforcement | Use case entry scope resolution and repository-level `ProjectScope` requirement |
| Observability | Safe structured logs and OpenTelemetry-compatible metrics |
| DTO compatibility | Package-level semantic versioning and migration notes |
| PBT | Fast-check generators for pure U1 logic and state transitions |

## Dashboard Resilience Pattern

U1 uses a timeout plus circuit breaker pattern for dependent dashboard sections.

Dependent sections:

| Section | Owner Port | Dashboard Contract |
|---|---|---|
| `recentJobs` | U4 summary port | `DashboardSection<JobSummaryRef[]>` |
| `approval` | U11 approval summary port | `DashboardSection<ApprovalDashboardSummary>` |
| `recentAuditEvents` | U11 audit summary port | `DashboardSection<AuditSummaryRef[]>` |
| `security` | U10 security summary port | `DashboardSection<SecuritySummaryRef>` |

Design rules:

- U1 reads project and workflow core state first.
- U1 calls U4/U10/U11 dependent summary ports in parallel.
- Each dependent port call has an exact 200ms timeout.
- A timed-out or failed dependent port produces `DashboardSection.status = "unavailable"` for that section only.
- A dependent section failure does not fail `getProjectDashboard` when project and workflow reads succeed.
- Circuit breaker state is tracked per dependency and section, not globally.
- When a circuit is open, U1 skips the port call and immediately returns `DashboardSection.status = "unavailable"` with `unavailableReason = "port-unavailable"`.
- U1 does not return stale summaries as `available` in MVP.

Circuit breaker policy:

| Attribute | MVP Value |
|---|---|
| Failure threshold | 5 consecutive timeout/error results per section dependency |
| Open duration | 30 seconds |
| Half-open probe | 1 request |
| Success in half-open | Close circuit and reset failure count |
| Failure in half-open | Reopen for 30 seconds |
| State persistence | Process-local in MVP |

Unavailable reason mapping:

| Dependency Result | DashboardUnavailableReason | Retryable |
|---|---|---|
| 200ms timeout | `timeout` | `true` |
| Circuit open | `port-unavailable` | `true` |
| Dependency returns permission denial | `permission-denied` | `false` |
| Dependency not implemented in current environment | `not-implemented` | `false` |
| Unexpected mapped failure | `unknown` | `true` |

## Dashboard NextAction Dependency Pattern

`NextAction` is calculated by U1 and must be rendered by U13 without recomputing workflow rules.

| Condition | NextAction Behavior |
|---|---|
| Approval summary unavailable and current transition requires approval | `actionType = "wait"`, `disabled = true`, `disabledReason = "DEPENDENCY_UNAVAILABLE"` |
| Security summary unavailable and current transition requires blocker evaluation | `actionType = "wait"`, `disabled = true`, `disabledReason = "DEPENDENCY_UNAVAILABLE"` |
| U4 jobs unavailable and current action requires job status | `actionType = "wait"`, `disabled = true`, `disabledReason = "DEPENDENCY_UNAVAILABLE"` |
| U4 jobs unavailable and current action does not require job status | NextAction remains enabled when approval and security are known |
| Workflow metadata unavailable | `actionType = "wait"`, `disabled = true`, `disabledReason = "WORKFLOW_METADATA_MISSING"` |

The dashboard remains `ok: true` for section-level dependency failure, but the next action can be disabled when U1 cannot safely determine the next user action.

## Workflow Transition Fail-Closed Pattern

U1 uses a no-retry fail-closed pattern for `canAdvanceStage` and `advanceStage`.

Design rules:

- U1 resolves `ProjectScope` at use case entry before evaluating transition state.
- U1 reads workflow state and transition metadata before calling approval or security ports.
- ApprovalGatePort and SecurityPolicyPort calls have exact 200ms timeouts.
- If both approval and security checks are required, U1 runs them in parallel.
- U1 performs no request-path retry.
- U1 never uses cached approval or security decisions to advance a stage.
- U1 starts the mutation transaction only after transition checks are allowed.
- A blocked or unknown dependency prevents advancement and returns a formal `U1ErrorCode`.

Failure mapping:

| Scenario | Public Error Code | Decision Reason | Metric Reason |
|---|---|---|---|
| ApprovalGatePort timeout | `WORKFLOW_APPROVAL_REQUIRED` | `approval-required` | `approval-check-timeout` |
| ApprovalGatePort unavailable/error | `WORKFLOW_APPROVAL_REQUIRED` | `approval-required` | `approval-check-unavailable` |
| SecurityPolicyPort timeout | `WORKFLOW_BLOCKED_BY_SECURITY` | `security-blocker` | `security-check-timeout` |
| SecurityPolicyPort unavailable/error | `WORKFLOW_BLOCKED_BY_SECURITY` | `security-blocker` | `security-check-unavailable` |
| Transition metadata unavailable | `WORKFLOW_METADATA_MISSING` | `invalid-transition` | `workflow-metadata-unavailable` |

The user-facing text for timeout/unavailable cases must say that U1 could not safely confirm the required condition. It must not imply that a human rejected the approval or that U10 found a blocker unless the dependency explicitly returned that result.

## Workflow Metadata Cache Pattern

U1 uses fixed TypeScript enums plus DB-backed workflow metadata with a process-local TTL cache.

Design rules:

- `WorkflowStageId`, `ApprovalGateType`, and `WorkflowPhaseId` remain fixed enum contracts in `packages/shared-types`.
- DB metadata stores stage display names, sort order, next action labels, enabled flags, and transition definitions.
- The process-local metadata cache TTL is 300 seconds.
- On cache miss or expired cache, U1 reloads metadata from the database.
- For transition decisions, expired metadata must not be used if reload fails.
- For dashboard display, if metadata reload fails and no fresh cache exists, U1 returns a disabled `NextAction` with `WORKFLOW_METADATA_MISSING`.
- Metadata reload failure emits `u1_workflow_transition_fail_closed_total{reason="workflow-metadata-unavailable"}` when it affects a transition.

This keeps runtime stage labels configurable while preventing unsafe transition advancement from stale or missing transition definitions.

## Project Listing Scalability Pattern

U1 lists workspace-accessible projects using cursor pagination and role-aware query shapes.

Listing rules:

- `workspaceId` is required.
- Default limit is 25.
- Maximum limit is 100.
- Default listing excludes `archived`.
- Sort order is `updatedAt DESC, id DESC`.
- Workspace `owner` and `builder` list all projects in the workspace.
- Workspace `viewer` lists only projects with matching `ProjectMembership`.

Query pattern:

| Actor Workspace Role | Query Pattern |
|---|---|
| `owner` / `builder` | Query `Project` by `(workspaceId, status, updatedAt, id)` |
| `viewer` | Join `ProjectMembership` to `Project` and filter by `ProjectMembership.actorId` plus `Project.workspaceId` |

The viewer query must not first expose project IDs outside the actor's scope to application code. Repository methods return only scoped projects.

## Signed Cursor Token Pattern

U1 uses an opaque base64url token containing a JSON payload and an HMAC signature.

Cursor payload:

```ts
type ProjectListCursorPayload = {
  version: 1;
  workspaceId: ID;
  actorId: ID;
  sort: "updatedAt_desc_id_desc";
  filtersHash: string;
  lastUpdatedAt: ISODateTime;
  lastId: ID;
  issuedAt: ISODateTime;
};
```

Token rules:

- The public cursor token is opaque to callers.
- U1 verifies HMAC before using cursor values.
- Cursor verification failure returns `PROJECT_NOT_FOUND_OR_DENIED` or a validation error mapped by the API schema, without exposing token internals.
- Cursor replay across a different workspace, actor, sort, or filter set is rejected by comparing payload fields and `filtersHash`.
- The signing key is resolved through runtime configuration by Infrastructure Design; it must not be hard-coded.

## Scope Enforcement Pattern

Scope enforcement occurs at the use case entry boundary.

Design rules:

- API handlers authenticate the request and pass an auth context to the use case.
- Use cases call the auth boundary to resolve `Actor`, `WorkspaceScopeDecision`, or `ProjectScopeDecision`.
- `createProject` and `listProjects` require `WorkspaceScope`.
- `getProject`, `getProjectDashboard`, `canAdvanceStage`, `advanceStage`, `blockStage`, and `archiveProject` require `ProjectScope`.
- Repository methods for project-owned records require `ProjectScope` and `TransactionContext`.
- Repository methods reject records outside `ProjectScope.projectId` with `PROJECT_SCOPE_VIOLATION`.
- Archived projects allow `read` and deny `mutate` / `archive` with `PROJECT_ARCHIVED_READONLY`.

`packages/auth-boundary` owns provider-neutral contracts. Concrete Auth.js or Cognito integration remains deferred.

## Audit Outbox Pattern

U1 uses the transactional outbox pattern with a U1-owned asynchronous dispatcher.

Mutation rules:

- `createProject`, `advanceStage`, `blockStage`, and `archiveProject` write project/workflow changes and an audit outbox entry in one PostgreSQL transaction.
- Outbox write failure aborts the mutation and returns `AUDIT_OUTBOX_WRITE_FAILED`.
- Dispatch to U11 is fully asynchronous and never attempted before returning the API response.
- Successful mutating APIs return `AUDIT_DISPATCH_PENDING` in `Result<T>.warnings` when the created outbox entry is still pending.

Dispatcher rules:

- The U1 Outbox Dispatcher leases eligible entries and marks them `dispatching`.
- The dispatcher delivers mapped audit events to U11 through `AuditPort`.
- U11 acceptance marks the entry `dispatched`.
- Retryable failures become `retry_scheduled`.
- Non-retryable failures or exhausted retries become `manual_replay_required`.
- Authorized manual close marks the entry `intentionally_closed` and preserves the record.

Outbox lifecycle:

| From Status | To Status | Trigger |
|---|---|---|
| `pending` | `dispatching` | Dispatcher leases entry |
| `pending` | `intentionally_closed` | Authorized close |
| `dispatching` | `dispatched` | U11 accepts event |
| `dispatching` | `retry_scheduled` | Retryable delivery failure |
| `dispatching` | `manual_replay_required` | Non-retryable failure |
| `retry_scheduled` | `dispatching` | `nextAttemptAt` reached |
| `retry_scheduled` | `manual_replay_required` | Automatic retry budget exhausted |
| `retry_scheduled` | `intentionally_closed` | Authorized close |
| `manual_replay_required` | `pending` | Authorized manual re-run |
| `manual_replay_required` | `dispatching` | Authorized immediate manual re-run |
| `manual_replay_required` | `intentionally_closed` | Authorized close |

## Audit Recovery Operation Pattern

U1 defines operation contracts for manual audit recovery. U13 may render the controls and U11 may provide audit detail views, but U1 owns the outbox state transition contract.

| Operation | Required Scope | Result |
|---|---|---|
| List unresolved outbox entries | Authorized workspace/project operations actor | Paginated refs without raw payload secrets |
| Manual re-run | Authorized operations actor | `manual_replay_required` or `retry_scheduled` becomes `pending` or `dispatching` |
| Intentionally close | Authorized operations actor plus reason | Entry becomes `intentionally_closed` |

Manual recovery operations must write an audit outbox event where feasible. If recovery audit write fails, the recovery mutation fails with `AUDIT_OUTBOX_WRITE_FAILED`.

## Observability Pattern

U1 uses API middleware and use case wrappers.

| Layer | Responsibility |
|---|---|
| API middleware | Request ID, auth context presence, request latency, response result, safe structured logs |
| Use case wrapper | Operation-level metric, domain error code, fail-closed reason, outbox status transition |
| Repository wrapper | Data access errors mapped to formal codes; no raw DB/provider payloads in logs |
| Dispatcher wrapper | Outbox lifecycle metrics and safe delivery result logs |

Required metrics:

| Metric | Emitted By |
|---|---|
| `u1_api_latency_ms` | API middleware |
| `u1_api_error_total` | API middleware / use case wrapper |
| `u1_dashboard_section_unavailable_total` | Dashboard aggregator |
| `u1_audit_outbox_current` | Dispatcher / status poller |
| `u1_audit_outbox_transitions_total` | Outbox repository wrapper |
| `u1_workflow_transition_fail_closed_total` | Workflow transition use case |

Logs must follow the approved structured logging data classification. Metrics must not include actor, workspace, project, slug, name, email, or free-text labels.

## DTO Compatibility Pattern

`packages/shared-types` uses package-level semantic versioning.

Rules:

- Individual DTOs do not carry `schemaVersion`.
- `packages/shared-types/CHANGELOG.md` is the authoritative migration note location.
- Any change to `Result<T>`, `ProjectDashboard`, `NextAction`, `ProjectScope`, `WorkspaceScope`, workflow stage names, or gate names requires a migration note.
- Removing or renaming a public field or error code is a breaking change.
- Adding optional fields is additive when existing consumers can ignore them safely.

## PBT Pattern

U1 uses `fast-check` with domain-specific generators under `packages/project-workflow/test/generators/`.

Generator organization:

| Generator | Target Property Area |
|---|---|
| `arbProjectName` | normalization, slug generation, uniqueness key |
| `arbWorkspaceId` | workspace-scoped uniqueness and listing |
| `arbActor` | actor and scope decision inputs |
| `arbWorkspaceScope` | workspace role properties |
| `arbProjectScope` | read/mutate/archive access properties |
| `arbWorkflowStageSequence` | configured transition order |
| `arbDashboardSection` | available/unavailable mapping |
| `arbAuditOutboxEntry` | lifecycle transitions |

CI execution:

| Profile | Minimum `numRuns` | Seed |
|---|---:|---|
| PR CI | 100 per property | `U1_PBT_SEED`, default `20260509` |
| Nightly CI | 1,000 per property | Generated or date-based seed logged to CI output |

A failing PBT run must print the seed, `numRuns`, property name, and shrunk counterexample.

## Security Baseline Validation

| Security Area | NFR Design Handling |
|---|---|
| Access control | Use case entry resolves workspace/project scope before repository access |
| Tenant isolation | Repositories require `ProjectScope` and reject mismatched records |
| Safe errors | Public outputs use formal `U1ErrorCode` values |
| Sensitive logging | Log classification prohibits idea text, target users, names, secrets, tokens, and raw provider errors |
| Auditability | Mutations require transactional outbox write |
| Dependency safety | Transition dependencies fail closed |

## Non-Goals

- U1 does not define U4 job execution internals.
- U1 does not define U10 security classification internals.
- U1 does not define U11 approval storage or audit storage internals.
- U1 does not define U13 component implementation internals.
- U1 does not choose Auth.js versus Cognito in NFR Design.
- U1 does not choose metrics backend/exporter in NFR Design.
