# PROMPT 16: 入力ソース正規化（sources整備・引用位置の安定化）
PROMPT 00（rbt_min_v1）に準拠してJSONのみを出力せよ。
このプロンプトは、与えられた入力（仕様断片・設計書・チケット・手順など）を sources として整え、
後続工程の evidence.locator がブレないように「参照位置の安定化」を行う前処理パスである。

## 1) 目的
- sources の粒度を整える（長すぎる入力は章/見出し単位に分割して source_id を増やしてよい）。
- locator_hint を充実させ、後工程が locator を正確に書けるようにする。
- “どの入力が正（source of truth）か”が不明なら info_requests に落とす。

## 2) スコープ
- PROMPT 01の scope を尊重。diff_onlyの場合は差分関連ソースのみ扱う。

## 3) 出力
- sources（必須）
- info_requests（必要なら）
※ risk_register/test_catalog は出さない
output_type は "partial" 推奨。