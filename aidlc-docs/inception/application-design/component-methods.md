# Component Methods

## Method Signature Conventions

- Method signatures are TypeScript-oriented because the MVP stack is Next.js, Node.js, TypeScript, and Prisma.
- This document defines high-level interfaces only. Detailed business rules will be defined later during Functional Design.
- All methods that perform long-running work return or reference an asynchronous `Job`.

## Shared Types

```ts
type ID = string;
type ISODateTime = string;

type Actor = {
  id: ID;
  displayName: string;
  role: "creator" | "operator" | "reviewer" | "system";
};

type Result<T> = {
  ok: boolean;
  value?: T;
  error?: UserFacingError;
};

type UserFacingError = {
  code: string;
  message: string;
  technicalDetailsRef?: ID;
  nextAction?: string;
};

type JobStatus = "queued" | "running" | "succeeded" | "failed" | "cancelled";

type Job = {
  id: ID;
  projectId: ID;
  type: string;
  status: JobStatus;
  progressLabel: string;
  createdAt: ISODateTime;
  updatedAt: ISODateTime;
};
```

## Project Workspace Component

```ts
createProject(input: CreateProjectInput, actor: Actor): Promise<Result<Project>>;
getProject(projectId: ID, actor: Actor): Promise<Result<Project>>;
listProjects(actor: Actor): Promise<Result<ProjectSummary[]>>;
getProjectDashboard(projectId: ID, actor: Actor): Promise<Result<ProjectDashboard>>;
archiveProject(projectId: ID, actor: Actor): Promise<Result<Project>>;
```

Purpose:
- Create and retrieve projects.
- Provide dashboard data to the GUI.
- Enforce the internal CRUD application MVP boundary.

## AI-DLC Workflow Orchestrator

```ts
initializeWorkflow(projectId: ID, actor: Actor): Promise<Result<WorkflowState>>;
getWorkflowState(projectId: ID, actor: Actor): Promise<Result<WorkflowState>>;
advanceStage(input: AdvanceStageInput, actor: Actor): Promise<Result<WorkflowState>>;
canAdvanceStage(input: AdvanceStageInput, actor: Actor): Promise<Result<AdvanceDecision>>;
blockStage(input: BlockStageInput, actor: Actor): Promise<Result<WorkflowState>>;
```

Purpose:
- Manage phase and stage progression.
- Check required approval gates before transitions.
- Prevent unapproved generation or deployment.

## Requirements Intake Component

```ts
saveIdea(input: SaveIdeaInput, actor: Actor): Promise<Result<Idea>>;
generateRequirementQuestions(projectId: ID, actor: Actor): Promise<Result<Job>>;
saveRequirementAnswers(input: RequirementAnswersInput, actor: Actor): Promise<Result<RequirementAnswerSet>>;
generateRequirementsDocument(projectId: ID, actor: Actor): Promise<Result<Job>>;
getRequirementsSummary(projectId: ID, actor: Actor): Promise<Result<RequirementsSummary>>;
```

Purpose:
- Capture natural language ideas.
- Generate and store clarification questions.
- Produce requirement artifacts from answers.

## Story and Artifact Manager

```ts
createArtifact(input: CreateArtifactInput, actor: Actor): Promise<Result<Artifact>>;
getArtifact(artifactId: ID, actor: Actor): Promise<Result<Artifact>>;
listArtifacts(projectId: ID, filter: ArtifactFilter, actor: Actor): Promise<Result<ArtifactSummary[]>>;
linkArtifacts(input: ArtifactLinkInput, actor: Actor): Promise<Result<ArtifactLink>>;
getTraceability(projectId: ID, actor: Actor): Promise<Result<TraceabilityMap>>;
```

Purpose:
- Store and link generated artifacts.
- Keep requirements, stories, plans, designs, tasks, and summaries traceable.

## Application Design Assistant

```ts
generateDesignQuestions(projectId: ID, actor: Actor): Promise<Result<Job>>;
saveDesignAnswers(input: DesignAnswersInput, actor: Actor): Promise<Result<DesignAnswerSet>>;
generateApplicationDesign(projectId: ID, actor: Actor): Promise<Result<Job>>;
getApplicationDesignSummary(projectId: ID, actor: Actor): Promise<Result<ApplicationDesignSummary>>;
```

Purpose:
- Support Application Design stage.
- Generate component, method, service, dependency, and consolidated design artifacts.

