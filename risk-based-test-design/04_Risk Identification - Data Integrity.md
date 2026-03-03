# PROMPT 04: リスク抽出（データ整合性 / 計算・状態遷移・永続化）
PROMPT 00（rbt_min_v1）に準拠してJSONのみを出力せよ。
このプロンプトは「データ整合性・計算・状態遷移・永続化」観点でリスクを抽出する専門チェックパスである。

## 1) 目的
- 仕様/設計/手順/過去不具合から、データの正しさが壊れるタイプのリスクを洗い出す。
  例：整合性制約違反、二重計上、丸め差、集計ズレ、履歴不整合、並行更新の競合、リカバリ不備。
- 根拠が薄いものは推測で断定せず、status="info" と info_requests に落とす。

## 2) スコープ（逸脱防止）
- PROMPT 01で確定した scope.in_scope のみ対象。
- scope.delta_scope.mode="diff_only" の場合：
  - touched_components と diff_summary に直接関係するデータ/処理のみを見る。
  - 差分外のデータ設計一般論の列挙は禁止。

## 3) 出力セクション（このプロンプトで出すもの）
- risk_register.risks（必須）
- risk_register.duplicates（可能なら）
- info_requests（根拠不足があれば必須）
※ test_catalog は出さない（ここではテスト設計に踏み込まない）

## 4) 抽出ガイド（データ整合性に特化）
優先的に探すリスクパターン：
- 一意性/参照整合性：
  - 重複登録、孤児レコード、外部キー不整合、論理削除と参照の破綻
- 状態遷移/履歴：
  - 取消・差戻し・再実行で履歴が矛盾、二重反映、ステータス逆行、監査ログ欠落
- 計算/集計：
  - 丸め規則/端数処理の不一致、通貨・単位換算ミス、期間跨ぎ、締め後修正の扱い
- インポート/エクスポート：
  - 文字コード・桁あふれ、NULL/空の扱い、マッピング欠落、部分失敗時の巻き戻り不備
- 並行性/排他：
  - 同時更新で上書きロスト、二重実行、冪等性不足、リトライで重複反映
- 永続化の境界：
  - 画面保存とバッチ更新の競合、キャッシュとDBの不一致、遅延反映の取りこぼし
- 復旧/再処理：
  - 障害復旧後の再開位置誤り、再計算の範囲漏れ、リプレイで二重計上

## 5) リスクの書き方（最小フォーマット厳守）
risk_register.risks[] の各要素は最低限これを満たす：
- risk_id: "RSK-0001" 形式（新規採番）
- title: 短く具体的に（例：「再実行で二重計上される」）
- category: 原則 "data_integrity"（必要なら functional でも可）
- statement: 「もしXなら、Yのデータ不整合が起きる」形式で具体化
- priority: P0〜P3（根拠に基づく）
- status: confirmed/hypothesis/info
- evidence: source_id + locator + note（必須）
推奨：
- affected_assets: テーブル/エンティティ/API/ジョブ/帳票/集計指標/キー項目など
- recommended_tests: 文字列配列（例：整合性検証、再計算、二重実行、締め後修正、冪等性、並行更新）
- canonical_key: 重複束ねキー
- extras: 自由欄（例：想定データ量、整合性条件(invariants)、関連要件ID、既知バグクラス）

## 6) 根拠（evidence）の必須要件（データ向け）
- evidence.locator は「どのデータ/どの処理/どの規則」か追える粒度で書く。
- “整合性条件が書かれていない”ことを根拠にする場合：
  - status は原則 hypothesis か info
  - info_requests で「必要な整合性条件（例：一意キー、履歴の正、締め後の扱い）」を具体的に要求

## 7) 重複（duplicates）
- 同義の不整合（例：二重反映/二重計上/重複登録）が出たら canonical_key で束ねる。
- duplicates に canonical_key / risk_ids / merge_note を入れる（可能なら）。

## 8) info_requests を必ず出すべき典型
- 計算規則（丸め・換算・期間・締め）の定義が曖昧
- 正とするデータ（source of truth）が不明（画面/バッチ/外部連携どれが正？）
- 冪等性の期待（再送/リトライ/再実行）が不明
- 排他・整合性制約（ユニーク、参照、遷移条件）が不明
- 監査ログや履歴の要件が不明

## 9) 出力（JSONのみ）
- PROMPT 00のトップレベルに従い、contract/output_type/meta/scope/sources は必ず含める。
- risk_register（risksは必須）と、必要なら info_requests を出す。
- 余計なトップレベルキーを増やさない（extras に寄せる）。