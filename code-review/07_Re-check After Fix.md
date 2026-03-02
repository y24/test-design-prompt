# PROMPT 07: Re-check After Fix (Diff-only Regression)
役割：修正後の差分のみを対象に、(1) 指摘が解消されたか、(2) 修正で新しい問題を入れていないか、を確認する。
PROMPT 00 の共通契約に従って JSON を1つだけ出力する。

## 入力（この順で与えられる想定）
- PR情報（任意）
- 「統合サマリ（PROMPT 06）の出力JSON」（任意だが推奨）
- Fix description（任意：何を直したか）
- fix diff（必須：修正コミットのdiff、またはレビュー対象範囲が分かる差分）
- 追加断片（任意）

## 手順
1) 対象範囲を fix diff に限定する（差分外は原則見ない）
2) もし PROMPT 06 の出力がある場合：
   - 各 finding について、修正差分が該当箇所に触れているかを確認
   - 触れているなら「解消/未解消/部分解消」を判断
   - 触れていないなら「未対応の可能性」として INFO で残す（断定しない）
3) 修正による副作用を検出（差分内に限る）：
   - 例：例外握り潰し、境界値欠落、ログに秘密、性能劣化、競合条件、ルール違反の新規追加など
   - ただし断定は根拠必須。怪しいだけなら INFO で「追加検証案」を出す
4) 最終判定（merge_recommendation）：
   - 以前の BLOCKER/HIGH が解消されていれば改善方向
   - 新規 BLOCKER が差分内で入ったら BLOCK

## 出力の作り方（スキーマへの落とし込み）
- findings の title には接頭辞を付ける：
  - [RESOLVED?] 解消したと思われる（根拠あり）
  - [NOT RESOLVED] まだ残っている（根拠あり）
  - [PARTIAL] 部分的
  - [NEW] 修正で新たに入った問題
  - [UNCERTAIN] diffから確定できない（INFO）
- recommendation.proposed_fix に「次に何を直す/確認する」を書く
- test_plan.required に「最小の再現/確認手順」を書く（特に回帰が怖い箇所）

## category 制約
- 元の finding がある場合はその category を踏襲
- [NEW] は実際の主因カテゴリ（SECURITY/CORRECTNESS/…）を付ける

## 制約
- fix diff 外のコードについて新規に掘らない（スコープ逸脱防止）
- 「直ったはず」などの希望的観測は禁止。根拠が無いなら INFO。