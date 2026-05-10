# Business Logic Model - U1 Project, Workflow, and Data Access Core

## Purpose

U1 provides project metadata, workflow state, stage transition decisions, shared contracts, and project-scoped data access boundaries for the AI-DLC GUI Application Development Platform. It must not absorb U2 artifact logic, U4 job execution, U10 security classification, U11 approval/audit storage, or U13 UI composition.

## Revision Summary

This revision resolves review findings by clarifying:

- Dashboard section availability contracts.
- Initial workflow stage and full stage transition table.
- Approval gate to stage transition mapping.
- Previously undefined API DTOs and port types.
- Formal U1 error code set.
- Workspace access and project scope model.
- Project-scoped repository contract.
- Audit outbox behavior on dispatch failure.
- Read vs mutate project scope decisions for archived projects.
- Slug uniqueness and collision behavior.
- Discriminated `Result<T>`.
- `NextAction` routing contract for U13.

## Owned Business Capabilities

| Capability | Description | U1 Owns | Delegated To |
|---|---|---|---|
| Workspace access boundary | Validate whether an actor can create/list projects in a workspace | Workspace membership, workspace scope decision | External auth provider if added later |
| Project creation | Create project metadata and initial workflow state | Input validation, project metadata, slug, lifecycle status, initial workflow | U2 receives seed; U11 handles audit storage through port |
| Project retrieval/listing | Return project data inside workspace/project scope | Scope checks, project read model | None |
| Project dashboard | Aggregate current workflow and lightweight section refs | Dashboard shape, section availability, workflow-derived next action | U4 jobs, U10 blockers, U11 audit/approval |
| Project archive | Stop future mutation without physical deletion | Archive transition, read-only behavior | U11 audit through outbox port |
| Workflow initialization | Create initial AI-DLC workflow state | Stage enum, metadata reference, initial completed stage | None |
| Workflow advancement | Decide whether a stage transition is allowed | Transition table, gate lookup, blocker lookup, state update | U10 blocker check, U11 gate status |
| Data access boundary | Force project scope into repository calls | Transaction boundary and repository contract | Domain units provide business facts |
| Shared contracts | Keep DTOs and domain refs stable | `Result<T>`, IDs, dashboard refs, scope decisions | All units consume |

## Business Processes

### Process 1: Create Project

**Goal**: Create a new development project and make it ready for requirements work.

**Input**

`CreateProjectInput`:

- `workspaceId`
- `name`
- `initialIdeaText`
- `targetUsers`
- `publicationScope`
- `appType`

**Rules**

1. Resolve `WorkspaceScope` for `operation = create_project`.
2. Validate field-specific creation inputs.
3. Default `appType` to `internal-crud-web-app`.
4. Reject unsupported app types with `PROJECT_TYPE_UNSUPPORTED`.
5. Normalize project name and check uniqueness within `(workspaceId, actor.id)`.
6. Generate slug and allocate the lowest available suffix within `(workspaceId, actor.id)`.
7. Create project with `status = draft`.
8. Initialize workflow with `workspace-detection` completed and `requirements-analysis` ready.
9. Write an audit outbox entry inside the same transaction.
10. Return `CreateProjectResult` with project, workflow, seed, and audit outbox ref.

**Failure Cases**

| Code | Meaning | UI Handling |
|---|---|---|
| `WORKSPACE_ID_REQUIRED` | workspace ID is missing | Show workspace selection error |
| `WORKSPACE_SCOPE_DENIED` | actor cannot create project in workspace | Show generic access denied message |
| `PROJECT_NAME_REQUIRED` | project name is blank | Field-level name error |
| `PROJECT_NAME_INVALID` | project name violates constraints | Field-level name error |
| `PROJECT_NAME_ALREADY_EXISTS` | normalized name conflict | Ask for another name |
| `PROJECT_SLUG_ALREADY_EXISTS` | slug suffix allocation failed | Ask for another name |
| `PROJECT_IDEA_REQUIRED` | initial idea is blank | Field-level idea error |
| `PROJECT_TARGET_USERS_REQUIRED` | target users are blank | Field-level target user error |
| `PROJECT_PUBLICATION_SCOPE_REQUIRED` | publication scope is missing | Field-level publication scope error |
| `PROJECT_PUBLICATION_SCOPE_UNSUPPORTED` | publication scope unsupported | Explain MVP boundary |
| `PROJECT_TYPE_UNSUPPORTED` | unsupported app type selected | Explain internal CRUD MVP boundary |
| `AUDIT_OUTBOX_WRITE_FAILED` | mutation could not be made auditable | Fail request and retry later |

