# AppRail

AIと一緒に業務アプリをつくれる開発支援プラットフォームです。

AppRailは、作りたいアプリのアイデア入力から要件整理、設計、実装、テスト、AWSへのデプロイまでを、AI-DLCの流れに沿ってGUIで進めるためのサービスです。AIに作業を任せつつ、要件・設計・実装差分・テスト結果・デプロイなどの重要な判断点では人が確認し、承認しながら安全に開発を進めることを目的としています。

MVPではまず社内向けCRUDアプリの作成に対象を絞り、要件整理からデプロイまでの一連の流れを確立します。その後、業務ポータル、予約・申請ワークフロー、分析ダッシュボード、顧客向けWebアプリなど、より幅広いアプリ種別へ展開していくことを想定しています。

## 想定ユーザー

MVPの主な対象は、社内向けCRUDアプリを作りたい非エンジニアです。

例えば、予約管理、在庫管理、問い合わせ管理、日報管理などの小規模な業務アプリを作りたいが、GitHub、AWS、CI/CD、環境変数などの技術的な手順に詳しくないユーザーを想定しています。

## MVPでできること

- 開発プロジェクトの作成
- 自然文によるアプリアイデア入力
- AIによる不足情報の質問生成
- 要件整理とMVPスコープ定義
- 画面一覧とデータモデルの生成
- 実装タスクの分解
- 要件、設計、実装差分、テスト結果、デプロイの確認と承認
- GitHubリポジトリ連携、ブランチ作成、Pull Request作成
- Next.jsベースのCRUDアプリ生成
- テスト実行と失敗理由の要約
- AWS Amplify Hostingへのデプロイ
- 承認履歴、GitHub操作履歴、デプロイ履歴の記録

## MVPで扱わないこと

- モバイルネイティブアプリやデスクトップアプリの生成
- 大規模SNS、リアルタイムアプリ、複雑な業務システム
- 金融、医療などの高リスク領域
- あらゆる技術スタックへの対応
- AIによる無承認の本番反映
- AIによる無承認のAWSリソース削除
- GitHubなしでの本番運用デプロイ

これらはMVPの対象外ですが、プロダクトとしてはCRUDアプリに固定せず、AI-DLCで安全に扱える範囲を広げながら、対応できるアプリ種別を段階的に増やす方針です。

## 設計方針

MVPは、Next.jsのUI/APIと分離されたバックグラウンドワーカーで構成します。ユーザー操作、軽量な入力検証、状態表示はUI/API側で扱い、AI生成、GitHub操作、テスト実行、AWS Amplifyデプロイなどの長時間処理は非同期ジョブとして実行します。

主要な技術方針は以下です。

- Frontend/API: Next.js, TypeScript, React
- UI: Tailwind CSS, shadcn/ui
- Backend: Next.js Route HandlersまたはHono互換構成
- Database: PostgreSQL + Prisma
- AI Provider: Amazon Bedrockを優先し、将来のプロバイダー追加に備えたAdapter構成
- Deployment: AWS Amplify Hosting
- Repository workflow: GitHub連携とPull Requestベースの変更確認

## 安全性と承認

AppRailは、AIに丸投げするためのツールではありません。AIの生成結果を人が確認し、必要な場面で承認するための安全なレールを提供します。

初期MVPでは、要件承認とデプロイ承認を必須ゲートとし、設計、実装差分、テスト結果は任意レビューとして扱います。AWSリソース作成、課金が発生する操作、外部公開、データベース変更、環境変数変更、リソース削除などは明示的な確認を必要とします。

## リポジトリ構成

```text
.
├── AGENTS.md
├── .aidlc-rule-details/
├── aidlc-docs/
└── README.md
```

主なディレクトリの役割は以下です。

| Path | 内容 |
|---|---|
| `AGENTS.md` | CodexがAI-DLCワークフローを理解するための入口ルール |
| `.aidlc-rule-details/` | AI-DLCの詳細ルール |
| `aidlc-docs/` | AppRailの要件、ユーザーストーリー、設計、計画などの成果物 |

現時点では、対象アプリケーションの実装コードはまだ作成されていません。実装コードは今後、`apps/` や `packages/` など、`aidlc-docs/` の外に配置する方針です。

## 主要ドキュメント

- [要件定義](aidlc-docs/inception/requirements/requirements.md)
- [ペルソナ](aidlc-docs/inception/user-stories/personas.md)
- [ユーザーストーリー](aidlc-docs/inception/user-stories/stories-ja.md)
- [アプリケーション設計](aidlc-docs/inception/application-design/application-design.md)
- [ユニット分解](aidlc-docs/inception/application-design/unit-of-work.md)
- [AI-DLC状態管理](aidlc-docs/aidlc-state.md)

## 現在の状態

AI-DLC上ではInceptionフェーズが完了し、ConstructionフェーズでU1 Project, Workflow, and Data Access Coreの設計成果物を作成している段階です。
