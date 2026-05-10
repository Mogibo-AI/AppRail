# Unit of Work Story Map

## 目的

このドキュメントは、MVPストーリー US-001 から US-029 を、主担当ユニットと副担当ユニットへ割り当てる。横断ストーリーは重複実装を避けるため、主担当を1つに固定し、必要な依存ユニットを副担当として記録する。

## Assignment Rules

- 各MVPストーリーは必ず1つのPrimary Unitを持つ。
- 複数ユニットに影響する場合はSecondary Unitsに記載する。
- U13 UI Experience Shellは多くのストーリーに関与するが、画面統合の副担当として扱う。
- 承認・監査に関するストーリーはU11をPrimary Unitにする。
- 安全性判定に関するストーリーはU10をPrimary Unitにする。

## Story Coverage by Unit

| Unit | Primary Stories | Count |
|---|---|---:|
| U1 Project, Workflow, and Data Access Core | US-001 | 1 |
| U2 Requirements, Stories, and Artifacts | US-002, US-003, US-004, US-006 | 4 |
| U3 Design and Task Planning | US-007, US-008, US-010 | 3 |
| U4 Job Orchestration Platform | None as primary; supports async workflows | 0 |
| U5 AI Provider Adapter | US-029 | 1 |
| U6 GitHub Integration | US-011 | 1 |
| U7 Code Generation and Diff Review | US-012, US-013, US-015 | 3 |
| U8 Testing and Quality | US-016, US-018 | 2 |
| U9 Deployment and Amplify Integration | US-019, US-020, US-022, US-023 | 4 |
| U10 Security Policy | US-028 | 1 |
| U11 Approval and Audit | US-005, US-009, US-014, US-017, US-021, US-027 | 6 |
| U12 Explanation and Operations Feedback | US-024, US-025, US-026 | 3 |
| U13 UI Experience Shell | None as primary; supports GUI integration | 0 |
| **Total** | **US-001 to US-029** | **29** |

## Full Story Map

