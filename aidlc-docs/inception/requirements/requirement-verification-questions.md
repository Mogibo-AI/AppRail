# Requirements Verification Questions

以下の質問に回答してください。各質問の `[Answer]:` の後に、該当する選択肢の文字を記入してください。選択肢に合わない場合は最後の `X) Other` を選び、同じ行または次の行に希望内容を記入してください。

## Question 1

初期MVPで最も優先する利用者は誰ですか？

A) 社内向けCRUDアプリを作りたい非エンジニア
B) 個人開発初心者・プログラミング学習者
C) PM / PdM が要件整理から実装依頼まで進める用途
D) AI-DLCを実践したいエンジニア
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 2

初期MVPで生成対象にするアプリ種別はどこまで絞りますか？

A) 社内向けCRUDアプリのみ
B) CRUDアプリに加えて簡単な公開Webフォームも含める
C) Next.jsで作れる小規模Webアプリ全般を対象にする
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 3

初期MVPにおけるAI実装の実行環境はどれを想定しますか？

A) サーバー側でAIエージェントを実行し、GitHubリポジトリへ変更を反映する
B) ユーザーのローカル開発環境またはCLIエージェントをGUIから操作する
C) まずはコード生成結果を画面に表示し、GitHub反映は後続フェーズにする
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 4

GitHub連携で初期MVPに必須とする範囲はどこまでですか？

A) リポジトリ作成、ブランチ作成、Pull Request作成まで
B) 既存リポジトリへの接続とPull Request作成まで
C) GitHub連携は後回しにし、まずはプロジェクト内で成果物を管理する
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 5

AWSデプロイの初期ターゲットはどれにしますか？

A) AWS Amplify Hosting を優先する
B) AWS App Runner を優先する
C) Amplify Hosting と App Runner の両方を初期から選べるようにする
D) 初期MVPではデプロイ前チェックまでにし、実デプロイは後続にする
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 6

初期MVPでデータベース作成をどこまで自動化しますか？

A) PostgreSQLとPrisma前提でDB作成・マイグレーションまでGUIから扱う
B) Prismaスキーマ生成まで行い、DB作成はユーザーまたは運用者が行う
C) 初期MVPでは永続化なし、またはモックDBで開始する
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 7

Human in the Loop の承認ゲートで、初期MVPから必須にするものはどれですか？

A) 要件承認、設計承認、実装差分承認、テスト結果承認、デプロイ承認
B) 要件承認、タスク承認、実装差分承認、デプロイ承認
C) 要件承認とデプロイ承認のみ必須にし、他は任意レビューにする
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 8

ユーザー向け表現の方針として、技術用語はどの程度隠しますか？

A) 原則として非技術者向けの表現だけを表示し、詳細は折りたたむ
B) 非技術者向け表現と技術用語を併記する
C) 初期MVPでは技術用語も表示し、ヘルプ文で補足する
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 9

認証・ユーザー管理は初期MVPで必要ですか？

A) 必須。CognitoまたはAuth.jsでログイン機能を実装する
B) 管理者ユーザーのみの簡易認証で開始する
C) 初期MVPでは認証なしのローカル/限定利用で開始する
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 10

このプラットフォーム自体の初期リリース形態はどれですか？

A) 単一のWebアプリとして提供する
B) Webアプリとバックグラウンドワーカーを分ける
C) ローカル実行のデスクトップ/CLI補助付きWebアプリとして提供する
X) Other (please describe after [Answer]: tag below)

[Answer]:X 初期リリースでは、Webアプリを中心に開発し、スマホ・デスクトップの両方で使いやすいレスポンシブUIとして提供する。その後、必要に応じてデスクトップアプリ化、スマホアプリ化を行う。

## Question 11

AIモデル/AI基盤は初期MVPで何を優先しますか？

A) Amazon Bedrock を主軸にする
B) Amazon Q Developer 連携を主軸にする
C) OpenAIなど複数プロバイダーを差し替え可能にする
D) 初期MVPでは抽象インターフェースだけ定義し、実AI接続は最小にする
X) Other (please describe after [Answer]: tag below)

[Answer]: X 初期MVPでは、AI基盤を特定プロバイダーに固定せず、AI Provider Adapter層を定義します。実接続はまずAmazon Bedrockを優先し、必要に応じてOpenAIなど1つの外部プロバイダーも追加できる設計にします。将来的には、ユーザーが自身のAIアカウント/APIキーを連携し、Bedrock、OpenAI、Anthropic、Geminiなどからモデルを選択できるBYO AI Account方式を目指します。

## Question 12

監査ログ・履歴管理で初期MVPから必須にする粒度はどれですか？

A) AIの全アクション、承認、GitHub操作、デプロイ履歴をすべて記録する
B) 承認、GitHub操作、デプロイ履歴を記録し、AI内部ログは要約のみ残す
C) 初期MVPではプロジェクトイベントと承認履歴のみ記録する
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 13

Security Baseline 拡張ルールをこのプロジェクトで有効化しますか？

A) Yes - セキュリティルールをブロッキング制約として強制する
B) No - PoC/プロトタイプとしてセキュリティ拡張は無効にする
X) Other (please describe after [Answer]: tag below)

[Answer]: X 初期MVPからSecurity Baseline拡張ルールは有効化します。ただし、すべてを一律にブロッキング制約にするのではなく、重大なセキュリティ違反のみブロッキングし、軽微な指摘は警告として扱います。特に、シークレットのハードコード、過剰なIAM権限、認証なしの外部公開、危険なAWSリソース削除、ユーザーデータの不適切な送信、未承認の本番デプロイはブロッキング対象にします。

## Question 14

Property-Based Testing 拡張ルールをこのプロジェクトで有効化しますか？

A) Yes - PBTルールをブロッキング制約として強制する
B) Partial - 純粋関数とシリアライズ往復など限定範囲で強制する
C) No - シンプルなCRUD/UI中心のためPBT拡張は無効にする
X) Other (please describe after [Answer]: tag below)

[Answer]: X 初期MVPでは、Property-Based Testing拡張ルールを限定範囲で有効化します。純粋関数、バリデーション、データ変換、シリアライズ/デシリアライズ、DynamoDB itemとdomain modelの相互変換、日付・金額・ステータスなどの正規化処理、AI生成成果物のスキーマ検証に対してPBTを強制します。一方で、UIコンポーネント、画面遷移、AWSデプロイ処理、GitHub API操作、LLM呼び出し、E2Eテストには初期MVPではPBTを強制せず、通常の単体テスト・統合テスト・E2Eテストで検証します。
