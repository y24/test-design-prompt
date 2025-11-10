# LLMによる業務シナリオ設計プロンプト・テンプレート

> **重要**：このガイドは `rules.md` の Phase 2（業務シナリオ設計）の詳細版です。全体のルール、用語定義、JSONスキーマ、リスク評価ルールなどは `rules.md` を参照してください。

## 0. ねらい

* 抽出済みの**テスト観点**（normal／quasi／abnormal）を入力に、ユーザーストーリーや業務フローに沿った**業務シナリオ**へ束ねる。
* 各シナリオは**前提（Given）— 行為（When）— 期待（Then）**で構造化し、**観点トレーサビリティ**と**リスクスコア**を付与する。
* 出力は **JSON**（構造は`rules.md` §7 に準拠。ここでは記述省略）と **Markdown** の二形式。

---

## 1. 前提（入力の貼り方）

* `test_viewpoint_identify_guide.md`の成果（観点リスト）を、参照可能な形で与える。
* 観点には `vp_id`、`test_class`、`risk_score`、`feature`、`source_refs` が含まれていること。
* 任意で**ユースケース図／業務フロー図の要約**や**ペルソナ**を与えると精度が上がる。

```markdown
### Inputs
- viewpoints: <<観点JSONの貼り付け（抜粋可）>>
- story_context:
  actors: ["一般ユーザー","管理者"]
  business_flow: ["会員登録→メール認証→初回ログイン→受注登録→出荷"]
  constraints: ["外部決済は除外","在庫は疑似データ"]
- selection_policy:
  risk_focus: "score>=4を必須カバー、score=3は代表シナリオに編入"
  max_scenarios: 20
```

---

## 2. プロンプト雛形

> **System** と **User** の2段構成。先に`rules.md`を読み込ませる前提。

### 2.1 System Prompt（固定）

```
あなたはソフトウェアQAのアナリスト。与えられた「テスト観点」を業務の流れに沿った「業務シナリオ」に束ねる。
必ず以下を守る：
- 出力は JSON と Markdown の二形式（JSON構造は rules.md §7 準拠、ここではスキーマ説明不要）。
- シナリオは Given/When/Then を核に、目的・終了条件・観点トレーサビリティ(covers_vp)・観測手段(observability)を含める（rules.md §4 Phase 2）。
- 単機能断片の列挙は不可。ユーザーストーリー／業務フローの開始条件→分岐→終了条件で構成する（rules.md §4 Phase 2）。
- リスクは取り込んだ観点の risk_score を合成（maxベース＋頻度補正）してシナリオ risk_score を決定（本ガイド§2.2参照）。
- 高リスク観点(score>=4)は必ずカバー。中低リスクは代表シナリオに編入してケース爆発を抑制（rules.md §11）。
- すべての表現は原文用語準拠。出典(source_refs)はシナリオにも残す。思考過程は出力しない（rules.md §6）。
```

### 2.2 User Prompt（テンプレート）

```
# タスク
以下のテスト観点をカバーする業務シナリオを設計してください。

# 入力
<viewpoints>{{VIEWPOINTS_BLOCK}}</viewpoints>
<story_context>{{STORY_CONTEXT_BLOCK}}</story_context>
<selection_policy>{{SELECTION_POLICY_BLOCK}}</selection_policy>

# 出力要件
1) JSON（rules.md準拠、スキーマ説明不要）
2) Markdown（各シナリオの要約と表）

# 設計方針
- 単機能チェックを束ねて、ユーザーの意図に沿う物語線にする。
- normal/quasi/abnormal を可能な範囲で1本に混在させ、分岐点で準正常・異常の回復やフォールバックを織り込む。
- 観点の重複は統合し、最小本数で高リスクを確実にカバー。

# リスク合成ルール
- 基本：`scenario_risk_score = max(covers_vp.risk_score)` を基本とし、主経路に登場する高頻度観点が多い場合は +1（上限5）。
- 法令/決済/監査系が含まれる場合は最低4。
- リスク評価の基本ルールは `rules.md` §5（リスク評価ルーブリック）を参照。

# 品質チェック
- 各シナリオに：objective, actor, given/when/then, steps, covers_vp, risk_score, end_condition, observability, source_refs。
- すべての risk>=4 観点が、少なくとも1本のシナリオにカバーされている。
- トレーサビリティの欠落なし、用語は原文準拠、実行可能性（データ前提の再現性）がある。
```

**差し替えパラメータ**

