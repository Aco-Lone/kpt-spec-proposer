# AGENTS.md

「要望 → 画面操作モック → KPT振り返り → （人間の判断で）仕様化（spec.md）」の反復サイクルを、カスタムエージェントで運用するリポジトリ。

## 全体プロセス

```
ラウンドN（繰り返し）:
  prototype-agent → モック更新（+ 必要なら rule.md を承認なしで更新）
  人間 → KPTのCSVを取り込み (kpt-log/roundN.csv コミット or 02_kpt-feedback に貼付け)
  kpt-agent → 集約 → 曖昧点があれば質問して中断 / なければ改善提案
  人間 → 改善提案をもとに次ラウンドのモック更新を prototype-agent に依頼

spec化（ラウンドとは独立。人間が「必要な機能を搭載できた」と判断したときのみ）:
  人間が spec-writer を手動起動 → spec.md を生成・更新 → 以降もループは継続
```

spec.md は「ゴール」ではなく「その時点のスナップショット」。

## エージェントの責任分界（越境しない）

| エージェント | 生成・更新する成果物 | 起動 |
|---|---|---|
| `prototype-agent` | モック本体、`request.md`、`prototype-notes.md`、`docs/design.md`（初回）、`rule.md`（承認不要・適宜） | 要望Issue作成時 / 人間がモック更新を依頼した時 |
| `kpt-agent` | `kpt-log/roundN.csv`、`kpt-summary.md`、`clarification-log.md` | KPTのCSVコミット時 / KPT Issue・回答コメント投稿時 |
| `spec-writer` | `spec.md` | 人間が明示的にメンション・アサインした時のみ |

## 厳守事項

- spec化のタイミングは人間の判断。`kpt-agent`は仕様化の可否を提案・判定せず、`spec-writer`を自動起動しない。`spec-writer`起動時の唯一のゲートは「`clarification-log.md`に spec / global の未回答質問がないこと」。
- `rule.md`の更新に人間の承認は不要。`prototype-agent`がデザイン判断の固まったラウンドで適宜更新する（収束判定はしない）。
- `kpt-agent`は曖昧点の質問を1セッション内でまとめて出し切る（複数ラウンドに分割しない）。
- KPTの生ログ（`kpt-log/roundN.csv`）は削除・改変しない。
- 成果物を混ぜない: `spec.md`=仕様（WHAT）、`rule.md`=feature独自のデザインルール（見た目・操作感）、`docs/design.md`=準拠する基本デザインシステム（Material Design 等の外部標準・基本トークン、リポジトリ横断・承認不要）。`spec.md`にデザインルールを書かない。
- 表の「生成・更新する成果物」以外を各エージェントが作成・編集しない。

## feature-nameの検証

- 許可形式は `^[a-z0-9]+(?:-[a-z0-9]+)*$`。`prototype-agent`と`kpt-agent`は、パスに使う前に検証する。
- 不正な場合は期待形式と受け取った値を通知し、スラッグ変換等の補正・推測をせず処理を中断する。

## ディレクトリ規約

- feature固有の成果物: `docs/specs/<feature-name>/`（`request.md` / `prototype-notes.md` / `kpt-log/roundN.csv` / `kpt-summary.md` / `clarification-log.md` / `spec.md` / `rule.md`）
- モック本体: `mocks/<feature-name>/`
- リポジトリ横断: `docs/design.md`
- ラウンド番号 N: `kpt-log/`内の`roundN.csv`の最大番号。貼り付け方式では既存最大+1で新規作成。ファイルが無ければRound1。

## 詳細ルールの正本（重複記述しない）

| ルール | 正本 |
|---|---|
| KPTのCSV形式・入力検証・生ログ保存・論点ID・対象(spec/rule)の付与・Statusの集約・新feature切り出しの判別・`kpt-summary.md`の書式 | `.github/skills/kpt-parsing/SKILL.md` |
| 曖昧点の判定パターン・質問生成ルール・`clarification-log.md`の仕様 | `.github/skills/ambiguity-detection/SKILL.md` |
| `spec.md` / `rule.md` の書式 | `.github/skills/spec-template/SKILL.md` / `design-rule-template/SKILL.md` |
| 変更履歴の追記規則 | `.github/skills/change-history-logging/SKILL.md` |
| モックの簡略化方針 / Webモックの技術スタック | `.github/skills/mock-scaffolding/SKILL.md` / `web-mock-stack/SKILL.md` |
| 設計判断の理由（なぜこの構成か） | `README.md`（エージェントは参照不要） |
