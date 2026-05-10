# Unit of Work Plan

## Purpose

This plan decomposes the AI-DLC GUI Application Development Platform into manageable units of work for development. A unit of work is a logical grouping of stories and components that can be designed, implemented, tested, and reviewed coherently.

## Source Artifacts

- `aidlc-docs/inception/requirements/requirements.md`
- `aidlc-docs/inception/user-stories/personas.md`
- `aidlc-docs/inception/user-stories/stories.md`
- `aidlc-docs/inception/user-stories/stories-ja.md`
- `aidlc-docs/inception/application-design/application-design.md`
- `aidlc-docs/inception/application-design/components.md`
- `aidlc-docs/inception/application-design/component-methods.md`
- `aidlc-docs/inception/application-design/services.md`
- `aidlc-docs/inception/application-design/component-dependency.md`

## Planned Unit Artifacts

- [x] Generate `aidlc-docs/inception/application-design/unit-of-work.md` with unit definitions, responsibilities, and greenfield code organization strategy.
- [x] Generate `aidlc-docs/inception/application-design/unit-of-work-dependency.md` with unit dependency matrix and sequencing.
- [x] Generate `aidlc-docs/inception/application-design/unit-of-work-story-map.md` mapping stories to units.
- [x] Validate unit boundaries and dependencies.
- [x] Ensure all MVP stories are assigned to units.

## Generation Summary

- **Generated At**: 2026-05-09T02:35:19Z
- **Generated Artifacts**:
  - `aidlc-docs/inception/application-design/unit-of-work.md`
  - `aidlc-docs/inception/application-design/unit-of-work-dependency.md`
  - `aidlc-docs/inception/application-design/unit-of-work-story-map.md`
- **Unit Count**: 13
- **MVP Story Coverage**: US-001 through US-029 assigned to primary units.
- **Validation Result**: No blocking issue found.

## Answer Analysis

- **Validation Result**: All questions answered.
- **Contradictions Detected**: None.
- **Ambiguities Detected**: None requiring follow-up.
- **Unit Decomposition Axis**: Hybrid of user journey and service responsibility.
- **Code Organization**: Monorepo structure: `apps/web/`, `apps/worker/`, `packages/{unit-name}/`.
- **UI Experience Shell**: Independent unit.
- **Security / Approval / Audit**: Security and Approval/Audit are separate units.
- **AI Job / Provider Adapter**: Job platform and Provider Adapter are separate units.
- **GitHub / Code Review**: GitHub integration and Code Generation/Diff Review are separate units.
- **Deployment**: Deployment Pre-Check and AWS Amplify Deployment are a single unit.
- **Operations Feedback**: MVP unit included, limited to log summary and improvement suggestion boundaries.
- **Recommended Implementation Sequence**: Core-first sequence.
- **Ownership Boundary**: Ownership boundaries are documented for future teams, but MVP implementation assumes one team.
- **Unit Interface Detail**: API, event, and job contracts are defined.
- **Cross-Cutting Stories**: Assigned to primary owner unit with secondary dependent units listed.

## Approved Planning Direction

After approval, unit artifacts will be generated using these decisions:

- Use a monorepo-oriented greenfield code organization.
- Keep UI shell independent from domain/service units.
- Split Security Policy from Approval/Audit to preserve governance boundaries.
- Split Job Orchestration from AI Provider Adapter to avoid provider coupling.
- Split GitHub Integration from Code Generation/Diff Review.
- Keep Deployment Pre-Check and Amplify Deployment together as a deployment unit.
- Include Operations Feedback as an MVP unit with limited scope.
- Sequence implementation from core foundations outward.

## Initial Unit Candidates

These are initial candidates only. Final units will be generated after the answers below are reviewed and approved.

| Candidate Unit                          | Scope                                                                                          |
| --------------------------------------- | ---------------------------------------------------------------------------------------------- |
| U1 Project and Workflow Core            | Project workspace, phase/stage/gate state, workflow progression                                |
| U2 Requirements, Stories, and Artifacts | Idea intake, questions, requirements, stories, artifact versioning and traceability            |
| U3 Design and Task Planning             | Application design assistant, task breakdown, design approval support                          |
| U4 AI Job and Provider Platform         | Job orchestrator, worker, AI Provider Adapter, Bedrock-first provider implementation           |
| U5 GitHub and Code Review               | Repository/branch/PR, diff retrieval, comments, check results, scope expansion review          |
| U6 Testing and Quality                  | Test orchestration, test result summaries, PBT declaration collection                          |
| U7 Deployment and Amplify Integration   | Deployment pre-check, app settings, Amplify app creation, GitHub connection, deployment result |
| U8 Security, Approval, and Audit        | Security policy, approval gates, audit history, blocker/warning classification                 |
| U9 Explanation and Operations Feedback  | Non-engineer language, error summaries, log summaries, improvement suggestions                 |
| U10 UI Experience Shell                 | Next.js screens, navigation, dashboard, review/approval/job status UI                          |

