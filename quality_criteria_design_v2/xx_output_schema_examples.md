# output_schema_examples.md

## 1. 目的

本ファイルは、非機能に関する品質基準策定プロセスで用いる**出力スキーマの例**を示す。  
各 `step guide` では、ステップ固有の目的・手順・完了条件を定義するが、  
本ファイルでは **レビューしやすく、トレーサブルで、境界が曖昧にならない出力形式** の例をまとめる。

本ファイルの目的は、以下である。

- 各ステップで出力すべき情報の粒度をそろえる
- 「品質基準」「確認方針」「根拠」「不足情報」の境界を明確にする
- 各ステップの成果物を、後続ステップへ引き継ぎやすくする
- AI エージェントの出力ぶれを抑える

---

## 2. 本ファイルの位置づけ

本ファイルは **出力形式の参考例** を示すものであり、  
絶対的な固定フォーマットではない。  
ただし、以下の原則はなるべく維持すること。

- 各レコードに識別子を付ける
- 根拠資料を追跡可能な粒度で示す
- 未確定事項を独立項目として持つ
- issue との関係を明示する
- 事実、解釈、提案を混ぜすぎない

---

## 3. 共通設計原則

## 3.1 最低限分けるべき情報

出力では、少なくとも以下を分けて扱うこと。

- 対象
- 内容本体
- 判断理由
- 根拠資料
- 不足情報 / 未確定事項
- 関連 issue
- ステータス

---

## 3.2 ID の付与ルール例

以下のような接頭辞を用いて、各レコードに一意な識別子を付けることを推奨する。

- 資料: `DOC-001`
- concern: `CONCERN-001`
- 品質基準候補: `QCAND-001`
- 最終品質基準: `QCRIT-001`
- 横断レビュー項目: `REVIEW-001`
- issue: `ISSUE-001`

ID は厳密な採番規則よりも、**追跡しやすさ** を優先する。

---

## 3.3 ステータスの扱い

成果物の成熟度や確定度を曖昧にしないため、必要に応じて以下のような値を使う。

- `draft`
- `provisional`
- `needs-confirmation`
- `confirmed`
- `reference-example`
- `undetermined`

---

## 3.4 根拠資料の表記原則

根拠資料は、可能な限り以下の粒度で示す。

- 文書名
- 章 / 節
- ページ
- 項番
- 表名 / 図番号
- 画面ID
- URL
- 版数 / 改訂日

例:

- `要件定義書 第2版 / 5.2節 / p.18`
- `画面仕様書 / 画面ID: SCR-INV-001 / エラーメッセージ欄`
- `運用手順書 / 3章 / 月次処理`

---

## 4. Step 1 の出力スキーマ例

## 4.1 入力資料一覧レコード

### 推奨項目

| 項目名 | 内容 |
|---|---|
| Document ID | 資料識別子 |
| Document Name | 資料名 |
| Document Type | 種別 |
| Summary | 主な内容 |
| Intended Use | 本プロセスでの用途 |
| Scope Notes | 参照可能範囲や注意点 |
| Evidence Quality | 根拠としての使いやすさに関する補足 |

### 例

| Document ID | Document Name | Document Type | Summary | Intended Use | Scope Notes | Evidence Quality |
|---|---|---|---|---|---|---|
| DOC-001 | 要件定義書 | requirements | 業務要件、機能要件、一部制約を記載 | 品質基準候補の背景把握 | 非機能記載の粒度は粗い | 業務前提の把握には有効 |

---

## 4.2 対象スコープ要約レコード

### 推奨項目

| 項目名 | 内容 |
|---|---|
| Scope Item | スコープ項目名 |
| Description | 内容 |
| Source | 根拠資料 |
| Status | 確定状態 |
| Notes | 補足 |

### 例

| Scope Item | Description | Source | Status | Notes |
|---|---|---|---|---|
| 対象システム | 連結会計Webアプリケーション | 要件定義書 1章 | confirmed |  |
| 想定利用者 | 経理担当者、管理者 | 要件定義書 2章 | provisional | 利用者属性の詳細は未確認 |
| 想定利用環境 | 社内PC利用 | 運用手順書 1章 | provisional | 対応ブラウザは別資料で不一致 |

