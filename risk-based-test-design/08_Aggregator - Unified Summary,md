# PROMPT 08: リスク統合（重複排除・正規化・優先度整合）
PROMPT 00（rbt_min_v1）に準拠してJSONのみを出力せよ。
このプロンプトは、複数のリスク抽出パス（PROMPT 03〜07 等）の出力を統合し、
重複排除・正規化・優先度の整合を行う「統合チェックパス」である。

## 1) 目的
- 複数パスで出た risk_register を統合し、次の状態にする：
  - 重複が束ねられ（canonical_key/duplicates）
  - statement が具体化され
  - priority/status が矛盾なく説明され
  - evidence が不足しているものは info に落ち、info_requests が整理される
- ここでは原則「新規に大量のリスクを増やさない」。
  例外：統合の過程で“統合しないと見落とす”重大ギャップが確定根拠つきで見つかった場合のみ追加可。

## 2) 入力
- 複数の JSON（PROMPT 03〜07の出力）：
  - sources / scope / risk_register / info_requests を含む
- もし入力にスコープが複数混在する場合：
  - PROMPT 01（オーケストレーター）の scope を最優先し、それ以外は out_of_scope 扱いにする。

## 3) スコープ（逸脱防止）
- scope.in_scope/out_of_scope を厳守。
- scope.delta_scope.mode="diff_only" の場合：
  - 差分に関係するリスク群のみ統合対象とする。
  - 差分外のリスクを「ついでに」整備しない。

## 4) 出力セクション（このプロンプトで出すもの）
- risk_register.risks（統合後の正規化リスク。可能なら “統合版” を出す）
- risk_register.duplicates（必須：束ねた結果を出す）
- info_requests（統合して重複排除したもの。必要なら優先度付け）
※ test_catalog は出さない

output_type は原則 "package" を推奨（統合結果として扱いやすい）。
ただし運用上、差分統合だけを返したい場合は "partial" でもよい（その場合は touched_risks を extras に記録）。

## 5) 統合手順（推奨アルゴリズム）
### 5.1 正規化
- title：短く、失敗モードが見える形に寄せる（冗長な背景は statement へ）
- statement：必ず「もしXなら、Yが起きる」形式を保つ
- category：最も支配的な性質で選ぶ（迷ったら other でなく、近いものへ寄せる）
- priority：P0〜P3。根拠（evidence）と rationale（extrasでも可）に矛盾させない
- status：
  - evidence が十分 → confirmed
  - evidence が断片的 → hypothesis
  - evidence 不足/確認待ち → info（info_requests必須）

### 5.2 canonical_key の付与
- 目的：同義・類似を束ねるためのキー
- ルール：英数字スネークケース推奨（例：`idempotency_double_apply`, `authz_tenant_boundary`）
- 同義判定の観点：
  - 同じ失敗モード（例：二重計上、権限境界漏れ、タイムアウト）
  - 同じ影響資産（同じAPI/同じジョブ/同じ帳票）
  - 条件違いは statement に残し、同一キーに束ねてもよい（merge_noteに違いを書く）

### 5.3 duplicates の生成（必須）
- duplicates[] に、canonical_key ごとの risk_ids を列挙する
- merge_note に「何が同じで何が違うか」を短く書く

### 5.4 evidence のマージ
- 統合後の1リスクに対し、複数の evidence を束ねる（最大でも必要十分に）
- evidence.note は「この根拠から何が言えるか」を明確に
- “記載なし”を根拠にする場合は、status を hypothesis/info に寄せる

### 5.5 優先度整合（priority）
- 各パスで priority が食い違う場合：
  - より高い priority に寄せるのではなく、根拠で裁定する
  - 根拠が揃わない場合は status を hypothesis/info にし、info_requests に落とす
- 重要：P0/P1 は “理由” が必要（evidence.note または extras.rationale）

## 6) info_requests の統合
- 重複する不足情報はまとめる（topic を正規化）
- related_risks を可能な限り付ける
- priority は、紐づくリスクの最大 priority に引っ張られやすいが、根拠で説明できない場合は上げない

## 7) 最低限の出力品質ゲート
- risks が空なら：
  - 仕様/入力が不足している可能性が高いので、info_requests を必ず出す
- すべてが status="info" なら：
  - “確定できない理由” を info_requests に集約し、確認手順を具体化する
- duplicates が空でもよいが、同義がありそうなら必ず作る（統合の役割）

## 8) 出力（JSONのみ）
- contract/output_type/meta/scope/sources を含める
- risk_register.risks / risk_register.duplicates を出す
- 必要なら info_requests を出す
- 余計なトップレベルキーは増やさない（extrasに寄せる）