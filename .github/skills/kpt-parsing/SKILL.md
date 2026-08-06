---
name: kpt-parsing
description: KPTのCSVを論点ID付きワイド表に集約するロジック（CSV形式・論点ID・対象・Status・新feature判別を含む）。kpt-agent が kpt-summary.md を更新するときに使う。
---

# skill: kpt-parsing

スプレッドシートで管理されたKPTのCSVを取り込み、論点IDを振ったワイド形式（1論点=1行）の表に変換するロジック。

## CSV形式
- ヘッダ7列固定・この順序: `機能カテゴリ,Keep,Keep状態,Problem,Problem状態,Try,Try状態`（UTF-8、1行=1論点、論点ID列は含めない）。
- 各行はKeep/Problem/Tryの少なくとも1つが非空。内容が空欄の種別は対応する状態列も空欄。
- 取り込み元は「コミット済みCSVパス」「Issue貼り付けCSV」「人間の自由文入力」のいずれか。自由文の場合はagentがKeep/Problem/Try・機能カテゴリを読み取り、本形式のCSVに整形してから以降の処理に進む。

## 処理
0. 入力をCSV形式で確定させ `docs/kpt/kpt-log/roundN.csv` として保存する（ラウンド番号はAGENTS.md「ディレクトリ規約」参照）。CSV入力はそのまま逐字保存し、自由文入力は整形結果を保存する。保存後は生ログとして扱い削除・改変しない。
1. 機能カテゴリに英小文字スラッグの略号（例: `theme`）を与える。対応は`kpt-summary.md`先頭の凡例で維持する。
2. 各論点に安定した**論点ID** `<略号>-<連番>`（機能カテゴリごとに一意）を付与する。
3. 各論点に**対象**（`spec`/`rule`）と**既存feature改善／新feature候補**の区分を付与する（下記）。
4. 過去ラウンドの`kpt-summary.md`があればマージする。同一論点は既存の論点IDを再利用し、解消済みはStatusを`解消済み(RoundN)`に更新する（行は削除しない）。
5. 長文の補足がある論点は`roundN-detail.md`に記載し、KPT一覧の詳細列にリンクする。

## Statusの決定
- Keepのみ（Problem・Tryが空欄）→ `維持`
- KeepとProblemが矛盾 → `矛盾あり`
- Try着手中／Problem対応進行中 → `対応中`
- Problem/Tryが解決済み → `解消済み(RoundN)`
- それ以外 → `未対応`

**許容値**: `維持` / `未対応` / `対応中` / `解消済み(RoundN)` / `矛盾あり`

## 対象の付与（spec / rule）
- 機能・データ・挙動・成功条件 → `spec`。配色・余白・文言・レイアウトのみ → `rule`。どちらとも取れる場合は`spec`を既定とし、判断できないものだけ`ambiguity-detection`で質問する。
- 一度確定した`対象`は同一論点IDで再利用する。

## 新feature切り出しの判別
- 既存feature`request.md`の「やりたいこと」の範囲内 → **既存feature改善**。範囲を超える新しい目的・画面・データを要求 → **新feature候補**。迷う場合は既存feature改善を既定とする。
- 新feature候補は`kpt-summary.md`末尾「次のアクション」に論点ID・想定feature-name案・根拠を列挙するのみで、切り出しの実行はしない。一度確定した論点は蒸し返さない。

## 曖昧点の扱い（KPT固有）
`ambiguity-detection`で一問ずつ対話的に確認する際、KPTでは特に以下を曖昧点として扱う。
- **対象（spec/rule）判別不能**: デフォルト規則（迷ったらspec）で決まらない場合のみ。
- **既存feature改善／新feature候補の判別不能**: デフォルト規則（迷ったら既存feature改善）で決まらない場合のみ。
- **既存rule.mdとの矛盾**: 改善提案が確定済み`rule.md`と衝突する場合。回答が得られるまで当該提案は保留し、`kpt-summary.md`の該当論点のStatusを`矛盾あり`とする。

## clarification-log.md の仕様
質問・回答の正規記録テーブル。`docs/specs/<feature-name>/clarification-log.md`に保持する。

| カラム | 説明 |
|---|---|
| 質問ID | feature内で単調増加の `Q-<3桁連番>`（例: Q-001）。一度採番したIDは再利用しない |
| 論点ID | kpt-summary.md の論点ID（例: theme-1） |
| 対象トラック | `spec` / `rule` / `global` の3値（下記） |
| 質問 | 質問文 |
| 推奨回答 | `ambiguity-detection`が提示した推奨とその理由 |
| 回答 | 人間の回答を転記（未回答時は空欄） |
| Status | `未回答` / `解消済み` の2値 |
| 質問ラウンド | 質問を起票したラウンド番号 |
| 解消ラウンド | 回答が確定し解消済みになったラウンド番号（未解消は空欄） |

**対象トラック**（clarification-log.md のみの区分。KPT自体の`対象`は `spec` / `rule` の2値）:
- `spec`: 仕様・機能要件に関わる曖昧点
- `rule`: デザインルール・見た目/操作感に関わる曖昧点
- `global`: specとruleの両方に関わるか、対象判別・新feature判別そのものが曖昧な質問。回答後にspec/ruleいずれかに確定される。

**運用規則**:
- 質問IDはfeature全体で通番（ラウンドをまたいでも連番を継続）。
- 回答を適用する際は、回答・Status（→`解消済み`）・解消ラウンドを同一処理パスで更新する。
- 曖昧点が解消されるまで改善提案には進まない。

## kpt-summary.md のフォーマット

```markdown
## 機能カテゴリ凡例
- `theme`: テーマ設定

## KPT一覧
| 論点ID | 機能カテゴリ | 対象 | Keep | Problem | Try | Status | 詳細 |
|---|---|---|---|---|---|---|---|
| theme-1 | テーマ設定 | rule |  | ダーク時のコントラストが低い | 配色プリセットを3種用意する | 対応中 | [Round1](kpt-log/round1-detail.md#theme-1) |

## ラウンド履歴

| ラウンド | 論点ID | 対象 | Keep | Problem | Try | 入力状態 | 集約後Status |
|---|---|---|---|---|---|---|---|
| Round1 | theme-1 | rule |  | ダーク時のコントラストが低い | 配色プリセットを3種用意する | Problem=未対応 / Try=未着手 | 未対応 |
```

- **KPT一覧**は各論点の最新スナップショットのみを保持する（毎ラウンド上書き）。
- **ラウンド履歴**は各ラウンドで集約した論点行を追記する（既存行は修正・削除しない）。
- `Keep` / `Problem` / `Try` 列は要約のみとし、長い原文・両論併記は当該ラウンドの`roundN-detail.md`に残す。
- 詳細列は、その論点の詳細を最後に追記したラウンドの`roundN-detail.md`へのリンクのみを保持する（過去分は各ラウンドのファイルに残る）。
- `集約後Status`の許容値は KPT一覧の `Status` と同じ。

## roundN-detail.md のフォーマット
論点単位の長文原文・両論併記を残す補足ファイル。該当ラウンドに補足が発生した場合のみ`docs/kpt/kpt-log/roundN-detail.md`を作成する。

```markdown
## theme-1
(CSV原文の補足、矛盾の両論併記など)
```

- `##`見出しは論点ID。生ログに準じ追記専用（既存内容は削除・改変しない）。
- ファイルはラウンドごとに分割する（累積ファイルにはしない）。
