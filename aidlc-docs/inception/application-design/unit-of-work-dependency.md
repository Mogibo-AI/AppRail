# Unit of Work Dependency

## 目的

このドキュメントは、`unit-of-work.md` で定義したユニット間の依存、実装順序、API/イベント/ジョブ契約を定義する。

## Dependency Principles

- UIはサービスAPIを呼び出し、Prismaへ直接アクセスしない。
- 長時間処理はU4 Job Orchestration Platformを経由する。
- AI生成はU5 AI Provider Adapterを経由する。
- 危険操作、外部公開、AWS操作、生成差分、承認前進行はU10 Security Policyを通す。
- 承認と監査はU11 Approval and Auditを通す。
- ユーザー向け説明はU12 Explanation and Operations Feedbackを通す。
- 外部連携のprovider固有・API固有の詳細をドメインユニットへ漏らさない。

## Unit Dependency Matrix

| Unit | Depends On | Dependency Type | Reason |
|---|---|---|---|
| U1 Project, Workflow, and Data Access Core | U10, U11 | Direct for gate-aware transitions | ワークフロー遷移時に安全判定と承認状態を参照する |
| U2 Requirements, Stories, and Artifacts | U1, U4, U5, U11, U12 | Direct | プロジェクト配下の成果物管理、AI質問生成、承認履歴、説明生成が必要 |
| U3 Design and Task Planning | U1, U2, U4, U5, U11, U12 | Direct | 承認済み要件を入力に設計とタスクを生成する |
| U4 Job Orchestration Platform | U1, U11 | Direct | ジョブ状態永続化と監査記録が必要 |
| U5 AI Provider Adapter | U10 | Direct | provider実行前の安全確認、設定妥当性確認が必要 |
| U6 GitHub Integration | U1, U4, U10, U11, U12 | Direct | 外部API操作、安全判定、監査、エラー説明が必要 |
| U7 Code Generation and Diff Review | U2, U3, U4, U5, U6, U10, U11, U12 | Direct | 承認済み要件/設計/タスクに基づき生成し、GitHub差分と安全判定を扱う |
| U8 Testing and Quality | U1, U4, U7, U11, U12 | Direct | 実装差分承認後にテストジョブを実行し、結果を承認対象にする |
| U9 Deployment and Amplify Integration | U1, U4, U6, U8, U10, U11, U12 | Direct | テスト承認、安全判定、GitHub接続、Amplifyジョブ、監査が必要 |
| U10 Security Policy | U1, U11 | Direct | project scoped policyと監査記録が必要 |
| U11 Approval and Audit | U1, U10 | Direct | 承認対象のproject scopeとblocker判定が必要 |
| U12 Explanation and Operations Feedback | U1, U4, U5, U9, U11 | Direct/Optional | 要約ジョブ、AI支援、デプロイ/ログ参照、監査が必要 |
| U13 UI Experience Shell | U1-U12 | API consumption | 画面は各ユニットの公開APIを統合する |

## Recommended Implementation Sequence