### Process 2: Initialize Workflow

**Goal**: Start a project at the first user-visible AI-DLC stage.

**Initial State**

- `workspace-detection` is a system bootstrap stage.
- New projects set `completedStages = ["workspace-detection"]`.
- New projects set `currentPhase = "inception"`.
- New projects set `currentStage = "requirements-analysis"`.
- New projects set `currentStageStatus = "ready"`.
- New projects set `projectStatus = "draft"`.

This avoids showing workspace detection as a user task while preserving it in the workflow history.

### Process 3: Get Project Dashboard

**Goal**: Provide U13 with enough state to render the dashboard without reimplementing workflow rules.

**Dashboard Shape**

- `project`: always available if project read succeeds.
- `workflow`: always available if project read succeeds.
- `nextAction`: always calculated by U1.
- `approval`: `DashboardSection<ApprovalDashboardSummary>`.
- `recentJobs`: `DashboardSection<JobSummaryRef[]>`.
- `recentAuditEvents`: `DashboardSection<AuditSummaryRef[]>`.
- `security`: `DashboardSection<SecuritySummaryRef>`.

**Unavailable Handling**

If U4/U10/U11 is unavailable, U1 returns `ok: true` for the dashboard and marks only the affected section:

```ts
{
  status: "unavailable",
  unavailableReason: "port-unavailable",
  message: "この情報は一時的に取得できません。",
  retryable: true
}
```

U1 does not fabricate empty successful data for unavailable sections.

### Process 4: Resolve NextAction

**Goal**: Give U13 routing-ready next action data.

**Rules**

1. If workflow is blocked by security, `actionType = blocked`, `target.unit = U10`, and `disabled = true`.
2. If a required approval is missing, `actionType = approve`, `target.unit = U11`, `requiredGate` is set, and `disabled = false`.
3. If the next stage belongs to another unit, `target.unit` points to that unit and `href` points to the intended UI route.
4. If a dependency section is unavailable and required for the next action, set `disabled = true` and `disabledReason = DEPENDENCY_UNAVAILABLE`.
5. U13 must follow `NextAction.target.href`; it must not recompute stage transition rules.

### Process 5: Advance Stage

**Goal**: Move the workflow only when transition, approval, security, and scope rules allow it.

**Decision Steps**

1. Resolve `ProjectScope` for `operation = mutate`.
2. Reject if project is archived.
3. Load current workflow state.
4. Find transition in `StageTransitionDefinition`.
5. If no transition exists, return `WORKFLOW_TRANSITION_INVALID`.
6. If transition requires a gate, call `ApprovalGatePort`.
7. If gate is not approved, return `WORKFLOW_APPROVAL_REQUIRED`.
8. Call `SecurityPolicyPort` for `operation = workflow-advance`.
9. If blockers exist, return `WORKFLOW_BLOCKED_BY_SECURITY`.
10. In a use case transaction, update workflow state and project status if configured.
11. Write audit outbox entry in the same transaction.
12. After commit, dispatch audit outbox asynchronously/retryably through U11.

**Audit Failure Rule**

- Audit outbox write failure before commit fails the API.
- Audit dispatch failure after commit does not fail the already successful API; return success with optional `AUDIT_DISPATCH_PENDING` warning and retry later.

### Process 6: Archive Project

**Goal**: Make a project read-only while preserving history.

**Rules**

