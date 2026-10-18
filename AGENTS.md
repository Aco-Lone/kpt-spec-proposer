# AGENTS.md

「概念設計 → 要件 → Spec・デザイン検討 → REQ単位のTDD（テストケース作成・プロトタイプ実装） → KPT振り返り → 要件・Spec・デザインへフィードバック」の反復サイクルを支援するプロンプト

## 全体プロセス

```
概念設計（機能横断・必要時に更新）:
  concept-designer → docs/knowledge/ の concept-class・user-story

featureごと（繰り返し）:
  requirements-writer → request.md・requirements.md（REQ、US/CC参照）
  spec-writer → spec.md（振る舞い・契約・異常系・AC） ‖ design-agent → design-token・design-rule（並行可）
  tdd-orchestrator → REQごとに繰り返す
    test-designer（サブ）→ test-cases.md・traceability.md（US〜TC）
    prototype-agent（サブ）→ Red→Green→Refactor、traceability.md（テストコード・実装状況）
  人間 → KPTの入力を取り込み (docs/kpt/kpt-log/roundN.csv コミット / 02_kpt-feedback へのCSV貼付け / 自由文記入)
  kpt-agent → 集約 → 曖昧点があれば質問して中断 / なければ改善提案（反映先エージェント別）
  人間 → 反映先のエージェント（concept-designer / requirements-writer / spec-writer / design-agent / tdd-orchestrator）を起動しループ継続
```

トレーサビリティ: `US → REQ → AC → TC → テストコード`。`docs/specs/<feature-name>/traceability.md`で追跡する。

## エージェントの責任分界（越境しない）

| エージェント | 生成・更新する成果物 | 起動 |
|---|---|---|
| `concept-designer` | `docs/knowledge/`のOKF概念（`concept-class`・`user-story`、承認不要） | 人間が明示的に起動した時のみ |
| `requirements-writer` | `request.md`、`requirements.md` | 人間が明示的に起動した時のみ（要望Issue作成時を含む） |
| `spec-writer` | `spec.md` | 人間が明示的にメンション・アサインした時のみ |
| `design-agent` | `docs/knowledge/`のOKF概念（`design-token`・`design-rule`、承認不要・適宜） | 人間が明示的に起動した時のみ |
| `tdd-orchestrator` | なし（サブエージェントの呼び出し・検証・REQ完了報告のみ） | 人間が明示的に起動した時のみ |
| `test-designer`（サブ） | `test-cases.md`、`traceability.md`のUS・REQ・AC・TC列 | `tdd-orchestrator`からの呼び出しのみ |
| `prototype-agent`（サブ） | モック本体・テストコード、`prototype-notes.md`、`traceability.md`のテストコード・実装状況列 | `tdd-orchestrator`からの呼び出しのみ |
| `kpt-agent` | `docs/kpt/kpt-log/roundN.csv`、`docs/kpt/kpt-log/roundN-detail.md`（補足がある場合のみ）、`docs/kpt/kpt-summary.md`、`clarification-log.md`、`docs/knowledge/`のOKF概念（`glossary-term`・`system-proposal`、承認不要・適宜） | KPTのCSVコミット時 / KPT Issue・回答コメント投稿時 |

## 厳守事項

