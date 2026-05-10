# NFR Design Plan - U1 Project, Workflow, and Data Access Core

## Purpose

U1 `Project, Workflow, and Data Access Core` のNFR Requirementsを、実装前に必要な論理コンポーネント、設計パターン、境界契約へ落とし込む。対象は、dashboard resilience、workflow transition fail-closed、audit outbox lifecycle、project listing scale、scope enforcement、observability、PBT execution designである。

## Source Artifacts

- `aidlc-docs/construction/u1-project-workflow-data-access-core/nfr-requirements/nfr-requirements.md`
- `aidlc-docs/construction/u1-project-workflow-data-access-core/nfr-requirements/tech-stack-decisions.md`
- `aidlc-docs/construction/u1-project-workflow-data-access-core/functional-design/business-logic-model.md`
- `aidlc-docs/construction/u1-project-workflow-data-access-core/functional-design/business-rules.md`
- `aidlc-docs/construction/u1-project-workflow-data-access-core/functional-design/domain-entities.md`
- `aidlc-docs/construction/u1-project-workflow-data-access-core/functional-design/frontend-components.md`

## Unit Context

| Item                    | Value                                                                            |
| ----------------------- | -------------------------------------------------------------------------------- |
| Unit ID                 | U1                                                                               |
| Unit Name               | Project, Workflow, and Data Access Core                                          |
| NFR Requirements Status | Approved by user on 2026-05-09T06:20:34Z                                         |
| NFR Design Focus        | Resilience patterns, performance patterns, security patterns, logical components |
| Primary Package Paths   | `packages/project-workflow/`, `packages/shared-types/`, `packages/data-access/`  |

## Plan Steps

- [x] Analyze U1 NFR Requirements artifacts.
- [x] Create this NFR Design plan and question set.
- [x] Collect user answers for U1 NFR Design questions.
- [x] Validate answers for completeness, contradictions, and ambiguity.
- [x] Generate `aidlc-docs/construction/u1-project-workflow-data-access-core/nfr-design/nfr-design-patterns.md`.
- [x] Generate `aidlc-docs/construction/u1-project-workflow-data-access-core/nfr-design/logical-components.md`.
- [x] Validate NFR Design against Security Baseline and Property-Based Testing applicability.
- [x] Update `aidlc-docs/aidlc-state.md` and `aidlc-docs/audit.md`.

## NFR Design Focus

- Dashboard section timeout/degrade pattern.
- Fail-closed transition dependency checks for U10/U11.
- Audit outbox dispatcher and lifecycle management.
- Project listing query pattern and cursor token design.
- Scope enforcement component placement.
- Auth adapter boundary and provider-neutral contracts.
- Observability instrumentation placement.
- Shared DTO compatibility workflow.
- PBT execution profile and generator organization.

## Questions

以下の質問に回答してください。各質問の `[Answer]:` の後に、該当する選択肢の文字を記入してください。選択肢に合わない場合は `X) Other` を選び、希望内容を記入してください。

## Question 1

dashboard dependent sectionのresilience patternはどれを採用しますか？

A) timeout-only patternにする。各sectionを200msで打ち切り、`DashboardSection.unavailable` にする
B) timeout + circuit breaker patternにする。連続失敗時は短時間openして即unavailableにする
C) timeout + stale summary fallback patternにする。古いsummaryがあればavailableとして返す
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 2

workflow transitionのApprovalGatePort/SecurityPolicyPort fail-closed patternはどれを採用しますか？

A) request内retryなし。200ms timeoutまたはerrorで即fail-closedにする
B) 1回だけ短いretryを行い、それでも失敗したらfail-closedにする
C) dependencyの状態をキャッシュし、直近の成功結果で判断する
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 3

audit outbox dispatcherはどの論理コンポーネントとして扱いますか？

A) U1内のOutbox Dispatcher Componentとして定義し、U11 AuditPortへ配送する
B) U11側のAudit Serviceがoutboxを読む。U1はoutbox table contractだけを定義する
C) U4 Job Orchestrationへdispatch jobを委譲し、U1はenqueue contractだけを定義する
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 4

audit outboxのmanual re-run / intentionally close操作はどのunit境界に置きますか？

A) U1がoperation contractを定義し、UI/実行はU11/U13に委譲する
B) U11が完全に所有し、U1はstatus参照だけにする
C) MVPではmanual re-run/closeのUIは作らず、DB/ops runbookだけで扱う
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 5

project listingでworkspace viewerがProjectMembership経由でアクセス可能projectを取得するquery patternはどれにしますか？

A) ProjectMembershipをjoinして1 queryで取得する
B) ProjectMembershipでprojectIdをページング取得し、Projectを2段階で取得する
C) viewer向けのProjectAccessReadModelを作り、一覧はread modelから取得する
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 6

cursor tokenの設計はどれにしますか？

A) `updatedAt` と `id` を含むopaque base64 JSONにする
B) Aに加えてHMAC署名を付け、改ざんを検出する
C) DB cursor IDを保存し、tokenはserver-side cursorへの参照だけにする
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 7

workflow metadataの読み取りpatternはどれにしますか？

A) 固定enum + in-memory static metadataにする
B) DB metadata tableを読み、process内TTL cacheを使う
C) DB metadata tableを毎回読む
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 8

scope enforcementはどの層に配置しますか？

A) API handlerでWorkspaceScope/ProjectScopeを解決し、use caseへ渡す
B) use case entryでWorkspaceScope/ProjectScopeを解決し、repositoryへ渡す
C) API handlerとuse case entryの両方で検証する
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 9

Auth adapterのNFR Design上の配置はどれにしますか？

A) `packages/project-workflow` にport interfaceだけ置き、実装はapp側に置く
B) `packages/data-access` にscope resolverとして置く
C) 専用 `packages/auth-boundary` を作る
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 10

observability instrumentationはどのpatternにしますか？

A) API middlewareでrequest/log/latencyを扱い、use caseはdomain eventだけ出す
B) API middlewareとuse case wrapperの両方で計測し、operation単位metricを出す
C) 各service method内に個別でlog/metricを実装する
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 11

shared DTO compatibility workflowはどれにしますか？

A) `packages/shared-types/CHANGELOG.md` にmigration noteを集約する
B) 各DTOファイルの近くにmigration noteを置く
C) ADRファイルだけで管理し、package changelogは作らない
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 12

PBT generator/test organizationはどれにしますか？

A) `packages/project-workflow/test/generators/` にU1 domain generatorsを集約する
B) `packages/shared-types/test/generators/` に全unit共通generatorsとして集約する
C) 各property test file内にgeneratorを置く
X) Other (please describe after [Answer]: tag below)

[Answer]: A
