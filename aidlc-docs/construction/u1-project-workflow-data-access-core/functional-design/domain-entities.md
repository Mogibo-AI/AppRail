# Domain Entities - U1 Project, Workflow, and Data Access Core

## Entity Overview

| Entity | Owned By | Purpose |
|---|---|---|
| `Workspace` | U1 | Groups projects under a user/team boundary |
| `WorkspaceMembership` | U1 | Defines actor access to a workspace |
| `WorkspaceScope` | U1 | Validated workspace access context |
| `Project` | U1 | Top-level development project and lifecycle metadata |
| `ProjectMembership` | U1 | Actor-to-project access boundary |
| `ProjectCreationSeed` | U1 transient / U2 consumer | Carries initial idea data to U2 without making U1 the authoritative idea owner |
| `WorkflowState` | U1 | Current AI-DLC phase/stage for a project |
| `WorkflowPhaseDefinition` | U1 | Fixed phase IDs with display metadata |
| `WorkflowStageDefinition` | U1 | Fixed stage IDs with display metadata and ordering |
| `StageTransitionDefinition` | U1 | Allowed stage transition and required approval gate |
| `AdvanceDecision` | U1 | Result of checking whether transition is allowed |
| `GateRequirement` | U1 contract / U11 implementation | Required approval gate reference for a transition |
| `ProjectDashboard` | U1 | Aggregated read model for GUI |
| `ProjectScope` | U1 | Validated access context for project-owned persistence |
| `TransactionContext` | U1 | Use case transaction context passed to repositories |
| `AuditOutboxEntryRef` | U1 contract / U11 implementation | Durable audit dispatch reference |
| `DomainEvent` | U1 shared | Event contract used by other units |

## Core Types

```ts
type ID = string;
type ISODateTime = string;

type UnitId =
  | "U1" | "U2" | "U3" | "U4" | "U5" | "U6" | "U7"
  | "U8" | "U9" | "U10" | "U11" | "U12" | "U13";

type Actor = {
  id: ID;
  displayName: string;
  role: "creator" | "operator" | "reviewer" | "system";
};

type Result<T> =
  | { ok: true; value: T; warnings?: UserFacingWarning[] }
  | { ok: false; error: UserFacingError };

type UserFacingWarning = {
  code: U1WarningCode;
  message: string;
  nextAction?: string;
};

type UserFacingError = {
  code: U1ErrorCode;
  message: string;
  fieldErrors?: FieldError[];
  technicalDetailsRef?: ID;
  nextAction?: string;
};

type FieldError = {
  field: string;
  code: U1ErrorCode;
  message: string;
};

type U1WarningCode =
  | "DASHBOARD_SECTION_UNAVAILABLE"
  | "AUDIT_DISPATCH_PENDING";
```

`Result<T>` is a discriminated union. `ok: true` always carries `value`; `ok: false` always carries `error`.

## Formal U1 Error Codes