* `{{VIEWPOINTS_BLOCK}}`：観点JSON（抜粋可、ただしvp_idとriskは必須）
* `{{STORY_CONTEXT_BLOCK}}`：アクター・業務フロー・制約
* `{{SELECTION_POLICY_BLOCK}}`：最大本数やリスク集中方針

---

## 3. シナリオ設計アルゴリズム（LLMが辿る手順）

> **注意**：この手順は `rules.md` §4 Phase 2 の詳細版です。全体の流れは `rules.md` を参照してください。

1. **ストーリー線の抽出**：`business_flow`をベースに主要経路（Happy Path）を定義。
2. **観点マッピング**：主要経路の各ステージに `viewpoints` を割り当て、空白（normal/quasi/abnormal）を検知。
3. **分岐の挿入**：準正常（回復可能）と異常（想定外）を**支流として接続**し、戻りやフォールバックを明記。
4. **リスク合成**：取り込んだ観点の `risk_score` を max ベースで合成、主経路頻度で補正（上記「リスク合成ルール」参照）。
5. **最小本数化**：重複区間を統合、代表データで束ね、max_scenarios を満たすよう調整。
6. **可観測性の付与**：UIメッセージID・ログキー・DB更新指標を明記（`rules.md` §6参照）。
7. **検収**：`rules.md` §4 Phase 2の検収ルールと§6を充足するまで自己修正。
8. **出力**：JSON → Markdown の順で提示。

---

## 4. Markdown出力フォーマット（最小例）

```markdown
## SC-001: 新規登録〜メール認証〜初回ログイン
- objective: 初回利用者が適切に登録・認証・ログインできること
- actor: 一般ユーザー
- given: 有効なメールアドレス、受信可能な環境
- when: 登録→認証リンク→初回ログインを実行
- then: ダッシュボードが表示され、ロールに応じたUIが反映される
- steps:
  1. 登録画面で必須入力→送信（正常）
  2. メール受信→認証リンクを開く（準正常：メール遅延時の再送確認）
  3. ログイン実行（異常：連続失敗時のロック確認→回復）
- covers_vp: VP-001, VP-002, VP-003
- risk_score: 4
- end_condition: ダッシュボード表示、またはロック後の回復確認完了
- observability: ["MSG-AUTH-200","LOG:auth.success","UI:dashboard.role-bound"]
- source_refs: [DOC-OPS-001 §3.2 p.12-13]
```

---

## 5. Few-shot（例示）

### 5.1 例A：ECサイト「受注→在庫チェック→出荷予約」

**前提となる観点（要約）**

* VP-010 normal：必須項目が満たされれば受注保存。
* VP-012 quasi：在庫不足時は**警告表示の上、保存は可能**（出荷時に再検証）。
* VP-013 abnormal：単価0や割引率>50%は保存拒否。

**期待されるシナリオ（Markdown）**

```markdown
## SC-101: 受注登録〜在庫警告を含む受注確定〜出荷予約
- objective: 在庫状況に応じた適切な受注確定と出荷予約の可否を検証する
- actor: 一般ユーザー
- given: 顧客・品目マスタ有効、在庫は一部不足、価格ルール有効
- when: 受注を登録し、在庫警告を確認しつつ保存→出荷予約を試みる
- then: 受注は保存され、出荷予約時に在庫再検証が行われ不備ならブロックされる
- steps:
  1. 必須項目を入力して受注保存（normal: VP-010）
  2. 在庫不足の警告を確認し保存続行（quasi: VP-012）
  3. 出荷予約時に再検証が走り、不足なら予約不可を確認（quasi派生）
  4. 単価0や割引>50%のデータでは保存自体が拒否されることを別枝で確認（abnormal: VP-013）
- covers_vp: VP-010, VP-012, VP-013
- risk_score: 4
- end_condition: 受注番号採番済／出荷予約の成否が仕様どおり判定
- observability: ["MSG-INV-WARN","MSG-SHIP-ERR","LOG:order.saved","LOG:ship.validate"]
- source_refs: [DOC-SRS-002 §5.1 p.44-47]
```

---

### 5.2 例B：会計ワークフロー「仕訳取込→承認→月次締め」

**前提となる観点（要約）**

* VP-201 normal：仕訳CSVのフォーマットに合致すれば取込成功。
* VP-202 quasi：軽微な行エラーはスキップし、サマリに件数・原因を記録。
* VP-203 abnormal：承認権限がないユーザーの承認操作は拒否。
* VP-204 abnormal：締め処理中に新規仕訳が入るとロールバックし再実行指示。

**期待されるシナリオ（Markdown）**