- Resolve `ProjectScope` for `operation = archive`.
- Archive is idempotent.
- Archived projects are readable.
- Archived projects reject mutate/archive operations with `PROJECT_ARCHIVED_READONLY`.
- Physical deletion is not part of MVP.

### Process 7: Run Project-Scoped Repository Operation

**Goal**: Ensure all project-owned data access is scoped by a validated `ProjectScope`.

**Rules**

- Resolve `ProjectScopeDecision` before repository access.
- Repositories accept `ProjectScope`, not a raw `projectId`.
- Repositories verify the target record belongs to `scope.projectId`.
- Mutating repositories require `scope.accessMode = read_write`.
- Use case owns transaction; repository receives `TransactionContext`.

## Stage Transition Model

| From Stage | To Stage | Required Gate | Project Status On Advance | Next Action Owner |
|---|---|---|---|---|
| `workspace-detection` | `requirements-analysis` | None | `draft` | U2 |
| `requirements-analysis` | `user-stories` | `requirements` | `in_progress` | U2 |
| `user-stories` | `workflow-planning` | None | `in_progress` | U1 |
| `workflow-planning` | `application-design` | None | `in_progress` | U3 |
| `application-design` | `units-generation` | `design` | `in_progress` | U3 |
| `units-generation` | `functional-design` | None | `in_progress` | U1 |
| `functional-design` | `nfr-requirements` | None | `in_progress` | U1 |
| `nfr-requirements` | `nfr-design` | None | `in_progress` | U1 |
| `nfr-design` | `infrastructure-design` | None | `in_progress` | U1 |
| `infrastructure-design` | `code-generation` | None | `in_progress` | U7 |
| `code-generation` | `test-execution` | `implementation-diff` | `in_progress` | U8 |
| `test-execution` | `deployment-precheck` | `test-result` | `in_progress` | U9 |
| `deployment-precheck` | `amplify-deployment` | `deployment` | `in_progress` | U9 |
| `amplify-deployment` | `operations` | None | `deployed` | U12 |

## Interfaces Owned by U1

### Project API

```ts
createProject(input: CreateProjectInput, actor: Actor): Promise<Result<CreateProjectResult>>;
getProject(projectId: ID, actor: Actor): Promise<Result<Project>>;
listProjects(input: ListProjectsInput, actor: Actor): Promise<Result<ProjectSummary[]>>;
getProjectDashboard(projectId: ID, actor: Actor): Promise<Result<ProjectDashboard>>;
archiveProject(projectId: ID, actor: Actor): Promise<Result<Project>>;
```

### Workflow API

```ts
initializeWorkflow(projectId: ID, actor: Actor): Promise<Result<WorkflowState>>;
getWorkflowState(projectId: ID, actor: Actor): Promise<Result<WorkflowState>>;
canAdvanceStage(input: AdvanceStageInput, actor: Actor): Promise<Result<AdvanceDecision>>;
advanceStage(input: AdvanceStageInput, actor: Actor): Promise<Result<WorkflowState>>;
blockStage(input: BlockStageInput, actor: Actor): Promise<Result<WorkflowState>>;
```

### Data Access API

```ts
runInTransaction<T>(input: TransactionInput<T>): Promise<Result<T>>;
getRepository<T extends ProjectScopedRepository<unknown>>(name: RepositoryName): T;
enforceWorkspaceScope(workspaceId: ID, operation: WorkspaceOperation, actor: Actor): Promise<Result<WorkspaceScopeDecision>>;
enforceProjectScope(projectId: ID, operation: ProjectOperation, actor: Actor): Promise<Result<ProjectScopeDecision>>;
```

### Required Ports

