# Services

## Service Layer Overview

The service layer coordinates UI/API requests, asynchronous jobs, domain components, external integrations, and persistence. The MVP uses a Next.js UI/API surface with a separated background worker so that all AI generation, GitHub, test, and deployment operations run asynchronously.

## Service Groups

| Service | Primary Components | Purpose |
|---|---|---|
| Project Service | Project Workspace, Data Access, Audit History | Project creation, project dashboard, project lifecycle metadata |
| Workflow Service | Workflow Orchestrator, Approval Gate, Audit History, Security Policy | Phase/stage progression and gate enforcement |
| Requirements Service | Requirements Intake, AI Provider Adapter, Artifact Manager, Explanation Component | Idea capture, questions, answers, requirements artifacts |
| Design Service | Application Design Assistant, Artifact Manager, AI Provider Adapter | Application design artifact generation |
| Task Service | Task Planning, Artifact Manager, Workflow Service | Implementation task breakdown and traceability |
| AI Job Service | Job Orchestrator, AI Provider Adapter, Artifact Manager | Asynchronous AI execution and status tracking |
| GitHub Service | GitHub Integration, Job Orchestrator, Explanation Component, Audit History | Repository, branch, PR, diff, comments, check results |
| Code Review Service | Code Generation, GitHub Integration, Security Policy, Explanation Component | Code generation, diff summary, scope expansion review |
| Test Service | Test Orchestration, Job Orchestrator, Explanation Component | Test execution, summaries, PBT visibility |
| Deployment Service | Deployment Pre-Check, AWS Amplify Deployment, Security Policy, Approval Gate | Deployment readiness and Amplify deployment |
| Audit Service | Audit History, Approval Gate | Operation history and approval traceability |
| Explanation Service | Explanation/Translation Component | Non-engineer language conversion |
| Operations Feedback Service | Operations Feedback, Explanation Component, Artifact Manager | Log summary and improvement suggestions |

## Orchestration Patterns

### Pattern 1: Approval-Gated Progression

1. UI requests stage advancement.
2. Workflow Service checks current phase/stage.
3. Workflow Service asks Approval Gate Component whether required approvals exist.
4. Workflow Service asks Security Policy Component whether blockers exist.
5. If allowed, Workflow Service advances the stage.
6. Audit Service records the transition.

### Pattern 2: Asynchronous AI Generation

1. UI requests AI generation.
2. Domain service validates the request and stage state.
3. Job Orchestrator creates a queued job.
4. Worker executes provider-agnostic generation through AI Provider Adapter.
5. Artifact Manager stores generated output.
6. Explanation Service creates non-engineer summaries.
7. Job status updates are exposed to the GUI.
8. Audit Service records a summarized event.

### Pattern 3: GitHub Review Flow

1. Code Review Service prepares generated changes.
2. Security Policy Component scans for blockers such as secrets.
3. GitHub Service creates or updates branch and Pull Request.
4. GitHub Service retrieves diff and check results.
5. Code Review Service generates a user-facing change summary.
6. Approval Gate Component records implementation diff approval.
7. Audit Service records GitHub operations and approval references.

### Pattern 4: Deployment Flow

1. User requests deployment preparation.
2. Deployment Service runs Deployment Pre-Check.
3. Security Policy Component blocks unsafe external publication or missing approval.
4. Approval Gate Component records deployment approval.
5. AWS Amplify Deployment Component creates/configures app, connects GitHub, and triggers deployment.
6. Job Orchestrator tracks deployment progress.
7. Explanation Service summarizes success or failure.
8. Audit Service records deployment history.

### Pattern 5: Operations Feedback Boundary

1. Operations Feedback Service receives deployment/runtime log references.
2. Explanation Service creates safe non-engineer summaries.
3. AI Provider Adapter can propose improvement candidates.
4. Artifact Manager stores suggestions as reviewable artifacts.
5. Next-cycle registration is noted as a boundary but not fully automated in MVP design.

## Service Interactions

| Caller | Callee | Interaction |
|---|---|---|
| UI/API | Project Service | Create/list/read project |
| UI/API | Workflow Service | Read workflow state and request advancement |
| Workflow Service | Approval Gate Component | Check required approvals |
| Workflow Service | Security Policy Component | Check blockers before transition |
| Requirements Service | AI Job Service | Generate questions and requirements |
| Design Service | AI Job Service | Generate design artifacts |
| Task Service | AI Job Service | Generate task breakdown |
| Code Review Service | GitHub Service | Branch, PR, diff, checks |
| Deployment Service | AWS Amplify Deployment Component | Create/connect/deploy/read status |
| Test Service | Job Orchestrator | Run tests asynchronously |
| All Services | Audit Service | Record operation summaries |
| All User-Facing Services | Explanation Service | Convert technical output to user-facing language |

## Data Ownership by Service

| Data Group | Owning Service |
|---|---|
| Project | Project Service |
| Workflow Phase/Stage/Gate State | Workflow Service |
| Approval | Approval Gate Component |
| AuditEvent | Audit Service |
| Idea, Requirement, Question, Answer | Requirements Service |
| Artifact and ArtifactLink | Story and Artifact Manager |
| Application Design Artifact | Design Service |
| Implementation Task | Task Service |
| AI Job | AI Job Service |
| GitHub Repository/Branch/PR metadata | GitHub Service |
| Diff Review and Scope Expansion Report | Code Review Service |
| Test Run and Test Result | Test Service |
| Deployment Pre-Check and Deployment Result | Deployment Service |
| Security Finding | Security Policy Component |
| Operations Feedback Summary | Operations Feedback Service |

## Transaction and Consistency Boundaries

- User-facing commands that change state must write audit events in the same logical operation.
- Approval decisions and workflow transitions are separate records but must be linked.
- Long-running job results are eventually consistent and surfaced through job status.
- GitHub and Amplify operations are external side effects and must record request, result, and error summaries.
- Security blockers must be evaluated before deployment and before applying risky code changes.

## Security Service Responsibilities

- Security Policy Component is called by services before risky actions.
- Services do not duplicate critical security classification logic.
- Components may perform local validation but must defer blocker/warning classification to Security Policy Component.
- Security findings are stored and linked to relevant approvals, deployments, or artifacts.

## PBT Design Responsibilities

- Domain components declare their PBT-applicable properties.
- Test Service collects declarations and plans property-based tests.
- Application Design records property categories at component level.
- Functional Design will define detailed PBT properties per unit.

