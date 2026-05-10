# Component Dependency

## Dependency Principles

- UI/API calls services, not persistence directly.
- Domain services use Data Access Component for Prisma operations.
- Long-running work goes through Job Orchestrator and Worker.
- AI generation goes through AI Provider Adapter.
- Risky operations call Security Policy Component before execution.
- User-facing technical output goes through Explanation/Translation Component.
- Approval records and audit events are separate but linked.

## Dependency Matrix

| Component | Depends On | Dependency Type | Reason |
|---|---|---|---|
| Project Workspace | Data Access, Audit History | Direct | Persist project and record creation/update events |
| AI-DLC Workflow Orchestrator | Approval Gate, Security Policy, Data Access, Audit History | Direct | Enforce gates and transitions |
| Requirements Intake | AI Provider Adapter, Artifact Manager, Explanation, Job Orchestrator | Direct | Generate questions/requirements asynchronously |
| Story and Artifact Manager | Data Access, Audit History | Direct | Store artifacts and traceability |
| Application Design Assistant | AI Provider Adapter, Artifact Manager, Job Orchestrator, Explanation | Direct | Generate design artifacts |
| Task Planning | Artifact Manager, AI Provider Adapter, Job Orchestrator | Direct | Generate implementation task breakdown |
| AI Provider Adapter | Provider Implementations, Security Policy | Direct | Provider execution and policy checks |
| Job Orchestrator and Worker | Data Access, Audit History, domain executors | Direct | Queue, run, track, and record jobs |
| GitHub Integration | Job Orchestrator, Security Policy, Explanation, Audit History | Direct | External GitHub operations |
| Code Generation and Diff Review | AI Provider Adapter, GitHub Integration, Security Policy, Explanation, Artifact Manager | Direct | Generate and review changes |
| Test Orchestration | Job Orchestrator, Explanation, Artifact Manager | Direct | Run and summarize tests |
| Deployment Pre-Check | Security Policy, Data Access, Explanation, Audit History | Direct | Validate readiness |
| AWS Amplify Deployment | Job Orchestrator, GitHub Integration, Security Policy, Explanation, Audit History | Direct | External Amplify operations |
| Security Policy | Data Access, Audit History | Direct | Store findings and policy decisions |
| Approval Gate | Data Access, Audit History, Security Policy | Direct | Approval storage and policy-aware decisions |
| Audit History | Data Access | Direct | Persist audit events |
| Explanation/Translation | AI Provider Adapter, static glossary | Optional | AI-assisted summaries and deterministic term mapping |
| Operations Feedback | AI Provider Adapter, Explanation, Artifact Manager, Audit History | Direct | Summarize logs and create improvement suggestions |
| Data Access | Prisma, PostgreSQL | Direct | Persistence boundary |

## Communication Patterns

| Interaction | Pattern |
|---|---|
| UI to service | Synchronous HTTP/API call |
| Service to worker | Asynchronous job enqueue |
| Worker to AI provider | Provider-adapter call |
| Worker to GitHub | External API call through GitHub Integration |
| Worker to Amplify | External API call through Amplify Deployment |
| Service to Security Policy | Synchronous policy evaluation |
| Service to Audit History | Synchronous audit event record for state changes |
| UI to job status | Polling or subscription |

## Textual Data Flow: Project Creation to Requirements

1. User enters project name and app idea in UI.
2. UI calls Project Service.
3. Project Service creates Project through Data Access.
4. Audit History records project creation.
5. Requirements Service saves idea.
6. Requirements Service enqueues AI question generation job.
7. Worker calls AI Provider Adapter.
8. Artifact Manager stores generated questions.
9. Explanation Component converts technical phrasing if needed.
10. UI displays questions and waits for answers.

## Textual Data Flow: Design to Code Review

1. User approves requirements and design gate.
2. Workflow Service advances to design/task stage.
3. Design Service generates component and data design artifacts through AI Job Service.
4. Artifact Manager stores design artifacts.
5. Task Service generates task breakdown.
6. Code Review Service enqueues code generation job.
7. Worker calls AI Provider Adapter and Code Generation.
8. Security Policy scans generated changes for blockers.
9. GitHub Integration creates branch and PR.
10. Code Review Service retrieves diff and generates summary.
11. Approval Gate records implementation diff decision.

## Textual Data Flow: Test and Deployment

1. User approves implementation diff.
2. Test Service enqueues test job.
3. Worker runs test commands and collects results.
4. Explanation Component summarizes failures.
5. Approval Gate records test result approval.
6. Deployment Pre-Check validates app settings, build readiness, DB connectivity, external publication, and security blockers.
7. Approval Gate records deployment approval.
8. AWS Amplify Deployment creates/configures app, connects GitHub, and triggers deployment.
9. Job Orchestrator tracks deployment status.
10. Audit History records deployment result.
11. UI displays public URL or failure summary.

## Textual Data Flow: Operations Feedback

1. Deployment or runtime log references are available.
2. Operations Feedback Component requests log summary.
3. Explanation Component produces non-engineer summary.
4. AI Provider Adapter can generate improvement candidates.
5. Artifact Manager stores improvement suggestions.
6. Workflow Service can later use approved suggestions as next-cycle input.

## Coupling and Boundary Notes

- GitHub Integration and AWS Amplify Deployment are external integration components and should be isolated behind service interfaces.
- AI Provider Adapter must not leak provider-specific request/response shapes into domain services.
- Security Policy Component owns blocker/warning classification; domain services provide facts and context.
- Explanation/Translation Component owns user-facing phrasing for technical results.
- Data Access Component owns Prisma and project scoping to avoid direct persistence coupling in UI/API components.

## Dependency Risks

| Risk | Affected Components | Mitigation |
|---|---|---|
| Provider-specific AI logic leaks into domain services | AI Provider Adapter, Requirements, Design, Code Generation | Keep provider-neutral interfaces and provider-specific implementation modules |
| External API failures stop user workflow without clear explanation | GitHub, Amplify, Explanation | Wrap errors and translate to user-facing next actions |
| Security logic becomes inconsistent | Security Policy, Deployment, Code Review | Centralize blocker/warning classification |
| Audit trail is incomplete | Approval, Audit, Workflow | Require audit event on every state-changing command |
| Job status becomes stale | Job Orchestrator, UI | Store job status transitions and expose polling/subscription |

