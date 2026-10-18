---
name: kpt-parsing
description: KPTのCSVを論点ID付きワイド表に集約するロジック（CSV形式・論点ID・対象・Status・新feature判別を含む）。kpt-agent が kpt-summary.md を更新するときに使う。
---

# skill: kpt-parsing

スプレッドシートで管理されたKPTのCSVを取り込み、論点IDを振ったワイド形式（1論点=1行）の表に変換するロジック。

## CSV形式
書式は[kpt-csv.md](templates/kpt-csv.md)に従う。取り込み元は「コミット済みCSVパス」「Issue貼り付けCSV」「人間の自由文入力」のいずれか。自由文の場合はagentがKeep/Problem/Try・機能カテゴリを読み取り、本形式のCSVに整形してから以降の処理に進む。

## 処理
0. 入力をCSV形式で確定させ `docs/kpt/kpt-log/roundN.csv` として保存する（ラウンド番号はAGENTS.md「ディレクトリ規約」参照）。CSV入力はそのまま逐字保存し、自由文入力は整形結果を保存する。保存後は生ログとして扱い削除・改変しない。
1. 機能カテゴリに英小文字スラッグの略号（例: `theme`）を与える。対応は`kpt-summary.md`先頭の凡例で維持する。
2. 各論点に安定した**論点ID** `<略号>-<連番>`（機能カテゴリごとに一意）を付与する。
3. 各論点に**対象**（`spec`/`knowledge`）と**既存feature改善／新feature候補**の区分を付与する（下記）。
4. 過去ラウンドの`kpt-summary.md`があればマージする。同一論点は既存の論点IDを再利用し、解消済みはStatusを`解消済み(RoundN)`に更新する（行は削除しない）。書式は[kpt-summary.md](templates/kpt-summary.md)に従う。
5. 長文の補足がある論点は[roundN-detail.md](templates/roundN-detail.md)に従い`roundN-detail.md`に記載し、KPT一覧の詳細列にリンクする。

## Statusの決定
- Keepのみ（Problem・Tryが空欄）→ `維持`
- KeepとProblemが矛盾 → `矛盾あり`
- Try着手中／Problem対応進行中 → `対応中`
- Problem/Tryが解決済み → `解消済み(RoundN)`
- それ以外 → `未対応`

**許容値**: `維持` / `未対応` / `対応中` / `解消済み(RoundN)` / `矛盾あり`

## 対象の付与（spec / knowledge）
- 機能・データ・挙動・成功条件 → `spec`（要件・Specへのフィードバック対象）。配色・余白・文言・レイアウトのみ → `knowledge`（`design-agent`によるOKF `design-rule`更新の対象、`okf-knowledge`参照）。どちらとも取れる場合は`spec`を既定とし、判断できないものだけ`ambiguity-detection`で質問する。
- 一度確定した`対象`は同一論点IDで再利用する。

## OKF知識の抽出（用語・システム構成案）
- KPT本文に新しいドメイン用語・略語が登場した場合、`okf-knowledge`に従い`docs/knowledge/glossary/`にOKF `glossary-term`として反映する。
- Tryが`spec`にも`knowledge`（design-rule）にも属さない、システム構成・インフラ・技術選定に関する提案の場合、`okf-knowledge`に従い`docs/knowledge/proposals/`にOKF `system-proposal`として反映する（確定仕様ではなく参考メモとして記載する）。
- いずれに該当するか判別できない場合のみ`ambiguity-detection`で質問する。

## 新feature切り出しの判別
- 既存feature`request.md`の「やりたいこと」の範囲内 → **既存feature改善**。範囲を超える新しい目的・画面・データを要求 → **新feature候補**。迷う場合は既存feature改善を既定とする。
- 新feature候補は`kpt-summary.md`末尾「次のアクション」に論点ID・想定feature-name案・根拠を列挙するのみで、切り出しの実行はしない。一度確定した論点は蒸し返さない。

## 曖昧点の扱い（KPT固有）
`ambiguity-detection`で一問ずつ対話的に確認する際、KPTでは特に以下を曖昧点として扱う。
- **対象（spec/knowledge）判別不能**: デフォルト規則（迷ったらspec）で決まらない場合のみ。
- **既存feature改善／新feature候補の判別不能**: デフォルト規則（迷ったら既存feature改善）で決まらない場合のみ。
- **既存design-ruleとの矛盾**: 改善提案が`docs/knowledge/`の確定済み`design-rule`と衝突する場合。回答が得られるまで当該提案は保留し、`kpt-summary.md`の該当論点のStatusを`矛盾あり`とする。

## clarification-log.md
質問・回答の正規記録。`docs/specs/<feature-name>/clarification-log.md`に保持し、書式は[clarification-log.md](templates/clarification-log.md)に従う。

**運用規則**:
- 回答を適用する際は、回答・Status（→`解消済み`）・解消ラウンドを同一処理パスで更新する。
- 曖昧点が解消されるまで改善提案には進まない。

## kpt-summary.md
書式は[kpt-summary.md](templates/kpt-summary.md)に従う。
- **KPT一覧**は毎ラウンド上書き、**ラウンド履歴**は追記専用。
- 改善提案の反映先は「次のアクション」に論点ID付きで記載する。

## roundN-detail.md
該当ラウンドに補足が発生した場合のみ`docs/kpt/kpt-log/roundN-detail.md`を[roundN-detail.md](templates/roundN-detail.md)に従い作成する。ラウンドごとに分割し、累積ファイルにはしない。
