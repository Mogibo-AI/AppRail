# User Stories

## Story Generation Decisions

- **Primary persona**: 社内向けCRUDアプリを作りたい非エンジニア
- **Generated app users**: 管理者と一般利用者の基本操作を含める
- **Breakdown**: Hybrid Journey + Feature
- **Acceptance criteria**: Given / When / Then plus checklist
- **Granularity**: Small stories suitable for implementation and testing
- **MVP lifecycle scope**: 企画、要件整理、設計、コード生成、テスト、デプロイ、ログ確認、改善提案
- **Approval gates**: 要件、設計、実装差分、テスト結果、デプロイを個別Story化
- **Security and PBT**: 横断Storyと関連Storyの受け入れ基準に併用
- **Post-MVP**: MVPとは別枠で短く記録

## MVP Stories

### Epic 1: Project Start and Idea Capture

#### US-001: Create a Development Project

As a non-engineer app creator, I want to create a new development project, so that I can manage my app idea, generated artifacts, approvals, and deployment history in one place.

**Acceptance Criteria**

Given I am on the project creation screen, when I enter a project name and create the project, then the system creates a project workspace and initial state.

Checklist:
- [ ] Project name and app description can be entered.
- [ ] Project state is created.
- [ ] Generated artifacts are scoped to the project.
- [ ] The app type is limited to internal CRUD applications.
- [ ] The UI avoids technical setup language unless shown in details.

#### US-002: Enter an App Idea in Natural Language

As a non-engineer app creator, I want to describe the app I want in natural language, so that I do not need to write technical requirements first.

**Acceptance Criteria**

Given I have created a project, when I enter an app idea such as a reservation or inventory app, then the system saves the idea as the source input for AI-DLC analysis.

Checklist:
- [ ] The input supports plain business language.
- [ ] The saved idea is traceable to later requirements.
- [ ] The UI indicates that the next step is to clarify the idea.
- [ ] Empty or vague input receives a user-friendly prompt to add detail.

#### US-003: Answer AI-Generated Requirement Questions

As a non-engineer app creator, I want the system to ask me missing requirement questions, so that the app is not built from an ambiguous idea.

**Acceptance Criteria**

Given I submitted an app idea, when the AI detects missing information, then it presents questions about purpose, users, MVP scope, screens, data, completion conditions, and risks.

Checklist:
- [ ] Questions are shown in GUI-friendly form.
- [ ] Answers are saved and linked to requirements.
- [ ] The user can choose an "Other" style answer when predefined options do not fit.
- [ ] The system detects unanswered required questions before proceeding.

#### US-004: Review Organized Requirements

As a non-engineer app creator, I want to review organized requirements, so that I can verify what the AI understood before design begins.

**Acceptance Criteria**

Given I have answered the requirement questions, when the AI organizes requirements, then I can see the app purpose, users, included features, excluded features, screens, data, completion conditions, and risks.

Checklist:
- [ ] Requirements are shown in non-technical language.
- [ ] Technical details are available in an expandable section.
- [ ] The user can request changes before approval.
- [ ] Requirements reference the original idea and answers.

#### US-005: Approve Requirements

As a non-engineer app creator, I want to explicitly approve requirements, so that downstream design and generation are based on agreed scope.

**Acceptance Criteria**

Given requirements are displayed, when I approve them, then the system records the approval and enables the next AI-DLC stage.

Checklist:
- [ ] Approval requires an explicit user action.
- [ ] Approval timestamp and actor are recorded.
- [ ] The system prevents code generation before requirements approval.
- [ ] The user can request changes instead of approving.

### Epic 2: Scope, Design, and Task Preparation

#### US-006: Review MVP Scope

As a non-engineer app creator, I want to review the MVP scope, so that I understand what will and will not be built first.

**Acceptance Criteria**

Given requirements are approved, when the system proposes an MVP scope, then it clearly separates included and excluded capabilities.

Checklist:
- [ ] MVP is constrained to internal CRUD applications.
- [ ] High-risk or complex features are marked out of scope.
- [ ] Post-MVP items are not mixed into MVP commitments.
- [ ] The user can request scope changes before design approval.

#### US-007: Review Generated Screen List

As a non-engineer app creator, I want to review generated screens, so that I can confirm the app will support the intended workflow.