```markdown
## SC-201: 仕訳取込〜承認〜月次締め（軽微エラーと権限制御を包含）
- objective: 取込エラーの扱い・承認権限・締めの整合性を業務線で検証する
- actor: 経理担当者
- given: 権限テーブル有効、CSVは一部不正行を含む
- when: 仕訳CSVを取込→スキップ件数を確認→承認→月次締めを実行
- then: 不正行はスキップされ、承認権限が正しく判定され、締め時の同時更新はロールバックされる
- steps:
  1. CSV取込（normal: VP-201）→サマリ出力を確認
  2. スキップ発生（quasi: VP-202）→原因集計と件数を確認
  3. 無権限ユーザーの承認試行が拒否されること（abnormal: VP-203）
  4. 締め処理中の新規仕訳投入でロールバック→再実行指示を確認（abnormal: VP-204）
- covers_vp: VP-201, VP-202, VP-203, VP-204
- risk_score: 5
- end_condition: 承認済み状態で月次締め完了または安全なロールバック
- observability: ["LOG:import.summary","MSG:auth.denied","LOG:close.rollback","DB:period.status=closed"]
- source_refs: [DOC-OPS-ACCT §7.3 p.88-95]
```

---

## 6. シナリオ選定と本数最適化

* **優先原則**：`risk_score>=4` を最優先で個別に確実カバー。
* **統合原則**：同一フロー段に属する中リスク（score=3）は代表データで束ねる。
* **上限管理**：`max_scenarios` を超える場合は、優先度（High>Med>Low）と**回復困難性**で削減。
* **非機能の編入**：可能なものは業務フロー内のステップとして埋め込み（例：大量件数、タイムアウト、メッセージ整合の確認タイミング）。

---

## 7. 品質検収チェックリスト

> **注意**：基本ルールは `rules.md` §4 Phase 2の検収ルールと§6（出力品質ルール）、§11（トレーサビリティ）を参照。

* [ ] 各シナリオに **objective / actor / given / when / then / steps / covers_vp / risk_score / end_condition / observability / source_refs**（`rules.md` §4 Phase 2検収ルール）
* [ ] すべての **risk>=4 観点**が少なくとも1本のシナリオでカバー（`rules.md` §11）
* [ ] 単機能の断片列挙になっていない（開始条件→分岐→終了条件で構成、`rules.md` §4 Phase 2）
* [ ] トレーサビリティ欠落なし（covers_vpに観点が正しく紐付く、`rules.md` §11）
* [ ] 用語は原文準拠、データ前提が再現可能（`rules.md` §6）
* [ ] 本数が `max_scenarios` 以内、かつ高リスク集中

---

## 8. 失敗パターンと回避策

* **症状**：観点をそのまま1対1で“テストケース化”

  * **回避**：観点は束ねて物語線を作る。分岐を内包し、準正常→回復、異常→フォールバックを組み込む。
* **症状**：高リスク観点がシナリオ外

  * **回避**：設計前に `risk>=4` の観点のみで**必須カバーマップ**を作り、まずそこから編成。
* **症状**：観測手段が曖昧

  * **回避**：UIメッセージID、ログキー、DB指標など**観測可能な証跡**を必ず記載。
* **症状**：ケース爆発

  * **回避**：代表データでまとめる／分岐は要所に限定／細粒度は別の低優先度シナリオに退避。

---

## 9. 実行例テンプレ（貼り替えて使う）

### 9.1 変数ブロック（人手）

```yaml
VIEWPOINTS_BLOCK: |
  [ { "vp_id":"VP-010","feature":"受注登録","test_class":"normal","risk_score":3 }, ... ]
STORY_CONTEXT_BLOCK: |
  actors: ["一般ユーザー","管理者"]
  business_flow: ["ログイン","受注登録","出荷予約"]
  constraints: ["外部決済除外"]
SELECTION_POLICY_BLOCK: |
  risk_focus: "score>=4必須"
  max_scenarios: 12
```

### 9.2 実行用プロンプト（User）

> 上記 **2.2 User Prompt** に、`VIEWPOINTS_BLOCK`／`STORY_CONTEXT_BLOCK`／`SELECTION_POLICY_BLOCK` を差し込んで送る。

---

## 10. 出力後の後処理（任意）

* **ギャップ検出**：`risk>=4` だが `covers_vp` に含まれない観点を抽出→追加シナリオを要求。
* **テーブル化**：Markdownの要約表を**優先度順**で並べ替えてレビュー用に配布。
* **実行設計**：手順の自動化が可能な部分（データ投入、ログ収集）にタグを付与し、後段の自動化計画へ接続。