| Order | Unit | 実装意図 | Completion Signal |
|---:|---|---|---|
| 1 | U1 Project, Workflow, and Data Access Core | 全ユニットのproject scope、状態、永続化境界を先に作る | project作成、workflow状態取得、Prisma境界が動く |
| 2 | U10 Security Policy | blocker/warning判定を早期に固定する | policy evaluation APIと代表的な分類テストがある |
| 3 | U11 Approval and Audit | 全ステージのHuman in the Loopを成立させる | approval decisionとaudit eventが保存される |
| 4 | U4 Job Orchestration Platform | AI/GitHub/Test/Deployを非同期化する土台を作る | job enqueue/status/retry/cancelが動く |
| 5 | U5 AI Provider Adapter | 生成系ユニットのprovider依存を閉じ込める | Bedrock-first adapterとstructured output validationが動く |
| 6 | U2 Requirements, Stories, and Artifacts | Inception入口の価値を成立させる | ideaから質問/要件/MVPスコープ成果物まで生成できる |
| 7 | U3 Design and Task Planning | Construction準備の成果物を作る | 画面/データ/タスク分解が生成される |
| 8 | U6 GitHub Integration | コード生成の格納先とレビュー導線を用意する | repo/branch/PR/diff取得が動く |
| 9 | U7 Code Generation and Diff Review | 実装差分レビュー体験を成立させる | Next.js template、task changes、diff summary、scope checkが動く |
| 10 | U8 Testing and Quality | 品質確認とテスト承認を成立させる | test job、result normalization、PBT対象検証が動く |
| 11 | U9 Deployment and Amplify Integration | AWS公開導線を成立させる | pre-check、Amplify deploy、public URL/result取得が動く |
| 12 | U12 Explanation and Operations Feedback | 非エンジニア向け説明と改善サイクルの入口を整える | error/log summaryとimprovement suggestion boundaryが動く |
| 13 | U13 UI Experience Shell | すべてのサービスAPIをGUIワークフローとして統合する | project dashboardからdeploy/operationsまで一連の画面がつながる |

## API Contract Categories

| Contract Category | Owner | Consumers | Notes |
|---|---|---|---|
| Project API | U1 | U2-U13 | project scope、dashboard、current workflow state |
| Workflow API | U1 | U11, U13 | stage transition、next action、gate status |
| Artifact API | U2 | U3, U7, U12, U13 | requirements、stories、design references、versioning |
| Job API | U4 | U2, U3, U6, U7, U8, U9, U12, U13 | enqueue、status、retry、cancel、progress |
| AI Provider API | U5 | U2, U3, U7, U12 | provider-neutral text/structured generation |
| GitHub API | U6 | U7, U9, U13 | repo、branch、PR、diff、checks |
| Code Review API | U7 | U8, U11, U13 | generated changes、diff summary、scope expansion result |
| Testing API | U8 | U9, U11, U13 | test run、quality summary、PBT result |
| Deployment API | U9 | U11, U12, U13 | settings、pre-check、deploy、deployment result |
| Security API | U10 | U1, U5, U6, U7, U8, U9, U11 | evaluate、classify、assert no blockers |
| Approval/Audit API | U11 | U1-U13 | approval decision、history、audit event |
| Explanation/Ops API | U12 | U2, U3, U6, U7, U8, U9, U13 | term mapping、error summary、log summary、improvement suggestions |

## Event Contracts

| Event | Owner | Required Consumers | Purpose |
|---|---|---|---|
| `project.created` | U1 | U11, U13 | プロジェクト作成履歴と画面更新 |
| `workflow.stageChanged` | U1 | U11, U13 | 現在ステージ変更の監査と画面更新 |
| `approval.requested` | U11 | U1, U13 | 承認待ち状態の開始 |
| `approval.approved` | U11 | U1, U13 | 次ステージ遷移可能化 |
| `approval.rejected` | U11 | U1, U13 | 修正ステータスへの戻し |
| `audit.recorded` | U11 | U13 | 履歴表示更新 |
| `job.enqueued` | U4 | U13 | 進捗表示開始 |
| `job.completed` | U4 | Domain owner, U13 | 成果物/結果の取得可能化 |
| `job.failed` | U4 | Domain owner, U12, U13 | 失敗要約と次アクション提示 |
| `requirements.documentGenerated` | U2 | U1, U11, U13 | 要件レビュー開始 |
| `design.generated` | U3 | U1, U11, U13 | 設計レビュー開始 |
| `tasks.generated` | U3 | U7, U13 | 実装タスク選択可能化 |
| `github.pullRequestCreated` | U6 | U7, U11, U13 | 差分レビュー開始 |
| `code.diffReady` | U7 | U8, U11, U13 | 実装差分承認開始 |
| `code.scopeExpansionDetected` | U7 | U10, U11, U13 | 未承認範囲の確認 |
| `tests.completed` | U8 | U9, U11, U13 | テスト結果承認開始 |
| `security.blockerRaised` | U10 | U1, U11, U13 | 進行ブロックと説明表示 |
| `deployment.preCheckCompleted` | U9 | U10, U11, U13 | デプロイ承認可否の判断 |
| `deployment.completed` | U9 | U11, U12, U13 | 公開URL表示と運用レビュー開始 |
| `deployment.failed` | U9 | U11, U12, U13 | 失敗要約と修正導線表示 |
| `ops.improvementSuggested` | U12 | U2, U11, U13 | 次サイクル候補作成 |

