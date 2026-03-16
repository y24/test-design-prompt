# issue_log_template.md

## 1. 目的

本ファイルは、非機能に関する品質基準策定プロセスにおいて発生する  
**不足情報、資料間矛盾、判断保留事項、確認待ち事項、差戻し事項** を継続管理するためのテンプレートである。

issue は、推測で埋めて先へ進まないための受け皿として機能する。  
本プロセスでは、定められないこと自体も重要な成果物であるため、  
issue は単なるメモではなく、レビュー可能で追跡可能な管理対象として扱う。

---

## 2. 本ファイルの位置づけ

本ファイルは、以下の目的で使用する。

- 不足情報を明示する
- 資料間矛盾を記録する
- 判断保留事項を継続管理する
- 後続ステップに影響する未解決事項を追跡する
- レビュー結果や追加資料に応じて状態更新する
- 最終成果物に残る未確定事項の根拠とする

本ファイルは、各 `step guide` から参照され、各ステップ終了時に必要に応じて更新される。

---

## 3. 記録対象

以下に該当するものは、issue として記録対象とする。

- 必要資料の不足
- 必要前提情報の不足
- 資料間の矛盾
- 対象範囲の不明確さ
- 品質基準候補の定義困難
- 目標値設定の根拠不足
- 確認方針の成立条件不足
- 品質特性分類の判断保留
- ステークホルダー確認待ち
- Step 3 / Step 4 などへの差戻しが必要な論点
- 最終成果物に未確定として残すべき論点

---

## 4. 記録対象外の目安

以下は、原則として issue にしなくてよい。

- 軽微な表記ゆれ
- 後続判断に影響しない文言調整
- その場で明確に解消できる単純な誤記
- 追跡管理の必要がない一時的な補足メモ

ただし、迷う場合は記録する側を優先する。

---

## 5. issue レコード項目

各 issue は、少なくとも以下の項目を持つこと。

| 項目名 | 必須 | 内容 |
|---|---|---|
| Issue ID | 必須 | 一意な識別子。例: ISSUE-001 |
| Title | 必須 | issue を短く表す見出し |
| Type | 必須 | issue の種別 |
| Detected Step | 必須 | 発見されたステップ |
| Status | 必須 | 現在状態 |
| Summary | 必須 | issue の内容概要 |
| Detail | 推奨 | 背景、具体的な問題内容、判断保留理由など |
| Impact | 必須 | 影響範囲 |
| Related Quality Characteristic | 推奨 | 関連する品質特性 |
| Related Target | 推奨 | 関連する業務、機能、画面、処理、運用など |
| Related Concern ID | 任意 | 関連 concern |
| Related Criterion Candidate ID | 任意 | 関連品質基準候補 |
| Related Criterion ID | 任意 | 最終品質基準との対応 |
| Evidence / Source | 推奨 | 根拠資料や参照箇所 |
| Required Action | 推奨 | 解消や確認のために必要な行動 |
| Required Additional Information | 推奨 | 必要な追加資料や確認事項 |
| Owner / Confirmation Point | 任意 | 誰に確認すべきか、どの論点として確認すべきか |
| Due Timing | 任意 | いつまでに確認が必要か |
| Resolution | 任意 | 解消内容 |
| Last Updated Step | 必須 | 最終更新ステップ |
| Notes | 任意 | 備考 |

---

## 6. 値の定義

### 6.1 Type の推奨値

`Type` は、原則として以下から選ぶ。

- `missing-information`
- `missing-document`
- `document-conflict`
- `scope-unclear`
- `classification-pending`
- `criterion-definition-pending`
- `target-setting-pending`
- `confirmation-policy-pending`
- `stakeholder-confirmation-needed`
- `cross-cutting-conflict`
- `step-back-required`
- `final-undetermined`

必要に応じて追加してよいが、増やしすぎないこと。

### 6.2 Detected Step / Last Updated Step の推奨値

- `Step1`
- `Step2`
- `Step3`
- `Step4`
- `Step5`
- `Step6`

### 6.3 Status の推奨値

- `open`  
  未解決。今後の確認や判断が必要
- `pending-review`  
  レビューまたは関係者確認待ち
- `in-progress`  
  解消に向けた整理中
