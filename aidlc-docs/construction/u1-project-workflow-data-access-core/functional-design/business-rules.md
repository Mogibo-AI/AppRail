# Business Rules - U1 Project, Workflow, and Data Access Core

## Scope

These rules define only the functional behavior owned by U1: project metadata, workflow state, stage transition decisions, workspace/project scoped access, and project-scoped data access boundaries. U1 does not own U2 artifacts, U4 jobs, U10 security classification, U11 approval/audit storage, or U13 UI implementation.

## Project Creation Rules

| Rule ID | Rule | Failure Code |
|---|---|---|
| U1-BR-001 | A project must belong to exactly one workspace. | `WORKSPACE_ID_REQUIRED` |
| U1-BR-002 | Actor must have `owner` or `builder` workspace role to create a project. | `WORKSPACE_SCOPE_DENIED` |
| U1-BR-003 | Project creation requires project name, natural-language app idea, target users, and publication scope. | Field-specific codes below |
| U1-BR-004 | Blank project name fails with a field-level error. | `PROJECT_NAME_REQUIRED` |
| U1-BR-005 | Blank initial idea fails with a field-level error. | `PROJECT_IDEA_REQUIRED` |
| U1-BR-006 | Blank target users fails with a field-level error. | `PROJECT_TARGET_USERS_REQUIRED` |
| U1-BR-007 | Missing publication scope fails with a field-level error. | `PROJECT_PUBLICATION_SCOPE_REQUIRED` |
| U1-BR-008 | Project name is normalized by trimming, collapsing internal whitespace, and comparing case-insensitively for uniqueness. | `PROJECT_NAME_INVALID` |
| U1-BR-009 | Project name must be unique within `(workspaceId, createdByActorId)`. | `PROJECT_NAME_ALREADY_EXISTS` |
| U1-BR-010 | The default app type is `internal-crud-web-app`. | N/A |
| U1-BR-011 | Any app type other than `internal-crud-web-app` is rejected in MVP. | `PROJECT_TYPE_UNSUPPORTED` |
| U1-BR-012 | Unauthenticated external publication is not a valid U1 project creation choice for MVP. | `PROJECT_PUBLICATION_SCOPE_UNSUPPORTED` |
| U1-BR-013 | A project is created with `status = draft`. | N/A |
| U1-BR-014 | Project creation must write an audit outbox entry in the same transaction. | `AUDIT_OUTBOX_WRITE_FAILED` |

## Slug and Uniqueness Rules

| Rule ID | Rule | Failure Code |
|---|---|---|
| U1-BR-015 | Slug is generated from normalized project name. | `PROJECT_NAME_INVALID` |
| U1-BR-016 | Slug must be unique within `(workspaceId, createdByActorId)`. | `PROJECT_SLUG_ALREADY_EXISTS` |
| U1-BR-017 | If slug collides but normalized name is different, append the lowest available suffix: `base`, `base-2`, `base-3`. | `PROJECT_SLUG_ALREADY_EXISTS` |
| U1-BR-018 | Slug max length is 72 characters; suffix allocation must preserve the max length. | `PROJECT_SLUG_ALREADY_EXISTS` |

## Workspace and Project Scope Rules

| Rule ID | Rule | Failure Code |
|---|---|---|
| U1-BR-019 | `listProjects` requires `workspaceId`; MVP does not expose all-workspace listing. | `WORKSPACE_ID_REQUIRED` |
| U1-BR-020 | Workspace membership is required before listing or creating projects. | `WORKSPACE_MEMBERSHIP_REQUIRED` |
| U1-BR-021 | Project read/mutate/archive operations require a `ProjectScopeDecision`. | `PROJECT_SCOPE_REQUIRED` |
| U1-BR-022 | Scope checks require actor and operation. | `PROJECT_SCOPE_ACTOR_REQUIRED`, `PROJECT_SCOPE_OPERATION_REQUIRED` |
| U1-BR-023 | Read access is allowed for archived projects if actor has project access. | N/A |
| U1-BR-024 | Mutate/archive access is denied for archived projects. | `PROJECT_ARCHIVED_READONLY` |
| U1-BR-025 | Repository methods receive `ProjectScope`, not only `projectId`. | `PROJECT_SCOPE_REQUIRED` |
| U1-BR-026 | Repository methods must reject records outside `ProjectScope.projectId`. | `PROJECT_SCOPE_VIOLATION` |

