# AppRail 技術環境ドキュメント

## 概要

このドキュメントは、AppRailの技術環境と制約を明示するための入力ドキュメントです。現在のAI-DLC成果物から逆生成しており、今後AI-DLCを再実行する際に、会話履歴だけに依存せず技術方針を読み込めるようにすることを目的としています。

## 言語

| 区分 | 言語 / ランタイム | バージョン / 制約 | 備考 |
|---|---|---|---|
| 必須 | TypeScript | スキャフォールド時に具体バージョンを固定する | UI、API、packages、workerの主要言語 |
| 必須 | Node.js | LTS系を前提に、スキャフォールド時に具体バージョンを固定する | Next.js、worker、テスト、ツール実行に使用 |
| MVPでは禁止 | Python, Go, Java, Ruby, PHP | 明示承認なしにアプリ本体のランタイムへ導入しない | MVPはTypeScript中心で統一する |

## パッケージマネージャー

| 方針 | 制約 |
|---|---|
| JavaScriptパッケージマネージャーはリポジトリ内で1つに統一する | npm, pnpm, yarnのlockfileを混在させない |
| lockfileをコミットする | 生成アプリの再現性を確保する |

## フレームワークとライブラリ

| 領域 | 必須 / 推奨 | 方針 | 備考 |
|---|---|---|---|
| Webフレームワーク | 必須 | Next.js | UI/APIの主要サーフェス |
| UIランタイム | 必須 | React | Next.jsを通じて利用 |
| スタイリング | 必須 | Tailwind CSS | MVPのUIスタイル |
| UIコンポーネント | 必須 | shadcn/ui | レビュー、承認、フォーム、ダッシュボードで利用 |
| APIハンドラー | 推奨 | Next.js Route Handlers または Hono互換構成 | 設計上のバックエンド境界に合わせる |
| ORM | 必須 | Prisma | PostgreSQLアクセスに利用 |
| データベース | 必須 | PostgreSQL | プラットフォームと生成CRUDアプリの基本DB |
| バリデーション | 必須 | Zod | API境界とservice/use case境界の検証 |
| 単体テスト | 必須 | Vitest | TypeScript packageとドメインロジックのテスト |
| Property-based testing | 対象領域で必須 | fast-check | バリデーション、正規化、ワークフロー、トレーサビリティ、スキーマ検証で利用 |
| AIプロバイダー連携 | 抽象化必須 | AI Provider Adapter | Amazon Bedrock優先、provider-neutralな設計 |

## クラウドサービス

| サービス | ステータス | 用途 |
|---|---|---|
| AWS Amplify Hosting | MVPで必須 | 生成されたレスポンシブWebアプリのデプロイ |
| Amazon Bedrock | 優先AIプロバイダー | AI Provider Adapter経由の生成処理 |
| PostgreSQL互換DB | 必須 | AppRailの永続化と生成アプリのデータモデル |
| GitHub | 必須連携 | リポジトリ接続、ブランチ作成、Pull Request作成、差分・チェック確認 |

## 禁止・制限する技術やパターン

| 項目 | ステータス | 理由 | 代替 |
|---|---|---|---|
| MVP生成アプリでのDynamoDB | MVPでは禁止 | データ生成と検証をPostgreSQL + Prismaに統一するため | PostgreSQL + Prisma |
| 無承認のAWSリソース削除 | 禁止 | 危険度が高いため | 明示的な承認とSecurity Policy評価 |
| 無承認の本番デプロイ | 禁止 | Human-in-the-loopの方針に反するため | デプロイ承認ゲート |
| ハードコードされたシークレット | 禁止 | セキュリティリスクが高いため | 環境変数やシークレット管理 |
| AI内部ログの生保存 | 制限 | 機密情報やノイズを含む可能性があるため | AIログ要約のみ保存 |
| DBや外部プロバイダーの生エラーをUIへ返すこと | 禁止 | 実装詳細や機密情報が漏れる可能性があるため | 形式化されたエラーコードとユーザー向け説明 |
| UIコンポーネントからPrismaへ直接アクセス | 禁止 | データアクセス境界を壊すため | Service APIとrepository wrapper |
| `ProjectScope` なしのproject-owned repositoryアクセス | 禁止 | テナント境界と権限境界を壊すため | `ProjectScope` を必須にする |

## アーキテクチャとリポジトリ構成

