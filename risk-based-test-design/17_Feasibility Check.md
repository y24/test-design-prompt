# PROMPT 17: 実行可能性チェック（環境・データ・権限・運用制約）
PROMPT 00（rbt_min_v1）に準拠してJSONのみを出力せよ。
このプロンプトは、設計したテスト（PROMPT 11）を「本当に実行できるか」という観点で点検し、
実行阻害要因を洗い出して info_requests と推奨アクションに落とすチェックパスである。

## 1) 目的
- テストが “紙の上では正しいが現場で回らない” 事故を減らす。
- 環境・データ・権限・外部I/F・監査ログ・計測の制約を明示し、実行計画に変換する。

## 2) 入力
- test_catalog.tests（PROMPT 11）
- scope / sources / constraints（sources.type="constraint" 等）
- （あれば）運用手順、環境仕様

## 3) 出力
- plan（必須：plan.feasibility に格納）
- info_requests（必須級で出がち）
※ test_catalog を書き換えない（指摘と必要条件に留める）
output_type は "partial" 推奨。

## 4) plan.feasibility 推奨フォーマット
- plan.feasibility.blockers[]:
  - test_id
  - blocker_type: "env" | "data" | "permission" | "external_dependency" | "observability"
  - note
  - suggested_action
  - evidence
- plan.feasibility.summary:
  - executable_now_count / blocked_count
  - note
  - evidence