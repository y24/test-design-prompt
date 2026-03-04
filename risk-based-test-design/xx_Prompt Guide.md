## 共通：どのPROMPTでも使える“投げ方”テンプレ

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
これから指定する PROMPT XX を実行してください。
出力はJSONのみ。説明文・Markdownは禁止。

【入力JSON（前工程の出力）】
<<ここに前の出力JSONを貼る。無い場合は省略>>

【追加入力（任意：文書・差分・ルールなど）】
<<ここに仕様断片やチケットを貼る。複数あるなら区切る>>
```

---

## 例1：新規RBT（フルコース寄り）の実行プロンプト例

### (1) PROMPT 01（入口：sourcesとscope固定）

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
PROMPT 01（オーケストレーター）を実行してください。
出力はJSONのみ。

【今回の前提（ヒント）】
- プロジェクト: 会計Webアプリ
- 対象リリース: 2026-04
- in_scope:
  - 仕訳入力
  - 仕訳承認
  - 仕訳検索（フィルタ機能含む）
- out_of_scope:
  - マスタ管理（別チーム）
  - 外部会計システムへの連携（今回はスタブ）
- delta_scope: none（新規RBT）

【入力（仕様/設計/手順/制約/過去不具合）】
--- DOC A: 仕様断片（仕訳検索）
（ここに文章を貼る）

--- DOC B: 設計断片（検索API/DB）
（ここに文章を貼る）

--- DOC C: 非機能（SLO/監査）
（ここに文章を貼る）

--- DOC D: 制約（期限/環境/データ）
（ここに文章を貼る）

--- DOC E: 過去不具合（チケット抜粋）
（ここに文章を貼る）
```

### (2) PROMPT 02（任意：入力が散らばってるとき）

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
PROMPT 02（対象理解）を実行してください。
出力はJSONのみ。

【入力JSON（PROMPT 01の出力）】
<<PROMPT01のJSONを貼る>>

【追加入力】
なし（PROMPT01で登録したsourcesを前提に作業）
```

### (3) PROMPT 03〜06（観点別リスク抽出：並列でもOK）

例：PROMPT 03（機能・業務ルール）

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
PROMPT 03（リスク抽出：機能・業務ルール）を実行してください。
出力はJSONのみ。

【入力JSON（PROMPT 01の出力）】
<<PROMPT01のJSONを貼る>>

【対象sources（ヒント）】
- 仕様: SRC-0001, SRC-0002
- 手順: SRC-0003
- 制約: SRC-0004
※ source_idはPROMPT01のsourcesから選ぶ
```

例：PROMPT 05（セキュリティ）

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
PROMPT 05（リスク抽出：セキュリティ/監査）を実行してください。
出力はJSONのみ。

【入力JSON（PROMPT 01の出力）】
<<PROMPT01のJSONを貼る>>

【対象sources（ヒント）】
- 権限/監査/NFR: SRC-0005
- 設計: SRC-0002
- 過去不具合: SRC-0006
```

### (4) PROMPT 07（任意：過去不具合があるなら）

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
PROMPT 07（過去不具合/障害 起点の再発リスク）を実行してください。
出力はJSONのみ。

【入力JSON（PROMPT 01の出力）】
<<PROMPT01のJSONを貼る>>

【対象sources（必須ヒント）】
- defect/incident系: SRC-0006, SRC-0007
```

### (5) PROMPT 08（統合：03〜06(+07)の出力をまとめて渡す）

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
PROMPT 08（リスク統合）を実行してください。
出力はJSONのみ。可能なら output_type="package" で統合結果を返してください。

【入力（統合対象JSON）】
--- OUTPUT 03
<<PROMPT03の出力JSON>>

--- OUTPUT 04
<<PROMPT04の出力JSON>>

--- OUTPUT 05
<<PROMPT05の出力JSON>>

--- OUTPUT 06
<<PROMPT06の出力JSON>>

--- OUTPUT 07（任意）
<<PROMPT07の出力JSON>>
```

### (6) PROMPT 09（評価：priority/statusの整合）

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
PROMPT 09（リスク評価）を実行してください。
出力はJSONのみ。変更したリスクだけなら output_type="partial" でOK。

【入力JSON（PROMPT 08の統合結果）】
<<PROMPT08のJSONを貼る>>
```

### (7) PROMPT 10（テスト方針/観点）

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
PROMPT 10（優先度付きテスト方針/観点）を実行してください。
出力はJSONのみ（planを必ず出す）。