- `resolved`  
  解消済み
- `closed`  
  解消済みで、以後の追跡不要
- `accepted-undetermined`  
  現時点では未確定のまま残すことを受容した状態

`resolved` と `closed` は区別してよい。  
記録として残しつつ追跡したいなら `resolved`、完全終了なら `closed`。

---

## 7. 記述ルール

### 7.1 Title は短く具体的に書く
悪い例:
- 不足
- 矛盾あり
- 要確認

望ましい例:
- 月次取込の想定データ件数が不明
- 対応ブラウザ記載が要件定義書と運用資料で不一致
- 監査ログ保持期間の定義根拠が不足

### 7.2 Summary と Detail を分ける
- `Summary` には一読で内容が分かる短い要約を書く
- `Detail` には背景、矛盾の内容、未確定理由などを補足する

### 7.3 Impact は具体的に書く
「影響あり」では弱い。  
以下のように、何にどう影響するかを書く。

- Step 4 の目標値設定ができない
- 互換性と移植性の候補整理に影響する
- 最終成果物で未確定事項として残る
- 対象範囲の確定ができないため Step 2 以降に波及する

### 7.4 Required Action は確認行為として書く
例:
- 利用部門に月次最大件数を確認する
- 正とする対応ブラウザ一覧を確認する
- 旧システム連携仕様の最新版を入手する
- 権限表の正本資料を特定する

### 7.5 Resolution は事実ベースで書く
例:
- 2026-03-XX 付の運用資料追補版で想定件数を確認し、Step 4 に反映した
- 要件定義書 第3版を正本とする旨の指示があり、競合資料を参考扱いへ変更した

---

## 8. 記録・更新タイミング

issue は、少なくとも以下のタイミングで記録または更新する。

### 8.1 Step 1
- 不足資料
- 対象範囲不明
- 利用環境不明
- 外部連携前提不明
- 対象外範囲不明

### 8.2 Step 2
- concern 抽出に必要な前提不足
- 特性分類保留
- concern の適用範囲不明
- 根拠資料不足

### 8.3 Step 3
- 品質基準候補へ変換困難
- 候補の粒度確定困難
- 候補の必要性は高いが表現確定不可

### 8.4 Step 4
- 目標値の根拠不足
- 確認方針の成立条件不足
- 暫定値の正式化に追加確認が必要

### 8.5 Step 5
- 候補間の重複・矛盾
- 横断的な抜け漏れ
- 差戻し必要論点
- 共通前提不足の再整理

### 8.6 Step 6
- 最終成果物に残る未確定事項
- accepted-undetermined として残す論点
- 最終レビュー重点確認事項

---

## 9. テンプレート

以下の表形式を基本テンプレートとして使用する。

| Issue ID | Title | Type | Detected Step | Status | Summary | Detail | Impact | Related Quality Characteristic | Related Target | Related Concern ID | Related Criterion Candidate ID | Related Criterion ID | Evidence / Source | Required Action | Required Additional Information | Owner / Confirmation Point | Due Timing | Resolution | Last Updated Step | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| ISSUE-001 |  |  | Step1 | open |  |  |  |  |  |  |  |  |  |  |  |  |  |  | Step1 |  |

---

## 10. 最小記録例

### 10.1 不足情報の例

| Issue ID | Title | Type | Detected Step | Status | Summary | Detail | Impact | Related Quality Characteristic | Related Target | Related Concern ID | Related Criterion Candidate ID | Related Criterion ID | Evidence / Source | Required Action | Required Additional Information | Owner / Confirmation Point | Due Timing | Resolution | Last Updated Step | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| ISSUE-003 | 月次取込の想定データ件数が不明 | missing-information | Step1 | open | 月次CSV取込処理の想定件数が資料上確認できない | 要件定義書と運用資料に取込運用の記述はあるが、件数条件やピーク条件が示されていない | Step4 の性能目標値設定に影響する | 性能効率性, 信頼性 | 月次CSV取込機能 | CONCERN-01 | QCAND-01 |  | 要件定義書 5.2節、運用手順書 3章 | 月次最大件数、平均件数、ピーク条件を確認する | 想定件数、同時実行有無、許容待ち時間 | 利用部門確認 | Step4 前 |  | Step3 |  |

