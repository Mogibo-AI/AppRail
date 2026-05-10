# Requirements: AI-DLC GUI Application Development Platform

## 1. Intent Analysis Summary

### 1.1 User Request

AWSのAI-DLCを活用し、一般ユーザーでもアイデア入力から要件整理、設計、実装、テスト、AWSへのデプロイ、運用改善までをGUI上で進められる、AI駆動型アプリ開発プラットフォームを開発する。

このプロジェクトはAI IDEそのものを作るものではなく、AI-DLCのプロセスをGUI化し、人間の確認と承認を挟みながらAIに安全に開発作業を任せるためのプラットフォームを作る。

### 1.2 Request Classification

| Item | Classification |
|---|---|
| Request Type | New Project |
| Project Type | Greenfield |
| Scope Estimate | System-wide |
| Complexity Estimate | Complex |
| Requirements Depth | Comprehensive |
| Primary Target | 社内向けCRUDアプリを作りたい非エンジニア |

### 1.3 Key MVP Decisions

| Decision Area | MVP Decision |
|---|---|
| Primary user | 社内向けCRUDアプリを作りたい非エンジニア |
| Generated app type | 社内向けCRUDアプリのみ |
| Initial app platform | Responsive web application |
| AI execution model | GUI上でコード生成結果を確認し、MVPからGitHub連携とAmplify実デプロイも含める |
| GitHub scope | 初期MVPからGitHub連携を含める |
| AWS deployment | AWS Amplify Hostingを優先し、実デプロイまで含める |
| Database | PostgreSQL + Prismaに絞る |
| Authentication | 初期利用はローカル/限定利用可。外部公開時は最低限の認証またはアクセス制限を必須にする |
| Required approval gates | 要件承認、デプロイ承認 |
| Recommended review gates | 設計、実装差分、テスト結果 |
| AI provider | AI Provider Adapterを定義し、実接続はAmazon Bedrockを優先。必要に応じて外部プロバイダーを追加可能にする |
| Audit scope | 承認、GitHub操作、デプロイ履歴を記録し、AI内部ログは要約のみ残す |
| Security extension | 有効。重大なセキュリティ違反はブロッキング、軽微な指摘は警告 |
| PBT extension | 限定範囲で有効。純粋関数、バリデーション、データ変換、シリアライズ、スキーマ検証に強制 |

## 2. Product Goal

ユーザーが作りたいアプリのアイデアを入力すると、AI-DLCの流れに沿って、企画、要件整理、MVP定義、設計、タスク分解、コード生成、テスト、デプロイ、結果確認までをGUIで進められる環境を提供する。

本プラットフォームは、AIに丸投げするためのツールではなく、AIの作業を人間が確認し、必要な場面で承認するための安全なレールを提供する。

## 3. MVP Scope

### 3.1 In Scope

- プロジェクト作成
- 自然文によるアイデア入力
- AIによる不足情報の質問生成
- 要件整理
- MVPスコープ定義
- 画面一覧生成
- PostgreSQL + Prisma前提のデータモデル生成
- 実装タスク生成
- 要件承認UI
- デプロイ承認UI
- 設計、実装差分、テスト結果の任意レビューUI
- GitHubリポジトリ連携
- ブランチ作成
- Pull Request作成
- Next.jsテンプレート生成
- AI Provider Adapter層
- Amazon Bedrock優先のAI接続
- 生成コードの画面表示
- 承認された変更のGitHub反映
- テスト実行
- テスト失敗理由のAI要約
- デプロイ前チェック
- AWS Amplify Hostingへの実デプロイ
- デプロイ結果表示
- 公開URL表示
- 承認履歴、GitHub操作履歴、デプロイ履歴の記録

### 3.2 Out of Scope

- あらゆる技術スタックへの対応
- モバイルネイティブアプリ生成
- デスクトップアプリ提供
- 大規模SNS、リアルタイムアプリ、複雑な業務システム
- 金融、医療など高リスク領域
- DynamoDBを初期MVPの生成・検証対象に含めること
- AIによる無承認の本番反映
- AIによる無承認のAWSリソース削除
- GitHubなしでの本番運用デプロイ
- 完全な運用監視プラットフォーム
- BYO AI Accountの本格提供
- 複雑なロールベース権限管理

## 4. Functional Requirements

### FR-001: Project Creation

ユーザーはGUIから新しい開発プロジェクトを作成できる。