| Story | Title | Epic | Primary Unit | Secondary Units | Notes |
|---|---|---|---|---|---|
| US-001 | 開発プロジェクトを作成する | Project Start and Idea Capture | U1 | U11, U13 | Project作成、workflow初期状態、audit記録、画面表示 |
| US-002 | 自然文でアプリアイデアを入力する | Project Start and Idea Capture | U2 | U1, U11, U13 | idea artifactとして保存し、project scopeへ紐付ける |
| US-003 | AIが生成した要件確認質問に回答する | Project Start and Idea Capture | U2 | U4, U5, U12, U13 | AI質問生成はjobとproviderを経由し、質問文は非エンジニア向け表現にする |
| US-004 | 整理された要件を確認する | Project Start and Idea Capture | U2 | U11, U12, U13 | requirements artifactをレビュー可能にする |
| US-005 | 要件を承認する | Project Start and Idea Capture | U11 | U1, U2, U10, U13 | Requirements Approval Gateを記録し、workflowを次ステージへ進める |
| US-006 | MVPスコープを確認する | Scope, Design, and Task Preparation | U2 | U3, U12, U13 | MVPに含める/含めない範囲を成果物化し、設計入力にする |
| US-007 | 生成された画面一覧を確認する | Scope, Design, and Task Preparation | U3 | U2, U12, U13 | 画面一覧と主要導線をレビューする |
| US-008 | PostgreSQLとPrismaのデータモデルを確認する | Scope, Design, and Task Preparation | U3 | U1, U2, U12, U13 | データモデル案とPrisma境界を確認する |
| US-009 | 設計を承認する | Scope, Design, and Task Preparation | U11 | U1, U3, U10, U13 | Design Approval Gateを記録する |
| US-010 | 実装タスク分解を確認する | Scope, Design, and Task Preparation | U3 | U2, U7, U12, U13 | tasksとstory traceabilityを作る |
| US-011 | GitHubリポジトリを接続または作成する | GitHub Integration and Code Generation | U6 | U1, U4, U10, U11, U12, U13 | GitHub外部操作はsecurity/audit/jobを通す |
| US-012 | Next.js CRUDテンプレートを生成する | GitHub Integration and Code Generation | U7 | U3, U4, U5, U6, U10, U13 | 承認済み設計に基づきtemplate diffを作る |
| US-013 | 生成コードを確認する | GitHub Integration and Code Generation | U7 | U6, U10, U11, U12, U13 | diff summaryと安全確認結果をレビューする |
| US-014 | 実装差分を承認する | GitHub Integration and Code Generation | U11 | U1, U7, U10, U13 | Implementation Diff Approval Gateを記録する |
| US-015 | AIによる未承認のスコープ拡張をブロックする | GitHub Integration and Code Generation | U7 | U2, U3, U10, U11, U13 | approved requirements/tasksとの差分比較で検知し、blocker化はU10に委譲する |
| US-016 | テストを実行する | Testing and Quality Review | U8 | U4, U7, U12, U13 | PR branch/refに対するtest jobを実行する |
| US-017 | テスト結果を承認する | Testing and Quality Review | U11 | U1, U8, U10, U12, U13 | Test Result Approval Gateを記録する |
| US-018 | PBT対象ロジックを検証する | Testing and Quality Review | U8 | U2, U3, U7, U10, U12 | pure functions、validation、schema normalizationなどに限定して検証する |
| US-019 | アプリ設定を行う | Deployment Preparation and Amplify Deployment | U9 | U10, U11, U12, U13 | `DATABASE_URL` などをGUI上のアプリ設定として扱う |
| US-020 | デプロイ前チェックを実行する | Deployment Preparation and Amplify Deployment | U9 | U8, U10, U11, U12, U13 | build/test/settings/security/external publicationを確認する |
| US-021 | デプロイを承認する | Deployment Preparation and Amplify Deployment | U11 | U1, U9, U10, U13 | Deployment Approval Gateを記録する |
| US-022 | AWS Amplify Hostingへデプロイする | Deployment Preparation and Amplify Deployment | U9 | U4, U6, U10, U11, U12, U13 | Amplify deploy jobを実行し、結果を監査する |
| US-023 | デプロイ結果と公開URLを確認する | Deployment Preparation and Amplify Deployment | U9 | U11, U12, U13 | 成功URLまたは失敗理由を表示する |
| US-024 | ログとエラーを分かりやすい言葉で確認する | Post-Deployment Review and Improvement Cycle | U12 | U4, U9, U13 | MVPではログ要約境界と表示導線を定義する |
| US-025 | 改善提案を作成する | Post-Deployment Review and Improvement Cycle | U12 | U2, U5, U11, U13 | 運用ログやエラーから次サイクル候補を作る |
| US-026 | GUI全体で非エンジニア向け表現を使う | Cross-Cutting Safety, Language, Audit, and AI Provider Controls | U12 | U13, all service units | glossaryとsummary rulesを全画面で利用する |
| US-027 | 承認と操作履歴を記録する | Cross-Cutting Safety, Language, Audit, and AI Provider Controls | U11 | U1, U4, U6, U7, U8, U9, U10, U12, U13 | approvalとaudit eventを分離して記録する |
| US-028 | セキュリティブロッカーと警告を適用する | Cross-Cutting Safety, Language, Audit, and AI Provider Controls | U10 | U7, U9, U11, U12, U13 | criticalはblocker、minorはwarningとして分類する |
| US-029 | AI Provider Adapterを設定する | Cross-Cutting Safety, Language, Audit, and AI Provider Controls | U5 | U2, U3, U4, U7, U10, U12 | Bedrock-firstだがprovider-neutral interfaceにする |

## Cross-Cutting Story Notes

| Story | Primary Unit Rationale | Cross-Cutting Dependencies |
|---|---|---|
| US-026 | 表現変換と用語マッピングはU12が所有する | U13が画面へ適用し、各サービスは技術情報をU12へ渡す |
| US-027 | 承認履歴と操作履歴はU11が所有する | 状態変更する全ユニットがaudit eventを発行する |
| US-028 | blocker/warning分類はU10が所有する | U7とU9が主要呼び出し元、U11が承認可否に利用する |
| US-029 | provider-neutral interfaceはU5が所有する | 生成を行うU2/U3/U7/U12が利用し、U4がjob実行を支える |

## Coverage Validation

- US-001 から US-029 まで全MVPストーリーを割り当て済み。
- 各ストーリーはPrimary Unitを1つだけ持つ。
- UI統合はU13、承認/履歴はU11、安全判定はU10、AI providerはU5に集約した。
- U4は主担当ストーリーを持たないが、AI/GitHub/Test/Deployを成立させる必須基盤ユニットとして扱う。

