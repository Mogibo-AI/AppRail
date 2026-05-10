# NFR Requirements Plan - U1 Project, Workflow, and Data Access Core

## Purpose

U1 `Project, Workflow, and Data Access Core` の非機能要件を定義する。対象は、project/workspace scope、workflow state、dashboard aggregation、transaction/outbox、Prisma data access boundary、shared TypeScript contract の品質要件である。

## Source Artifacts

- `aidlc-docs/construction/u1-project-workflow-data-access-core/functional-design/business-logic-model.md`
- `aidlc-docs/construction/u1-project-workflow-data-access-core/functional-design/business-rules.md`
- `aidlc-docs/construction/u1-project-workflow-data-access-core/functional-design/domain-entities.md`
- `aidlc-docs/construction/u1-project-workflow-data-access-core/functional-design/frontend-components.md`
- `aidlc-docs/inception/application-design/unit-of-work.md`
- `aidlc-docs/inception/application-design/unit-of-work-dependency.md`

## Unit Context

| Item                     | Value                                                                                                        |
| ------------------------ | ------------------------------------------------------------------------------------------------------------ |
| Unit ID                  | U1                                                                                                           |
| Unit Name                | Project, Workflow, and Data Access Core                                                                      |
| Functional Design Status | Approved by user on 2026-05-09T05:21:45Z                                                                     |
| NFR Focus                | Performance, scalability, availability, security, audit durability, maintainability, PBT framework selection |
| Primary Package Paths    | `packages/project-workflow/`, `packages/shared-types/`, `packages/data-access/`                              |

## Plan Steps

- [x] Analyze U1 functional design artifacts.
- [x] Create this NFR requirements plan and question set.
- [x] Collect user answers for U1 NFR requirements questions.
- [x] Validate answers for completeness, contradictions, and ambiguity.
- [x] Generate `aidlc-docs/construction/u1-project-workflow-data-access-core/nfr-requirements/nfr-requirements.md`.
- [x] Generate `aidlc-docs/construction/u1-project-workflow-data-access-core/nfr-requirements/tech-stack-decisions.md`.
- [x] Validate NFR requirements against Security Baseline and Property-Based Testing applicability.
- [x] Update `aidlc-docs/aidlc-state.md` and `aidlc-docs/audit.md`.

## Answer Analysis

- **Reviewed At**: 2026-05-09T06:01:07Z
- **Validation Result**: All questions answered with valid options.
- **Contradictions Detected**: None.
- **Ambiguities Requiring Follow-Up**: None.
- **Selected NFR Decisions**:
  - Support around 1,000 projects per workspace with indexes and cursor pagination.
  - Target P95 500ms for U1 major APIs.
  - Revised dashboard dependent section timeout to exact 200ms to preserve the dashboard end-to-end P95 500ms target.
  - No explicit MVP availability SLO; prioritize recovery and auditability.
  - Transactional audit outbox is mandatory; dispatch failures remain until manual re-run.
  - Auth integration is provider-neutral at U1; Auth.js/Cognito selection is deferred.
  - Project/workflow/outbox mutations must use a single PostgreSQL transaction.
  - Validate at API and service/use case boundaries; repositories require `ProjectScope`.
  - Use `fast-check` for TypeScript PBT.
  - Run PR CI PBT with fixed seed and at least 100 `numRuns`; run nightly PBT with logged seed and at least 1,000 `numRuns`.
  - Require structured logs, request ID, error code, audit outbox status, API latency metric, and dashboard section unavailable metric.
  - Avoid breaking shared DTO/schema changes during MVP; require schema version and migration notes.

## Generated Artifacts

- `aidlc-docs/construction/u1-project-workflow-data-access-core/nfr-requirements/nfr-requirements.md`
- `aidlc-docs/construction/u1-project-workflow-data-access-core/nfr-requirements/tech-stack-decisions.md`

## Revision History

### 2026-05-09T06:08:26Z - Review Change Request Applied

- Split dashboard latency into end-to-end target, U1 local processing budget, and dependent port timeout.
- Added measurable MVP load profile.
- Fixed cursor pagination default limit to 25 and max limit to 100.
- Clarified `listProjects` as workspace-accessible project listing and aligned indexes.
- Added fail-closed behavior for ApprovalGatePort and SecurityPolicyPort timeout/unavailable cases.
- Clarified fully asynchronous audit outbox dispatch and `Result<T>.warnings` usage.
- Added audit outbox status lifecycle, retry count, `nextAttemptAt`, manual re-run, and intentional close rules.
- Split audit outbox metrics into gauge and transition counter with cardinality limits.
- Defined structured log safe/hashed/prohibited data.
- Chose package-level semantic versioning for shared DTO compatibility.
- Aligned Dashboard unavailable state with `NextAction.disabled` and `DEPENDENCY_UNAVAILABLE`.
- Added PR/nightly PBT `numRuns`, seed policy, reproduction requirements, and required property areas.

