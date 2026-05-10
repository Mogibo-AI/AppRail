# Unit of Work

## 目的

このドキュメントは、AI-DLC GUI Application Development Platform のMVPを、設計・実装・テスト・レビューしやすい作業単位へ分解する。ユニットは、承認済みのアプリケーション設計、コンポーネント設計、ユーザーストーリー、ユニット生成計画に基づいて定義する。

## 入力成果物

- `aidlc-docs/inception/requirements/requirements.md`
- `aidlc-docs/inception/user-stories/stories.md`
- `aidlc-docs/inception/user-stories/stories-ja.md`
- `aidlc-docs/inception/application-design/application-design.md`
- `aidlc-docs/inception/application-design/components.md`
- `aidlc-docs/inception/application-design/component-methods.md`
- `aidlc-docs/inception/application-design/services.md`
- `aidlc-docs/inception/application-design/component-dependency.md`
- `aidlc-docs/inception/plans/unit-of-work-plan.md`

## 分解方針

- ユーザージャーニーとサービス責務のハイブリッドで分解する。
- UI Experience Shell は独立ユニットとして扱う。
- Security Policy と Approval/Audit は別ユニットにする。
- Job Orchestration と AI Provider Adapter は別ユニットにする。
- GitHub Integration と Code Generation/Diff Review は別ユニットにする。
- Deployment Pre-Check と AWS Amplify Deployment は同一ユニットにする。
- Operations Feedback はMVPユニットに含めるが、ログ要約と改善提案の境界定義に絞る。
- 横断ストーリーは主担当ユニットを1つ定め、副担当ユニットを依存として記録する。

## Greenfield Code Organization Strategy

対象アプリケーションコードは `aidlc-docs/` の外に配置する。MVP実装は単一チーム前提だが、将来のチーム分担に備えて所有境界を明確にする。

| Path | 役割 | 主な所有ユニット |
|---|---|---|
| `apps/web/` | Next.js UI/API、画面、Route Handlers、BFF的なAPI境界 | U13, 各ドメインユニット |
| `apps/worker/` | 非同期ジョブ実行、AI/GitHub/Test/Deployワーカー | U4 |
| `packages/project-workflow/` | プロジェクト、ワークフロー状態、フェーズ/ステージ遷移 | U1 |
| `packages/requirements-artifacts/` | アイデア入力、質問、要件、ストーリー、成果物管理 | U2 |
| `packages/design-task-planning/` | 画面設計、データ設計、タスク分解 | U3 |
| `packages/job-platform/` | ジョブキュー、状態、リトライ、キャンセル、進捗 | U4 |
| `packages/ai-provider/` | provider-neutral AI interface、Bedrock provider | U5 |
| `packages/github-integration/` | GitHub repo/branch/PR/diff/checks | U6 |
| `packages/code-generation-review/` | テンプレート生成、コード生成、差分要約、スコープ逸脱検知 | U7 |
| `packages/testing-quality/` | テスト実行、結果正規化、PBT対象宣言 | U8 |
| `packages/deployment-amplify/` | アプリ設定、デプロイ前チェック、Amplifyデプロイ | U9 |
| `packages/security-policy/` | blocker/warning分類、危険操作判定 | U10 |
| `packages/approval-audit/` | 承認ゲート、監査イベント、履歴 | U11 |
| `packages/explanation-ops/` | 非エンジニア向け表現、エラー要約、ログ要約、改善提案境界 | U12 |
| `packages/shared-types/` | 共有型、イベント型、ジョブ契約、DTO | U1 governs, all units consume |
| `packages/data-access/` | Prisma repository、transaction、project scoped persistence | U1 governs, all units consume |

## Unit Overview