## Project Lifecycle Rules

| Rule ID | Rule | Notes |
|---|---|---|
| U1-BR-027 | Allowed project statuses are `draft`, `in_progress`, `deployed`, `paused`, and `archived`. | Fixed enum in MVP |
| U1-BR-028 | `draft` means the project exists and is ready for requirements work. | Initial state |
| U1-BR-029 | `in_progress` means the project has active AI-DLC work after initial setup. | Requirements through deployment pre-check |
| U1-BR-030 | `deployed` means at least one successful Amplify deployment result exists. | U9 signals through a workflow result/ref |
| U1-BR-031 | `paused` means work is intentionally stopped but can later resume. | Post-MVP UI controls may expand this |
| U1-BR-032 | `archived` means the project is no longer mutable. | Physical deletion is not allowed |
| U1-BR-033 | Archive is idempotent. | Repeating archive does not create a second state mutation |
| U1-BR-034 | Archived projects remain readable for history and audit. | Required for traceability |

## Workflow Initialization Rules

| Rule ID | Rule | Failure Code |
|---|---|---|
| U1-BR-035 | Workflow state must exist for every active project. | `WORKFLOW_STATE_MISSING` |
| U1-BR-036 | `workspace-detection` is completed by U1 during project creation. | N/A |
| U1-BR-037 | The first user-visible stage for a new project is `requirements-analysis` with status `ready`. | N/A |
| U1-BR-038 | Phase and stage identifiers are fixed enums in MVP. | `WORKFLOW_STAGE_UNKNOWN` |
| U1-BR-039 | Stage display name, order, and next action label may be read from metadata tables. | `WORKFLOW_METADATA_MISSING` |

## Workflow Transition Table

| From Stage | To Stage | Required Gate | From Status Required | To Status On Advance | Project Status Update |
|---|---|---|---|---|---|
| `workspace-detection` | `requirements-analysis` | None | `completed` | `ready` | `draft` |
| `requirements-analysis` | `user-stories` | `requirements` | `completed` | `ready` | `in_progress` |
| `user-stories` | `workflow-planning` | None | `completed` | `ready` | `in_progress` |
| `workflow-planning` | `application-design` | None | `completed` | `ready` | `in_progress` |
| `application-design` | `units-generation` | `design` | `completed` | `ready` | `in_progress` |
| `units-generation` | `functional-design` | None | `completed` | `ready` | `in_progress` |
| `functional-design` | `nfr-requirements` | None | `completed` | `ready` | `in_progress` |
| `nfr-requirements` | `nfr-design` | None | `completed` | `ready` | `in_progress` |
| `nfr-design` | `infrastructure-design` | None | `completed` | `ready` | `in_progress` |
| `infrastructure-design` | `code-generation` | None | `completed` | `ready` | `in_progress` |
| `code-generation` | `test-execution` | `implementation-diff` | `completed` | `ready` | `in_progress` |
| `test-execution` | `deployment-precheck` | `test-result` | `completed` | `ready` | `in_progress` |
| `deployment-precheck` | `amplify-deployment` | `deployment` | `completed` | `ready` | `in_progress` |
| `amplify-deployment` | `operations` | None | `completed` | `ready` | `deployed` |

## Workflow Transition Rules

| Rule ID | Rule | Failure Code |
|---|---|---|
| U1-BR-040 | Stage advancement must follow the configured transition table. | `WORKFLOW_TRANSITION_INVALID` |
| U1-BR-041 | A transition requiring approval cannot advance until U11 reports the required `GateRequirement` as approved. | `WORKFLOW_APPROVAL_REQUIRED` |
| U1-BR-042 | A transition cannot advance while U10 reports blocking findings for the transition context. | `WORKFLOW_BLOCKED_BY_SECURITY` |
| U1-BR-043 | U1 must not provide admin override for missing approval gates in MVP. | N/A |
| U1-BR-044 | If no next stage exists, advancement fails without mutation. | `WORKFLOW_ALREADY_AT_FINAL_STAGE` |
| U1-BR-045 | Every successful stage advancement writes an audit outbox entry in the same transaction. | `AUDIT_OUTBOX_WRITE_FAILED` |
| U1-BR-046 | Failed advancement caused by missing approval or blockers should enqueue a non-critical audit event when U11 is available, but failure to enqueue that non-critical event must not mask the original decision. | N/A |