| Error Code | Usage |
|---|---|
| `WORKSPACE_ID_REQUIRED` | `workspaceId` is missing |
| `WORKSPACE_SCOPE_DENIED` | Actor cannot access the workspace for the requested operation |
| `WORKSPACE_MEMBERSHIP_REQUIRED` | Actor has no workspace membership |
| `PROJECT_NAME_REQUIRED` | Project name is blank |
| `PROJECT_NAME_INVALID` | Project name violates format or length rules |
| `PROJECT_NAME_ALREADY_EXISTS` | Normalized project name already exists in the workspace/actor scope |
| `PROJECT_SLUG_ALREADY_EXISTS` | Slug collision cannot be resolved within allowed suffix attempts |
| `PROJECT_IDEA_REQUIRED` | Initial idea text is blank |
| `PROJECT_TARGET_USERS_REQUIRED` | Target users are blank |
| `PROJECT_PUBLICATION_SCOPE_REQUIRED` | Publication scope is missing |
| `PROJECT_PUBLICATION_SCOPE_UNSUPPORTED` | Publication scope is not supported by MVP |
| `PROJECT_TYPE_UNSUPPORTED` | App type is not supported by MVP |
| `PROJECT_NOT_FOUND_OR_DENIED` | Project does not exist or actor cannot access it |
| `PROJECT_ARCHIVED_READONLY` | Mutation attempted against an archived project |
| `PROJECT_SCOPE_REQUIRED` | Project scoped repository call lacks `ProjectScope` |
| `PROJECT_SCOPE_ACTOR_REQUIRED` | Project scope check lacks actor |
| `PROJECT_SCOPE_OPERATION_REQUIRED` | Project scope check lacks requested operation |
| `PROJECT_SCOPE_MUTATION_DENIED` | Actor has read access but cannot mutate |
| `PROJECT_SCOPE_VIOLATION` | Query attempted outside validated project scope |
| `WORKFLOW_STATE_MISSING` | Project has no workflow state |
| `WORKFLOW_STAGE_UNKNOWN` | Stage identifier is not known |
| `WORKFLOW_METADATA_MISSING` | Stage metadata is missing |
| `WORKFLOW_TRANSITION_INVALID` | Requested transition is not configured |
| `WORKFLOW_APPROVAL_REQUIRED` | Required approval gate is not approved |
| `WORKFLOW_BLOCKED_BY_SECURITY` | U10 reports blocking findings |
| `WORKFLOW_ALREADY_AT_FINAL_STAGE` | No next stage exists |
| `TRANSACTION_FAILED` | Use case transaction failed |
| `TRANSACTION_CONTEXT_REQUIRED` | Repository mutation lacks transaction context |
| `UNSAFE_QUERY_REJECTED` | Unsafe data access path rejected |
| `DATA_ACCESS_ERROR` | Data access error after safe mapping |
| `AUDIT_OUTBOX_WRITE_FAILED` | Durable audit outbox write failed inside transaction |

```ts
type U1ErrorCode =
  | "WORKSPACE_ID_REQUIRED"
  | "WORKSPACE_SCOPE_DENIED"
  | "WORKSPACE_MEMBERSHIP_REQUIRED"
  | "PROJECT_NAME_REQUIRED"
  | "PROJECT_NAME_INVALID"
  | "PROJECT_NAME_ALREADY_EXISTS"
  | "PROJECT_SLUG_ALREADY_EXISTS"
  | "PROJECT_IDEA_REQUIRED"
  | "PROJECT_TARGET_USERS_REQUIRED"
  | "PROJECT_PUBLICATION_SCOPE_REQUIRED"
  | "PROJECT_PUBLICATION_SCOPE_UNSUPPORTED"
  | "PROJECT_TYPE_UNSUPPORTED"
  | "PROJECT_NOT_FOUND_OR_DENIED"
  | "PROJECT_ARCHIVED_READONLY"
  | "PROJECT_SCOPE_REQUIRED"
  | "PROJECT_SCOPE_ACTOR_REQUIRED"
  | "PROJECT_SCOPE_OPERATION_REQUIRED"
  | "PROJECT_SCOPE_MUTATION_DENIED"
  | "PROJECT_SCOPE_VIOLATION"
  | "WORKFLOW_STATE_MISSING"
  | "WORKFLOW_STAGE_UNKNOWN"
  | "WORKFLOW_METADATA_MISSING"
  | "WORKFLOW_TRANSITION_INVALID"
  | "WORKFLOW_APPROVAL_REQUIRED"
  | "WORKFLOW_BLOCKED_BY_SECURITY"
  | "WORKFLOW_ALREADY_AT_FINAL_STAGE"
  | "TRANSACTION_FAILED"
  | "TRANSACTION_CONTEXT_REQUIRED"
  | "UNSAFE_QUERY_REJECTED"
  | "DATA_ACCESS_ERROR"
  | "AUDIT_OUTBOX_WRITE_FAILED";
```

## Workspace Access Model