## Task Planning Component

```ts
generateTaskPlan(projectId: ID, actor: Actor): Promise<Result<Job>>;
listTasks(projectId: ID, actor: Actor): Promise<Result<ImplementationTask[]>>;
getTask(taskId: ID, actor: Actor): Promise<Result<ImplementationTask>>;
markTaskRisk(input: TaskRiskInput, actor: Actor): Promise<Result<ImplementationTask>>;
linkTaskToStory(input: TaskStoryLinkInput, actor: Actor): Promise<Result<TaskStoryLink>>;
```

Purpose:
- Create implementation tasks from approved design.
- Maintain traceability to stories and requirements.

## AI Provider Adapter

```ts
listProviders(actor: Actor): Promise<Result<AIProviderInfo[]>>;
getProviderCapabilities(providerId: ID, actor: Actor): Promise<Result<AIProviderCapabilities>>;
generateText(input: GenerateTextInput, actor: Actor): Promise<Result<AITextResponse>>;
generateStructured<T>(input: GenerateStructuredInput<T>, actor: Actor): Promise<Result<T>>;
validateProviderConfig(providerId: ID, actor: Actor): Promise<Result<ProviderConfigStatus>>;
```

Purpose:
- Provide a provider-neutral interface.
- Support Bedrock first while allowing future providers.

## Job Orchestrator and Worker

```ts
enqueueJob(input: EnqueueJobInput, actor: Actor): Promise<Result<Job>>;
getJob(jobId: ID, actor: Actor): Promise<Result<Job>>;
listJobs(projectId: ID, filter: JobFilter, actor: Actor): Promise<Result<Job[]>>;
cancelJob(jobId: ID, actor: Actor): Promise<Result<Job>>;
retryJob(jobId: ID, actor: Actor): Promise<Result<Job>>;
```

Purpose:
- Run all AI, GitHub, test, and deployment work asynchronously.
- Expose progress to the GUI.

## GitHub Integration Component

```ts
connectRepository(input: ConnectRepositoryInput, actor: Actor): Promise<Result<GitHubRepository>>;
createRepository(input: CreateRepositoryInput, actor: Actor): Promise<Result<GitHubRepository>>;
createBranch(input: CreateBranchInput, actor: Actor): Promise<Result<GitHubBranch>>;
createPullRequest(input: CreatePullRequestInput, actor: Actor): Promise<Result<GitHubPullRequest>>;
getPullRequestDiff(prId: ID, actor: Actor): Promise<Result<GitHubDiff>>;
listPullRequestChecks(prId: ID, actor: Actor): Promise<Result<GitHubCheckRun[]>>;
createPullRequestComment(input: PullRequestCommentInput, actor: Actor): Promise<Result<GitHubComment>>;
```

Purpose:
- Manage GitHub operations required for generated code review.
- Retrieve diffs and checks for GUI summaries.

## Code Generation and Diff Review Component

```ts
generateCrudTemplate(projectId: ID, actor: Actor): Promise<Result<Job>>;
generateCodeForTask(input: GenerateCodeInput, actor: Actor): Promise<Result<Job>>;
summarizeDiff(input: DiffSummaryInput, actor: Actor): Promise<Result<DiffSummary>>;
detectScopeExpansion(input: ScopeExpansionInput, actor: Actor): Promise<Result<ScopeExpansionReport>>;
prepareImplementationReview(input: ImplementationReviewInput, actor: Actor): Promise<Result<ImplementationReview>>;
```

Purpose:
- Generate templates and task-specific changes.
- Summarize changes and detect unapproved scope expansion.

## Test Orchestration Component

```ts
runTests(input: RunTestsInput, actor: Actor): Promise<Result<Job>>;
getTestResult(testRunId: ID, actor: Actor): Promise<Result<TestResult>>;
summarizeTestFailure(input: TestFailureSummaryInput, actor: Actor): Promise<Result<TestFailureSummary>>;
declarePbtProperties(input: PbtDeclarationInput, actor: Actor): Promise<Result<PbtPropertySet>>;
getPbtCoverage(projectId: ID, actor: Actor): Promise<Result<PbtCoverageSummary>>;
```

Purpose:
- Run and summarize tests.
- Track PBT-applicable declarations from domain components.

## Deployment Pre-Check Component