---

## 4.3 不足情報 / 要確認事項レコード

### 推奨項目

| 項目名 | 内容 |
|---|---|
| Item ID | 識別子 |
| Category | 不足 / 矛盾 / 判断保留など |
| Summary | 概要 |
| Impact | 影響範囲 |
| Related Step | 影響する後続ステップ |
| Related Issue | 関連 issue |

### 例

| Item ID | Category | Summary | Impact | Related Step | Related Issue |
|---|---|---|---|---|---|
| GAP-001 | missing-information | 同時利用者数が不明 | 性能効率性の具体化に影響 | Step4 | ISSUE-003 |

---

## 5. Step 2 の出力スキーマ例

## 5.1 concern レコード

### 推奨項目

| 項目名 | 内容 |
|---|---|
| Concern ID | concern 識別子 |
| Title | concern 見出し |
| Summary | concern の概要 |
| Target | 対象 |
| Related Quality Characteristic | 関係する品質特性 |
| Background | 背景・文脈 |
| Evidence References | 根拠資料 |
| Missing Information | 不足情報 |
| Related Issue | 関連 issue |

### 例

| Concern ID | Title | Summary | Target | Related Quality Characteristic | Background | Evidence References | Missing Information | Related Issue |
|---|---|---|---|---|---|---|---|---|
| CONCERN-001 | 大量CSV取込時の処理時間 | 月次取込の遅延が業務進行に影響しそう | CSV取込機能、月次運用 | 性能効率性, 信頼性 | 締め前に一括実行される想定 | 要件定義書 5.2節、運用手順書 3章 | 想定件数、ピーク条件 | ISSUE-003 |

---

## 5.2 特性別サマリ例

### 推奨項目

| 品質特性 | 主な concern | 備考 |
|---|---|---|
| 性能効率性 | CONCERN-001, CONCERN-004 | 件数前提不足あり |
| セキュリティ | CONCERN-003 | 監査要件の詳細不足 |

---

## 6. Step 3 の出力スキーマ例

## 6.1 品質基準候補レコード

### 推奨項目

| 項目名 | 内容 |
|---|---|
| Criterion Candidate ID | 候補識別子 |
| Quality Characteristic | 品質特性 |
| Target | 対象 |
| Criterion Title | 見出し |
| Criterion Summary | 概要説明 |
| Rationale | 判断理由 |
| Evidence References | 根拠資料 |
| Related Concern ID | 対応 concern |
| Undetermined Points | 未確定事項 |
| Related Issue | 関連 issue |
| Status | 確定状態 |

### 例

| Criterion Candidate ID | Quality Characteristic | Target | Criterion Title | Criterion Summary | Rationale | Evidence References | Related Concern ID | Undetermined Points | Related Issue | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| QCAND-001 | 性能効率性 | 月次CSV取込機能 | 月次取込処理が業務進行を阻害しない性能 | 月次運用で実施される取込処理は、業務担当者の待機や後続作業の停滞を招かない性能を備えること | 締め前運用への影響が大きいため | 要件定義書 5.2節、運用手順書 3章 | CONCERN-001 | 想定件数、許容待ち時間 | ISSUE-003 | provisional |

---

## 6.2 候補化できなかった concern の例

### 推奨項目

| 項目名 | 内容 |
|---|---|
| Concern ID | 対象 concern |
| Reason | 候補化困難な理由 |
| Missing Information | 不足情報 |
| Suggested Follow-up | 必要な次対応 |
| Related Issue | 関連 issue |

### 例

| Concern ID | Reason | Missing Information | Suggested Follow-up | Related Issue |
|---|---|---|---|---|
| CONCERN-009 | 対象範囲が曖昧で基準候補に変換しづらい | 導入先差分の定義 | 対象環境一覧の確認 | ISSUE-014 |

---

## 7. Step 4 の出力スキーマ例

## 7.1 目標値 / 確認方針レコード

### 推奨項目