## Design Constraints

- Target architecture is Next.js UI/API plus separated background worker.
- All AI generation, GitHub, test, and deployment work should be asynchronous jobs.
- PostgreSQL + Prisma is the MVP data store.
- Units should support explicit approval gates and auditability.
- Security blocker logic must stay centralized through the Security Policy Component.
- Domain components should declare PBT-applicable properties.
- Code organization must keep application code outside `aidlc-docs/`.

## Questions

以下の質問に回答してください。各質問の `[Answer]:` の後に、該当する選択肢の文字を記入してください。選択肢に合わない場合は `X) Other` を選び、希望内容を記入してください。

## Question 1

ユニット分解の主軸はどれを優先しますか？

A) AI-DLCフェーズ/ユーザージャーニー単位で分ける
B) 技術コンポーネント/サービス責務単位で分ける
C) ユーザージャーニーとサービス責務のハイブリッドで分ける
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 2

コード構成はどの形を優先しますか？

A) Greenfield multi-unit monolithとして、`src/{unit-name}/` と `tests/{unit-name}/` に分ける
B) UI/APIとWorkerをトップレベルで分け、その中をユニット別に分ける
C) 将来の独立デプロイを見据えて、`apps/web/`, `apps/worker/`, `packages/{unit-name}/` のmonorepo構成にする
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 3

UI Experience Shellを独立ユニットにしますか？

A) 独立ユニットにする。画面、ナビゲーション、レビューUIをまとめて扱う
B) 各ドメインユニットにUIを含める
C) MVPではUIをProject/Workflow系ユニットに含め、後で分離する
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 4

Security、Approval、Auditは1つの横断ユニットにまとめますか？

A) 1つの横断ユニットにまとめる
B) SecurityとApproval/Auditを別ユニットに分ける
C) 各ユニットに分散し、共通ライブラリだけ作る
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 5

AI Job OrchestratorとAI Provider Adapterは同じユニットにしますか？

A) 同じユニットにする。AI実行基盤としてまとめる
B) Job基盤とProvider Adapterを別ユニットにする
C) Provider AdapterはCode Generation系ユニットに含める
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 6

GitHub連携とCode Generation/Diff Reviewは同じユニットにしますか？

A) 同じユニットにする。コードレビュー体験としてまとめる
B) GitHub連携とCode Generation/Diff Reviewを別ユニットにする
C) GitHub連携はDeployment側にも強く関係するため、外部連携ユニットとしてまとめる
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 7

Deployment Pre-CheckとAWS Amplify Deploymentは同じユニットにしますか？

A) 同じユニットにする。デプロイ準備から実行まで一体で扱う
B) Pre-CheckとAmplify Deploymentを別ユニットにする
C) Pre-CheckはSecurityユニットに含め、AmplifyだけDeploymentユニットにする
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 8

Operations FeedbackはMVPユニットとして扱いますか？

A) MVPユニットにするが、ログ要約と改善提案の境界定義に絞る
B) MVPユニットにして、ログ取得、要約、改善提案まで含める
C) Post-MVP候補に移し、MVPユニットからは外す
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 9

ユニット実装の推奨順序はどれを優先しますか？

A) Core基盤から順に実装する: Project/Workflow -> Security/Audit -> AI Job -> Requirements -> Design/Task -> GitHub/Code -> Test -> Deploy -> Operations/UI
B) ユーザージャーニー順に実装する: Project -> Requirements -> Design -> Code -> Test -> Deploy -> Operations, 横断基盤は必要時に追加する
C) UIプロトタイプを先に作り、その後にバックエンド/Worker/外部連携を埋める
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 10

チーム/所有境界を想定したユニット分けは必要ですか？

A) 必要。将来チーム分担しやすい境界にする
B) MVPでは少人数前提で、実装しやすさを優先する
C) 設計上は所有境界を示すが、実装は単一チーム前提にする
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 11

ユニット間インターフェースはどの粒度で定義しますか？

A) TypeScript型とサービスメソッド単位で定義する
B) API/イベント/ジョブ契約まで明確に定義する
C) MVPでは主要データ型とサービス責務に留める
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 12

ストーリー割当で、横断ストーリーはどう扱いますか？

A) 主担当ユニットに割り当て、副担当ユニットを依存として記載する
B) 横断ユニットにまとめて割り当てる
C) 関連する全ユニットに重複して割り当てる
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Approval Gate

回答は検証済みで、矛盾や追加確認が必要な曖昧さはありません。計画承認後に以下を生成します。

- `aidlc-docs/inception/application-design/unit-of-work.md`
- `aidlc-docs/inception/application-design/unit-of-work-dependency.md`
- `aidlc-docs/inception/application-design/unit-of-work-story-map.md`