## Approval Gate Dependency Rules

| Rule ID | Gate Type | Transition Controlled | U1 Responsibility | U11 Responsibility |
|---|---|---|---|---|
| U1-BR-047 | `requirements` | `requirements-analysis` -> `user-stories` | Ask `ApprovalGatePort` if approved | Store and evaluate requirement approval |
| U1-BR-048 | `design` | `application-design` -> `units-generation` | Ask `ApprovalGatePort` if approved | Store and evaluate design approval |
| U1-BR-049 | `implementation-diff` | `code-generation` -> `test-execution` | Ask `ApprovalGatePort` if approved | Store and evaluate implementation diff approval |
| U1-BR-050 | `test-result` | `test-execution` -> `deployment-precheck` | Ask `ApprovalGatePort` if approved | Store and evaluate test result approval |
| U1-BR-051 | `deployment` | `deployment-precheck` -> `amplify-deployment` | Ask `ApprovalGatePort` if approved | Store and evaluate deployment approval |

## Data Access Rules

| Rule ID | Rule | Failure Code |
|---|---|---|
| U1-BR-052 | Transaction boundaries are defined at use case level. | `TRANSACTION_FAILED` |
| U1-BR-053 | Repository methods receive a transaction context rather than opening their own transaction by default. | `TRANSACTION_CONTEXT_REQUIRED` |
| U1-BR-054 | Every repository query for project-owned data requires `ProjectScope`. | `PROJECT_SCOPE_REQUIRED` |
| U1-BR-055 | Data access must use parameterized Prisma operations and must not expose raw SQL string concatenation as a normal repository path. | `UNSAFE_QUERY_REJECTED` |
| U1-BR-056 | Shared repositories must return domain results, not raw provider-specific database errors. | `DATA_ACCESS_ERROR` |

## Audit Failure Rules

| Rule ID | Rule | API Behavior |
|---|---|---|
| U1-BR-057 | Mutating use cases must write an audit outbox entry within the same transaction as the project/workflow mutation. | If outbox write fails, fail the transaction with `AUDIT_OUTBOX_WRITE_FAILED` |
| U1-BR-058 | Dispatching the audit outbox to U11 after transaction commit is retryable. | Dispatch failure does not fail an already successful API response |
| U1-BR-059 | Successful API responses may include `AUDIT_DISPATCH_PENDING` warning when dispatch is pending or retrying. | API remains successful with warning |

## Dashboard Rules

| Rule ID | Rule |
|---|---|
| U1-BR-060 | Dashboard must include current phase, current stage, project status, and `NextAction`. |
| U1-BR-061 | Approval, recent jobs, recent audit events, and security sections must use `DashboardSection<T>`. |
| U1-BR-062 | If U4/U10/U11 ports are unavailable, only that section is marked `status: "unavailable"` with `unavailableReason`; the dashboard response remains `ok: true`. |
| U1-BR-063 | Dashboard must not expose raw technical logs or provider-specific error payloads. |
| U1-BR-064 | U13 must use `NextAction.target.href` and must not recompute workflow transition rules. |

## Validation Rules