| 項目名 | 内容 |
|---|---|
| Criterion Candidate ID | 対象候補ID |
| Quality Characteristic | 品質特性 |
| Target | 対象 |
| Criterion Title | 見出し |
| Target Value or Confirmation Policy | 目標値または確認方針 |
| Type | target-value / confirmation-policy / mixed / undetermined |
| Rationale | 判断理由 |
| Evidence References | 根拠資料 |
| Confidence Status | 確定状態 |
| Missing Information | 不足情報 |
| Related Issue | 関連 issue |

### 例

| Criterion Candidate ID | Quality Characteristic | Target | Criterion Title | Target Value or Confirmation Policy | Type | Rationale | Evidence References | Confidence Status | Missing Information | Related Issue |
|---|---|---|---|---|---|---|---|---|---|---|
| QCAND-001 | 性能効率性 | 月次CSV取込機能 | 月次取込処理が業務進行を阻害しない性能 | 想定件数条件のもとで処理完了時間の目標値を定める。具体値は現時点では要確認 | target-value | 業務待ち時間の合意が必要なため | 要件定義書 5.2節、運用手順書 3章 | needs-confirmation | 想定件数、ピーク条件 | ISSUE-003 |

---

## 7.2 数値化できた項目一覧の例

| Criterion Candidate ID | Target Value Summary | Status | Notes |
|---|---|---|---|
| QCAND-005 | 監査ログ保持期間を数値で定める | provisional | 保持期間規程未提示 |

---

## 7.3 確認方針中心の項目一覧の例

| Criterion Candidate ID | Confirmation Focus | Status | Notes |
|---|---|---|---|
| QCAND-002 | 誤操作防止、導線理解、メッセージ理解 | provisional | 実ユーザー属性未確認 |

---

## 8. Step 5 の出力スキーマ例

## 8.1 横断レビュー項目レコード

### 推奨項目

| 項目名 | 内容 |
|---|---|
| Review Item ID | レビュー項目識別子 |
| Issue Type | duplicate / overlap / conflict / gap / granularity / unresolved |
| Related Record ID | 該当候補ID群 |
| Summary | 問題概要 |
| Detail | 詳細説明 |
| Impact | 影響範囲 |
| Recommended Action | 推奨対応 |
| Related Issue | 関連 issue |
| Status | open / note / needs-fix / escalate |

### 例

| Review Item ID | Issue Type | Related Record ID | Summary | Detail | Impact | Recommended Action | Related Issue | Status |
|---|---|---|---|---|---|---|---|---|
| REVIEW-001 | duplicate | QCAND-001, QCAND-007 | 月次取込性能に関する候補が重複 | 見出しは異なるが対象・背景・確認方向がほぼ同一 | レビュー負荷増加、重複管理 | 1候補へ統合 | ISSUE-017 | needs-fix |

---

## 8.2 未確定事項の横断整理例

| Common Missing Information | Related Record ID | Impact | Recommended Handling | Related Issue |
|---|---|---|---|---|
| 対応ブラウザ一覧 | QCAND-008, QCAND-010 | 互換性、移植性の具体化に影響 | 共通 issue に集約 | ISSUE-014 |

---

## 9. Step 6 の出力スキーマ例

## 9.1 最終品質基準レコード

### 推奨項目

| 項目名 | 内容 |
|---|---|
| Criterion ID | 最終品質基準ID |
| Quality Characteristic | 品質特性 |
| Target | 対象 |
| Criterion Title | 見出し |
| Criterion Summary | 概要説明 |
| Target Value or Confirmation Policy | 目標値または確認方針 |
| Type | target-value / confirmation-policy / mixed / undetermined |
| Rationale | 判断理由 |
| Evidence References | 根拠資料 |
| Undetermined Points | 未確定事項 |
| Related Issue | 関連 issue |
| Status | confirmed / provisional / reference-example / needs-confirmation / undetermined |

### 例

| Criterion ID | Quality Characteristic | Target | Criterion Title | Criterion Summary | Target Value or Confirmation Policy | Type | Rationale | Evidence References | Undetermined Points | Related Issue | Status |
|---|---|---|---|---|---|---|---|---|---|---|---|
| QCRIT-001 | 性能効率性 | 月次CSV取込機能 | 月次取込処理が業務進行を阻害しない性能 | 月次運用で実施される取込処理は、業務担当者の待機や後続処理の停滞を招かない性能を備えること | 想定件数条件のもとで処理完了時間の目標値を設定する。具体値は暫定扱い | target-value | 締め前運用への影響が大きいため | 要件定義書 5.2節、運用手順書 3章 | 想定件数、許容待ち時間 | ISSUE-003 | needs-confirmation |

