# Requirements Clarification Questions

回答内容を確認した結果、初期MVPのスコープと技術前提に影響する確認点があります。各質問の `[Answer]:` の後に、該当する選択肢の文字を記入してください。選択肢に合わない場合は `X) Other` を選び、希望内容を記入してください。

## Clarification 1: GitHub連携なしでのAmplifyデプロイ

Question 3 と Question 4 では、初期MVPではAI生成コードを画面表示中心にし、GitHub連携は後回しにする回答でした。一方で Question 5 では、AWS Amplify Hosting を初期ターゲットにする回答でした。

Amplify Hosting はGitHub連携で運用する構成が一般的ですが、GitHub連携を後回しにする場合、初期MVPのデプロイ体験をどう扱うか確認が必要です。

### Clarification Question 1

初期MVPでのAWS Amplifyデプロイはどの範囲にしますか？

A) 実デプロイまで行う。ただしGitHub連携なしで、生成コードのビルド成果物または手動アップロード相当の方式を使う
B) Amplifyデプロイ設計とデプロイ前チェックまで行い、実デプロイはGitHub連携を追加する後続フェーズに回す
C) 初期MVPからGitHub連携を復活させ、Amplifyへの実デプロイまで含める
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Clarification 2: データベース技術の範囲

Question 6 では PostgreSQL と Prisma を前提にDB作成・マイグレーションまでGUIから扱う回答でした。一方で Question 14 のPBT対象には DynamoDB item と domain model の相互変換が含まれていました。

初期MVPのDB技術をPostgreSQLに絞るのか、DynamoDBも含めるのか確認が必要です。

### Clarification Question 2

初期MVPのデータベース技術はどこまで対象にしますか？

A) PostgreSQL + Prisma のみに絞る。DynamoDB変換は将来拡張の例として扱う
B) PostgreSQL + Prisma を主軸にしつつ、DynamoDB変換もPBT対象として設計だけ含める
C) PostgreSQL と DynamoDB の両方を初期MVPの生成・検証対象にする
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Clarification 3: 認証なしMVPと外部公開の安全境界

Question 9 では、初期MVPは認証なしのローカル/限定利用で開始する回答でした。一方で Question 5 では Amplify Hosting を初期ターゲットにしており、Question 13 では認証なしの外部公開をブロッキング対象にする回答でした。

このため、初期MVPでプラットフォーム本体や生成アプリをどの範囲まで外部公開してよいか確認が必要です。

### Clarification Question 3

認証なしの初期MVPで許可する公開範囲はどれですか？

A) ローカル実行または限定ネットワークのみ。外部公開は行わない
B) プラットフォーム本体はローカル/限定利用、生成アプリのみ一時的な検証URLとして外部公開を許可する
C) 外部公開する場合は、初期MVPでも最低限の認証またはアクセス制限を必須にする
X) Other (please describe after [Answer]: tag below)

[Answer]: C
