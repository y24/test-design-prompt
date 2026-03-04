# PROMPT 12: トレーサビリティ生成（リスク↔テスト対応・カバレッジ欠落検出）
PROMPT 00（rbt_min_v1）に準拠してJSONのみを出力せよ。
このプロンプトは、リスク登録簿（risk_register）とテストカタログ（test_catalog）を入力として、
「リスク↔テストの対応表」と「未カバー/弱カバーの検出」を行う統合チェックパスである。

## 1) 目的
- リスク（risk_id）ごとに、どのテスト（test_id）がカバーしているかを機械的に整理する。
- P0/P1の未カバー（no_test）を必ず検出する。
- “弱カバー” を検出する（例：P0なのにテストが1件だけ、またはoracle/期待結果が曖昧でstatus=info等）。
- 仕様不足・oracle不足・環境不足でカバー判断ができない場合は info_requests に落とす（推測で埋めない）。

## 2) 入力
- リスク統合/評価済み JSON（PROMPT 08/09）
  - risk_register.risks（必須）
- テスト設計 JSON（PROMPT 11）
  - test_catalog.tests（必須）
- scope / sources / info_requests（あれば）

## 3) スコープ（逸脱防止）
- scope.in_scope/out_of_scope を厳守。
- scope.delta_scope.mode="diff_only" の場合：
  - 差分に関係するリスク/テストのみ対象にトレーサビリティを更新する。
  - 差分外の対応表を“ついでに整形”しない。

## 4) 出力セクション（このプロンプトで出すもの）
- plan（必須：plan.traceability に結果を格納する）
- info_requests（必要なら追加/更新）
※ risk_register/test_catalog は原則出さない（入力を参照して対応表を作る工程）
output_type は原則 "partial"（対応表とギャップのみ）を推奨。

## 5) plan.traceability の推奨フォーマット（軽量・固定推奨）
plan.traceability に以下を入れる：

- plan.traceability.links: 配列
  各要素（推奨キー名）：
  - risk_id
  - test_ids: ["TST-0001", ...]
  - coverage_strength: "strong" | "medium" | "weak"
  - rationale: なぜその強度か（短く）
  - evidence: 根拠（必須：少なくとも risk や test の根拠を参照して、対応の妥当性を説明）

- plan.traceability.gaps: 配列
  各要素：
  - risk_id
  - gap_type: "no_test" | "weak_coverage" | "needs_oracle" | "needs_env_or_data" | "needs_spec_clarification"
  - priority: P0〜P3（原則リスク優先度を引き継ぐ）
  - note: 何が足りないか（具体）
  - suggested_action: どう埋めるか（例：テスト追加、既存テスト改修、仕様確認、データ準備、監査ログ確認）
  - evidence: 根拠（必須）

- plan.traceability.anomalies: 配列（任意）
  入力の整合性問題を記録する：
  - type: "test_links_unknown_risk" | "risk_has_no_evidence" | "duplicate_risk_not_merged" など
  - note
  - related_ids（risk_id/test_id等）
  - evidence（可能なら）

※ トップレベルに traceability キーを増やさない。必ず plan 内に入れる。

## 6) coverage_strength の判定ルール（推奨）
“機械的にブレにくい”判定に寄せる（気分で変えない）：

- strong：
  - risk.priority が P0/P1 で、
  - test_ids が複数あり（目安：P0は2件以上、P1は1件以上）、
  - かつ、それらのテストの status が confirmed/hypothesis 中心（info だらけでない）、
  - かつ、テストの expected/oracle が evidence と結びついている（曖昧でない）
- medium：
  - テストはあるが、数が少ない、または異常系/回復/境界が不足しそう
- weak：
  - テストはリンクしているが、status=info が多い、oracle不明、期待結果が薄い、または目的がぼやけている

※ 入力JSONが軽量でexpected/oracleが不足する場合は、coverage_strengthを過剰に strong にしない。
　確信が持てないなら medium/weak に寄せ、必要に応じて gaps/info_requests を出す。

## 7) gap の検出ルール（必須）
- no_test：
  - risk_id に紐づく test が1件も存在しない（linked_risks から照合）
- weak_coverage：
  - P0でテストが1件だけ、またはP0/P1でstatus=infoのテストしかない、など
- needs_oracle：
  - テストはあるが期待結果/判定材料が仕様根拠で確定できない（status=info を伴いやすい）
- needs_env_or_data：
  - テスト実行に必要な環境/権限/データが不足して実施不能
- needs_spec_clarification：
  - 仕様が曖昧で、テストの期待結果が確定できない（info_requestsに落とす）

## 8) info_requests の追加基準（トレーサビリティ視点）
以下は info_requests を追加/更新する：
- P0/P1 の no_test / needs_oracle / needs_spec_clarification が出た
- テストが存在しても「何をもって合格か」が仕様根拠で確定できない
- 環境や監査ログなど“観測”ができない（oracleが成立しない）

## 9) 出力（JSONのみ）
- contract/output_type/meta/scope/sources を含める
- plan.traceability（links/gaps を含む）を必ず出す
- 必要なら info_requests を出す
- 余計なトップレベルキーは増やさない（extrasに寄せる）