---

## 9.2 最終サマリ例

### 品質特性別サマリ

| 品質特性 | 主な品質基準ID | コメント |
|---|---|---|
| 性能効率性 | QCRIT-001, QCRIT-004 | 件数前提が未確定 |
| 使用性 | QCRIT-002 | 確認方針中心 |
| セキュリティ | QCRIT-003 | ログ保持期間は未確定 |

### 未確定事項一覧

| Related Criterion ID | Undetermined Summary | Impact | Required Follow-up | Related Issue |
|---|---|---|---|---|
| QCRIT-003 | 監査ログ保持期間が未確定 | セキュリティ基準の最終化に影響 | 運用規程確認 | ISSUE-022 |

---

## 10. 境界を明確にするための記述ルール

## 10.1 「品質基準」と「確認方針」を混ぜない

悪い例:

- 月次取込を速くし、JMeterで確認すること

この書き方だと、基準と確認手段が混ざる。

望ましい例:

- 品質基準: 月次取込処理が業務進行を阻害しない性能を備えること
- 目標値または確認方針: 想定件数条件のもとで処理完了時間を確認対象とする

---

## 10.2 「根拠」と「判断理由」を混ぜない

悪い例:

- 要件定義書にあるので重要である

この書き方だと、資料参照と判断が分離されていない。

望ましい例:

- 根拠資料: 要件定義書 5.2節
- 判断理由: 締め前運用に直結し、遅延が後続作業へ波及するため

---

## 10.3 「不足情報」と「issue」を混ぜない

悪い例:

- 想定件数が不明なので ISSUE-003

この書き方だと、情報不足の内容と issue 管理が未分離。

望ましい例:

- 不足情報: 想定件数、ピーク条件
- 関連 issue: ISSUE-003

---

## 11. 推奨する最終レコード形

最終的には、以下の形を最重要スキーマとして扱うことを推奨する。

| 項目名 | 内容 |
|---|---|
| Criterion ID | 一意な品質基準ID |
| Quality Characteristic | 品質特性 |
| Target | 対象 |
| Criterion Title | 品質基準の見出し |
| Criterion Summary | 品質基準の概要説明 |
| Target Value or Confirmation Policy | 目標値または確認方針 |
| Type | 種別 |
| Rationale | 判断理由 |
| Evidence References | 根拠資料 |
| Undetermined Points | 未確定事項 |
| Related Issue | 関連 issue |
| Status | ステータス |

この形を維持すると、少なくとも次の境界が崩れにくい。

- 品質基準
- 具体化内容
- 根拠
- 不足情報
- issue 管理

---

## 12. 運用上の注意

### 12.1 例を絶対ルールにしない
本ファイルは出力の安定化のための参考例である。  
実資料や対象領域に応じて、列追加や補足は許容する。

### 12.2 ただし列の意味は崩さない
たとえば `Rationale` に根拠資料を書いたり、  
`Undetermined Points` に推奨対応を書いたりしないこと。  
列名と中身がずれると、見た目だけ表で中身は霧、になる。

### 12.3 表と箇条書きは併用してよい
一覧性が必要なものは表、説明補足が必要なものは箇条書き、のように併用してよい。  
ただし、どちらでも情報の境界は保つこと。

### 12.4 ステップをまたいで同じ ID を維持する
concern、候補、最終基準、issue の対応関係が追えるように、  
可能な限り ID の連続性や対応関係を残すこと。

---

## 13. 本ファイルの責務外

以下は本ファイルの責務外である。

- 各ステップの詳細手順
- ステップ進行制御
- issue の運用ルール詳細
- 特性ごとの分析観点詳細
- 実行プロンプトそのもの

これらは `rules.md`、`orchestrator.md`、各 `step guide`、`xx_issue_log_template.md` に委ねる。