```ts
type WorkspaceRole = "owner" | "builder" | "viewer";

type WorkspaceOperation =
  | "create_project"
  | "list_projects"
  | "read_workspace";

type Workspace = {
  id: ID;
  name: string;
  createdByActorId: ID;
  createdAt: ISODateTime;
  updatedAt: ISODateTime;
};

type WorkspaceMembership = {
  workspaceId: ID;
  actorId: ID;
  role: WorkspaceRole;
  createdAt: ISODateTime;
};

type WorkspaceScope = {
  workspaceId: ID;
  actorId: ID;
  role: WorkspaceRole;
  allowedOperations: WorkspaceOperation[];
};

type WorkspaceScopeDecision =
  | { allowed: true; scope: WorkspaceScope }
  | {
      allowed: false;
      workspaceId?: ID;
      operation: WorkspaceOperation;
      reason: "workspace-not-found-or-denied" | "membership-required" | "operation-denied";
      code: U1ErrorCode;
    };
```

**Rules**

- `owner` and `builder` can create projects.
- `viewer` can list and read projects but cannot create projects.
- `listProjects` must receive a `workspaceId`; cross-workspace listing is not an MVP U1 API.

## Project

```ts
type ApplicationType =
  | "internal-crud-web-app"
  | "future-unsupported";

type ApplicationTypeSupportStatus =
  | "supported-in-mvp"
  | "blocked-in-mvp";

type PublicationScope =
  | "internal-only"
  | "restricted-external";

type ProjectStatus =
  | "draft"
  | "in_progress"
  | "deployed"
  | "paused"
  | "archived";

type Project = {
  id: ID;
  workspaceId: ID;
  name: string;
  normalizedName: string;
  slug: string;
  appType: ApplicationType;
  appTypeSupportStatus: ApplicationTypeSupportStatus;
  publicationScope: PublicationScope;
  targetUsersSummary: string;
  status: ProjectStatus;
  createdByActorId: ID;
  createdAt: ISODateTime;
  updatedAt: ISODateTime;
  archivedAt?: ISODateTime;
};
```

**Ownership Boundary**

U1 owns project metadata, lifecycle, slug, scope, and workflow state. U2 owns the authoritative idea, requirements, and artifacts. U1 accepts `initialIdeaText` during creation only to validate the start of the project and produce `ProjectCreationSeed`.

## Project Creation DTOs

```ts
type CreateProjectInput = {
  workspaceId: ID;
  name: string;
  initialIdeaText: string;
  targetUsers: string;
  publicationScope: PublicationScope;
  appType?: ApplicationType;
};

type CreateProjectResult = {
  project: Project;
  workflow: WorkflowState;
  seed: ProjectCreationSeed;
  audit: AuditOutboxEntryRef;
};

type ListProjectsInput = {
  workspaceId: ID;
  status?: ProjectStatus;
  limit?: number;
  cursor?: string;
};
```

## Slug and Uniqueness

```ts
type ProjectSlug = string;

type ProjectUniquenessKey = {
  workspaceId: ID;
  createdByActorId: ID;
  normalizedName: string;
};

type ProjectSlugKey = {
  workspaceId: ID;
  createdByActorId: ID;
  slug: ProjectSlug;
};
```

**Rules**

- `normalizedName` enforces user/workspace-level duplicate names.
- `slug` is generated from `normalizedName`.
- `slug` must be unique within `(workspaceId, createdByActorId)`.
- If the base slug collides but `normalizedName` does not, U1 appends the lowest available numeric suffix: `base`, `base-2`, `base-3`, and so on.
- Slug max length is 72 characters. When adding a suffix, U1 truncates the base part to keep the full slug within the limit.
- If no suffix can be allocated within implementation limits, return `PROJECT_SLUG_ALREADY_EXISTS`.

## ProjectCreationSeed

```ts
type ProjectCreationSeed = {
  projectId: ID;
  initialIdeaText: string;
  targetUsers: string;
  publicationScope: PublicationScope;
  createdByActorId: ID;
  createdAt: ISODateTime;
};
```

## ProjectMembership

```ts
type ProjectRole = "owner" | "editor" | "viewer";

type ProjectMembership = {
  projectId: ID;
  actorId: ID;
  role: ProjectRole;
  createdAt: ISODateTime;
};
```

## WorkflowState