MVPは、Next.jsのUI/APIサーフェスと分離されたバックグラウンドworkerで構成します。AI生成、GitHub操作、テスト実行、デプロイなどの長時間処理は非同期ジョブとして実行します。

想定するアプリケーションコード配置:

```text
apps/web/
  Next.js UI、画面、API handlers、BFF的なroute境界

apps/worker/
  AI、GitHub、テスト、デプロイ、Operations Feedbackのジョブ実行

packages/project-workflow/
  プロジェクト作成、ワークフロー状態、ステージ遷移、ダッシュボード集約

packages/requirements-artifacts/
  アイデア、質問、回答、要件、MVPスコープ、成果物、ストーリー

packages/design-task-planning/
  画面設計、データ設計、タスク分解

packages/job-platform/
  ジョブキュー、状態、リトライ、キャンセル、進捗

packages/ai-provider/
  provider-neutralなAIインターフェースとBedrock実装

packages/github-integration/
  リポジトリ、ブランチ、Pull Request、差分、コメント、チェック

packages/code-generation-review/
  テンプレート生成、コード生成、差分要約、スコープ逸脱検知

packages/testing-quality/
  テスト実行、結果正規化、property-based testing宣言

packages/deployment-amplify/
  アプリ設定、デプロイ前チェック、Amplifyデプロイ

packages/security-policy/
  blocker/warning分類、危険操作判定

packages/approval-audit/
  承認ゲート、監査イベント、操作履歴

packages/explanation-ops/
  非エンジニア向け説明、エラー要約、ログ要約、改善提案

packages/shared-types/
  共有DTO、イベント、ジョブ契約、プロジェクト/ワークフロー契約

packages/data-access/
  Prisma client境界、transaction、project-scoped repository
```

`aidlc-docs/` はドキュメント出力専用です。アプリケーションコードは `aidlc-docs/` の中に生成しません。

## ドメイン境界

| 境界 | ルール |
|---|---|
| Project and Workflow | プロジェクトメタデータ、ライフサイクル、ワークフロー状態、次アクション、共有契約、データアクセス境界を所有する |
| Requirements | アイデア入力、AI質問、回答、要件、MVPスコープ、成果物トレーサビリティを所有する |
| Design and Tasks | 画面一覧、データモデル、設計成果物、タスク分解を所有する |
| Jobs | 非同期実行、進捗、リトライ、キャンセル、ジョブ状態を所有する |
| AI Provider | provider-neutralなAI生成インターフェースを所有する |
| GitHub | リポジトリ、ブランチ、Pull Request、差分、コメント、チェック連携を所有する |
| Security | blocker/warning分類と危険操作判定を所有する |
| Approval and Audit | 人間の承認ゲートと監査可能な操作記録を所有する |
| Explanation | 非エンジニア向け表現、技術用語変換、エラー要約を所有する |

## データと永続化

| 領域 | 方針 |
|---|---|
| 主データベース | PostgreSQL |
| ORM | Prisma |
| トランザクション | project/workflow/outboxの整合性が必要なmutation use caseでは1つのPostgreSQL transactionを使う |
| Repository pattern | project-owned repositoryは `ProjectScope` を必須とし、raw `projectId` だけではアクセスさせない |
| Project listing | cursor pagination、default limit 25、maximum limit 100 |
| Project uniqueness | `(workspaceId, createdByActorId)` 内で正規化済みproject nameとslugを一意にする |
| Audit durability | 状態変更mutationと同じtransaction内でaudit outboxを書き込む |

## セキュリティ

| 領域 | 要件 |
|---|---|
| 認証 | 初期はprovider-neutralなAuth adapterを定義し、具体プロバイダー選定は後続設計に委ねる |
| 認可 | workspace/project操作の前に `WorkspaceScope` と `ProjectScope` を解決する |
| 入力検証 | API境界とservice/use case境界でZod schemaによる検証を行う |
| 外部公開 | 認証またはアクセス制限なしの外部公開をブロックする |
| シークレット | token、cookie、API key、DB credentialなどをハードコードしない |
| エラー処理 | Public APIは生のprovider errorではなく、形式化されたエラーコードとユーザー向けメッセージを返す |
| ログ | idea text、target-user free text、project name、workspace name、actor display name、生成成果物、token、secret、raw DB error、raw provider payloadをログに出さない |
| Security Policy | blocker/warning分類はSecurity Policy Componentに集約する |