### 10.2 資料矛盾の例

| Issue ID | Title | Type | Detected Step | Status | Summary | Detail | Impact | Related Quality Characteristic | Related Target | Related Concern ID | Related Criterion Candidate ID | Related Criterion ID | Evidence / Source | Required Action | Required Additional Information | Owner / Confirmation Point | Due Timing | Resolution | Last Updated Step | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| ISSUE-014 | 対応ブラウザ記載が資料間で不一致 | document-conflict | Step1 | pending-review | 要件定義書と運用資料で対応ブラウザの記載が一致しない | 要件定義書では Edge / Chrome、運用資料では Edge のみ記載。どちらを正とするか不明 | 互換性、移植性、使用性の整理に影響する | 互換性, 移植性, 使用性 | 利用環境全体 |  | QCAND-08 | QCRIT-08 | 要件定義書 3.4節、運用資料 1.2節 | 正本資料を確認する | 対応ブラウザ一覧、版数、適用範囲 | プロジェクト管理資料確認 | Step4 前 |  | Step4 | 版差異の可能性あり |

### 10.3 最終未確定事項の例

| Issue ID | Title | Type | Detected Step | Status | Summary | Detail | Impact | Related Quality Characteristic | Related Target | Related Concern ID | Related Criterion Candidate ID | Related Criterion ID | Evidence / Source | Required Action | Required Additional Information | Owner / Confirmation Point | Due Timing | Resolution | Last Updated Step | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| ISSUE-022 | 監査ログ保持期間が未確定 | target-setting-pending | Step4 | accepted-undetermined | 監査ログ保持期間の目標値を現時点では確定できない | ログ出力の必要性は確認できるが、保持期間に関する規程・運用基準が提示されていない | セキュリティ基準の一部が暫定のまま残る | セキュリティ | 承認・設定変更機能 | CONCERN-03 | QCAND-02 | QCRIT-02 | 要件定義書 7章、画面仕様書 権限表 | 運用規程または監査要件を確認する | 保持期間基準、閲覧権限、削除ルール | 情報管理ルール確認 | Step6 後も継続 | Step6 時点では未確定として最終成果物へ反映 | Step6 | 最終レビューで重点確認 |

---

## 11. サマリ用の推奨ビュー

最終成果物やレビュー時には、必要に応じて以下の観点で issue を再整理してよい。

### 11.1 ステータス別ビュー
- open
- pending-review
- in-progress
- resolved
- closed
- accepted-undetermined

### 11.2 種別別ビュー
- 資料不足
- 資料矛盾
- 候補化困難
- 目標値設定困難
- 横断矛盾
- 最終未確定

### 11.3 影響範囲別ビュー
- Step 2 以降へ影響
- Step 4 の具体化へ影響
- 最終成果物へ影響
- 後続の検証設計へ影響

### 11.4 特性別ビュー
- 性能効率性関連
- 互換性関連
- 使用性関連
- 信頼性関連
- セキュリティ関連
- 保守性関連
- 移植性関連

---

## 12. 運用上の注意

### 12.1 issue を推測の免罪符にしない
issue を書いたから適当に埋めてよい、にはならない。  
issue は未解決を未解決として可視化するためのもの。

### 12.2 同じ論点を重複起票しすぎない
同一論点が複数候補へ影響する場合は、  
共通 issue としてまとめ、個別候補から参照することを優先する。

### 12.3 解消済みでも記録は残す
解消した issue を削除しない。  
状態を更新して履歴として残すこと。

### 12.4 最終的に unresolved が残ってもよい
重要なのは、未解決を隠さないこと。  
むしろ、未解決が見えるほうが文書として健全なことも多い。

### 12.5 issue と本文の対応を切らない
品質基準候補や最終カタログ側から、関連 issue を追えるようにしておくこと。

---

## 13. 完了条件

本テンプレートは、以下を満たすように利用する。

- 各ステップで必要な issue が記録されている
- issue の状態が更新されている
- 各 issue の影響範囲が明示されている
- 品質基準候補または最終品質基準との関係が追跡できる
- 最終成果物に残る未確定事項が issue として可視化されている