**Acceptance Criteria**

Given MVP scope is available, when the AI generates screens, then each screen includes purpose, main actions, and related data.

Checklist:
- [ ] Screen list covers administrator and general user basics for the generated CRUD app.
- [ ] Screen descriptions use non-engineer language.
- [ ] Each screen maps to a user need or requirement.
- [ ] Missing obvious CRUD screens are flagged for review.

#### US-008: Review PostgreSQL and Prisma Data Model

As a non-engineer app creator, I want to review a simplified data model, so that I can confirm the app stores the right business information.

**Acceptance Criteria**

Given the screen list and requirements are available, when the AI generates a data model, then it shows entities, fields, relationships, and constraints for PostgreSQL and Prisma.

Checklist:
- [ ] Data model is limited to PostgreSQL + Prisma for MVP.
- [ ] Business labels are shown before database terminology.
- [ ] Required fields and relationships are visible.
- [ ] DynamoDB is not included in the MVP data model.

#### US-009: Approve Design

As a non-engineer app creator, I want to approve the proposed design, so that implementation does not begin from an unreviewed structure.

**Acceptance Criteria**

Given screen and data design are displayed, when I approve the design, then the system records the decision and allows task decomposition or code planning to continue.

Checklist:
- [ ] Design approval is a distinct story and UI action.
- [ ] The user can request design changes before approval.
- [ ] Approval history includes timestamp and actor.
- [ ] Security warnings are visible before approval.

#### US-010: Review Implementation Task Breakdown

As a non-engineer app creator, I want implementation work broken into understandable tasks, so that I can track what the AI will build.

**Acceptance Criteria**

Given design is approved, when the AI decomposes work, then each task has purpose, affected area, and expected result.

Checklist:
- [ ] Tasks are small enough to connect to implementation and testing.
- [ ] Tasks use GUI wording before technical file names.
- [ ] Tasks can be traced to requirements or screens.
- [ ] Risky tasks are marked for explicit review.

### Epic 3: GitHub Integration and Code Generation

#### US-011: Connect or Create a GitHub Repository

As a platform operator or non-engineer app creator, I want to connect or create a GitHub repository, so that generated code can be tracked and reviewed.

**Acceptance Criteria**

Given project setup is ready, when I connect GitHub, then the system can create or connect a repository and prepare a branch for generated changes.

Checklist:
- [ ] GitHub setup uses least-privilege permissions.
- [ ] The UI describes Pull Requests as "変更内容の確認".
- [ ] Repository and branch operations are recorded in history.
- [ ] GitHub setup errors are summarized in non-engineer language.

#### US-012: Generate a Next.js CRUD Template

As a non-engineer app creator, I want the platform to generate a Next.js CRUD template, so that I can start from a working baseline.

**Acceptance Criteria**

Given requirements, design, and GitHub setup are ready, when template generation runs, then the system creates a Next.js, TypeScript, Tailwind CSS, shadcn/ui, Prisma, PostgreSQL-ready project.

Checklist:
- [ ] Template supports administrator and general user basics.
- [ ] Backend uses Next.js Route Handlers or Hono-compatible structure.
- [ ] Lock files and dependency versions are tracked.
- [ ] No secrets are hardcoded.

#### US-013: Review Generated Code

As a technical reviewer or non-engineer app creator, I want to review generated code through a summarized view, so that I can understand what changed before approving it.

**Acceptance Criteria**

Given AI generated code, when I open the review view, then I see a human-readable summary, changed areas, risk notes, and technical details on demand.

Checklist:
- [ ] Changes are summarized before raw diffs.
- [ ] Technical details are available but not the default focus.
- [ ] Scope expansion is highlighted.
- [ ] Security and test implications are shown.

#### US-014: Approve Implementation Diff

As a non-engineer app creator, I want to approve implementation diffs, so that only accepted changes are applied to GitHub.

**Acceptance Criteria**

Given generated code changes are ready, when I approve the implementation diff, then the system records approval and allows the changes to be pushed or included in a Pull Request.

Checklist:
- [ ] Approval is explicit.
- [ ] Unapproved changes are not pushed.
- [ ] Approval history records actor, timestamp, and summary.
- [ ] The user can reject or request changes.

