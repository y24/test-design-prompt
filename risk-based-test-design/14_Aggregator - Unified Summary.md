# PROMPT 14: 統合サマリ（優先度・実施推奨・不足情報・Go/No-Go）
PROMPT 00（rbt_min_v1）に準拠してJSONのみを出力せよ。
このプロンプトは、これまでの成果（リスク、テスト、トレーサビリティ、監査指摘）を集約し、
「いま何をやるべきか」「何が足りないか」「出荷/実施判断はどうか」を短く判断可能な形に畳み込む統合チェックパスである。

## 1) 目的
- RBTの成果物を“意思決定に使える形”にする。
- 重複排除済みの優先リスク、推奨テスト、未カバー、監査NG、不足情報をまとめる。
- Go/No-Go は断定しすぎない。根拠で支えられる範囲で "go/conditional/no_go" を出す。

## 2) 入力
- リスク統合/評価済み JSON（PROMPT 08/09）
- テスト設計 JSON（PROMPT 11）
- トレーサビリティ JSON（PROMPT 12：plan.traceability）
- 規約監査 JSON（PROMPT 13：plan.audit）
- info_requests（各工程からの不足情報）
- scope / sources

## 3) スコープ（逸脱防止）
- scope.in_scope/out_of_scope を厳守。
- scope.delta_scope.mode="diff_only" の場合：
  - 差分に関係する要素のサマリだけを出す（全体総括にしない）。
  - ただし、差分により P0 相当の新規リスク/未カバーが出た場合は必ず前面に出す。

## 4) 出力セクション（このプロンプトで出すもの）
- plan（必須：plan.summary に統合サマリを格納する）
- info_requests（必要なら統合して更新）
※ risk_register/test_catalog を再掲しない（要約に徹する）
output_type は原則 "partial"（サマリのみ）。必要なら "package" でもよい。

## 5) plan.summary の推奨フォーマット（軽量・固定推奨）
plan.summary に以下を入れる：

### 5.1 plan.summary.decision（意思決定）
- status: "go" | "conditional" | "no_go"
- rationale: なぜその判断か（短く、根拠に紐づける）
- blocking_items: 配列（"conditional/no_go" の要因。具体的に）
- evidence: 根拠（必須）

判断の目安（根拠がある範囲で使う）：
- no_go:
  - P0未カバー（no_test）や、must_fix級の規約違反が残っている
  - 重大な仕様未確定で、P0テストの期待結果が確定できない（needs_spec_clarificationが解消不能）
- conditional:
  - P0は一応あるが弱い/情報不足が残る、P1未カバーが一部、should_fixが残る
- go:
  - P0がカバーされ、重大な監査NGがなく、不足情報が意思決定を阻害しない

### 5.2 plan.summary.top_risks（上位リスク一覧）
配列。各要素：
- risk_id
- title
- priority
- coverage_status: "covered" | "weak" | "not_covered" | "unknown"
- key_tests: ["TST-...."]（あれば）
- note: 一言（なぜ重要/何が懸念か）
- evidence: 根拠（必須）

※ coverage_status は plan.traceability.links/gaps を参照して決める。
  確信が持てないなら "unknown"。

### 5.3 plan.summary.recommended_execution（実施推奨）
- now: 直近でやるべき（P0中心）のテストID/活動
- next: 余裕があればやる（P1/P2）
- defer: 今回は見送る（P3やスコープ外）
各要素は：
- item_type: "test" | "activity" | "fix" | "info"
- ref_id: "TST-0001" / "RSK-0003" / "INF-0002" 等
- reason: 理由（短く）
- evidence: 根拠（必須）

### 5.4 plan.summary.coverage_gaps（カバレッジ欠落の要点）
配列。各要素：
- risk_id
- gap_type: "no_test" | "weak_coverage" | "needs_oracle" | "needs_env_or_data" | "needs_spec_clarification"
- priority
- suggested_action
- evidence

（PROMPT 12 の gaps を要約して重要順に）

### 5.5 plan.summary.audit_blockers（規約監査の阻害要因）
配列（must_fix/should_fix中心）。各要素：
- severity
- rule_ref
- target_id
- suggested_fix
- evidence

（PROMPT 13 の findings を要約して重要順に）

### 5.6 plan.summary.info_gaps（不足情報の集約）
- critical: P0/P1相当の不足情報（INF-xxxx）
- others: それ以外
各要素：
- info_id
- topic
- how_to_confirm
- related_risks（可能なら）
- evidence（不足の根拠として、どこが欠けているか）

※ 既存の info_requests を統合して重複排除する。

## 6) まとめ方のルール（読みやすさと正確さ）
- 数を絞る：
  - top_risks は原則 5〜15件（P0優先）
  - coverage_gaps は P0/P1 を最優先
- evidence を削らない：
  - サマリでも evidence を必須（根拠なしの“それっぽい結論”は禁止）
- 不確実性の表現：
  - 迷ったら status を conditional/unknown 側に寄せ、info_requests に落とす

## 7) 出力（JSONのみ）
- contract/output_type/meta/scope/sources を含める
- plan.summary（上の推奨フォーマット）を必ず出す
- 必要なら info_requests を出す（統合後）
- 余計なトップレベルキーは増やさない（extrasに寄せる）