```ts
type WorkflowPhaseId =
  | "inception"
  | "construction"
  | "operations";

type WorkflowStageId =
  | "workspace-detection"
  | "requirements-analysis"
  | "user-stories"
  | "workflow-planning"
  | "application-design"
  | "units-generation"
  | "functional-design"
  | "nfr-requirements"
  | "nfr-design"
  | "infrastructure-design"
  | "code-generation"
  | "test-execution"
  | "deployment-precheck"
  | "amplify-deployment"
  | "operations";

type WorkflowStageStatus =
  | "ready"
  | "in_progress"
  | "waiting_for_user"
  | "completed"
  | "blocked"
  | "skipped";

type WorkflowState = {
  projectId: ID;
  currentPhase: WorkflowPhaseId;
  currentStage: WorkflowStageId;
  currentStageStatus: WorkflowStageStatus;
  completedStages: WorkflowStageId[];
  skippedStages: WorkflowStageId[];
  updatedAt: ISODateTime;
};
```

**Initial Workflow State**

`workspace-detection` is a system bootstrap stage. For a new user-created project, U1 marks `workspace-detection` as completed during project creation and sets the first user-visible current stage to `requirements-analysis`.

```ts
const initialWorkflowState: Pick<WorkflowState, "currentPhase" | "currentStage" | "currentStageStatus" | "completedStages"> = {
  currentPhase: "inception",
  currentStage: "requirements-analysis",
  currentStageStatus: "ready",
  completedStages: ["workspace-detection"],
};
```

## Workflow Definitions

```ts
type ApprovalGateType =
  | "requirements"
  | "design"
  | "implementation-diff"
  | "test-result"
  | "deployment";

type WorkflowPhaseDefinition = {
  id: WorkflowPhaseId;
  displayName: string;
  sortOrder: number;
  enabledInMvp: boolean;
};

type WorkflowStageDefinition = {
  id: WorkflowStageId;
  phaseId: WorkflowPhaseId;
  displayName: string;
  sortOrder: number;
  nextActionLabel: string;
  enabledInMvp: boolean;
};

type StageTransitionDefinition = {
  fromStage: WorkflowStageId;
  toStage: WorkflowStageId;
  requiredGate?: ApprovalGateType;
  fromStatusRequired: WorkflowStageStatus;
  toStatusOnAdvance: WorkflowStageStatus;
  projectStatusOnAdvance?: ProjectStatus;
};

type GateRequirement = {
  gateType: ApprovalGateType;
  projectId: ID;
  transition: Pick<StageTransitionDefinition, "fromStage" | "toStage">;
  requiredDecision: "approved";
  ownerUnit: "U11";
};
```

## StageTransition and AdvanceDecision

```ts
type StageTransition = {
  projectId: ID;
  fromPhase: WorkflowPhaseId;
  fromStage: WorkflowStageId;
  toPhase: WorkflowPhaseId;
  toStage: WorkflowStageId;
};

type AdvanceDecisionReason =
  | "allowed"
  | "approval-required"
  | "security-blocker"
  | "invalid-transition"
  | "project-archived"
  | "project-scope-denied"
  | "final-stage";

type AdvanceDecision = {
  projectId: ID;
  transition: StageTransition;
  allowed: boolean;
  reason: AdvanceDecisionReason;
  requiredGate?: GateRequirement;
  blockerRefs?: ID[];
  nextAction: NextAction;
};

type AdvanceStageInput = {
  projectId: ID;
  expectedCurrentStage: WorkflowStageId;
  requestedNextStage?: WorkflowStageId;
};

type BlockStageInput = {
  projectId: ID;
  stage: WorkflowStageId;
  reason: AdvanceDecisionReason;
  blockerRefs?: ID[];
};
```

## ProjectDashboard