| Unit | 名称 | 所有責務 | 対応コンポーネント | Primary Stories |
|---|---|---|---|---|
| U1 | Project, Workflow, and Data Access Core | プロジェクト作成、ワークフロー状態、共有型、Prisma永続化境界 | Project Workspace, AI-DLC Workflow Orchestrator, Data Access | US-001 |
| U2 | Requirements, Stories, and Artifacts | アイデア、質問、要件、MVPスコープ、成果物/ストーリー管理 | Requirements Intake, Story and Artifact Manager | US-002, US-003, US-004, US-006 |
| U3 | Design and Task Planning | 画面一覧、データモデル、設計成果物、タスク分解 | Application Design Assistant, Task Planning | US-007, US-008, US-010 |
| U4 | Job Orchestration Platform | 長時間処理の非同期実行、進捗、リトライ、キャンセル | Job Orchestrator and Worker | Supporting unit |
| U5 | AI Provider Adapter | Bedrock-firstのprovider-neutral AI実行基盤 | AI Provider Adapter | US-029 |
| U6 | GitHub Integration | repo作成/接続、branch、PR、diff、check results | GitHub Integration | US-011 |
| U7 | Code Generation and Diff Review | Next.jsテンプレート生成、コード生成、差分確認、スコープ逸脱検知 | Code Generation and Diff Review | US-012, US-013, US-015 |
| U8 | Testing and Quality | テスト実行、結果要約、PBT対象ロジック検証 | Test Orchestration | US-016, US-018 |
| U9 | Deployment and Amplify Integration | アプリ設定、デプロイ前チェック、Amplifyデプロイ、公開URL | Deployment Pre-Check, AWS Amplify Deployment | US-019, US-020, US-022, US-023 |
| U10 | Security Policy | blocker/warning分類、危険操作と外部公開の判定 | Security Policy | US-028 |
| U11 | Approval and Audit | 要件/設計/実装差分/テスト/デプロイ承認、操作履歴 | Approval Gate, Audit History | US-005, US-009, US-014, US-017, US-021, US-027 |
| U12 | Explanation and Operations Feedback | 非エンジニア向け表現、エラー/ログ要約、改善提案境界 | Explanation/Translation, Operations Feedback | US-024, US-025, US-026 |
| U13 | UI Experience Shell | Next.js画面、ナビゲーション、カード、チェックリスト、承認UI | UI shell using all service APIs | Supporting unit |

## Unit Details

### U1: Project, Workflow, and Data Access Core

**Scope**

- 開発プロジェクトの作成、取得、一覧、現在状態の管理。
- AI-DLCのPhase、Stage、Approval Gate参照、次アクション判定。
- `packages/shared-types/` と `packages/data-access/` の基本方針管理。
- Prisma repositoryとtransaction境界の提供。

**Out of Scope**

- 要件や設計の生成内容そのもの。
- 承認判断の保存詳細。これはU11が所有する。
- セキュリティ分類ロジック。これはU10が所有する。

**Primary Interfaces**

- API: `createProject`, `getProjectDashboard`, `getWorkflowState`, `advanceWorkflow`
- Events: `project.created`, `workflow.stageChanged`
- Data: `Project`, `WorkflowPhase`, `WorkflowStage`, `WorkflowGateRef`

### U2: Requirements, Stories, and Artifacts

**Scope**

- 自然文アイデア入力。
- AIによる要件確認質問生成依頼。
- 回答保存、要件整理、MVPスコープ、除外スコープの管理。
- 要件、ストーリー、成果物バージョン、トレーサビリティ管理。

**Out of Scope**

- 詳細設計と実装タスク分解。これはU3が所有する。
- AI provider固有の呼び出し。これはU5が所有する。

**Primary Interfaces**

- API: `submitIdea`, `generateRequirementQuestions`, `submitRequirementAnswers`, `generateRequirements`, `getArtifact`
- Jobs: `requirements.generateQuestions`, `requirements.generateDocument`
- Events: `requirements.questionsGenerated`, `requirements.documentGenerated`

### U3: Design and Task Planning

**Scope**

- 画面一覧、データモデル、API案、AWS構成案などの設計生成。
- 実装タスク分解、タスクとストーリーの対応付け。
- 設計レビューに必要な説明データの提供。

**Out of Scope**

- 実コード生成。これはU7が所有する。
- GitHub操作。これはU6が所有する。