#### US-015: Block Unapproved AI Scope Expansion

As a technical reviewer, I want the system to detect when AI expands scope, so that unintended functionality is not silently added.

**Acceptance Criteria**

Given generated code contains behavior not mapped to approved requirements, when the system reviews the diff, then it marks the change as scope expansion and requires approval or removal.

Checklist:
- [ ] Scope expansion is shown separately from expected changes.
- [ ] Unapproved production-impacting changes are blocked.
- [ ] The user can accept, reject, or request explanation.
- [ ] The audit log records the decision.

### Epic 4: Testing and Quality Review

#### US-016: Run Tests

As a non-engineer app creator, I want to run tests from the GUI, so that I can confirm the generated app works before deployment.

**Acceptance Criteria**

Given implementation changes are ready, when I run tests, then the system executes configured tests and displays pass, fail, or not-run status.

Checklist:
- [ ] Unit, integration, and E2E test categories can be represented.
- [ ] Test status is understandable without reading raw logs first.
- [ ] Failures include a concise AI-generated cause summary.
- [ ] Logs are available in details.

#### US-017: Approve Test Results

As a non-engineer app creator, I want to approve test results, so that deployment only proceeds after I understand the test outcome.

**Acceptance Criteria**

Given test results are available, when I approve them, then the system records approval and unlocks deployment preparation.

Checklist:
- [ ] Test result approval is a distinct gate.
- [ ] Failed tests cannot be approved without explicit override and reason.
- [ ] The system explains risk in non-engineer language.
- [ ] Approval history is recorded.

#### US-018: Validate PBT-Applicable Logic

As a technical reviewer, I want PBT-applicable logic to be identified and tested, so that validation, data transformation, serialization, normalization, and schema checks are robust.

**Acceptance Criteria**

Given generated code includes PBT-applicable functions, when test planning or execution runs, then the system includes property-based tests for the configured scope.

Checklist:
- [ ] PBT applies to pure functions, validation, data transformations, serialization/deserialization, normalization, and AI-generated artifact schema validation.
- [ ] PBT does not block UI components, screen transitions, AWS deployment operations, GitHub API calls, LLM calls, or E2E tests in MVP.
- [ ] PBT failures show a reproducible summary when available.
- [ ] PBT coverage is visible in test results.

### Epic 5: Deployment Preparation and Amplify Deployment

#### US-019: Configure App Settings

As a non-engineer app creator, I want to provide required app settings, so that the generated app can connect to services without exposing secrets.

**Acceptance Criteria**

Given deployment preparation starts, when required settings such as database connection are missing, then the system explains what is missing and how to provide it.

Checklist:
- [ ] Environment variables are shown as "アプリ設定".
- [ ] `DATABASE_URL` missing state is explained in non-engineer language.
- [ ] Secrets are never shown in full after entry.
- [ ] Settings changes require confirmation when they affect deployment.

#### US-020: Run Deployment Pre-Checks

As a non-engineer app creator, I want deployment pre-checks to run before release, so that unsafe or incomplete deployments are blocked.

**Acceptance Criteria**

Given the app is ready for deployment, when pre-checks run, then the system checks build status, app settings, database connectivity, external publication boundary, and security blockers.

Checklist:
- [ ] External publication without authentication or access restriction is blocked.
- [ ] Hardcoded secrets are blocking findings.
- [ ] Overbroad IAM, dangerous AWS deletion, improper user data transmission, and unapproved production deployment are blocking findings.
- [ ] Minor security findings are shown as warnings.

#### US-021: Approve Deployment

As a non-engineer app creator, I want to explicitly approve deployment, so that the app is not published by accident.

**Acceptance Criteria**

Given pre-checks pass, when I approve deployment, then the system records deployment approval and allows AWS Amplify Hosting deployment.

Checklist:
- [ ] Deployment approval is a distinct gate.
- [ ] The UI shows whether external publication is involved.
- [ ] Potential cost-impacting operations require confirmation.
- [ ] Approval history records actor, timestamp, and target environment.

#### US-022: Deploy to AWS Amplify Hosting

As a non-engineer app creator, I want the platform to deploy through AWS Amplify Hosting, so that I can publish the generated app from the GUI.