- 工程の順序は「概念設計 → 要件 → Spec・デザイン → TDD」。上流の成果物が無いまま下流に進まない。KPTで上流に影響する論点が出た場合は、該当する上流の成果物を先に更新してから下流（テスト・実装）を更新する。
- 概念設計・要件・Spec・デザイン検討やTDDの起動は人間の判断。`kpt-agent`は各エージェントを自動起動せず、反映先の提案を行うのみ。`spec-writer`起動時のゲートは「`requirements.md`があり、`clarification-log.md`に spec / global の未回答質問がないこと」。
- TDDはREQ単位で進める。`tdd-orchestrator`は`requirements.md`の記載順（依存REQを先に）に全REQを自走し、REQごとに完了報告する。人間の確認は待たず次のREQへ進むが、検証に失敗したREQで停止し後続REQには進まない。REQ完了は、当該REQの全AC・TCが緑で、`traceability.md`に未紐付けがないこと。
- テストを先に書く。TCの無い機能・テストの無い実装を追加しない。TCの期待値は`prototype-agent`が変更せず、誤りは`test-designer`へ差し戻す。
- トレーサビリティを切らない。`REQ`は元`US-`を、`AC`は`REQ`を、`TC`は`AC`を、テストコードはテスト名の`[TC-<連番>]`で`TC`を参照する。IDは再利用しない。廃止した要件・AC・TCは削除せず状態で表す。
- `requirements.md`・`spec.md`は変更履歴を持ち、旧内容を削除せず差分を追記する。
- `spec.md`は Design by Contract で記述し、全ての事前条件に異常系を定め、契約・異常系の全項目をACで検証する。異常系のACも必ずテストケース化する。
- `docs/knowledge/`（OKF）の更新に人間の承認は不要。`concept-designer`（`concept-class`・`user-story`）・`design-agent`（`design-token`・`design-rule`）・`kpt-agent`（`glossary-term`・`system-proposal`）がそれぞれ適宜更新する（収束判定はしない）。常に最新の内容に上書きし、変更履歴・ログは持たない。
- `kpt-agent`は`ambiguity-detection`に従い一問ずつ対話的に曖昧点を確認し、1セッション内で解消しきる（複数セッションに分割しない）。
- KPTの生ログ（`docs/kpt/kpt-log/roundN.csv`）は削除・改変しない。
- 成果物を混ぜない: `requirements.md`=要件（何を満たすか）、`spec.md`=振る舞い・契約（事前条件・事後条件・不変条件）・異常系・受け入れ基準、`test-cases.md`=検証手順、`docs/knowledge/`=OKF（Open Knowledge Format）に倣ったナレッジバンドル。概念クラス（`concept-class`）、機能横断ユーザーストーリー（`user-story`）、feature独自のデザインルール（`design-rule`）、準拠する基本デザインシステム（`design-token`）、KPT由来の用語（`glossary-term`）、KPTでは対応できないシステム構成案（`system-proposal`、確定仕様ではなく参考メモ）をリポジトリ横断で記録する。`requirements.md`・`spec.md`にこれらを複製しない（IDで参照する）。
- 表の「生成・更新する成果物」以外を各エージェントが作成・編集しない。
- `test-designer`・`prototype-agent`を直接起動しない。必ず`tdd-orchestrator`経由で呼び出す。

## ディレクトリ規約

- feature固有の成果物: `docs/specs/<feature-name>/`（`request.md` / `requirements.md` / `spec.md` / `test-cases.md` / `traceability.md` / `prototype-notes.md` / `clarification-log.md`）
- KPT関連: `docs/kpt/kpt-log/roundN.csv` / `docs/kpt/kpt-log/roundN-detail.md` / `docs/kpt/kpt-summary.md`
- モック本体・テストコード: `mocks/<feature-name>/`（`unit`: `src/**/*.test.tsx`、`e2e`: `e2e/*.spec.ts`）
- リポジトリ横断: `docs/knowledge/`（OKFナレッジバンドル: `concept-class` / `user-story` / `design-token` / `design-rule` / `glossary-term` / `system-proposal`）
- ラウンド番号 N: `docs/kpt/kpt-log/`内の`roundN.csv`の最大番号。貼り付け方式では既存最大+1で新規作成。ファイルが無ければRound1。

## 詳細ルールの正本（重複記述しない）

各スキルは`SKILL.md`（手順）と`templates/`（記載内容の規定）で構成する。

| ルール | 正本 |
|---|---|
| KPTのCSV形式（自由文入力の整形含む）・生ログ保存・論点ID・対象(spec/knowledge)の付与・Statusの集約・新feature切り出しの判別・KPT固有の曖昧点パターン・`clarification-log.md`・`kpt-summary.md` | `.github/skills/kpt-parsing/` |
| 対話的な質問生成ルール（一問ずつ・推奨回答提示） | `.github/skills/ambiguity-detection/` |
| 概念クラス・横断ユーザーストーリーの導出手順 | `.github/skills/concept-modeling/` |
| `request.md`・`requirements.md`の手順と書式 | `.github/skills/requirements-template/` |
| `spec.md`の手順と書式 | `.github/skills/spec-template/` |
| `test-cases.md`の手順と書式 | `.github/skills/test-case-template/` |
| `traceability.md`の手順と書式（更新担当・検証） | `.github/skills/traceability-matrix/` |
| REQ単位のTDD（オーケストレーション・Red/Green/Refactor・REQ完了報告） | `.github/skills/tdd-workflow/` |
| `docs/knowledge/`（OKF）6種のconceptのフォーマット・更新原則 | `.github/skills/okf-knowledge/` |
| 変更履歴の追記規則（`requirements.md`・`spec.md`） | `.github/skills/change-history-logging/` |
| モックの簡略化方針と`prototype-notes.md` / Webモックの技術スタックとテスト基盤 | `.github/skills/mock-scaffolding/` / `.github/skills/web-mock-stack/` |
