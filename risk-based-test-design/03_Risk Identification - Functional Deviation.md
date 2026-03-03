# PROMPT 03: リスク抽出（機能・仕様逸脱 / 業務ルール）
PROMPT 00（rbt_min_v1）に準拠してJSONのみを出力せよ。
このプロンプトは「機能・仕様・業務ルール」観点でリスクを抽出する専門チェックパスである。

## 1) 目的
- 仕様/設計/手順/ユーザーストーリー/過去不具合から、
  「機能が期待通りでない」「業務ルールを破る」「境界条件で破綻する」タイプのリスクを洗い出す。
- “推測で埋めない”。根拠が弱いなら status="info" に落とし info_requests を出す。

## 2) スコープ（逸脱防止）
- 対象は PROMPT 01 で確定した scope.in_scope のみ。
- scope.delta_scope.mode="diff_only" の場合：
  - 差分に直接関係する機能・フロー・データだけを見る。
  - 差分外の網羅的なリスク列挙は禁止。

## 3) 出力セクション（このプロンプトで出すもの）
- risk_register.risks（必須）
- risk_register.duplicates（可能なら）
- info_requests（根拠不足があれば必須）
※ test_catalog は出さない（ここではテスト設計に踏み込まない）

## 4) 抽出のガイド（機能/業務ルールに特化）
次のパターンを優先的に探す：
- 業務ルール違反：入力制約、計算/丸め、必須/任意、状態遷移、締め処理、取消/差戻し
- 境界条件：0/負数/最大桁/最大件数/NULL/空/特殊文字/タイムゾーン/うるう年
- 例外系：通信断、二重送信、部分失敗、リトライ、タイムアウト、排他、重複登録
- 不整合：画面/バッチ/API間の仕様齟齬、表示と保存の差、検索と詳細の差
- ユースケース欠落：主要シナリオの未定義、戻る/再実行、権限別の分岐
- 過去不具合の再発：同種のバグクラス、周辺機能への波及

## 5) リスクの書き方（最小フォーマット厳守）
risk_register.risks[] の各要素は最低限これを満たす：
- risk_id: "RSK-0001" 形式（新規採番）
- title: 一目で分かる短い題名
- category: 原則 "functional"（データ寄りなら data_integrity へ寄せてもよい）
- statement: 「もしXなら、Yが起きる」形式で具体的に
- priority: P0〜P3（根拠に基づく）
- status: confirmed/hypothesis/info
- evidence: source_id + locator + note（必須）
推奨：
- recommended_tests: 文字列配列（例：E2Eシナリオ、境界値、状態遷移、異常系、リトライ検証）
- affected_assets: 影響範囲（画面、API、ジョブ、DBテーブル、帳票、業務プロセス名など）
- canonical_key: 重複束ねキー（英数字/スネークケース推奨）
- extras: 自由欄（例：想定ユーザー、関連要件ID、検出方法のメモ）

## 6) 根拠（evidence）の最小要件
- すべてのリスクに evidence を付ける。
- locator は再現可能に（章番号、見出し、ページ、チケットキー、ログ箇所など）。
- 仕様に書かれていない“欠落”を根拠にする場合：
  - evidence.note に「記載が見当たらない」旨を明記し、
  - status は原則 hypothesis か info とし、
  - 確定に必要な info_requests を必ず出す。

## 7) 重複（duplicates）の扱い
- 同義のリスクが複数出た場合、canonical_key を同じにする。
- duplicates に以下を入れる（可能なら）：
  - canonical_key
  - risk_ids
  - merge_note（表現違い/原因違い/影響違い など）

## 8) info_requests の必須条件
次のいずれかに当てはまる場合は必ず info_requests を出す：
- 優先度を決めるのに必要な前提（業務重要度、影響範囲、締め/監査の扱い）が不足
- 正常系/異常系の仕様が曖昧（期待結果が確定できない）
- データ仕様（キー、一意制約、状態遷移、整合性条件）が不明

## 9) 出力（JSONのみ）
- contract/output_type/meta/scope/sources は必ず含める（PROMPT 00に従う）。
- risk_register（risksは必須）と、必要なら info_requests を出す。
- 余計なトップレベルキーを勝手に増やさない（extras に寄せる）。