**Primary Interfaces**

- API: `generateApplicationDesign`, `generateTaskBreakdown`, `getDesignSummary`
- Jobs: `design.generateArtifacts`, `tasks.generateBreakdown`
- Events: `design.generated`, `tasks.generated`

### U4: Job Orchestration Platform

**Scope**

- AI、GitHub、テスト、デプロイなどの長時間処理を非同期ジョブとして扱う。
- ジョブ状態、進捗、結果、失敗、リトライ、キャンセルを管理する。
- `apps/worker/` の実行境界を提供する。

**Out of Scope**

- 各ジョブのドメイン処理本体。ジョブ実行先のユニットが所有する。
- provider固有のAI処理。これはU5が所有する。

**Primary Interfaces**

- API: `enqueueJob`, `getJobStatus`, `cancelJob`, `retryJob`
- Events: `job.enqueued`, `job.started`, `job.completed`, `job.failed`, `job.cancelled`
- Data: `Job`, `JobAttempt`, `JobLogRef`

### U5: AI Provider Adapter

**Scope**

- Bedrock-firstのAI provider実装。
- provider-neutralな生成、構造化出力、model metadata取得。
- AI応答のschema validationとprovider固有エラーの正規化。

**Out of Scope**

- プロンプト内容の業務責務。各ドメインユニットが所有する。
- UI表示文言。これはU12とU13が所有する。

**Primary Interfaces**

- API: `generateText`, `generateStructured`, `getModelCapabilities`
- Jobs: consumed by U2, U3, U7, U12 through U4
- Data: `AiProviderConfig`, `AiGenerationRequest`, `AiGenerationResult`

### U6: GitHub Integration

**Scope**

- GitHubリポジトリ作成または接続。
- branch、Pull Request、diff、comments、check resultsの取得。
- GitHub操作結果の監査イベント化。

**Out of Scope**

- 差分の意味的レビュー。これはU7が所有する。
- デプロイ判断。これはU9とU11が所有する。

**Primary Interfaces**

- API: `connectRepository`, `createRepository`, `createBranch`, `createPullRequest`, `getDiff`, `getCheckResults`
- Jobs: `github.createRepository`, `github.preparePullRequest`
- Events: `github.repositoryConnected`, `github.pullRequestCreated`, `github.checksUpdated`

### U7: Code Generation and Diff Review

**Scope**

- Next.js CRUDテンプレート生成。
- 承認済みタスク単位のコード生成。
- 生成差分の非エンジニア向け要約。
- 承認範囲外のスコープ拡張検知。

**Out of Scope**

- GitHub APIの直接操作。これはU6が所有する。
- テスト実行。これはU8が所有する。

**Primary Interfaces**

- API: `generateTemplate`, `generateTaskImplementation`, `summarizeDiff`, `detectScopeExpansion`
- Jobs: `code.generateTemplate`, `code.generateTaskChanges`, `code.summarizeDiff`
- Events: `code.diffReady`, `code.scopeExpansionDetected`

### U8: Testing and Quality

**Scope**

- ユニット、integration、必要最小限のE2Eテスト実行。
- テスト結果の正規化、失敗理由の要約入力。
- PBT対象ロジックの宣言と検証。

**Out of Scope**

- UI、AWS操作、GitHub API、LLM呼び出し自体へのPBT強制。
- デプロイ可否の最終判断。これはU9/U10/U11が所有する。

**Primary Interfaces**

- API: `runTests`, `getTestResult`, `getQualitySummary`, `registerPbtProperties`
- Jobs: `tests.run`, `tests.normalizeResults`
- Events: `tests.completed`, `tests.failed`, `quality.pbtEvaluated`

### U9: Deployment and Amplify Integration

**Scope**

- アプリ設定、環境変数、DB接続、外部公開境界のチェック。
- デプロイ前チェックとAWS Amplify Hostingへのデプロイ。
- デプロイ結果、公開URL、失敗理由の取得。

**Out of Scope**