【入力JSON（評価済みリスク）】
<<PROMPT08+09を統合したもの、またはPROMPT08のpackage>>

【補足（ヒント）】
- 今回はP0/P1中心。P2は重要画面のみ。P3は基本見送り。
- 自動化候補はE2E中心で検討。
```

### (8) PROMPT 11（テスト設計：リスク＋方針を両方渡す）

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
PROMPT 11（テスト設計）を実行してください。
出力はJSONのみ（test_catalog.tests必須）。

【入力JSON（リスク）】
<<PROMPT08/09のJSON>>

【入力JSON（方針 plan）】
<<PROMPT10のJSON>>
```

### (9) PROMPT 12（トレーサビリティ：リスク＋テスト）

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
PROMPT 12（トレーサビリティ生成）を実行してください。
出力はJSONのみ（plan.traceability必須）。

【入力JSON（リスク）】
<<PROMPT08/09のJSON>>

【入力JSON（テスト）】
<<PROMPT11のJSON>>
```

### (10) PROMPT 13（任意：規約があるときだけ）

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
PROMPT 13（規約監査）を実行してください。
出力はJSONのみ（plan.audit必須）。

【監査対象JSON】
--- RISKS
<<PROMPT08/09のJSON>>
--- TESTS
<<PROMPT11のJSON>>
--- TRACE
<<PROMPT12のJSON>>

【ルール文書】
--- RULES
（ここにテスト設計ルール全文）
```

### (11) PROMPT 14（統合サマリ：全部まとめて判断に落とす）

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
PROMPT 14（統合サマリ）を実行してください。
出力はJSONのみ（plan.summary必須）。

【入力】
--- RISKS
<<PROMPT08/09のJSON>>
--- TESTS
<<PROMPT11のJSON>>
--- TRACE
<<PROMPT12のJSON>>
--- AUDIT（任意）
<<PROMPT13のJSON（あれば）>>
```

---

## 例2：PR差分チェック（最短コース）の実行プロンプト例

### (1) PROMPT 01（diff_only宣言が命）

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
PROMPT 01（オーケストレーター）を実行してください。
出力はJSONのみ。

【前提】
- delta_scope: diff_only
- touched_components（分かる範囲で）:
  - 検索フィルタUI
  - 検索API /search
  - DB: journal 테ーブルのindex

【入力：差分情報】
--- PR説明
（PR概要を貼る）

--- 変更点メモ（diff_summary）
（変更した仕様/挙動の要約を貼る）

--- 変更ファイル一覧（任意）
（ファイル名やモジュール名）
```

### (2) PROMPT 15（差分再チェック：ハブ）

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
PROMPT 15（差分再チェック）を実行してください。
出力はJSONのみ。delta_scope.mode="diff_only" を必ず維持。

【既存成果物（package推奨）】
<<前回の統合済みRBTパッケージ（RISKS+TESTS+TRACE+SUMMARYなど）>>

【入力JSON（PROMPT 01 diff_onlyの出力）】
<<今回のPROMPT01のJSON>>

【追加：差分の根拠】
--- PR説明/差分抜粋
（該当箇所を貼る）
```

### (3) 15の結果で必要な専門パスだけ追加（全部diff_only）

例：権限・監査に触れたなら PROMPT 05（diff_only）

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
PROMPT 05（セキュリティ/監査のリスク抽出）を diff_only で実行してください。
出力はJSONのみ。

【入力JSON（PROMPT 01 diff_only）】
<<PROMPT01のJSON>>
【差分根拠】
（該当変更点の文章/抜粋）
```

その後、必要なら（影響範囲だけ）
**08→09（統合/評価）→11→12→14** を同じ要領で回す。

---

## 例3：情報が増えたとき（仕様追記/NFR追加/障害報告追加）

```text
PROMPT 00（rbt_min_v1）の契約に従ってください。
PROMPT 01 を実行してください。出力はJSONのみ。

【目的】
- sourcesに追加登録して、影響がある工程だけ回す

【追加入力】
--- 仕様追記
（貼る）

--- NFR追記
（貼る）

--- 障害報告
（貼る）
```

次に、追加情報の種類で

* 仕様追記→PROMPT 03/04
* NFR追記→PROMPT 06
* 障害追加→PROMPT 07
  …を回す（必要なら08/09/11/12/14）。