```ts
runPreChecks(input: DeploymentPreCheckInput, actor: Actor): Promise<Result<DeploymentPreCheckReport>>;
checkAppSettings(projectId: ID, actor: Actor): Promise<Result<AppSettingsCheck>>;
checkExternalPublication(input: PublicationCheckInput, actor: Actor): Promise<Result<SecurityDecision>>;
checkDatabaseConnectivity(projectId: ID, actor: Actor): Promise<Result<DatabaseConnectivityCheck>>;
```

Purpose:
- Validate deployment readiness.
- Block unsafe external publication and missing required settings.

## AWS Amplify Deployment Component

```ts
createAmplifyApp(input: CreateAmplifyAppInput, actor: Actor): Promise<Result<AmplifyApp>>;
connectAmplifyToGitHub(input: AmplifyGitHubConnectionInput, actor: Actor): Promise<Result<AmplifyConnection>>;
triggerDeployment(input: TriggerDeploymentInput, actor: Actor): Promise<Result<Job>>;
getDeploymentStatus(deploymentId: ID, actor: Actor): Promise<Result<DeploymentStatus>>;
getDeploymentResult(deploymentId: ID, actor: Actor): Promise<Result<DeploymentResult>>;
```

Purpose:
- Create/configure Amplify applications.
- Trigger and track deployments.

## Security Policy Component

```ts
evaluateAction(input: SecurityActionInput, actor: Actor): Promise<Result<SecurityDecision>>;
evaluateDeployment(input: SecurityDeploymentInput, actor: Actor): Promise<Result<SecurityDecision>>;
scanForSecrets(input: SecretScanInput, actor: Actor): Promise<Result<SecurityFinding[]>>;
classifyFinding(input: SecurityFindingInput, actor: Actor): Promise<Result<SecurityFindingClassification>>;
```

Purpose:
- Centralize blocking/warning policy.
- Provide security decisions to other services.

## Approval Gate Component

```ts
requestApproval(input: ApprovalRequestInput, actor: Actor): Promise<Result<Approval>>;
recordApprovalDecision(input: ApprovalDecisionInput, actor: Actor): Promise<Result<Approval>>;
getApproval(approvalId: ID, actor: Actor): Promise<Result<Approval>>;
listApprovals(projectId: ID, filter: ApprovalFilter, actor: Actor): Promise<Result<Approval[]>>;
```

Purpose:
- Manage explicit approval gates separate from audit events.

## Audit History Component

```ts
recordAuditEvent(input: AuditEventInput, actor: Actor): Promise<Result<AuditEvent>>;
listAuditEvents(projectId: ID, filter: AuditEventFilter, actor: Actor): Promise<Result<AuditEvent[]>>;
getAuditSummary(projectId: ID, actor: Actor): Promise<Result<AuditSummary>>;
linkAuditToApproval(input: AuditApprovalLinkInput, actor: Actor): Promise<Result<AuditApprovalLink>>;
```

Purpose:
- Store traceable history for approvals, GitHub operations, deployment, and AI summaries.

## Explanation/Translation Component

```ts
translateTerm(input: TermTranslationInput, actor: Actor): Promise<Result<TermTranslation>>;
summarizeTechnicalError(input: TechnicalErrorInput, actor: Actor): Promise<Result<UserFacingError>>;
createNextAction(input: NextActionInput, actor: Actor): Promise<Result<UserFacingNextAction>>;
translateServiceResult<T>(input: ServiceResultTranslationInput<T>, actor: Actor): Promise<Result<UserFacingResult>>;
```

Purpose:
- Convert technical output into non-engineer language.

## Operations Feedback Component

```ts
summarizeLogs(input: LogSummaryInput, actor: Actor): Promise<Result<Job>>;
generateImprovementSuggestions(input: ImprovementSuggestionInput, actor: Actor): Promise<Result<Job>>;
getOperationsFeedback(projectId: ID, actor: Actor): Promise<Result<OperationsFeedbackSummary>>;
```

Purpose:
- Define MVP boundaries for log summary and improvement suggestions.
- Full next-cycle automation remains later-stage detail.

## Data Access Component

```ts
runInTransaction<T>(input: TransactionInput<T>): Promise<Result<T>>;
getRepository<T>(name: RepositoryName): Repository<T>;
saveDomainEvent(input: DomainEventInput): Promise<Result<void>>;
enforceProjectScope(projectId: ID, actor: Actor): Promise<Result<ProjectScopeDecision>>;
```

Purpose:
- Encapsulate Prisma persistence and project scoping.

