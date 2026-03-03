# PROMPT 09: リスク評価（優先度付け・確度/根拠の明確化）
PROMPT 00（rbt_min_v1）に準拠してJSONのみを出力せよ。
このプロンプトは、統合済みリスク（PROMPT 08 など）に対して、
priority/status を再点検し、根拠と不足情報を整理する「評価チェックパス」である。

## 1) 目的
- 各リスクの priority（P0〜P3）と status（confirmed/hypothesis/info）を整合させる。
- “高優先度の理由” を evidence と紐づけて明示する（根拠なしの格上げ禁止）。
- 判断に必要な情報が不足する場合は、status="info" に落とし info_requests を強化する。
- ここでは原則、新規リスクを増やさない（評価の場で話を増やすと収拾がつかない）。

## 2) 入力
- 統合済みの JSON（推奨：PROMPT 08 の output_type="package"）
  - risk_register.risks
  - info_requests
  - scope / sources

## 3) スコープ（逸脱防止）
- scope.in_scope/out_of_scope を厳守。
- scope.delta_scope.mode="diff_only" の場合：
  - 差分関連リスクのみ評価対象。
  - 差分外リスクの“ついで評価”は禁止。

## 4) 出力セクション（このプロンプトで出すもの）
- risk_register.risks（評価後：priority/status/evidence/statement の微修正を含む）
- info_requests（必要なら更新・追加。重複はまとめる）
※ test_catalog は出さない
output_type は原則 "partial"（変更があったリスクだけ）を推奨。
ただし、運用上 “評価済み一式” が欲しい場合は "package" でもよい。

## 5) 評価の考え方（軽量だが実務寄り）
priority は “危険の大きさ” で決めるが、根拠で説明できる範囲に留める。

### 5.1 優先度を上げやすい根拠の例（evidenceに紐づける）
- 主要業務フロー/クリティカルデータに直結（締め処理、決済、請求、権限境界、監査ログ等）
- 過去障害・過去不具合で実害が出ている（defect/incident ソース）
- 非機能要件（SLO/RTO/RPO/監査要件）に抵触する恐れが明示されている
- 回避不能/復旧困難/サイレント破壊（気づきにくい破綻）を示唆する記述がある

### 5.2 優先度を下げる/保留にする典型
- 影響範囲が限定され、回避策が明確（ただし根拠がある場合）
- 発生条件が特殊で、通常運用では起こりにくい（根拠がある場合）
- 仕様が未確定で、判断材料が不足（→ status="info" + info_requests）

### 5.3 status の裁定ルール
- confirmed：
  - evidence が「仕様上の要件」または「過去事象」または「設計上の明示」を直接支える
- hypothesis：
  - evidence はあるが、実装/設定/詳細仕様の確認が必要
- info：
  - priority を根拠付きで判断できない、または statement 自体が確定できない

## 6) リスクごとの必須チェック（品質ゲート）
各 risk について次を点検し、必要なら修正する：
- statement が曖昧すぎないか（条件Xと結果Yが具体か）
- priority が statement と evidence に対して過大/過小でないか
- evidence が「その結論に効いている」形になっているか（単なる関係薄い引用の羅列禁止）
- status が evidence の強さに合っているか
- recommended_tests がリスクの性質に合っているか（ただしテスト設計は次工程）

## 7) 出力フォーマット上の注意（重要）
- 既存 risk_id は変更しない。
- 修正したリスクだけを出す場合（output_type="partial"）：
  - risk_register.risks に “変更した risk” のみ列挙する
  - extras に `changed_fields`（配列）と `change_note`（文字列）を入れることを推奨
- info_requests を追加/更新した場合：
  - 必ず related_risks を可能な限り付ける

## 8) 出力（JSONのみ）
- contract/output_type/meta/scope/sources を含める
- risk_register.risks（評価後の更新分）を出す
- 必要なら info_requests を出す
- 余計なトップレベルキーを増やさない（extrasに寄せる）