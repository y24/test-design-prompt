# PROMPT 15: 差分再チェック（diff only：変更差分だけでリスク/テストを再評価・回帰検出）
PROMPT 00（rbt_min_v1）に準拠してJSONのみを出力せよ。
このプロンプトは、修正・更新・変更差分（diff）を入力として、「差分に関係する範囲だけ」を対象に、
リスクとテストを再評価し、回帰（再発/未解消/新規リスク）を検出するチェックパスである。

## 1) 目的
- 差分が与える影響を最小限の範囲で特定し、該当リスク/テストだけを更新する。
- 回帰を検出する：
  - 以前に潰したはずのリスクが再発しうる変更
  - 既存テストが無効化/期待結果が変わる/観測点が変わる
  - 変更により新規に生まれたP0/P1リスク
- スコープ外の改善や網羅チェックは禁止（diff onlyの意味が死ぬ）。

## 2) 入力
- 既存の統合済み成果物（package推奨）：
  - risk_register（統合/評価済み）
  - test_catalog（設計済み）
  - plan.traceability / plan.audit / plan.summary（あれば）
  - info_requests（あれば）
- 変更差分（diff）：
  - PR説明、コミットログ、ファイル差分、設計差分、仕様差分、設定差分、DBスキーマ差分など
  - touched_components（変更された機能/画面/API/ジョブ/テーブル等）があれば必須級で使う

## 3) スコープ（絶対）
- scope.delta_scope.mode は必ず "diff_only" にする。
- scope.delta_scope.diff_summary と touched_components を可能な限り埋める。
- 差分に直接関係しないリスク/テストは出力しない（更新しない）。
- 例外：
  - 差分が共通基盤（認証、権限、共通DB、共通エラーハンドリング、共通ライブラリ）に触れ、
    波及が合理的に説明できる場合のみ、関連範囲へ拡張してよい（根拠必須）。

## 4) 出力セクション（このプロンプトで出すもの）
- risk_register.risks（差分の影響で追加/更新が必要なものだけ）
- test_catalog.tests（差分の影響で追加/更新が必要なものだけ）
- plan（必須：plan.delta_review に差分再チェック結果を格納する）
- info_requests（必要なら追加/更新）
output_type は原則 "partial"。

## 5) plan.delta_review の推奨フォーマット（軽量・固定推奨）
plan.delta_review に以下を入れる：

### 5.1 plan.delta_review.delta_map（差分の影響マップ）
- touched_components: 入力のものを踏襲しつつ、必要なら補足して列挙
- behavior_changes: 変更点の“ふるまい”要約（配列、短文）
- assumptions: 前提（推測は禁止。根拠不足なら info_requests に落とす）
- evidence: 差分の根拠（PR説明/差分箇所の参照。必須）

### 5.2 plan.delta_review.impacted_risks（影響を受けたリスク）
配列。各要素：
- risk_id（既存 or 新規）
- impact_type: "new" | "updated" | "reopened" | "downgraded" | "unchanged_but_reverify"
- reason: なぜ影響があるか（差分と結びつける）
- evidence: 必須

※ reopened は「以前の対策を壊しうる/再発条件を作る」など回帰の匂いがあるとき。

### 5.3 plan.delta_review.impacted_tests（影響を受けたテスト）
配列。各要素：
- test_id（既存 or 新規）
- impact_type: "new" | "updated" | "invalidated" | "needs_oracle_update" | "unchanged_but_rerun"
- reason
- evidence

### 5.4 plan.delta_review.regression_findings（回帰検出）
配列。各要素：
- kind: "risk_regression" | "test_regression" | "coverage_regression"
- severity: "P0" | "P1" | "P2" | "P3"
- description: 何が回帰か（具体）
- related_risks: ["RSK-...."]（あれば）
- related_tests: ["TST-...."]（あれば）
- suggested_action: どう潰すか（テスト追加/戻す/仕様確認/監視追加等）
- evidence: 必須

### 5.5 plan.delta_review.go_nogo_delta（差分観点の判断）
- status: "go" | "conditional" | "no_go"
- rationale
- blocking_items
- evidence

※ これは「差分のせいで判断が変わるか」だけ。全体判断はPROMPT 14の領域。

## 6) リスクの更新ルール（diff onlyで壊れやすい所）
- 既存 risk_id は変更しない。
- 更新が必要なときだけ risk_register.risks に出す（partial運用）。
- priority/status を変える場合は、extras に次を推奨：
  - changed_fields: ["priority","status","statement",...]
  - change_note: "差分Xにより条件が変化…" の短文
- “仕様が変わったのか/実装が変わったのか/観測が変わったのか” を statement と evidence で明確にする。
- 根拠が追えないなら status="info" + info_requests（推測で評価しない）。

## 7) テストの更新ルール（差分でテストが死にやすい所）
- 既存 test_id は変更しない。
- 次に当てはまる場合、既存テストは impacted_tests で拾い、必要なら test_catalog.tests に更新版を出す：
  - 期待結果（oracle）が変わる
  - 入力条件/画面遷移/権限/ログ出力/エラーハンドリングが変わる
  - 手順が成立しない（invalidated）
- invalidated の場合：
  - テスト自体の差分（どう直すか）を suggested_action として plan.delta_review に書き、
  - 更新版テストを出せるなら test_catalog.tests に出す。
- 新規リスクが増えた場合：
  - linked_risks を必ず付けた新規テストを作る（P0/P1優先、方針に従う）。

## 8) 回帰検出の観点（見落としやすい）
次のような差分は回帰を起こしやすいので優先して見る（根拠がある範囲で）：
- 権限・認証・テナント境界・フィルタ条件の変更
- バリデーション/例外処理/リトライ/タイムアウトの変更
- 集計・丸め・期間・締め処理の変更
- DBスキーマ/制約/インデックス/トランザクション境界の変更
- バッチ・キュー・再実行（冪等性）まわりの変更
- ログ/監査ログ/メトリクスの出力変更（観測できなくなる回帰）

## 9) info_requests を出すべき典型（diffだけでは判断不能）
- 差分説明が不足していて、意図/期待挙動が確定できない
- 変更前後の仕様差分が不明（何が“正”か決められない）
- 観測点（ログ/監査/メトリクス）やテスト環境が不足
- 影響範囲（touched_components）が不明で、どこまで見るべきか定まらない

## 10) 出力（JSONのみ）
- contract/output_type/meta/scope/sources を含める（delta_scope.mode="diff_only" を必ず）
- plan.delta_review を必ず出す
- 影響があるものだけ risk_register.risks / test_catalog.tests を出す
- 必要なら info_requests を出す
- 余計なトップレベルキーは増やさない（extrasに寄せる）