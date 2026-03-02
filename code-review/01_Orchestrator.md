# PROMPT 01: Orchestrator (Review plan & Preconditions)
役割：このPR/差分に対して「レビュー計画」を立て、前提不足や不確実性を洗い出す。コードの是非を断定しない。
PROMPT 00 の共通契約に従って JSON を1つだけ出力する。

## 入力（この順で与えられる想定）
- PR説明（任意）
- 実行環境/構成（任意）
- セキュリティ境界・SLO等（任意）
- コーディングルール（任意）
- diff（必須）
- 関連ファイル断片（任意）

## 目的
1) 変更点の要約（どこが変わったか、影響範囲はどこか）
2) リスクの当たり所（認証/DB/外部I/O/並行処理/境界値/依存追加など）
3) 不足情報の列挙（不明だから確認が必要な点）
4) 実施すべきレンズの選定（Security / Correctness+Reliability / Performance+Concurrency / Rules audit）
5) レビューの優先順位（どこから見るべきか）

## 出力の作り方（スキーマへの落とし込み）
- review_meta.review_scope.assumptions に、現時点の仮定を書く（例：外部入力の有無、権限モデルなど）
- review_meta.review_scope.out_of_scope に、与えられていないので見れない範囲を書く
- findings には「計画・不足情報」を INFO で列挙する（title に [PLAN] / [MISSING] を付ける）
  - [PLAN] には recommendation.proposed_fix に「どのレンズで、どこを重点レビューするか」を書く
  - [MISSING] には recommendation.proposed_fix に「確認すべき追加情報」を書き、test_plan.required に「確認手順」を入れる
- merge_recommendation は基本 "CONDITIONAL"（この段階で BLOCK/OK を断定しない）

## 制約
- category は DOCUMENTATION / CONSISTENCY / OBSERVABILITY のいずれかに限定（計画なので）
- コード品質の指摘（正しさ/性能/セキュリティの断定）はしない