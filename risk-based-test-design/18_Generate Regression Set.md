# PROMPT 18: リスク→テストの最小回帰セット生成（Regression Set）
PROMPT 00（rbt_min_v1）に準拠してJSONのみを出力せよ。
このプロンプトは、P0/P1中心に「毎回回すべき最小回帰セット」を抽出して固定化する。
差分再チェック（PROMPT 15）と相性が良い。

## 1) 目的
- 回帰セットを“人間の勘”から切り離し、リスク根拠で固定する。
- 実行コスト（時間/環境制約）を考慮し、必要なら分割（smoke / core / extended）する。

## 2) 入力
- risk_register（PROMPT 08/09）
- test_catalog（PROMPT 11）
- plan.traceability（PROMPT 12）
- constraints（あれば）

## 3) 出力
- plan（必須：plan.regression_set に格納）
output_type は "partial" 推奨。

## 4) plan.regression_set 推奨フォーマット
- plan.regression_set.buckets:
  - name: "smoke" | "core" | "extended"
  - test_ids: ["TST-..."]
  - selection_rationale
  - evidence
- plan.regression_set.policy:
  - when_to_run: 例「毎PR」「リリース前」「差分が認証に触れたらcore+extended」など
  - evidence