## NFR Assessment Focus

- Project creation and dashboard API latency.
- Workspace/project listing scale and pagination.
- Dashboard dependent-port timeout behavior.
- Transaction and audit outbox durability.
- Authorization and project scope enforcement.
- Prisma/PostgreSQL indexing and uniqueness constraints.
- Error handling and availability behavior for U4/U10/U11 dependency failures.
- TypeScript validation, schema validation, and shared DTO stability.
- PBT framework and seed/reproducibility expectations.

## Questions

以下の質問に回答してください。各質問の `[Answer]:` の後に、該当する選択肢の文字を記入してください。選択肢に合わない場合は `X) Other` を選び、希望内容を記入してください。

## Question 1

MVPで想定する1 workspaceあたりのproject数はどの程度を基準にしますか？

A) 100件以下を主対象にし、単純なページングで十分とする
B) 1,000件程度までを対象にし、indexとcursor paginationを必須にする
C) 10,000件以上も見据え、検索/絞り込み最適化をNFRに含める
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 2

U1の主要APIの応答時間目標はどれにしますか？

A) P95 300ms以下を目標にする
B) P95 500ms以下を目標にする
C) P95 1秒以下を目標にする
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 3

`getProjectDashboard` でU4/U10/U11のdependent section取得が遅い場合、どのタイムアウト方針にしますか？

A) 各dependent sectionを300ms程度で打ち切り、`DashboardSection.unavailable` にする
B) 各dependent sectionを500ms程度で打ち切り、`DashboardSection.unavailable` にする
C) dashboard全体の完全性を優先し、dependent section完了まで待つ
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 4

U1 core APIの可用性目標はどれにしますか？

A) MVPは99.5%相当を目標にする
B) MVPでも99.9%相当を目標にする
C) 明示SLOは置かず、障害時の復旧性と監査可能性を優先する
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 5

Audit outboxの耐久性とretry方針はどれにしますか？

A) transaction内outbox書き込み必須、dispatch失敗は24時間retry
B) transaction内outbox書き込み必須、dispatch失敗は7日間retry
C) transaction内outbox書き込み必須、dispatch失敗は手動再実行できるまで保持
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 6

workspace/project scopeの認証・認可連携はMVPでどの前提にしますか？

A) Auth.js sessionを第一候補にし、U1はActor/Scope抽象だけに依存する
B) Amazon Cognitoを第一候補にし、U1はActor/Scope抽象だけに依存する
C) provider-neutralなAuth adapterを定義し、Auth.js/Cognitoの選定は後続NFR Designで決める
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 7

project/workflow mutationの整合性要件はどれにしますか？

A) 単一PostgreSQL transactionでproject/workflow/outboxを更新することを必須にする
B) project/workflowはtransaction、outboxはbest effortにする
C) eventual consistencyを許容し、ジョブで補正する
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 8

Data Access層のvalidation方針はどれにしますか？

A) API境界のみschema validationし、repositoryでは型を信頼する
B) API境界とservice/use case境界でschema validationし、repositoryはProjectScopeを必須にする
C) API/service/repositoryすべてでschema validationを行う
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 9

U1のPBTフレームワークはどれにしますか？

A) TypeScript標準として `fast-check` を採用する
B) PBTは後続で再評価し、今はフレームワークを未確定にする
C) PBTはVitestのparameterized tests中心にし、専用PBT frameworkは使わない
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 10

PBTのCI実行方針はどれにしますか？

A) 通常CIで軽量seed固定、nightlyで試行回数を増やす
B) 通常CIで十分な試行回数を実行し、nightlyは設けない
C) ローカルのみ必須にし、CIでは当面実行しない
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 11

U1のobservability要件として、どこまでをMVPに含めますか？

A) structured log、request ID、error code、audit outbox statusを必須にする
B) Aに加えてAPI latency metric、dashboard section unavailable metricを必須にする
C) Bに加えてSLO dashboardとalert ruleまで必須にする
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 12

shared DTO/schemaの互換性管理はどの方針にしますか？

A) 破壊的変更はMVP中も避け、schema versionとmigration noteを必須にする
B) MVP中は破壊的変更を許容するが、変更履歴を残す
C) code generation前までは自由に変更し、以降に互換性管理を始める
X) Other (please describe after [Answer]: tag below)

[Answer]: A