Acceptance Criteria:
- ユーザーはプロジェクト名と作りたいアプリの概要を入力できる。
- システムはプロジェクト状態、履歴、生成成果物をプロジェクト単位で管理する。
- 初期MVPでは対象アプリ種別を社内向けCRUDアプリに制限する。

### FR-002: Idea Input

ユーザーは自然文で作りたいアプリのアイデアを入力できる。

Acceptance Criteria:
- 入力欄は非エンジニアでも使いやすい表現にする。
- 入力内容は後続の要件整理、画面設計、データ設計、タスク分解の起点として保存される。

### FR-003: AI Requirements Question Generation

AIは入力されたアイデアを分析し、不足情報を質問として生成する。

Acceptance Criteria:
- 質問は目的、想定ユーザー、MVP範囲、画面、データ、完成条件、リスクを補える内容にする。
- 質問はGUI上で回答しやすい形式にする。
- ユーザーの回答は要件定義に反映される。

### FR-004: Requirements Organization

AIはユーザー入力と回答をもとに、要件を整理して表示する。

Acceptance Criteria:
- アプリの目的、対象ユーザー、解決したい課題、MVPに含める機能、MVPで除外する機能を表示する。
- ユーザーは要件を確認し、必要に応じて修正できる。
- 要件承認は初期MVPの必須ゲートとする。

### FR-005: MVP Scope Definition

AIは初期MVPに含める範囲と除外する範囲を提示する。

Acceptance Criteria:
- スコープは社内向けCRUDアプリに収まるように制限される。
- 高リスク領域や複雑な機能はMVP外として明示される。
- ユーザーはMVPスコープを承認または変更依頼できる。

### FR-006: Screen List Generation

AIはMVP要件に基づいて画面一覧を生成する。

Acceptance Criteria:
- 画面一覧には画面名、目的、主な操作、関連データを含める。
- 技術用語は原則として非技術者向け表現で表示し、必要な詳細は折りたたむ。

### FR-007: Data Model Generation

AIはPostgreSQL + Prisma前提でデータモデルを生成する。

Acceptance Criteria:
- エンティティ、主要フィールド、リレーション、制約を生成する。
- Prismaスキーマ生成に進める情報を含める。
- 初期MVPではDynamoDBを生成対象に含めない。

### FR-008: Task Decomposition

AIは要件と設計をもとに実装タスクを分解する。

Acceptance Criteria:
- タスクはユーザーがGUI上で確認できる粒度に分解される。
- 各タスクには目的、対象ファイルまたは対象機能、期待結果を含める。
- タスクは後続のコード生成とテスト計画に接続される。

### FR-009: Human Approval Gates

システムは重要な判断点に人間の承認ゲートを設ける。

Acceptance Criteria:
- 初期MVPでは要件承認とデプロイ承認を必須とする。
- 設計、実装差分、テスト結果は任意レビューとして表示する。
- AWSリソース作成、課金が発生する操作、外部公開、データベース変更、環境変数変更、リソース削除は明示的な確認を必要とする。
- 未承認の本番デプロイは禁止する。

### FR-010: GitHub Integration

初期MVPからGitHub連携を提供する。

Acceptance Criteria:
- GitHubリポジトリの作成または接続ができる。
- ブランチ作成ができる。
- 生成・修正されたコードをPull Requestとして提示できる。
- GUI上ではPull Requestを「変更内容の確認」として表現する。

### FR-011: Next.js Template Generation

システムは社内向けCRUDアプリ用のNext.jsテンプレートを生成する。

Acceptance Criteria:
- TypeScript、React、Tailwind CSS、shadcn/uiを前提にする。
- BackendはNext.js Route HandlersまたはHonoを利用できる設計にする。
- PrismaとPostgreSQLを利用できる構成にする。

### FR-012: AI Code Generation

AIは承認されたタスクまたはMVP範囲に基づいてコードを生成する。

Acceptance Criteria:
- 生成コードはGUI上で確認できる。
- ユーザーが承認した変更のみGitHubへ反映される。
- AIが仕様を勝手に拡張した場合は、差分として明示し、承認なしに反映しない。

### FR-013: Test Execution and Summary

システムは生成コードに対してテストを実行し、結果をGUIに表示する。

