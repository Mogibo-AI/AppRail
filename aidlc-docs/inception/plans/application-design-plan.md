# Application Design Plan

## Purpose

This file defines the Application Design stage plan. It collects design decisions before generating component, method, service, dependency, and consolidated application design artifacts.

## Source Artifacts

- `aidlc-docs/inception/requirements/requirements.md`
- `aidlc-docs/inception/user-stories/personas.md`
- `aidlc-docs/inception/user-stories/stories.md`
- `aidlc-docs/inception/user-stories/stories-ja.md`
- `aidlc-docs/inception/plans/execution-plan.md`

## Design Scope

The design covers a greenfield AI-DLC GUI platform for non-engineers to create, review, test, deploy, and improve internal CRUD applications. The MVP includes GitHub integration, AWS Amplify Hosting deployment, PostgreSQL + Prisma, AI Provider Adapter with Amazon Bedrock first, explicit approval gates, audit history, security blockers/warnings, and limited PBT enforcement.

## Planned Design Artifacts

- [x] `aidlc-docs/inception/application-design/components.md`
- [x] `aidlc-docs/inception/application-design/component-methods.md`
- [x] `aidlc-docs/inception/application-design/services.md`
- [x] `aidlc-docs/inception/application-design/component-dependency.md`
- [x] `aidlc-docs/inception/application-design/application-design.md`
- [x] Validate design completeness and consistency

## Answer Analysis

- **Validation Result**: All design questions answered.
- **Contradictions Detected**: None.
- **Ambiguities Detected**: None requiring follow-up.
- **Architecture Boundary**: Next.js UI/API plus separated background worker.
- **Workflow State Granularity**: Separate phase, stage, and approval gate entities.
- **AI Execution**: Asynchronous jobs for all AI generation with GUI progress polling or subscription.
- **GitHub Responsibility**: Repository connection, branch, PR, diff retrieval, comments, and check results.
- **Amplify Responsibility**: Amplify app creation, GitHub connection, deploy trigger, and result retrieval.
- **Data Ownership**: PostgreSQL + Prisma with logical groups for project management, AI artifacts, and audit.
- **Approval and Audit**: Separate Approval and AuditEvent entities.
- **Security Policy Placement**: Dedicated Security Policy Component used by services.
- **AI Provider Adapter**: Generic provider interface designed for Bedrock and future providers.
- **Explanation Responsibility**: Dedicated Explanation/Translation Component.
- **PBT Responsibility**: Domain components declare their own PBT-applicable properties.
- **Operations Feedback**: Define log summary and improvement suggestion component boundaries in MVP.

## Planned Design Focus

- Component boundaries for the AI-DLC GUI platform
- Workflow orchestration and state transitions
- AI Provider Adapter responsibilities
- GitHub integration responsibilities
- AWS Amplify deployment responsibilities
- PostgreSQL + Prisma data management responsibilities
- Approval gate and audit responsibilities
- Security blocker/warning enforcement
- PBT-applicable validation and schema-check responsibilities
- Non-engineer language and explanation responsibilities

## Initial Component Candidates

- Project Workspace Component
- AI-DLC Workflow Orchestrator
- Requirements Intake Component
- Story and Artifact Manager
- Application Design Assistant
- Task Planning Component
- AI Provider Adapter
- GitHub Integration Component
- Code Generation and Diff Review Component
- Test Orchestration Component
- Deployment Pre-Check Component
- AWS Amplify Deployment Component
- Security Policy Component
- Audit History Component
- Operations Feedback Component

## Design Questions

以下の質問に回答してください。各質問の `[Answer]:` の後に、該当する選択肢の文字を記入してください。選択肢に合わない場合は `X) Other` を選び、希望内容を記入してください。

## Question 1

MVPのアーキテクチャ境界はどの形を優先しますか？

A) Next.js単体アプリ内にUI、Route Handlers、サービス層、Prismaをまとめる
B) Next.js UI/APIとバックグラウンドワーカーを分ける
C) Next.jsフロントエンドと独立Backend APIを分ける
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 2

AI-DLCワークフロー状態の管理はどの粒度にしますか？

A) プロジェクト単位の単一状態モデルで、現在フェーズと成果物状態を管理する
B) フェーズ、ステージ、承認ゲートを個別エンティティとして管理する
C) イベントソーシング寄りに、状態はイベント履歴から復元する
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 3

AI生成処理の実行方式はどれを優先しますか？

A) 同期実行を基本にし、長時間処理のみジョブ化する
B) すべて非同期ジョブとして扱い、進捗をGUIでポーリングまたは購読する
C) MVPでは同期実行中心にし、ジョブ基盤は後続に回す
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 4

GitHub連携コンポーネントの責務はどこまで持たせますか？

A) リポジトリ接続、ブランチ作成、Pull Request作成まで
B) Aに加えて、差分取得、コメント、チェック結果取得まで
C) GitHub操作は最小限にし、詳細レビューはGitHub側に遷移させる
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 5

AWS Amplifyデプロイコンポーネントはどの責務を持ちますか？

A) Amplifyアプリ作成、GitHub接続、デプロイトリガー、結果取得まで持つ
B) 既存Amplifyアプリへの接続とデプロイ結果取得を中心にする
C) MVPではデプロイ状態表示を中心にし、作成操作は運用者が行う
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 6

PostgreSQL + Prismaのデータ所有境界はどれを優先しますか？

A) すべての業務データを単一DBで管理し、サービス層で責務を分ける
B) プロジェクト管理、AI成果物、監査ログを論理スキーマまたはテーブル群で明確に分ける
C) 監査ログだけ別ストアに分離し、他は単一DBにする
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 7

承認ゲートと監査ログの設計方針はどれを優先しますか？

A) ApprovalエンティティとAuditEventエンティティを分けて管理する
B) AuditEventに承認も含めて一本化する
C) 承認は状態として管理し、監査ログは要約だけ残す
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 8

セキュリティポリシーの適用場所はどこに置きますか？

A) 専用Security Policy Componentで判定し、各サービスが呼び出す
B) 各サービス内に個別ルールとして実装する
C) デプロイ前チェックに集約し、通常操作中は警告中心にする
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 9

AI Provider Adapterの初期設計はどの抽象度にしますか？

A) Bedrock実装を優先しつつ、共通インターフェースは最小限にする
B) Bedrock/OpenAI等を想定した汎用インターフェースを最初から定義する
C) LLM呼び出しだけでなく、ツール実行・成果物検証もAdapterに含める
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 10

非エンジニア向け表現の責務はどのコンポーネントに持たせますか？

A) UIコンポーネント側で表示変換する
B) Explanation/Translation専用コンポーネントを作り、各サービス結果を変換する
C) 各ドメインサービスが非技術者向け説明を返す
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 11

PBT対象の設計責務はどこに置きますか？

A) Testing and Quality ComponentがPBT対象を管理する
B) 各ドメインコンポーネントが自身のPBT対象を宣言する
C) Functional Design段階でのみ扱い、Application Designでは対象範囲の記述に留める
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 12

Operations FeedbackのMVP設計範囲はどこまで含めますか？

A) ログ要約と改善提案のコンポーネント境界だけ定義する
B) ログ取得、要約、改善提案、次サイクル登録まで設計する
C) Application DesignではPost-MVP候補として扱い、詳細設計は後続に回す
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Next-Step Gate

Application Design成果物は生成済みです。レビュー後、承認または修正依頼を行ってください。