```ts
type ApprovalGatePort = {
  getRequiredGate(projectId: ID, transition: StageTransition): Promise<Result<GateRequirement | null>>;
  isGateApproved(projectId: ID, gateType: ApprovalGateType): Promise<Result<boolean>>;
  getApprovalDashboardSummary(projectId: ID): Promise<Result<ApprovalDashboardSummary>>;
};

type SecurityPolicyPort = {
  hasBlockingFindings(projectId: ID, context: SecurityEvaluationContext): Promise<Result<boolean>>;
  getSecuritySummary(projectId: ID): Promise<Result<SecuritySummaryRef>>;
};

type AuditPort = {
  enqueueAuditEvent(input: AuditEventInput, tx: TransactionContext): Promise<Result<AuditOutboxEntryRef>>;
  listRecentAuditEvents(projectId: ID, limit: number): Promise<Result<AuditSummaryRef[]>>;
};

type JobSummaryPort = {
  listRecentJobs(projectId: ID, limit: number): Promise<Result<JobSummaryRef[]>>;
};
```

## Testable Properties

| Area | Property Category | Property |
|---|---|---|
| Result | Invariant | `ok: true` implies `value` exists; `ok: false` implies `error` exists |
| Project name normalization | Idempotence | `normalizeProjectName(normalizeProjectName(name)) = normalizeProjectName(name)` |
| Project slug generation | Invariant | Generated slug is non-empty, lowercase, URL-safe, and within max length for every valid project name |
| Slug collision | Invariant | Collision resolution chooses the lowest available suffix |
| Project uniqueness key | Invariant | Equivalent normalized names produce the same uniqueness key within the same workspace/user |
| Project status transition | Stateful / Invariant | Archived projects never transition back to mutable states |
| Workflow initialization | Invariant | New workflow has `workspace-detection` completed and `requirements-analysis` ready |
| Workflow transition | Stateful / Invariant | A required unapproved gate always produces `AdvanceDecision.allowed = false` |
| Workflow transition | Invariant | A transition never skips configured stage order |
| Dashboard section availability | Invariant | Unavailable dependent ports produce unavailable dashboard sections, not dashboard failure |
| Archive project | Idempotence | Archiving an already archived project leaves observable state unchanged |
| Project repository | Invariant | Project-owned repository methods require `ProjectScope` |

## Security Baseline Compliance

| Rule | Status | Rationale |
|---|---|---|
| SECURITY-05 Input Validation | Compliant | U1 defines field-specific validation, length constraints, enum allowlists, and safe repository contracts |
| SECURITY-08 Application-Level Access Control | Compliant | Workspace and project scope decisions are required before data access |
| SECURITY-11 Secure Design Principles | Compliant | U10/U11 are ports; U1 does not own security classification or approval storage |
| SECURITY-13 Data Integrity | Compliant | Mutations require durable audit outbox writes inside transactions |
| Other Security Rules | N/A at Functional Design | Infrastructure, HTTP headers, network, logging, auth credential storage, and CI/CD details are handled in later construction stages |

## PBT Compliance

| Rule | Status | Rationale |
|---|---|---|
| PBT-01 Property Identification During Design | Compliant | Testable properties are identified for result shape, normalization, slug allocation, scope, workflow transitions, and dashboard availability |
| PBT-02 Round-Trip Properties | N/A | U1 does not define serialization/deserialization pairs as business functions in this stage |
| PBT-03 Invariant Properties | Compliant | Invariants are documented for project uniqueness, workflow order, dashboard consistency, and project scope |
| PBT-07 Generator Quality | Planned for Code Generation | Domain generators are required for project names, actors, workspace IDs, project statuses, dashboard sections, and stage sequences |
| PBT-08 Shrinking and Reproducibility | Planned for Code Generation | fast-check configuration will preserve shrinking and seed reproducibility |
| PBT-09 Framework Selection | Deferred to NFR Requirements | TypeScript PBT framework selection will be documented in NFR Requirements |

## Open Constraints for Later Units

- U10 must implement `SecurityPolicyPort` with blocker/warning classification.
- U11 must implement `ApprovalGatePort` and `AuditPort`.
- U4 must implement `JobSummaryPort`.
- U13 must use `NextAction.target.href` without duplicating workflow rules.
- U2 must take `ProjectCreationSeed` and store the authoritative idea artifact.