## Job Contracts

| Job | Owner | Requires | Produces |
|---|---|---|---|
| `requirements.generateQuestions` | U2 | Project, idea text, AI Provider config | Question artifact, audit event |
| `requirements.generateDocument` | U2 | Question answers, project context | Requirements artifact, MVP scope |
| `design.generateArtifacts` | U3 | Approved requirements | Screen list, data model, API/deploy design |
| `tasks.generateBreakdown` | U3 | Approved design | Implementation task list, story-task traceability |
| `github.createRepository` | U6 | GitHub account/installation, project metadata | Repository connection |
| `github.preparePullRequest` | U6 | Repository connection, branch metadata | Pull Request reference |
| `code.generateTemplate` | U7 | Approved design, repository connection | Initial Next.js template diff |
| `code.generateTaskChanges` | U7 | Approved task, approved scope, repository connection | Task diff and diff summary |
| `code.summarizeDiff` | U7 | Pull Request diff | Non-engineer diff summary, scope analysis |
| `tests.run` | U8 | Pull Request branch/ref, test configuration | Test result and quality summary |
| `deployment.preCheck` | U9 | App settings, test approval, security facts | Pre-check result |
| `deployment.amplifyDeploy` | U9 | Deployment approval, GitHub ref, Amplify config | Deployment result and public URL |
| `ops.summarizeLogs` | U12 | Deployment/log references | Plain-language log summary |
| `ops.suggestImprovements` | U12 | Requirements, logs, recent errors | Improvement candidates |

## Gate Dependencies

| Gate | Owner | Blocking Inputs |
|---|---|---|
| Requirements Approval | U11 | Requirements artifact from U2, security findings if any |
| Design Approval | U11 | Design artifacts from U3, MVP scope from U2 |
| Implementation Diff Approval | U11 | Diff summary from U7, scope analysis from U7, security findings from U10 |
| Test Result Approval | U11 | Test result from U8, failure summary from U12 |
| Deployment Approval | U11 | Deployment pre-check from U9, blocker status from U10, test approval from U8 |

## Dependency Risks and Mitigations

| Risk | Affected Units | Mitigation |
|---|---|---|
| U1が肥大化して全責務を吸収する | U1, all | U1はproject/workflow/data boundariesに限定し、生成や外部連携を持たない |
| provider固有レスポンスが各ユニットへ漏れる | U5, U2, U3, U7, U12 | U5のDTOで正規化し、structured output schemaを共有する |
| SecurityとApprovalが混ざり、判断根拠が曖昧になる | U10, U11 | U10は判定、U11は承認/履歴に限定する |
| UIがドメインロジックを持つ | U13, all | U13はAPI統合と状態表示に限定する |
| 非同期ジョブ失敗時にユーザーが次アクションを理解できない | U4, U12, U13 | `job.failed` をU12で要約し、U13で次アクションとして表示する |

## Validation Summary

- 承認済みのcore-first実装順序を反映した。
- API、イベント、ジョブ契約をユニット間インターフェースとして定義した。
- U13は独立したUI Experience Shellとして、全ユニットのAPI consumerに限定した。
- デプロイ前チェックとAmplifyデプロイはU9に統合した。

