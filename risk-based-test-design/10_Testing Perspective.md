# PROMPT 10: 優先度付きテスト方針/観点（カバレッジ設計）
PROMPT 00（rbt_min_v1）に準拠してJSONのみを出力せよ。
このプロンプトは、評価済みリスク（PROMPT 08/09 等）を入力として、
「どのリスクを、どの深さで、どのテスト種別で攻めるか」を方針化するチェックパスである。
ここで決めた方針は、次工程（PROMPT 11: テスト設計）の“制約”として機能する。

## 1) 目的
- P0〜P3の優先度に応じて、テストの厚み（深さ・数・種別・レベル）を定義する。
- 「テスト観点（レビュー軸）」を、リスクカテゴリごとに具体化して列挙する。
- テスト資産（既存テストケース/チェックリスト）がある場合は、活用方針（流用/改修/再設計）を決める。
- 根拠なしの断定は禁止。方針を確定できない要素は info_requests に落とす。

## 2) 入力
- 統合/評価済みの JSON（推奨：PROMPT 08 or 09 の出力）
  - risk_register.risks（必須）
  - scope / sources
  - info_requests（あれば）
- （任意）既存テスト資産（sources.type="test_asset"）

## 3) スコープ（逸脱防止）
- scope.in_scope/out_of_scope を厳守。
- scope.delta_scope.mode="diff_only" の場合：
  - 差分に関係するリスク群の“方針差分”だけを出す（影響のない領域の方針を改変しない）。
  - 既存方針を全体最適化しようとしない（回帰判断が壊れる）。

## 4) 出力セクション（このプロンプトで出すもの）
- plan（必須：この中にテスト方針/観点を格納する）
- info_requests（必要なら追加/更新）
※ risk_register/test_catalog は原則出さない（方針策定に集中する）
output_type は原則 "partial"（方針のみ）を推奨。

## 5) plan の推奨フォーマット（機械処理しやすく、軽量）
plan の中に以下を入れる（キー名はこのまま推奨）：

- plan.test_policy:
  - priority_policy: P0〜P3ごとの基本方針（短文）
  - test_levels: 使用するテストレベル（例：unit/integration/system/e2e/exploratory/operational）
  - selection_rules: どのリスクをテスト化するかの採用基準（例：P0/P1は必須、P2は条件付き、P3は任意）
  - regression_policy: 回帰方針（例：P0/P1は回帰セット固定、差分時は影響範囲回帰を追加）
  - evidence: 方針の根拠（sourcesへの参照。NFRや監査要件など）

- plan.viewpoints:
  リスクカテゴリごとの「観点セット」。各要素は：
  - category: "functional" | "data_integrity" | "security" | "performance" | "reliability" | "operability" | ...
  - checklist: 観点の箇条書き（具体的に。抽象語だけにしない）
  - recommended_test_types: そのカテゴリで優先するテスト種別（例：状態遷移、境界値、権限マトリクス、負荷、障害注入、運用リハ等）

- plan.coverage_targets:
  “厚み”の定義（優先度ごとの最低ライン）。各要素は：
  - priority: "P0"〜"P3"
  - minimum_coverage: 例「主要正常系+代表異常系+境界」「同時実行/再実行」「監査ログ」など
  - notes: 補足（例：P0は自動化候補を必ず検討、など）

- plan.risk_to_test_intent:
  テスト設計（PROMPT 11）で迷わないための意図付け（ただしテストIDはここでは作らない）
  各要素は：
  - risk_id
  - intent: そのリスクをどう検証するかの要約（例：E2Eで再現、APIで権限境界、負荷でSLO検証）
  - suggested_level: 推奨レベル（e2e/system/integration/operational 等）
  - must_have_oracles: 観測すべき指標/判定材料（例：監査ログ、整合性条件、エラーハンドリング）
  - evidence: 根拠（必須）

※ plan の中身は多少拡張してよいが、トップレベルに新しいキーを増やさない（PROMPT 00準拠）。

## 6) 方針策定の具体ルール（軽量だが事故を減らす）
- P0:
  - 原則「落とせない」。正常系だけでなく、代表異常系・境界・回復（再実行/リトライ/取消）を含める。
  - 判定（oracle）が弱いなら、まず info_requests（仕様/ログ/整合性条件）を立てる。
- P1:
  - 主要シナリオ+重要分岐を押さえる。P0の補助線として“穴埋め”。
- P2:
  - 条件付き（使用頻度・影響範囲・変更頻度が高いなら上げる）。回帰セットに入れるかは合理化する。
- P3:
  - 実施は任意。時間があれば探索/軽いスモークで触る、程度に留める。

## 7) info_requests を出すべき典型（方針が固まらない）
- NFR/SLO/RTO等の目標がない（性能/可用性の優先度が決められない）
- 監査ログ要件やデータ分類が不明（セキュリティ/監査の厚みが決められない）
- 正とする整合性条件（invariants）が不明（データ整合性のoracleが決められない）
- テスト環境/データ制約が不明（実行可能性が判断できない）

## 8) 出力（JSONのみ）
- contract/output_type/meta/scope/sources を含める
- plan（上の推奨フォーマット）を必ず出す
- 必要なら info_requests を出す
- 余計なトップレベルキーは増やさない（extrasに寄せる）