- 無承認のAWS操作。
- App RunnerなどMVP外デプロイ方式の実装。

**Primary Interfaces**

- API: `saveAppSettings`, `runDeploymentPreCheck`, `deployToAmplify`, `getDeploymentResult`
- Jobs: `deployment.preCheck`, `deployment.amplifyDeploy`
- Events: `deployment.preCheckCompleted`, `deployment.started`, `deployment.completed`, `deployment.failed`

### U10: Security Policy

**Scope**

- blocker/warning分類。
- hardcoded secrets、過剰IAM、認証なし外部公開、危険な削除、未承認本番反映などの判定。
- リスクの重大度と次アクションの構造化。

**Out of Scope**

- 承認履歴の保存。これはU11が所有する。
- UI表現。これはU12/U13が所有する。

**Primary Interfaces**

- API: `evaluateSecurityPolicy`, `classifyFinding`, `assertNoBlockingFindings`
- Events: `security.findingCreated`, `security.blockerRaised`
- Data: `SecurityFinding`, `SecurityDecision`

### U11: Approval and Audit

**Scope**

- 要件、設計、実装差分、テスト結果、デプロイの承認ゲート。
- 操作履歴、承認履歴、GitHub/デプロイ/AIジョブの監査イベント。
- 承認対象、actor、timestamp、decision、summaryの保存。

**Out of Scope**

- セキュリティ分類そのもの。これはU10が所有する。
- ドメイン成果物の生成。各ドメインユニットが所有する。

**Primary Interfaces**

- API: `requestApproval`, `submitApprovalDecision`, `getApprovalHistory`, `recordAuditEvent`
- Events: `approval.requested`, `approval.approved`, `approval.rejected`, `audit.recorded`
- Data: `Approval`, `ApprovalDecision`, `AuditEvent`

### U12: Explanation and Operations Feedback

**Scope**

- 技術用語を非エンジニア向け表現に変換する。
- エラー、テスト失敗、GitHub失敗、デプロイ失敗、ログの要約。
- 運用改善提案の境界定義と候補作成。

**Out of Scope**

- 本格的なログ収集基盤。
- Post-MVPの高度な運用分析ダッシュボード。

**Primary Interfaces**

- API: `translateTechnicalTerm`, `summarizeError`, `summarizeLogs`, `suggestImprovements`
- Jobs: `ops.summarizeLogs`, `ops.suggestImprovements`
- Events: `explanation.generated`, `ops.improvementSuggested`

### U13: UI Experience Shell

**Scope**

- Next.js画面、ナビゲーション、プロジェクトダッシュボード、ウィザード、カード、チェックリスト。
- 承認UI、ジョブ進捗UI、差分確認UI、デプロイ結果UI。
- 各ドメインユニットのAPIを呼び出すユーザー体験の統合。

**Out of Scope**

- ドメインロジックと外部API処理。
- Prismaへの直接アクセス。

**Primary Interfaces**

- Screens: Project dashboard, requirements wizard, design review, task review, code review, test review, deployment review, operations feedback.
- API consumption: U1からU12の公開API。

## Ownership Boundary

MVP実装は単一チーム前提とする。ただし将来の分担に備え、以下の所有境界を守る。

- U1は共有型とデータアクセス方針を管理するが、各ユニットの業務ルールを吸収しない。
- U4はジョブ基盤だけを所有し、ジョブ内部の業務処理は各ユニットに委譲する。
- U5はprovider固有処理を閉じ込め、ドメインユニットへprovider固有レスポンスを漏らさない。
- U10は安全性の判定を所有し、U11は承認と履歴を所有する。
- U13は画面と体験を所有し、サービスロジックや永続化ロジックを持たない。

## Validation Summary

- MVPストーリー US-001 から US-029 はすべて主担当ユニットへ割り当て済み。
- 承認済みの分割方針に反する統合はない。
- Unit間の詳細依存と実装順序は `unit-of-work-dependency.md` に定義する。
- ストーリー対応の完全な一覧は `unit-of-work-story-map.md` に定義する。

