# Functional Design Plan - U1 Project, Workflow, and Data Access Core

## Purpose

U1 `Project, Workflow, and Data Access Core` の詳細な業務ロジック設計を行う。対象は、プロジェクト作成、AI-DLCワークフロー状態、Phase/Stage/Gateの遷移、共有型、Prisma永続化境界である。

## Source Artifacts

- `aidlc-docs/inception/application-design/unit-of-work.md`
- `aidlc-docs/inception/application-design/unit-of-work-dependency.md`
- `aidlc-docs/inception/application-design/unit-of-work-story-map.md`
- `aidlc-docs/inception/application-design/components.md`
- `aidlc-docs/inception/application-design/component-methods.md`
- `aidlc-docs/inception/application-design/services.md`
- `aidlc-docs/inception/application-design/component-dependency.md`
- `aidlc-docs/inception/user-stories/stories-ja.md`

## Unit Context

| Item                  | Value                                                                                                  |
| --------------------- | ------------------------------------------------------------------------------------------------------ |
| Unit ID               | U1                                                                                                     |
| Unit Name             | Project, Workflow, and Data Access Core                                                                |
| Primary Story         | US-001: 開発プロジェクトを作成する                                                                     |
| Supporting Stories    | US-002 to US-029 depend on U1 for project scope, workflow state, shared types, or persistence boundary |
| Primary Components    | Project Workspace Component, AI-DLC Workflow Orchestrator, Data Access Component                       |
| Primary Package Paths | `packages/project-workflow/`, `packages/shared-types/`, `packages/data-access/`                        |
| App/API Consumers     | `apps/web/`, later domain packages                                                                     |

## Plan Steps

- [x] Analyze U1 business logic boundaries from unit and component artifacts.
- [x] Create this functional design plan and question set.
- [x] Collect user answers for U1 functional design questions.
- [x] Validate answers for completeness, contradictions, and ambiguity.
- [x] Generate `aidlc-docs/construction/u1-project-workflow-data-access-core/functional-design/business-logic-model.md`.
- [x] Generate `aidlc-docs/construction/u1-project-workflow-data-access-core/functional-design/business-rules.md`.
- [x] Generate `aidlc-docs/construction/u1-project-workflow-data-access-core/functional-design/domain-entities.md`.
- [x] Generate `aidlc-docs/construction/u1-project-workflow-data-access-core/functional-design/frontend-components.md` only for U1-facing shell requirements such as project creation form and workflow status display boundaries.
- [x] Validate functional design artifacts against Security Baseline and Property-Based Testing applicability.
- [x] Update `aidlc-docs/aidlc-state.md` and `aidlc-docs/audit.md`.

## Answer Analysis

- **Reviewed At**: 2026-05-09T03:59:32Z
- **Validation Result**: All questions answered with valid options.
- **Contradictions Detected**: None.
- **Ambiguities Requiring Follow-Up**: None.
- **Selected Design Decisions**:
  - Project type defaults to `internal-crud-web-app`; future values can exist but are blocked in MVP.
  - Project creation requires project name, natural-language idea, target users, and publication scope.
  - Project name is unique within same user/workspace.
  - Project lifecycle states are `draft`, `in_progress`, `deployed`, `paused`, and `archived`.
  - Phase/stage uses fixed enum plus metadata table for display/order.
  - Required approval gates are strict blockers.
  - U10/U11 dependencies are interfaces in U1 and implemented by their own units.
  - Transaction boundary is use case level.
  - Repository queries for project-owned data require `ProjectScope`, derived from `projectId`, actor, and operation.
  - Physical project deletion is not allowed in MVP; archive only.
  - Dashboard includes phase/stage, next action, approval pending state, recent jobs, recent audit events, and blocker presence.
  - PBT applies to workflow/status transitions, project input normalization, slug generation, and uniqueness decisions.

## Generated Artifacts

- `aidlc-docs/construction/u1-project-workflow-data-access-core/functional-design/business-logic-model.md`
- `aidlc-docs/construction/u1-project-workflow-data-access-core/functional-design/business-rules.md`
- `aidlc-docs/construction/u1-project-workflow-data-access-core/functional-design/domain-entities.md`
- `aidlc-docs/construction/u1-project-workflow-data-access-core/functional-design/frontend-components.md`

## Revision History

### 2026-05-09T04:17:10Z - Review Change Request Applied

