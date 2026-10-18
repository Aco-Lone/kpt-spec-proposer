## 機能カテゴリ凡例
<!-- 必須: 機能カテゴリと略号の対応。略号は英小文字スラッグ -->
- `<略号>`: <機能カテゴリ名>

## KPT一覧
<!--
必須。各論点の最新スナップショットのみ（毎ラウンド上書き）。
- 論点ID: `<略号>-<連番>`
- 対象: `spec` / `knowledge`
- Keep / Problem / Try: 要約のみ。長い原文・両論併記は roundN-detail.md に残す
- Status: `維持` / `未対応` / `対応中` / `解消済み(RoundN)` / `矛盾あり`
- 詳細: その論点の詳細を最後に追記したラウンドの roundN-detail.md へのリンクのみ
-->
| 論点ID | 機能カテゴリ | 対象 | Keep | Problem | Try | Status | 詳細 |
|---|---|---|---|---|---|---|---|
| theme-1 | テーマ設定 | knowledge |  | ダーク時のコントラストが低い | 配色プリセットを3種用意する | 対応中 | [Round1](kpt-log/round1-detail.md#theme-1) |

## ラウンド履歴
<!--
必須。各ラウンドで集約した論点行を追記する。既存行は修正・削除しない。
- 入力状態: 入力された Keep/Problem/Try の状態（例: Problem=未対応 / Try=未着手）
- 集約後Status: KPT一覧の Status と同じ許容値
-->
| ラウンド | 論点ID | 対象 | Keep | Problem | Try | 入力状態 | 集約後Status |
|---|---|---|---|---|---|---|---|
| Round1 | theme-1 | knowledge |  | ダーク時のコントラストが低い | 配色プリセットを3種用意する | Problem=未対応 / Try=未着手 | 未対応 |

## 次のアクション
<!--
必須: 改善提案の要点と、反映先エージェント別の整理を書く。
- 反映先: `concept-designer` / `requirements-writer` / `spec-writer` / `design-agent` / `tdd-orchestrator`
- 新feature候補があれば、論点ID・想定feature-name案・根拠を列挙する（切り出しの実行はしない）
-->
