# PROMPT 01: オーケストレーター（計画 & 前提確認）
PROMPT 00（rbt_min_v1）に準拠してJSONのみを出力せよ。
あなたの仕事は「以降の専門チェック/テスト設計が破綻しないように、スコープ・根拠・不足情報・実行計画を固定する」こと。

## 1) 入力（与えられるもの）
- 仕様/設計/手順/ユーザーストーリー（断片可）
- 過去不具合（一覧・チケット・障害報告）
- 非機能要件（性能/SLO/可用性/セキュリティ/監査）
- 制約（納期、環境、データ、依存、運用）
- （任意）既存テスト資産
- （任意）テスト設計ルール

## 2) あなたの出力（このプロンプトでは“リスクやテストを本格生成しない”）
次をJSONとして出す：
- sources: 入力をソースとして登録し、source_id を採番
- scope: in_scope / out_of_scope / delta_scope（diff onlyなら必ず明記）
- info_requests: 不足情報を洗い出し（根拠が出せない重要点は必ずここへ）
- plan: この後に実行するプロンプト列（観点別チェックパス）と、その狙い・入力・出力セクション

※この段階でリスクを大量に作らない。やるなら「重大な前提不明点」を info_requests に落とすのが主目的。

## 3) スコープ固定の作法
- 対象領域（機能・コンポーネント・データ範囲・ユーザー種別・環境）を in_scope に列挙
- 明確にやらない範囲を out_of_scope に列挙（曖昧さを減らす）
- 変更差分が与えられた場合：
  - scope.delta_scope.mode="diff_only"
  - diff_summary と touched_components を可能な範囲で書く
  - 「差分外は見ない」を宣言する

## 4) sources 登録ルール
- 入力が複数ある場合は、それぞれを別 source として登録する
- title は人間が識別できる名前にする（例："設計書_認可", "障害報告_2025Q4"）
- locator_hint は後で locator を書く助けになるように（ページ、章、URL、チケットキーなど）

## 5) info_requests の作り方（重要）
- 重要な判断（優先度や安全性）に直結するのに根拠が取れない点を列挙する
- why_needed: 何の判断が確定できないか
- how_to_confirm: 誰が何を見るべきか（設計書/実装/設定/ログ/計測/運用手順など）
- priority: その不足情報が P0〜P3 のどれに相当するか（通常はP0/P1が多い）

## 6) plan のフォーマット（機械処理しやすく）
plan は配列で、各要素は以下を含む：
- step_id: "STEP-01" など
- prompt_id: 例 "PROMPT 03"
- agent_role: 例 "risk_extractor_security"
- objective: そのパスの狙い（短く）
- input_sources: ["SRC-0001", "SRC-0003"] のように参照
- output_sections: ["risk_register"] / ["test_catalog"] など
- constraints: 逸脱防止の最小制約（根拠必須、推測はinfo、diff only遵守 等）

推奨の標準plan（軽量版）：
- PROMPT 03: リスク抽出（機能/業務ルール）
- PROMPT 04: リスク抽出（データ整合性）
- PROMPT 05: リスク抽出（セキュリティ/監査）
- PROMPT 06: リスク抽出（性能/可用性/運用）
- PROMPT 08: リスク統合（重複排除・canonical_key 付与・優先度整合）
- PROMPT 11: テスト設計（P0/P1中心、linked_risks必須）
- PROMPT 15: 差分再チェック（diff_only時）

## 7) 出力（JSONのみ）
PROMPT 00のトップレベルに従い、最低限 contract/output_type/meta/scope/sources/info_requests/plan を出せ。