- Added dashboard `DashboardSection<T>` availability contract.
- Clarified workflow initial state: `workspace-detection` is completed during project creation and `requirements-analysis` is first user-visible stage.
- Added explicit transition table with required gates and status updates.
- Aligned approval gates to MVP stage enum.
- Defined missing DTOs, port types, and refs.
- Standardized U1 error codes and field-level validation codes.
- Added workspace access model with `WorkspaceMembership` and `WorkspaceScope`.
- Updated repository contract to require `ProjectScope`.
- Defined audit outbox behavior and retry semantics.
- Added read/mutate/archive operation-aware `ProjectScopeDecision`.
- Defined slug uniqueness and suffix collision behavior.
- Replaced boolean/optional `Result<T>` with discriminated union.
- Expanded `NextAction` with route-ready target data.

## Design Focus

- Project lifecycle and allowed project states.
- Project creation constraints for MVP internal CRUD applications.
- AI-DLC workflow state model across Inception, Construction, and Operations placeholder.
- Stage transition rules and approval gate dependencies.
- Data access ownership rules for project-scoped persistence.
- Shared type ownership and cross-unit contract stability.
- Audit and Security dependency boundaries without absorbing U10/U11 responsibilities.

## Questions

以下の質問に回答してください。各質問の `[Answer]:` の後に、該当する選択肢の文字を記入してください。選択肢に合わない場合は `X) Other` を選び、希望内容を記入してください。

## Question 1

MVPのプロジェクト種別は、U1の業務ルールとしてどこまで制限しますか？

A) `internal-crud-web-app` の1種類だけを許可する
B) `internal-crud-web-app` を既定にし、将来拡張用の種別フィールドは持つがMVPでは他種別をブロックする
C) 複数種別を登録可能にするが、MVP対応可否を別フラグで判定する
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 2

プロジェクト作成時に必須にする入力はどれにしますか？

A) プロジェクト名と自然文アイデアのみ
B) プロジェクト名、自然文アイデア、対象ユーザー、公開範囲
C) プロジェクト名、自然文アイデア、対象ユーザー、公開範囲、想定データ種別
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 3

プロジェクト名の一意性はどう扱いますか？

A) 同一ユーザー/同一ワークスペース内で一意にする
B) 全システムで一意にする
C) 表示名は重複可にし、内部IDだけで一意にする
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 4

プロジェクトのライフサイクル状態はどの粒度で管理しますか？

A) `draft`, `active`, `archived` の最小構成にする
B) `draft`, `in_progress`, `deployed`, `paused`, `archived` にする
C) AI-DLC phase/stageだけで管理し、別のproject statusは持たない
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 5

AI-DLCのPhase/StageはU1でどのように表現しますか？

A) 固定enumとして定義する
B) DB上の設定テーブルとして定義し、将来追加できるようにする
C) 固定enumと設定テーブルを併用し、MVPはenum、表示名や順序はDBで管理する
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 6

ワークフロー遷移時の承認ゲート確認はどの厳しさにしますか？

A) 必須ゲートが承認済みでなければ絶対に遷移不可にする
B) 管理者だけは警告付きで遷移できるようにする
C) MVPではログ記録のみ行い、ブロックは後続ユニットで実装する
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 7

U10 Security PolicyとU11 Approval/Auditが未実装の初期段階で、U1はどう依存を扱いますか？

A) U1に最小のinterfaceだけ定義し、実装はU10/U11で差し替える
B) U1内に一時的な簡易実装を置き、後でU10/U11へ移す
C) U10/U11が実装されるまでU1の遷移機能は作らない
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 8

Data Access Componentのトランザクション境界はどう定義しますか？

A) 各サービスメソッド単位でtransactionを張る
B) use case単位でtransactionを張り、repositoryはtransaction contextを受け取る
C) repository単位で必要な時だけtransactionを張る
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 9

Project scoped persistenceの強制はどの層で行いますか？

A) 各repositoryの全queryで `projectId` を必須にする
B) service層で `projectId` を検証し、repositoryは汎用queryにする
C) DB Row Level Securityを前提にする
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 10

プロジェクト削除はMVPでどう扱いますか？

A) 物理削除は不可にし、archiveのみ許可する
B) 通常はarchive、管理者のみ物理削除を許可する
C) MVPでは削除/アーカイブを実装しない
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 11

U1のダッシュボード状態に含める情報はどれを必須にしますか？

A) 現在phase/stage、次アクション、承認待ち有無
B) Aに加えて直近ジョブ、直近監査イベント、ブロッカー有無
C) Bに加えて成果物一覧、PR、デプロイURLまで含める
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 12

U1でProperty-Based Testing対象にするロジックはどれを優先しますか？

A) ワークフロー状態遷移とproject status遷移だけ
B) Aに加えてproject作成入力の正規化、slug生成、一意性判定を含める
C) U1ではPBTを使わず通常ユニットテストのみとする
X) Other (please describe after [Answer]: tag below)

[Answer]: B