## 承認とガバナンス

| ゲート | MVPでの扱い |
|---|---|
| 要件承認 | 後続の設計・生成へ進む前に必須 |
| 設計承認 | レビュー可能にし、ワークフロー遷移上必要な場合はゲートとして扱う |
| 実装差分承認 | 生成された変更を次工程へ進める前に必須 |
| テスト結果承認 | デプロイ準備前にレビュー可能にする |
| デプロイ承認 | AWS Amplifyデプロイ前に必須 |
| 危険なAWS操作 | 明示的な確認とSecurity Policy評価を必須にする |

## テスト

| テスト種別 | ツール / 方針 | 要件 |
|---|---|---|
| Unit test | Vitest | package-level logicに必須 |
| Property-based test | fast-check | property宣言対象のドメイン領域に必須 |
| Integration test | 実装時に詳細定義 | service境界と重要フローに必要 |
| E2E test | 後続で詳細定義 | ユーザージャーニー検証に有効だが、初期はdomain invariantを優先 |
| テスト要約 | Explanation component / AI-assisted summary | 失敗内容を非エンジニア向けに説明する |

Property-based testingの対象:

- Project nameの正規化。
- Slug生成と一意性。
- Scope decision。
- Workflow stage transition。
- Dashboard section mapping。
- Artifact serializationとtraceability invariant。
- Security finding classification。
- Test result normalization。

## CIと品質ゲート

| ゲート | 要件 |
|---|---|
| Type check | 生成されたTypeScript packages/appsに必須 |
| Lint | scaffold作成後に必須 |
| Unit test | package logicに必須 |
| PR CIでのPBT | `fast-check` を `numRuns >= 100`、固定/ログ出力されたseedで実行 |
| Nightly CIでのPBT | `fast-check` を `numRuns >= 1000`、ログ出力されたseedで実行 |
| Build | デプロイ承認前に必須 |
| Deployment pre-check | AWS Amplifyデプロイ前に必須 |

## 可観測性

| 領域 | 方針 |
|---|---|
| Logs | structured JSON-compatible logs |
| Metrics | OpenTelemetry-compatibleなmetric nameとlabel |
| 必須メトリクス | API latency、API error count、dashboard section unavailable count、audit outbox current count、audit outbox transition count、fail-closed transition count |
| Dashboard dependency | U4 jobs、U10 security、U11 approval/auditの各sectionは個別にdegradeする |
| Timeout | dashboard dependent sectionとtransition checkはbounded timeoutを持ち、必要条件が確認できない場合はfail closedにする |

## 性能とスケール目標

| 領域 | 目標 |
|---|---|
| Active workspaces | MVP load profileで100 active workspaces |
| Projects per workspace | 最大1,000 projects |
| Concurrent authenticated users | 環境全体で50 active users |
| Sustained U1 API load | 合計20 requests/second |
| Burst U1 API load | 最大60秒間、50 requests/second |
| Core API latency | MVP load profileでP95 <= 500ms |
| Dashboard local processing | U1 local workでP95 <= 250ms |
| Dashboard dependent section timeout | dependent sectionごとに200ms |

## サンプルコード方針

現時点では対象アプリケーションのscaffoldが未生成のため、具体的なサンプルコードはまだありません。scaffoldまたはcode generation時に、以下のパターン例を追加します。

- Zod validation付きの典型的なNext.js route handler。
- `Result<T>` を返すproject-scoped service/use case。
- `ProjectScope` を必須とするPrisma repository method。
- Vitest unit test。
- `fast-check` property test。
- Explanation layerを通したuser-facing error mapping。

## 未決定事項

| 未決定事項 | 決定タイミング | 理由 |
|---|---|---|
| Package manager | Project scaffold | repo tooling作成時に固定する必要があるため |
| Node.jsの具体バージョン | Project scaffold | runtimeとdeployment targetに合わせて固定するため |
| Auth.js vs Cognito | Auth担当unitの設計または実装時 | U1ではprovider-neutralなauth contractだけが必要なため |
| Metrics backend/exporter | Infrastructure design | 最終的なruntime/deployment構成に依存するため |
| Alert ruleとSLO dashboard | NFR designまたはInfrastructure design | MVPではrecoveryとauditabilityを優先しているため |
| CI workflow file layout | Code generation / Build and Test | 生成されるrepo構成に依存するため |
