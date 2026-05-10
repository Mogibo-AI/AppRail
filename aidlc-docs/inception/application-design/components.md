# Components

## Design Decisions Applied

| Area | Decision |
|---|---|
| Architecture boundary | Next.js UI/API plus separated background worker |
| Workflow state | Separate entities for phases, stages, and approval gates |
| AI execution | Asynchronous jobs with GUI progress polling or subscription |
| GitHub scope | Repository connection, branch, PR, diff, comments, and check results |
| Amplify scope | Amplify app creation, GitHub connection, deployment trigger, and result retrieval |
| Data ownership | PostgreSQL + Prisma with clear logical table groups for project management, AI artifacts, and audit |
| Approval/audit | Separate `Approval` and `AuditEvent` entities |
| Security policy | Dedicated Security Policy Component called by services |
| AI provider abstraction | Generic provider interface designed for Bedrock and future providers |
| Explanation responsibility | Dedicated Explanation/Translation Component |
| PBT responsibility | Domain components declare PBT-applicable properties |
| Operations feedback | Define log summary and improvement suggestion boundaries only in MVP |

## Component Overview

| Component | Purpose | Primary Responsibilities | Primary Interfaces |
|---|---|---|---|
| Project Workspace Component | Manage projects and top-level workspaces | Project creation, workspace metadata, application type constraints, project dashboard state | Project API, Workflow Orchestrator, Artifact Manager |
| AI-DLC Workflow Orchestrator | Control phase, stage, gate, and transition state | State machine, stage progression, gate blocking, transition validation | Project Workspace, Approval Gate, Audit History, Job Orchestrator |
| Requirements Intake Component | Convert app ideas into structured requirements | Idea capture, AI question generation request, answer capture, requirements artifact creation | AI Provider Adapter, Artifact Manager, Explanation Component |
| Story and Artifact Manager | Store and expose AI-DLC documentation artifacts | Artifact versioning, file/document metadata, traceability between requirements/stories/design | Requirements Intake, Design Assistant, Audit History |
| Application Design Assistant | Generate high-level design artifacts | Screen/data/component design requests, design summaries, design review state | AI Provider Adapter, Artifact Manager, Explanation Component |
| Task Planning Component | Create implementation task breakdown | Task generation, task traceability, task risk labels, task-to-story mapping | Design Assistant, Workflow Orchestrator |
| AI Provider Adapter | Abstract LLM provider access | Provider-agnostic generation, model selection, prompt execution, structured response parsing | Job Orchestrator, Requirements Intake, Design Assistant, Code Generation |
| Job Orchestrator and Worker | Run long-running operations asynchronously | AI jobs, GitHub jobs, test jobs, deployment jobs, progress, retries, cancellation | Workflow Orchestrator, AI Provider Adapter, GitHub Integration, Amplify Deployment |
| GitHub Integration Component | Manage repository and review workflow | Repository connection/creation, branch creation, PR creation, diff retrieval, comments, check results | Job Orchestrator, Code Generation, Audit History |
| Code Generation and Diff Review Component | Generate and review code changes | Template generation, code generation request, diff summary, scope expansion detection | AI Provider Adapter, GitHub Integration, Security Policy, Testing Component |
| Test Orchestration Component | Run and summarize tests | Unit/integration/E2E execution, PBT applicable declarations, result summary, failure explanation | Job Orchestrator, Explanation Component, Security Policy |
| Deployment Pre-Check Component | Validate deployment readiness | App setting checks, build status, DB connectivity, external publication boundary, security blockers | Security Policy, Amplify Deployment, Audit History |
| AWS Amplify Deployment Component | Deploy generated app through Amplify | Amplify app creation, GitHub connection, deployment trigger, deployment result retrieval | Job Orchestrator, Deployment Pre-Check, GitHub Integration |
| Security Policy Component | Centralize safety and security decisions | Blocking/warning classification, secret checks, IAM policy checks, external publication checks, production approval checks | All service components, Deployment Pre-Check, Audit History |
| Approval Gate Component | Manage explicit human approvals | Requirement, design, implementation diff, test result, deployment approvals | Workflow Orchestrator, Audit History, Security Policy |
| Audit History Component | Record auditable operations | Approval records, GitHub operations, deployment history, summarized AI logs | All service components |
| Explanation/Translation Component | Convert technical results into non-engineer language | Error summaries, technical term mapping, user-facing next actions | UI, Requirements Intake, GitHub, Tests, Deployment |
| Operations Feedback Component | Provide post-deployment review boundaries | Log summary boundary, improvement suggestion boundary, next-cycle candidate creation boundary | Amplify Deployment, Explanation Component, Artifact Manager |
| Data Access Component | Encapsulate Prisma persistence | Repository layer, transactions, logical table grouping, query boundaries | All domain components |

## Component Responsibilities

### Project Workspace Component

- Creates and manages development projects.
- Enforces MVP app type constraints: internal CRUD applications only.
- Provides project dashboard data: current phase, next action, recent approvals, recent jobs.
- Owns project-level metadata but does not own detailed workflow state.

