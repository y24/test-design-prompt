# PROMPT 11: テスト設計（P0/P1中心・リスク起点でテストケース/チャーター化）
PROMPT 00（rbt_min_v1）に準拠してJSONのみを出力せよ。
このプロンプトは、評価済みリスク（PROMPT 08/09）とテスト方針（PROMPT 10）を入力として、
優先度の高いリスクから「実行可能なテストケース/テストチャーター」を設計するチェックパスである。

## 1) 目的
- リスク（risk_id）に紐づく形で test_catalog.tests を作る（linked_risks 必須）。
- 原則 P0/P1 を優先して設計する（P2/P3は方針に従い、必要なものだけ）。
- “根拠なしの期待結果” を書かない。oracle（判定材料）が不足するなら status="info" と info_requests を出す。

## 2) 入力
- リスク統合/評価済み JSON（PROMPT 08 or 09）
  - risk_register.risks
  - info_requests（あれば）
- テスト方針 JSON（PROMPT 10）
  - plan.test_policy / plan.viewpoints / plan.risk_to_test_intent など
- （任意）既存テスト資産（sources.type="test_asset"）
  - 流用できる場合は、testの extras に `reuse_of` を記録する（IDや参照を残す）

## 3) スコープ（逸脱防止）
- scope.in_scope/out_of_scope を厳守。
- scope.delta_scope.mode="diff_only" の場合：
  - 影響を受けるリスク（touched_componentsに紐づく）に対するテストのみ追加/更新する。
  - 差分外のテストを“ついでに整備”しない。

## 4) 出力セクション（このプロンプトで出すもの）
- test_catalog.tests（必須）
- info_requests（必要なら追加/更新）
※ risk_register は原則出さない（評価済みを前提にする）
output_type は原則 "partial"（追加/更新したテストのみ）を推奨。
統合済み一式が必要なら "package" でもよい。

## 5) テストの採用基準（方針の実装）
- P0:
  - 原則、各リスクに対して最低1件以上のテストを作る（重大リスクの未カバー禁止）。
  - 正常系だけでなく、代表異常系/境界/回復（再実行・取消・リトライ等）を含める（plan.coverage_targetsに従う）。
- P1:
  - 主要分岐と重要データ条件をカバーする。
- P2/P3:
  - plan.test_policy.selection_rules に従う（条件付き・任意）。
  - 迷う場合は作らず、coverage_gaps相当の不足（この軽量版では info_requests で代替）として理由を残す。

## 6) test_catalog.tests[] の必須キー（PROMPT 00準拠）
各テストは最低限以下を満たす：
- test_id: "TST-0001" 形式（新規採番）
- title: 具体（失敗モード/観測点が分かる）
- priority: P0〜P3（原則、紐づくリスクの最大 priority を上限にする。根拠なく上げない）
- status: confirmed/hypothesis/info
- linked_risks: ["RSK-0001", ...]（必須。最低1つ）
- evidence: source_id + locator + note（必須）
推奨キー：
- steps: 実行手順（配列）
- expected: 期待結果（配列）
- extras: 自由欄（level, technique, test_data, env, oracle, automation_candidate, timebox_minutes, reuse_of 等）

## 7) “期待結果（oracle）” の扱い（事故防止の中核）
- expected/oracle は、仕様・設計・運用要件など evidence を伴うものだけを書く。
- 仕様にない期待結果を“それっぽく”書かない。
- oracle が不足する場合：
  - test.status="info"
  - info_requests に「何を確認すれば期待結果が確定するか」を追加
  - それでも可能な範囲で「観測可能な事実」（例：エラーコード、ログ出力、DB整合性チェック）を候補として extras.oracle_candidates に書いてよい（ただし断定しない）

## 8) リスク↔テストの紐づけ戦略（実務寄り）
- 1テストで複数リスクをカバーしてよいが、次を守る：
  - linked_risks を増やしすぎない（原則1〜3）
  - 目的がぼやけるなら分割する
- 逆に、1リスクに複数テストが必要な場合（P0で多い）：
  - 例：正常系/異常系/並行/再実行/運用復旧 などに分ける
- plan.risk_to_test_intent がある場合は、それに沿って test.extras.intent_ref を付けてもよい

## 9) 既存テスト資産の取り込み（任意）
- 既存資産に同等テストがある場合：
  - 新規で作らず、流用方針にするのが基本（重複を増やさない）
  - ただし、既存が曖昧/根拠不足なら改修案として新規作成してよい
- 流用する場合：
  - extras.reuse_of に参照（例：既存テストID、ファイル名、行番号）を入れる
  - evidence に test_asset を参照する（source_idで）

## 10) info_requests を必ず出すべき典型（テスト設計が詰まる）
- 期待結果が確定できない（仕様が曖昧、例外時の挙動が未定義）
- データ準備が不明（テストデータの作り方、マスタ条件、期間条件）
- 環境制約が不明（本番同等の負荷/外部I/F/監査ログ/権限設定）
- 観測手段が不明（ログ、監査ログ、メトリクス、DB参照の可否）

## 11) 出力（JSONのみ）
- contract/output_type/meta/scope/sources を含める
- test_catalog.tests（必須）を出す
- 必要なら info_requests を出す
- 余計なトップレベルキーは増やさない（extrasに寄せる）