# AGENTS.md

「要望 → 画面操作モック → KPT振り返り → （人間の判断で）仕様化（spec.md）」の反復サイクルを支援するプロンプト

## 全体プロセス

```
ラウンドN（繰り返し）:
  prototype-agent → モック更新（+ 必要なら rule.md を承認なしで更新）
  人間 → KPTの入力を取り込み (docs/kpt/kpt-log/roundN.csv コミット / 02_kpt-feedback へのCSV貼付け / 自由文記入)
  kpt-agent → 集約 → 曖昧点があれば質問して中断 / なければ改善提案
  人間 → 次のいずれか:
    a) 改善提案をもとに次ラウンドのモック更新を prototype-agent に依頼（ループ継続）
    b) 「必要な機能を搭載できた」と判断した場合は、当該ラウンドの最終ステップとして spec-writer を起動
       → spec.md を生成・更新 → 本開発（実装）チームへハンドオフし、当該featureのループは終了
```

## エージェントの責任分界（越境しない）

| エージェント | 生成・更新する成果物 | 起動 |
|---|---|---|
| `prototype-agent` | モック本体、`request.md`、`prototype-notes.md`、`docs/design.md`（初回）、`rule.md`（承認不要・適宜） | 要望Issue作成時 / 人間がモック更新を依頼した時 |
| `kpt-agent` | `docs/kpt/kpt-log/roundN.csv`、`docs/kpt/kpt-log/roundN-detail.md`（補足がある場合のみ）、`docs/kpt/kpt-summary.md`、`clarification-log.md` | KPTのCSVコミット時 / KPT Issue・回答コメント投稿時 |
| `spec-writer` | `spec.md` | 人間が明示的にメンション・アサインした時のみ。ラウンドの最終ステップとしてのみ起動され、完了後は本開発チームへハンドオフし当該featureのループを終了 |

## 厳守事項

- spec化のタイミングは人間の判断。`kpt-agent`は仕様化の可否を提案・判定せず、`spec-writer`を自動起動しない。`spec-writer`起動時の唯一のゲートは「`clarification-log.md`に spec / global の未回答質問がないこと」。
- spec化を行う場合は当該ラウンドの最終ステップとして行う。`spec.md`生成後は本開発（実装）チームへハンドオフし、当該feature（スコープ）のプロトタイピング/KPTループはそこで終了する。
- `rule.md`の更新に人間の承認は不要。`prototype-agent`がデザイン判断の固まったラウンドで適宜更新する（収束判定はしない）。
- `kpt-agent`は`ambiguity-detection`に従い一問ずつ対話的に曖昧点を確認し、1セッション内で解消しきる（複数セッションに分割しない）。
- KPTの生ログ（`docs/kpt/kpt-log/roundN.csv`）は削除・改変しない。
- 成果物を混ぜない: `spec.md`=仕様（WHAT）、`rule.md`=feature独自のデザインルール（見た目・操作感）、`docs/design.md`=準拠する基本デザインシステム（Material Design 等の外部標準・基本トークン、リポジトリ横断・承認不要）。`spec.md`にデザインルールを書かない。
- 表の「生成・更新する成果物」以外を各エージェントが作成・編集しない。

## ディレクトリ規約

- feature固有の成果物: `docs/specs/<feature-name>/`（`request.md` / `prototype-notes.md` / `clarification-log.md` / `spec.md` / `rule.md`）
- KPT関連: `docs/kpt/kpt-log/roundN.csv` / `docs/kpt/kpt-log/roundN-detail.md` / `docs/kpt/kpt-summary.md`
- モック本体: `mocks/<feature-name>/`
- リポジトリ横断: `docs/design.md`
- ラウンド番号 N: `docs/kpt/kpt-log/`内の`roundN.csv`の最大番号。貼り付け方式では既存最大+1で新規作成。ファイルが無ければRound1。

## 詳細ルールの正本（重複記述しない）

| ルール | 正本 |
|---|---|
| KPTのCSV形式（自由文入力の整形含む）・生ログ保存・論点ID・対象(spec/rule)の付与・Statusの集約・新feature切り出しの判別・KPT固有の曖昧点パターン・`clarification-log.md`の仕様・`kpt-summary.md`の書式 | `.github/skills/kpt-parsing/SKILL.md` |
| 対話的な質問生成ルール（一問ずつ・推奨回答提示） | `.github/skills/ambiguity-detection/SKILL.md` |
| `spec.md` / `rule.md` の書式 | `.github/skills/spec-template/SKILL.md` / `design-rule-template/SKILL.md` |
| 変更履歴の追記規則 | `.github/skills/change-history-logging/SKILL.md` |
| モックの簡略化方針 / Webモックの技術スタック | `.github/skills/mock-scaffolding/SKILL.md` / `web-mock-stack/SKILL.md` |
