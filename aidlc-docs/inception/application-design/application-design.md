# Application Design

## Summary

The MVP is designed as a responsive Next.js platform with a separated background worker. The UI/API surface handles user interaction, lightweight request validation, and status display. Long-running AI generation, GitHub operations, tests, and AWS Amplify deployments run as asynchronous jobs.

The design uses clear component boundaries for workflow state, approvals, audit, AI provider abstraction, GitHub, code generation, testing, deployment, security policy, explanation/translation, and operations feedback.

## Major Design Decisions

| Area | Decision |
|---|---|
| Runtime shape | Next.js UI/API plus separated worker |
| Job model | All AI generation, GitHub, test, and deployment work runs asynchronously |
| Workflow state | Phase, stage, and approval gate are separate entities |
| Data store | PostgreSQL + Prisma with logical table groups |
| Approval/audit | Approval and AuditEvent are separate but linked |
| Security | Dedicated Security Policy Component |
| AI provider | Generic interface with Bedrock-first implementation |
| Explanation | Dedicated Explanation/Translation Component |
| PBT | Domain components declare applicable properties |
| Operations feedback | MVP defines log summary and improvement suggestion boundaries |

## Primary Components

1. Project Workspace Component
2. AI-DLC Workflow Orchestrator
3. Requirements Intake Component
4. Story and Artifact Manager
5. Application Design Assistant
6. Task Planning Component
7. AI Provider Adapter
8. Job Orchestrator and Worker
9. GitHub Integration Component
10. Code Generation and Diff Review Component
11. Test Orchestration Component
12. Deployment Pre-Check Component
13. AWS Amplify Deployment Component
14. Security Policy Component
15. Approval Gate Component
16. Audit History Component
17. Explanation/Translation Component
18. Operations Feedback Component
19. Data Access Component

## Logical Architecture

### UI/API Layer

- Next.js UI
- Next.js Route Handlers or Hono-compatible API handlers
- Non-engineer user experience
- Project dashboard, review screens, approval screens, job status screens

### Service Layer

- Project Service
- Workflow Service
- Requirements Service
- Design Service
- Task Service
- GitHub Service
- Code Review Service
- Test Service
- Deployment Service
- Audit Service
- Explanation Service
- Operations Feedback Service

### Worker Layer

- AI generation jobs
- GitHub operation jobs
- Test execution jobs
- Deployment jobs
- Log summary and improvement suggestion jobs

### Integration Layer

- AI Provider Adapter with Bedrock-first provider implementation
- GitHub API integration
- AWS Amplify Hosting integration
- PostgreSQL/Prisma persistence

### Policy and Governance Layer

- Security Policy Component
- Approval Gate Component
- Audit History Component
- PBT declarations and testing policy

## Data Model Groups

The MVP uses a single PostgreSQL database with Prisma, logically grouped by responsibility.

| Data Group | Example Entities |
|---|---|
| Project management | Project, ProjectMember, ProjectSetting |
| Workflow state | WorkflowPhase, WorkflowStage, StageTransition, GateState |
| Requirements and stories | Idea, Question, Answer, Requirement, StoryReference |
| Artifacts | Artifact, ArtifactVersion, ArtifactLink |
| Design and tasks | DesignArtifact, ComponentDefinition, ImplementationTask |
| Jobs | Job, JobLogSummary, JobArtifact |
| GitHub | GitHubRepository, GitHubBranch, GitHubPullRequest, GitHubCheckRun |
| Deployment | DeploymentTarget, DeploymentPreCheck, DeploymentRun, DeploymentResult |
| Approval | Approval, ApprovalDecision |
| Audit | AuditEvent |
| Security | SecurityFinding, SecurityDecision |
| Operations feedback | OperationsSummary, ImprovementSuggestion |

## Key User Journeys

### Create Project and Requirements

1. User creates project.
2. User enters app idea.
3. Requirements Service generates questions.
4. User answers questions.
5. Requirements artifact is generated.
6. User approves requirements.

### Design and Task Preparation

1. Design Service generates application design artifacts.
2. User reviews and approves design.
3. Task Service generates implementation tasks.
4. Units Generation decomposes work for Construction.

### Code, Review, and Test

1. Code Review Service generates template or task-specific code.
2. GitHub Service creates branch and Pull Request.
3. Code Review Service summarizes diff and scope expansion.
4. User approves implementation diff.
5. Test Service runs tests and summarizes result.
6. User approves test result.

### Deploy and Review

1. Deployment Service runs pre-checks.
2. Security Policy blocks unsafe publication or risky operations.
3. User approves deployment.
4. Amplify Deployment Component triggers deployment.
5. User sees deployment result and public URL.
6. Operations Feedback Component summarizes logs and improvement candidates.

## Security Design

- Security Policy Component centralizes blocker/warning decisions.
- Blocking findings prevent progression until resolved or explicitly handled.
- Deployment Pre-Check calls Security Policy Component before deployment approval.
- Code Review Service calls Security Policy Component before generated changes proceed.
- Audit History records security-relevant decisions.

Blocking conditions include:
- Hardcoded secrets
- Overbroad IAM permissions
- External publication without authentication or access restriction
- Dangerous AWS resource deletion
- Improper user data transmission
- Unapproved production deployment

## PBT Design

PBT is declared by domain components and coordinated by Test Orchestration.

Applicable areas:
- Requirement/question schema validation
- Artifact serialization and deserialization
- Workflow transition invariants
- Task traceability invariants
- Scope expansion mapping
- Security finding classification
- Test result normalization
- Explanation term mapping

Non-applicable in MVP:
- UI components
- Screen transitions
- AWS deployment operations
- GitHub API calls
- LLM calls
- E2E tests

## Artifact References

- Component definitions: `components.md`
- Component methods: `component-methods.md`
- Service orchestration: `services.md`
- Dependencies and data flow: `component-dependency.md`

## Design Completeness Check

- [x] Component boundaries identified.
- [x] Service responsibilities defined.
- [x] Method signatures documented at high level.
- [x] Component dependencies documented.
- [x] Data ownership groups identified.
- [x] Asynchronous job model documented.
- [x] Security policy placement documented.
- [x] PBT responsibility documented.
- [x] Non-engineer explanation responsibility documented.
- [x] Operations feedback MVP boundary documented.

## Extension Compliance Summary

### Security Baseline

No blocking security finding exists in the Application Design artifacts.

Relevant compliance points:
- Dedicated Security Policy Component.
- Security checks before deployment and risky code changes.
- Approval and audit are separate and linked.
- External publication without authentication or access restriction is blocked.

### Property-Based Testing

No blocking PBT finding exists in the Application Design artifacts.

Relevant compliance points:
- Domain components declare PBT-applicable properties.
- Test Orchestration collects declarations.
- Detailed PBT properties will be expanded during Functional Design and Code Generation planning.