**Acceptance Criteria**

Given deployment is approved and GitHub is connected, when deployment starts, then the system triggers AWS Amplify Hosting deployment and tracks status.

Checklist:
- [ ] Amplify deployment uses GitHub integration.
- [ ] Deployment status is displayed in the GUI.
- [ ] Deployment failures are summarized in non-engineer language.
- [ ] Deployment history is recorded.

#### US-023: View Deployment Result and Public URL

As a non-engineer app creator, I want to see deployment result and URL, so that I know whether the app is available.

**Acceptance Criteria**

Given deployment finishes, when I view deployment result, then I see success or failure, public URL if available, and next recommended action.

Checklist:
- [ ] Successful deployment shows the published URL.
- [ ] Failed deployment shows likely cause and next action.
- [ ] External access restrictions are visible.
- [ ] Deployment result links to relevant history.

### Epic 6: Post-Deployment Review and Improvement Cycle

#### US-024: Review Logs and Errors in Plain Language

As a non-engineer app creator, I want deployment and runtime errors summarized in plain language, so that I can understand what needs attention.

**Acceptance Criteria**

Given the app has deployment or runtime logs, when I open the operations view, then the system shows a plain-language summary and links to details.

Checklist:
- [ ] CloudWatch-style logs are shown as "エラー内容" or equivalent user-facing wording.
- [ ] Sensitive data is not displayed in summaries.
- [ ] The summary separates user action needed from technical detail.
- [ ] Related deployment or test history is linked.

#### US-025: Create Improvement Suggestions

As a non-engineer app creator, I want the AI to suggest improvements after deployment, so that I can start the next AI-DLC cycle.

**Acceptance Criteria**

Given deployment result, logs, or user feedback exist, when improvement analysis runs, then the system proposes improvement tasks that can be reviewed before entering the next cycle.

Checklist:
- [ ] Suggestions are separated from automatically approved work.
- [ ] Each suggestion includes reason and expected benefit.
- [ ] Suggestions can become new project tasks or next-cycle requirements.
- [ ] High-risk suggestions require explicit review.

### Epic 7: Cross-Cutting Safety, Language, Audit, and AI Provider Controls

#### US-026: Use Non-Engineer Language Across the GUI

As a non-engineer app creator, I want technical concepts translated into familiar language, so that I can make decisions without needing development background.

**Acceptance Criteria**

Given a screen contains technical concepts, when the UI displays them, then it uses non-engineer terms first and provides technical detail only on demand.

Checklist:
- [ ] Pull Request is shown as "変更内容の確認".
- [ ] CI/CD is shown as "自動チェック・自動公開".
- [ ] Environment variables are shown as "アプリ設定".
- [ ] CloudWatch logs are shown as "エラー内容".
- [ ] Technical detail is available in expandable sections.

#### US-027: Track Approval and Operation History

As a platform operator, I want approval, GitHub, and deployment history recorded, so that decisions and changes are auditable.

**Acceptance Criteria**

Given a user approves or performs a tracked operation, when the action completes, then the system records actor, timestamp, action type, and summary.

Checklist:
- [ ] Requirement, design, implementation diff, test result, and deployment approvals are recorded.
- [ ] GitHub operations are recorded.
- [ ] Deployment operations are recorded.
- [ ] AI internal logs are summarized rather than stored as full raw logs.

#### US-028: Enforce Security Blockers and Warnings

As a platform operator, I want critical security issues blocked and minor findings warned, so that the platform remains safe without stopping every low-risk issue.

**Acceptance Criteria**

Given the system detects security findings, when a finding is critical, then it blocks progression until resolved or explicitly handled according to policy.

Checklist:
- [ ] Hardcoded secrets are blocking.
- [ ] Overbroad IAM permissions are blocking.
- [ ] External publication without authentication or access restriction is blocking.
- [ ] Dangerous AWS resource deletion is blocking.
- [ ] Improper user data transmission is blocking.
- [ ] Unapproved production deployment is blocking.
- [ ] Minor findings are warnings with priority guidance.

#### US-029: Configure AI Provider Adapter

As a platform operator, I want AI provider access abstracted through an adapter, so that the MVP can prioritize Amazon Bedrock while allowing future provider expansion.

**Acceptance Criteria**