Acceptance Criteria:
- テスト成功、失敗、未実行を分かりやすく表示する。
- 失敗時はログの生表示だけでなく、AIによる原因要約を表示する。
- 重要なビジネスシナリオには通常の例示ベーステストを含める。
- PBT対象範囲ではproperty-based testを実行できる構成にする。

### FR-014: Deployment Pre-Check

デプロイ前に必要な設定や危険な状態を検査する。

Acceptance Criteria:
- 環境変数、DB接続、ビルド可否、外部公開条件を確認する。
- 認証またはアクセス制限なしで外部公開しようとした場合はブロックする。
- `DATABASE_URL` 未設定などの問題は非技術者向けに説明する。

### FR-015: AWS Amplify Deployment

初期MVPではAWS Amplify Hostingへの実デプロイを優先する。

Acceptance Criteria:
- GitHub連携を利用してAmplify Hostingへデプロイできる。
- デプロイ前にユーザーの明示的な承認を必要とする。
- デプロイ結果、公開URL、失敗理由をGUIに表示する。

### FR-016: Audit and History

システムは重要な操作履歴を追跡可能にする。

Acceptance Criteria:
- 承認履歴を記録する。
- GitHub操作履歴を記録する。
- デプロイ履歴を記録する。
- AI内部ログは全文保存ではなく、意思決定に必要な要約を残す。

### FR-017: AI Provider Adapter

AI基盤を特定プロバイダーに固定しないため、AI Provider Adapter層を定義する。

Acceptance Criteria:
- 初期MVPではAmazon Bedrock接続を優先する。
- 必要に応じてOpenAIなど外部プロバイダーを追加できる構造にする。
- 将来的なBYO AI Account方式を妨げない設計にする。

## 5. Non-Functional Requirements

### NFR-001: Safety

AIが生成したコードやインフラ操作を、ユーザーの明示的な承認なしに本番反映してはならない。

### NFR-002: Security

Security Baseline拡張は有効とする。初期MVPでは以下をブロッキング対象にする。

- シークレットのハードコード
- 過剰なIAM権限
- 認証またはアクセス制限なしの外部公開
- 危険なAWSリソース削除
- ユーザーデータの不適切な送信
- 未承認の本番デプロイ

軽微なセキュリティ指摘は警告として表示し、対応優先度を示す。

### NFR-003: Usability

非エンジニア向けの表現を原則とし、技術用語は必要な場合のみ詳細表示に含める。

Example mappings:

| Technical Term | GUI Term |
|---|---|
| Inception | 企画を固める |
| Construction | 作る |
| Operation | 改善・運用する |
| Pull Request | 変更内容の確認 |
| CI/CD | 自動チェック・自動公開 |
| Environment Variables | アプリ設定 |
| CloudWatch Logs | エラー内容 |

### NFR-004: Auditability

承認、GitHub操作、デプロイ履歴は追跡可能でなければならない。AI内部ログは要約を残し、ユーザーが判断した根拠を確認できるようにする。

### NFR-005: Testability

通常の単体テスト、統合テスト、E2Eテストを基本とし、限定範囲でPBTを強制する。

PBTを強制する対象:
- 純粋関数
- バリデーション
- データ変換
- シリアライズ/デシリアライズ
- 日付、金額、ステータスなどの正規化処理
- AI生成成果物のスキーマ検証

PBTを初期MVPで強制しない対象:
- UIコンポーネント
- 画面遷移
- AWSデプロイ処理
- GitHub API操作
- LLM呼び出し
- E2Eテスト

### NFR-006: Maintainability

AI Provider Adapter、GitHub連携、AWSデプロイ、要件生成、コード生成、監査ログは責務を分離して設計する。

### NFR-007: Deployment Reliability

デプロイ前チェックを通過しない場合、実デプロイを実行してはならない。失敗時は非技術者向けの原因説明と次の操作を提示する。

### NFR-008: Accessibility and Responsive UI

初期リリースはWebアプリを中心にし、スマホ・デスクトップの両方で使いやすいレスポンシブUIとして提供する。

### NFR-009: Cost Awareness

AWSリソース作成や課金が発生する可能性のある操作では、実行前にユーザー確認を行う。

## 6. Data Requirements

初期MVPではPostgreSQL + Prismaを採用する。

Primary data groups:
- Project
- User input / idea
- Requirement
- Question and answer
- MVP scope
- Screen definition
- Data model definition
- Implementation task
- Generated artifact
- Review and approval
- GitHub operation
- Deployment
- Audit event