```ts
type DashboardUnavailableReason =
  | "port-unavailable"
  | "timeout"
  | "not-implemented"
  | "permission-denied"
  | "unknown";

type DashboardSection<T> =
  | {
      status: "available";
      data: T;
      updatedAt: ISODateTime;
    }
  | {
      status: "unavailable";
      unavailableReason: DashboardUnavailableReason;
      message: string;
      retryable: boolean;
      updatedAt?: ISODateTime;
    };

type ProjectDashboard = {
  project: ProjectSummary;
  workflow: WorkflowState;
  nextAction: NextAction;
  approval: DashboardSection<ApprovalDashboardSummary>;
  recentJobs: DashboardSection<JobSummaryRef[]>;
  recentAuditEvents: DashboardSection<AuditSummaryRef[]>;
  security: DashboardSection<SecuritySummaryRef>;
};
```

U1 owns the dashboard contract and section availability shape. U4/U10/U11 own the actual job, security, approval, and audit details.

```ts
type ProjectSummary = {
  id: ID;
  workspaceId: ID;
  name: string;
  slug: string;
  status: ProjectStatus;
  appType: ApplicationType;
  publicationScope: PublicationScope;
};

type NextActionType =
  | "answer_questions"
  | "review"
  | "approve"
  | "run_job"
  | "configure"
  | "wait"
  | "blocked";

type NextActionTarget = {
  unit: UnitId;
  stage: WorkflowStageId;
  refType: "project" | "stage" | "gate" | "job" | "artifact" | "security-finding";
  refId?: ID;
  href: string;
};

type NextAction = {
  label: string;
  actionType: NextActionType;
  target: NextActionTarget;
  disabled: boolean;
  disabledReason?: U1ErrorCode | "DEPENDENCY_UNAVAILABLE";
  requiredGate?: ApprovalGateType;
};

type ApprovalDashboardSummary = {
  hasPendingApproval: boolean;
  pendingGate?: ApprovalGateType;
  pendingGateRef?: ID;
};

type JobSummaryRef = {
  id: ID;
  type: string;
  status: "queued" | "running" | "succeeded" | "failed" | "cancelled";
  progressLabel: string;
  updatedAt: ISODateTime;
};

type AuditSummaryRef = {
  id: ID;
  action: string;
  actorDisplayName: string;
  createdAt: ISODateTime;
};

type SecuritySummaryRef = {
  hasBlockers: boolean;
  blockerCount: number;
  warningCount: number;
  blockerRefs: ID[];
};
```

## Scope Decisions

```ts
type ProjectOperation = "read" | "mutate" | "archive";

type ProjectScope = {
  projectId: ID;
  workspaceId: ID;
  actorId: ID;
  actorProjectRole: ProjectRole;
  operation: ProjectOperation;
  accessMode: "read_only" | "read_write";
};

type ProjectScopeDecision =
  | {
      allowed: true;
      scope: ProjectScope;
    }
  | {
      allowed: false;
      projectId: ID;
      operation: ProjectOperation;
      reason:
        | "not-found-or-not-allowed"
        | "archived-readonly"
        | "mutation-denied"
        | "invalid-actor";
      code: U1ErrorCode;
    };
```

**Rules**

- Read operations are allowed for archived projects if the actor has access.
- Mutate/archive operations are denied for archived projects with `PROJECT_ARCHIVED_READONLY`.
- Repository calls receive `ProjectScope`, not only `projectId`.

## Transaction and Repository Contracts

```ts
type RepositoryName =
  | "workspace-membership"
  | "project"
  | "project-membership"
  | "workflow-state"
  | "workflow-definition"
  | "audit-outbox";

type TransactionContext = {
  id: ID;
  startedAt: ISODateTime;
};

type TransactionInput<T> = {
  name: string;
  actor: Actor;
  run: (tx: TransactionContext) => Promise<Result<T>>;
};

type ProjectScopedRepository<T, CreateInput = unknown, UpdateInput = unknown> = {
  findMany(scope: ProjectScope, tx: TransactionContext): Promise<Result<T[]>>;
  findOne(scope: ProjectScope, id: ID, tx: TransactionContext): Promise<Result<T | null>>;
  create(scope: ProjectScope, input: CreateInput, tx: TransactionContext): Promise<Result<T>>;
  update(scope: ProjectScope, id: ID, input: UpdateInput, tx: TransactionContext): Promise<Result<T>>;
};
```

