# PROMPT 07: リスク抽出（過去不具合/障害 起点の再発リスク）
PROMPT 00（rbt_min_v1）に準拠してJSONのみを出力せよ。
このプロンプトは「過去不具合・障害報告・チケット」から再発/類似クラスのリスクを抽出する専門チェックパスである。

## 1) 目的
- 過去不具合（defect）・障害（incident）・問い合わせ・運用トラブルを主語にして、
  「同じクラスの失敗がまた起きる」リスクを抽出する。
- “一般論”に逃げない。必ず過去事象（sources.type=defect/incident 等）の根拠を張る。
- 根拠が弱い/情報不足なら status="info" と info_requests に落とす。

## 2) スコープ（逸脱防止）
- PROMPT 01で確定した scope.in_scope の範囲に紐づく事象だけを扱う。
- scope.delta_scope.mode="diff_only" の場合：
  - 差分に関係する領域（touched_components）と近接領域の事象だけを見る。
  - 差分外の全履歴から網羅的に掘り返すのは禁止（ノイズが増える）。

## 3) 出力セクション（このプロンプトで出すもの）
- risk_register.risks（必須）
- risk_register.duplicates（可能なら）
- info_requests（根拠不足があれば必須）
※ test_catalog は出さない（ここではテスト設計に踏み込まない）

## 4) 抽出ガイド（過去不具合起点）
優先する観点（再発しやすいものを上に寄せる）：
- 頻出/再オープン/類似チケット多数（同じ失敗モードが繰り返し）
- 重大（P0/P1相当）：データ不整合、権限漏れ、締め処理停止、復旧困難、監査指摘、顧客影響大
- 根本原因クラス（例）：
  - 境界条件（NULL/空/最大桁/時刻）
  - 排他/同時実行/冪等性/リトライ
  - 状態遷移（取消・差戻し・再実行）
  - 権限/テナント境界/IDOR
  - 集計/丸め/換算/期間跨ぎ
  - 検索条件・フィルタ・UI反映の不整合
  - 設定差分・環境依存・運用手順漏れ
- “修正したはず”なのに再発：回帰リスク（テスト不足/カバレッジ穴の可能性）

## 5) リスクの書き方（最小フォーマット厳守）
risk_register.risks[] の各要素は最低限これを満たす：
- risk_id: "RSK-0001" 形式（新規採番）
- title: 再発の姿が見える題名（例：「再実行で二重計上が再発する」）
- category: 事象の性質に合わせて（functional/data_integrity/security/performance/reliability/operability/compliance/other）
- statement: 「過去にAが起きた。条件Xが残る/再導入されると、同様にBが起きる」形式で具体化
- priority: P0〜P3（根拠に基づく：頻度×影響×未解消度）
- status: confirmed/hypothesis/info
- evidence: source_id + locator + note（必須。チケットキーや障害報告番号はlocatorに入れる）
推奨：
- recommended_tests: 文字列配列（例：回帰、同時実行、再実行、境界値、権限マトリクス、データ整合性検証）
- affected_assets: 影響範囲（画面/API/ジョブ/データ/顧客影響点など）
- canonical_key: 同じバグクラスを束ねるキー
- extras:
  - linked_tickets: ["BUG-123", "INC-45"] のような配列
  - recurrence_hint: "reopened" / "similar_many" / "once" など自由記述
  - root_cause_class: "idempotency" / "boundary" / "authz" 等（自由記述）

## 6) 根拠（evidence）の要件（チケット起点）
- 1リスクにつき最低1件は defect/incident 系の source を evidence に含めること。
- locator にはチケットキー/障害報告番号/該当コメント位置など、追える情報を入れる。
- チケットに原因/再現条件/修正内容が不足している場合：
  - status は hypothesis または info
  - info_requests で「何を確認すれば再発条件が確定するか」を具体化する。

## 7) 重複（duplicates）
- 同一クラス（例：二重計上、権限漏れ、再実行不備）が表現違いで出たら canonical_key で束ねる。
- duplicates に canonical_key / risk_ids / merge_note を入れる（可能なら）。

## 8) info_requests を必ず出すべき典型
- チケットが「何が起きたか」しか書いておらず、再現条件が不明
- 根本原因（設計/実装/運用）の分類が不明
- 修正内容と影響範囲が不明（どこに波及したか）
- 回帰防止策（テスト追加・監視追加）が不明/未実施
- “暫定対応”で終わって恒久対策が不明

## 9) 出力（JSONのみ）
- PROMPT 00のトップレベルに従い、contract/output_type/meta/scope/sources は必ず含める。
- risk_register（risksは必須）と、必要なら info_requests を出す。
- 余計なトップレベルキーを増やさない（extras に寄せる）。