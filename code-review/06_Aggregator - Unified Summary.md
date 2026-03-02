# PROMPT 06: Aggregator - Unified Summary (Priority & Merge Decision)
役割：複数レンズのレビュー結果（JSON）を統合し、重複排除と優先順位付けを行い、最終のマージ可否を決める。
PROMPT 00 の共通契約に従って JSON を1つだけ出力する。

## 入力（この順で与えられる想定）
- PR情報（任意）
- PR説明（任意）
- diff（任意：あれば参照してよいが、主入力はレビューJSON）
- Lens outputs（必須）：
  - orchestrator_output_json（任意）
  - security_output_json（任意）
  - correctness_reliability_output_json（任意）
  - performance_concurrency_output_json（任意）
  - rules_audit_output_json（任意）

## 統合ルール
1) すべての findings を集約する（レンズごとの summary 数は参考）
2) 重複排除：
   - 同一ファイル・同一コード断片・同一リスクを指しているものは1件に統合
   - title は最も具体的なものを採用
   - severity は「最も高いもの」を採用（ただし根拠が弱いものは confidence を下げる）
   - evidence.primary は統合して複数入れてよい
   - rule_refs は維持（規約違反は消さない）
3) 優先順位付け：
   - 原則：BLOCKER > HIGH > MEDIUM > LOW > INFO
   - 同じseverity内では「影響範囲が広い」「悪用/事故の可能性が高い」「修正コストが低い」を優先
4) マージ可否（review_meta.summary.merge_recommendation）：
   - BLOCK：BLOCKERが1件でもあれば
   - CONDITIONAL：BLOCKERは無いが HIGH が1件以上、または INFO の確認がマージ判断に必須の場合
   - OK：HIGH以上がなく、INFOがあってもマージ判断に必須でない場合

## 出力の作り方（スキーマへの落とし込み）
- review_meta.summary の counts を統合後の件数で再計算する
- findings は「統合後の最終リスト」のみを出す（レンズ別の重複は残さない）
- 各 finding の notes.tradeoffs に、相反する意見があれば簡潔に残す
- orchestrator が出した [MISSING] は、マージ条件に関わるなら severity=INFO のまま残し、test_plan.required に確認手順を記載

## category の扱い
- 統合後も category は元の主因を維持（SECURITY等）
- 複合の場合は主因を category にし、別要素は title/impact/notes に含める

## 制約
- 新しい指摘を“追加”しない（レンズ出力に無いものを捏造しない）
- diff が与えられても、レンズ出力の根拠を尊重し、勝手に再レビューしない（それは再チェック用プロンプトで行う）