Given the platform needs AI generation, when provider configuration is set, then the system uses the AI Provider Adapter rather than hardcoding provider-specific logic into product workflows.

Checklist:
- [ ] Amazon Bedrock is the priority initial provider.
- [ ] Provider-specific details are isolated behind an adapter.
- [ ] Adding another provider does not require rewriting AI-DLC workflow screens.
- [ ] BYO AI Account is not required in MVP but is not blocked by the design.

## INVEST Review

| Story Range | Independent | Negotiable | Valuable | Estimable | Small | Testable |
|---|---|---|---|---|---|---|
| US-001 to US-005 | Yes | Yes | Yes | Yes | Yes | Yes |
| US-006 to US-010 | Yes | Yes | Yes | Yes | Yes | Yes |
| US-011 to US-015 | Yes | Yes | Yes | Yes | Yes | Yes |
| US-016 to US-018 | Yes | Yes | Yes | Yes | Yes | Yes |
| US-019 to US-023 | Yes | Yes | Yes | Yes | Yes | Yes |
| US-024 to US-029 | Yes | Yes | Yes | Yes | Yes | Yes |

## Story to Requirement Traceability

| Requirement | Story IDs |
|---|---|
| FR-001 Project Creation | US-001 |
| FR-002 Idea Input | US-002 |
| FR-003 AI Requirements Question Generation | US-003 |
| FR-004 Requirements Organization | US-004, US-005 |
| FR-005 MVP Scope Definition | US-006 |
| FR-006 Screen List Generation | US-007 |
| FR-007 Data Model Generation | US-008 |
| FR-008 Task Decomposition | US-010 |
| FR-009 Human Approval Gates | US-005, US-009, US-014, US-017, US-021 |
| FR-010 GitHub Integration | US-011, US-014, US-022 |
| FR-011 Next.js Template Generation | US-012 |
| FR-012 AI Code Generation | US-013, US-014, US-015 |
| FR-013 Test Execution and Summary | US-016, US-017, US-018 |
| FR-014 Deployment Pre-Check | US-019, US-020 |
| FR-015 AWS Amplify Deployment | US-021, US-022, US-023 |
| FR-016 Audit and History | US-027 |
| FR-017 AI Provider Adapter | US-029 |
| NFR-001 Safety | US-005, US-009, US-014, US-017, US-020, US-021, US-028 |
| NFR-002 Security | US-020, US-028 |
| NFR-003 Usability | US-026 |
| NFR-004 Auditability | US-027 |
| NFR-005 Testability | US-016, US-018 |
| NFR-007 Deployment Reliability | US-020, US-022, US-023 |
| NFR-008 Accessibility and Responsive UI | US-007, US-012, US-026 |
| NFR-009 Cost Awareness | US-021 |

## Post-MVP Candidate Stories

These are intentionally not written at MVP-level detail.

- PM-001: As a user, I want to bring my own AI account/API key, so that I can choose Bedrock, OpenAI, Anthropic, Gemini, or another provider.
- PM-002: As a user, I want mobile native app generation, so that I can create apps beyond responsive web.
- PM-003: As a platform operator, I want App Runner deployment support, so that backend-heavy apps can be deployed outside Amplify Hosting.
- PM-004: As an administrator, I want advanced role-based access control, so that larger teams can govern who may approve, deploy, or modify settings.
- PM-005: As a user, I want richer operations dashboards, so that logs, metrics, incidents, and improvement tasks are managed in one place.
- PM-006: As a user, I want support for additional databases and data stores, so that DynamoDB or other storage options can be selected when appropriate.

## Extension Compliance Summary

### Security Baseline

Status: Enabled with custom severity.

- Blocking security concerns are represented in US-020 and US-028.
- Approval and audit requirements are represented in US-005, US-009, US-014, US-017, US-021, and US-027.
- External publication without authentication or access restriction is explicitly blocked.

No blocking security finding exists in the user story artifacts.

### Property-Based Testing

Status: Enabled with custom partial enforcement.

- PBT-applicable scope is represented in US-018.
- PBT is included in test result visibility and traceability through US-016 and US-017.
- Non-applicable areas for MVP are explicitly excluded in US-018.

No blocking PBT finding exists in the user story artifacts.