### AI-DLC Workflow Orchestrator

- Manages phases, stages, and transitions as first-class entities.
- Blocks progression when required gates are not approved.
- Evaluates whether the next action is allowed.
- Delegates approval storage to Approval Gate Component and audit storage to Audit History Component.

### Requirements Intake Component

- Captures natural-language app ideas.
- Requests AI-generated clarification questions.
- Stores answers and produces structured requirement artifacts.
- Uses Explanation/Translation Component for non-engineer wording.
- Declares PBT-applicable properties for answer validation and artifact schema validation.

### Story and Artifact Manager

- Stores generated AI-DLC artifacts, including requirements, personas, stories, design docs, and plans.
- Tracks artifact versions and source relationships.
- Links requirements to stories, stories to tasks, and design artifacts to implementation units.

### Application Design Assistant

- Generates high-level components, service definitions, and dependency maps.
- Produces design artifacts after design questions are answered.
- Does not define detailed business rules; those belong to Functional Design.

### Task Planning Component

- Breaks approved design into user-understandable implementation tasks.
- Tracks task source requirements and story IDs.
- Labels risky tasks that need explicit review.

### AI Provider Adapter

- Defines provider-agnostic interfaces for text generation, structured output generation, and model metadata.
- Provides Bedrock as the first concrete provider.
- Leaves room for OpenAI or other providers without changing workflow components.

### Job Orchestrator and Worker

- Runs all AI generation, GitHub, test, and deployment operations as asynchronous jobs.
- Provides progress, status, retry, failure, and cancellation interfaces.
- Stores job summaries and references detailed logs through artifacts or audit events.

### GitHub Integration Component

- Manages GitHub repository creation or connection.
- Creates branches and Pull Requests.
- Retrieves diffs, comments, and check results.
- Converts technical GitHub errors through Explanation/Translation Component.

### Code Generation and Diff Review Component

- Generates Next.js CRUD templates and task-specific code changes.
- Summarizes diffs for non-engineer review.
- Detects scope expansion by comparing generated changes to approved requirements and tasks.
- Calls Security Policy Component before changes can move toward deployment.

### Test Orchestration Component

- Runs configured tests asynchronously.
- Displays test status and summaries.
- Includes PBT where domain components declare applicable properties.
- Does not enforce PBT for UI components, AWS deployment operations, GitHub API calls, LLM calls, or E2E tests in MVP.

### Deployment Pre-Check Component

- Confirms build readiness, app settings, database connectivity, external publication boundary, and security blockers.
- Blocks external publication without authentication or access restriction.
- Explains missing settings such as `DATABASE_URL` in non-engineer language.

### AWS Amplify Deployment Component

- Creates or configures Amplify applications.
- Connects Amplify to GitHub.
- Triggers deployments and retrieves deployment results.
- Records deployment status and public URL.

### Security Policy Component

- Centralizes security checks that services must call before risky actions.
- Classifies findings as blocking or warning.
- Blocks hardcoded secrets, overbroad IAM, unauthenticated external publication, dangerous AWS deletion, improper user data transmission, and unapproved production deployment.

### Approval Gate Component

- Manages explicit approvals independently from audit events.
- Supports requirement, design, implementation diff, test result, and deployment gates.
- Records actor, timestamp, target, summary, and decision.

### Audit History Component

- Records auditable operation history.
- Stores approval references, GitHub operation summaries, deployment summaries, and AI internal log summaries.
- Does not store raw AI logs by default.

### Explanation/Translation Component

- Converts technical outputs into non-engineer language.
- Provides term mappings such as Pull Request to "変更内容の確認" and environment variables to "アプリ設定".
- Generates next-action guidance for errors and warnings.

### Operations Feedback Component

- MVP scope defines only component boundaries for log summary and improvement suggestions.
- Full log ingestion, next-cycle registration, and richer dashboards remain later-stage detail or Post-MVP.

### Data Access Component

- Encapsulates Prisma queries and transactions.
- Maintains logical table group boundaries for project management, AI artifacts, workflow state, approvals, audit, jobs, GitHub, and deployment.

## PBT Property Declarations at Component Level

| Component | PBT-Applicable Properties |
|---|---|
| Requirements Intake Component | Question/answer schema validation, requirement artifact schema validation |
| Story and Artifact Manager | Artifact serialization/deserialization, traceability mapping invariants |
| AI-DLC Workflow Orchestrator | Allowed state transitions, idempotent transition checks |
| Task Planning Component | Task-to-requirement traceability invariants |
| Code Generation and Diff Review Component | Scope mapping validation, structured diff summary schema validation |
| Test Orchestration Component | Test result normalization and summary schema validation |
| Security Policy Component | Finding classification invariants |
| Explanation/Translation Component | Term mapping completeness and deterministic normalization |

## Security Compliance Summary

- Security checks are centralized in the Security Policy Component.
- Each risky component must call Security Policy Component before performing or approving risky operations.
- No blocking security finding exists at Application Design component level.