Project-owned repositories must reject calls where `scope.projectId` does not match the target record.

## Port and Ref Types

```ts
type SecurityEvaluationContext = {
  projectId: ID;
  transition?: StageTransition;
  operation: "workflow-advance" | "project-create" | "project-archive" | "dashboard-read";
};

type AuditEventInput = {
  projectId: ID;
  workspaceId: ID;
  actorId: ID;
  action:
    | "project.created"
    | "project.archived"
    | "workflow.stageChanged"
    | "workflow.stageBlocked";
  summary: string;
  occurredAt: ISODateTime;
  targetRef?: {
    type: "project" | "workflow-stage";
    id: ID;
  };
};

type AuditOutboxEntryRef = {
  id: ID;
  status: "pending" | "dispatched" | "failed";
  retryable: boolean;
};
```

## Domain Events

```ts
type DomainEvent =
  | ProjectCreatedEvent
  | ProjectArchivedEvent
  | WorkflowStageChangedEvent
  | WorkflowStageBlockedEvent;

type ProjectCreatedEvent = {
  type: "project.created";
  projectId: ID;
  actorId: ID;
  occurredAt: ISODateTime;
};

type ProjectArchivedEvent = {
  type: "project.archived";
  projectId: ID;
  actorId: ID;
  occurredAt: ISODateTime;
};

type WorkflowStageChangedEvent = {
  type: "workflow.stageChanged";
  projectId: ID;
  fromStage: WorkflowStageId;
  toStage: WorkflowStageId;
  actorId: ID;
  occurredAt: ISODateTime;
};

type WorkflowStageBlockedEvent = {
  type: "workflow.stageBlocked";
  projectId: ID;
  stage: WorkflowStageId;
  reason: AdvanceDecisionReason;
  actorId: ID;
  occurredAt: ISODateTime;
};
```

## Entity Relationships

| Relationship | Cardinality | Rule |
|---|---|---|
| Workspace to WorkspaceMembership | 1:N | Workspace access is checked before project creation/listing |
| Workspace to Project | 1:N | A project belongs to one workspace |
| Project to ProjectMembership | 1:N | A project can have multiple actors with roles |
| Project to WorkflowState | 1:1 | Every active project has one current workflow state |
| WorkflowPhaseDefinition to WorkflowStageDefinition | 1:N | Stages belong to phases |
| WorkflowStageDefinition to StageTransitionDefinition | 1:N | Allowed next stages are explicit |
| Project to DomainEvent | 1:N | Critical state changes emit or enqueue audit events |

## Persistence Notes

- PostgreSQL + Prisma is the target persistence approach.
- Unique constraint: `(workspaceId, createdByActorId, normalizedName)`.
- Unique constraint: `(workspaceId, createdByActorId, slug)`.
- Recommended indexes:
  - `WorkspaceMembership.workspaceId`
  - `WorkspaceMembership.actorId`
  - `Project.workspaceId`
  - `Project.createdByActorId`
  - `Project.status`
  - `WorkflowState.projectId`
  - `ProjectMembership.projectId`
  - `ProjectMembership.actorId`
- Physical deletion is not modeled for project-owned records in MVP.

## Testable Properties

| Entity/Type | Property |
|---|---|
| Result | `ok: true` always has `value`; `ok: false` always has `error` |
| Project | `normalizedName` is stable and idempotent |
| Project | `slug` is URL-safe for all valid names |
| Project | slug suffix allocation chooses the lowest available suffix |
| Project | unsupported `appType` cannot produce `supported-in-mvp` |
| ProjectStatus | `archived` is terminal for mutation workflows |
| WorkflowState | new project starts with `workspace-detection` completed and `requirements-analysis` ready |
| WorkflowState | current stage is always a known `WorkflowStageId` |
| AdvanceDecision | missing approval or blocker always means `allowed = false` |
| ProjectDashboard | dashboard phase/stage mirrors `WorkflowState` |
| DashboardSection | unavailable sections carry `unavailableReason` and do not fail the whole dashboard |
| ProjectScopedRepository | project-owned query cannot execute without `ProjectScope` |