| Field | Required | Validation | Failure Code |
|---|---|---|---|
| `workspaceId` | Yes | Valid ID, actor has workspace access | `WORKSPACE_ID_REQUIRED`, `WORKSPACE_SCOPE_DENIED` |
| `name` | Yes | 1 to 80 characters after normalization, no control characters | `PROJECT_NAME_REQUIRED`, `PROJECT_NAME_INVALID` |
| `initialIdeaText` | Yes | 1 to 4000 characters after trimming | `PROJECT_IDEA_REQUIRED` |
| `targetUsers` | Yes | 1 to 1000 characters after trimming | `PROJECT_TARGET_USERS_REQUIRED` |
| `publicationScope` | Yes | One of allowed MVP values | `PROJECT_PUBLICATION_SCOPE_REQUIRED`, `PROJECT_PUBLICATION_SCOPE_UNSUPPORTED` |
| `appType` | Yes | Defaults to `internal-crud-web-app`; unsupported values rejected | `PROJECT_TYPE_UNSUPPORTED` |
| `actor.id` | Yes | Valid ID | `PROJECT_SCOPE_ACTOR_REQUIRED` |
| `operation` | Yes for scope checks | `read`, `mutate`, or `archive` | `PROJECT_SCOPE_OPERATION_REQUIRED` |

## Error Handling Rules

| Rule ID | Rule |
|---|---|
| U1-BR-065 | Every public method returns discriminated `Result<T>`. |
| U1-BR-066 | Field validation uses the field-specific error codes listed in Formal U1 Error Codes. |
| U1-BR-067 | Internal database or provider details must not be returned to UI callers. |
| U1-BR-068 | Authorization failures must not reveal whether a project exists outside the actor's scope. |

## Security Rule Compliance

| Security Rule | U1 Functional Design Status | Notes |
|---|---|---|
| SECURITY-05 | Compliant | Required fields, length bounds, enum allowlists, and repository safety rules are defined |
| SECURITY-08 | Compliant | Workspace and project scope enforcement is required before access |
| SECURITY-09 | Compliant at design level | Production error details are not exposed through public `Result<T>` errors |
| SECURITY-11 | Compliant | Security-critical policy logic is isolated behind U10; U1 does not scatter blocker classification |
| SECURITY-13 | Compliant | Project and workflow mutations write durable audit outbox entries |
| SECURITY-01, SECURITY-02, SECURITY-03, SECURITY-04, SECURITY-06, SECURITY-07, SECURITY-10, SECURITY-12, SECURITY-14 | N/A for U1 Functional Design | These are addressed in NFR, Infrastructure Design, Code Generation, or Build and Test |

## PBT Rule Compliance

| PBT Rule | U1 Functional Design Status | Notes |
|---|---|---|
| PBT-01 | Compliant | Testable properties are identified below and in `business-logic-model.md` |
| PBT-02 | N/A | No business-owned round-trip pair in U1 |
| PBT-03 | Compliant | Invariants are defined for workflow, project status, uniqueness, dashboard availability, and scope |
| PBT-07 | Planned | Domain-specific generators are required during code generation |
| PBT-08 | Planned | Shrinking and seed reproducibility are required during code generation/build |
| PBT-09 | Deferred | Framework selection belongs to NFR Requirements |

## Testable Properties for Code Generation

| Property ID | Function/Area | Category | Property |
|---|---|---|---|
| U1-PROP-001 | `normalizeProjectName` | Idempotence | Applying normalization twice equals applying it once |
| U1-PROP-002 | `createProjectSlug` | Invariant | Valid input always produces lowercase URL-safe slug within max length |
| U1-PROP-003 | `allocateProjectSlug` | Invariant | Slug collision resolution chooses the lowest available suffix and preserves max length |
| U1-PROP-004 | `projectUniquenessKey` | Invariant | Names equivalent after normalization produce identical keys in same workspace/actor scope |
| U1-PROP-005 | `canAdvanceStage` | Invariant | Missing required approval always denies advancement |
| U1-PROP-006 | `canAdvanceStage` | Invariant | Blocking security finding always denies advancement |
| U1-PROP-007 | `advanceStage` | Stateful | Valid stage sequences never skip configured order |
| U1-PROP-008 | `archiveProject` | Idempotence | Repeating archive yields the same final observable state |
| U1-PROP-009 | `getProjectDashboard` | Invariant | Dashboard current phase/stage equals persisted workflow state |
| U1-PROP-010 | `getProjectDashboard` | Invariant | Unavailable dependent ports produce unavailable sections, not dashboard failure |
| U1-PROP-011 | Project repository methods | Invariant | Project-owned repository calls cannot execute without `ProjectScope` |

## Blocking Findings

No blocking Security Baseline or PBT findings are present in this revised U1 Functional Design.