DynamoDBは初期MVPの生成・検証対象に含めない。DynamoDB itemとdomain modelの相互変換は将来拡張またはPBT例として扱う。

## 7. Security Requirements

### 7.1 External Publication Boundary

認証またはアクセス制限なしの外部公開は禁止する。外部公開する場合は、初期MVPでも最低限の認証またはアクセス制限を必須にする。

### 7.2 Secrets

APIキー、トークン、AWS認証情報、DB接続文字列などのシークレットをソースコードに含めてはならない。

### 7.3 Deployment Approval

本番相当環境へのデプロイ、外部公開、AWSリソース作成、DB変更、リソース削除は明示的な承認を必要とする。

### 7.4 Least Privilege

AWSやGitHubの権限は必要最小限に制限する。ワイルドカード権限は原則禁止し、必要な場合は理由を記録する。

### 7.5 Logging

ログにパスワード、トークン、個人情報、接続文字列を出力してはならない。

## 8. Acceptance Criteria for MVP

MVPは以下を満たしたとき完了とみなす。

1. ユーザーが社内向けCRUDアプリのアイデアを入力できる。
2. AIが不足情報を質問し、ユーザー回答を要件に反映できる。
3. MVPスコープ、画面一覧、データモデル、実装タスクを生成できる。
4. ユーザーが要件を承認できる。
5. GitHubリポジトリ連携、ブランチ作成、Pull Request作成ができる。
6. Next.js + TypeScript + Prisma + PostgreSQL前提のテンプレートを生成できる。
7. AIが生成したコードをGUI上で確認できる。
8. 承認された変更のみGitHubに反映できる。
9. テストを実行し、結果と失敗理由の要約を表示できる。
10. デプロイ前チェックを実行できる。
11. 外部公開時に認証またはアクセス制限がない場合はブロックできる。
12. ユーザーの明示的な承認後、AWS Amplify Hostingへデプロイできる。
13. デプロイ結果と公開URLを表示できる。
14. 承認、GitHub操作、デプロイ履歴を記録できる。

## 9. Risks and Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| MVP範囲が広がりすぎる | 実装が複雑化し完了しにくい | 社内向けCRUDアプリに限定する |
| AIが仕様を勝手に拡張する | 期待と異なる成果物になる | 要件承認とデプロイ承認を必須にし、差分を表示する |
| 認証なしで外部公開される | 情報漏えいにつながる | 外部公開時は認証またはアクセス制限を必須にする |
| GitHub/AWS連携が非エンジニアに難しい | ユーザー体験が悪化する | GUIでは非技術者向け表現に変換する |
| AIプロバイダー固定による将来制約 | 拡張性が下がる | AI Provider Adapter層を設ける |
| AWS課金操作の誤実行 | 予期しないコストが発生する | 課金可能性のある操作は明示確認を必須にする |

## 10. Extension Compliance Summary

### Security Baseline

Status: Enabled with custom severity.

Applicable at requirements stage:
- SECURITY-05 Input Validation: Requirements include validation and schema checks.
- SECURITY-06 Least Privilege: Requirements include least-privilege constraints.
- SECURITY-08 Application-Level Access Control: Requirements block unauthenticated external publication.
- SECURITY-10 Supply Chain Security: Requirements imply GitHub and CI/test governance, to be detailed in design.
- SECURITY-12 Authentication and Credential Management: Requirements prohibit hardcoded secrets and require access restriction for external publication.

No blocking security finding exists at this stage. Detailed controls must be expanded during design and construction.

### Property-Based Testing

Status: Enabled with custom partial enforcement.

Applicable at requirements stage:
- PBT scope is defined for pure functions, validation, transformations, serialization, normalization, and generated artifact schema validation.
- PBT is explicitly not required for UI components, AWS deployment operations, GitHub API operations, LLM calls, or E2E tests in the MVP.

No blocking PBT finding exists at this stage. Specific PBT properties must be identified during functional design and code generation planning.

## 11. Next Stage Recommendation

User Stories should execute next.

Reasoning:
- This is a new user-facing product.
- The project has multiple personas, including non-engineers, admins/operators, and AI-assisted developers.
- The workflows are complex and benefit from acceptance criteria.
- Human approval gates and GUI journeys require clear user-centered stories.
- Story artifacts will improve later screen design, task decomposition, and testing.
