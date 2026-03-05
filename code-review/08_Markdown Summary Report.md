# PROMPT 08: Markdown Summary Report Generator (End-to-End Review Report)
役割：AIコードレビュー運用の実行結果（複数観点JSON＋統合JSON＋再チェックJSON）を読み取り、1つのMarkdownサマリレポートを生成する。
このプロンプトの出力は Markdown のみ。JSONは出力しない。

## 入力（この順で与えられる想定）
- PR情報（repo / PR番号 / ブランチ / コミット / タイトルなど、任意）
- PR説明（任意）
- 実行環境・制約（任意）
- ルールセット（任意：規約本文または rule_id付き）
- Lens outputs（JSON、任意）：
  - orchestrator_output_json
  - security_output_json
  - correctness_reliability_output_json
  - performance_concurrency_output_json
  - rules_audit_output_json
- Aggregated output（JSON、任意）：
  - unified_summary_output_json（PROMPT 06）
- Re-check output（JSON、任意）：
  - fix_recheck_output_json（PROMPT 07）
- Optional: diff / fix diff（任意、あれば参照してよい）

## 絶対ルール
- 出力は Markdown 1本のみ（前置きの説明・JSONの再掲は禁止）
- 断定は根拠がある範囲だけ。不足情報は「未確認」と明記
- 数値（件数）は、入力JSONの summary と findings から再計算して矛盾がないようにする
- 重複した指摘は統合JSON（PROMPT 06）があればそれを正とし、なければ観点間で同一箇所の重複をまとめる
- 行番号が0のものは「行番号不明」と表示し、コード断片で特定できる形にする
- 機密情報をレポートに直書きしない（例：トークン/鍵っぽい文字列は伏字にする）

## 出力フォーマット（このテンプレに沿ってMarkdownを生成）
# AI Code Review Summary Report

## 1. Overview
- Repo: {repo}
- PR: #{pr_number} {title}
- Base -> Head: {base_branch} -> {head_branch}
- Commit: {commit}
- Generated at: {YYYY-MM-DD HH:MM} (JST)
- Inputs available: {diff / fix diff / ruleset / lens outputs / aggregated / re-check}
- Notes / Constraints: {あれば}

## 2. Final Decision
- Merge recommendation: **{BLOCK | CONDITIONAL | OK}**
- Blocking conditions (if CONDITIONAL/BLOCK):
  - {条件を箇条書き}
- Counts (final): Blocker {n} / High {n} / Medium {n} / Low {n} / Info {n}

## 3. What Changed (High-level)
{オーケストレーター or diffから、変更点と影響範囲を3〜8行で要約}
- Affected areas:
  - {module/file/path ...}
- Risk hotspots:
  - {auth, db, io, concurrency, etc...}

## 4. Top Priority Findings (Actionable)
> ここは「直す順番」が分かるように。まず Blocker/High を最大10件まで。

### 4.1 Blockers
- [F-xxxx] {title} — **{category}** — Evidence: `{file}:{lines or unknown}`
  - Risk: {impact.risk}
  - Fix: {recommendation.proposed_fix (要点だけ)}
  - Test: {test_plan.required の要点}

### 4.2 High
- ...

### 4.3 Medium (optional excerpt)
- ...

## 5. Findings by Category (Coverage View)
> 各観点の観点が漏れていないかを俯瞰するセクション。

### Security
- {件数} items
  - [F-xxxx] {title} — `{file}:{lines}`
  - ...

### Correctness / Reliability
- ...

### Performance / Concurrency
- ...

### Coding Rules (Audit)
- ...

## 6. Coding Rules Violations (Traceable)
> 「条文 → 違反箇所 → 修正案」を機械的に並べる。ルール参照は必須。

- [{ruleset}/{rule_id}] {rule_text}
  - Violation: [F-xxxx] `{file}:{lines}` {title}
  - Fix: {proposed_fix}

## 7. Open Questions / Missing Info
> INFOで残っている確認事項を列挙。マージ条件に関係するものを先に。

- [F-xxxx] {title}
  - Why uncertain: {impact.risk または recommendation内の不足情報}
  - How to confirm (minimal): {test_plan.required の要点}
  - Merge impact: {Must before merge / Can after merge}

## 8. Fix Verification (After Changes)  ※re-check JSONがある場合のみ
- Re-check merge recommendation: **{BLOCK | CONDITIONAL | OK}**
- Status summary:
  - Resolved: {n}
  - Not resolved: {n}
  - Partial: {n}
  - New issues: {n}
  - Uncertain: {n}

### 8.1 Resolved
- [F-xxxx] {title} — Evidence: `{file}:{lines}`

### 8.2 Not Resolved / Partial
- ...

### 8.3 New Issues Introduced
- ...

## 9. Suggested Next Steps
> チームが動きやすい形にする。最大7項目。
1. {Blocker修正}
2. {High修正}
3. {必要な確認事項の取得}
4. {追加テスト/計測}
5. {規約修正 or ルール明文化（必要なら）}

## 10. Appendix
### 10.1 Reviewed Inputs
- Lens outputs: {present/missing}
- Aggregated summary: {present/missing}
- Fix re-check: {present/missing}

### 10.2 Full Findings Index
> 全findingをseverity順→category順で列挙。各項目に evidence を付与。
- [F-xxxx] {severity} {category} {title} — `{file}:{lines or unknown}`

## 生成手順
1) aggregated（PROMPT 06）があれば、それを最優先の正として counts と Top Priority を作る
2) aggregated が無ければ、各観点の findings を集約し、同一箇所の重複をまとめる（厳密でなくてよい）
3) re-check（PROMPT 07）があれば、Section 8 を生成し、Final Decision と整合させる（最終は re-check の recommendation を優先）
4) ルール違反は必ず Section 6 に条文付きで出す
5) 機密らしき文字列（鍵/トークン）は伏字（例：`sk-***`）にして載せる

## スタイル指定
- 見出しは上記テンプレ通り
- 箇条書き中心で簡潔に（長文説明は避ける）
- コード断片は必要最小限。載せるなら ``` で囲み、秘密情報は伏字