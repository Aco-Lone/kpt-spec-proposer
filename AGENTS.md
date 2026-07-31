# AGENTS.md

このリポジトリは「シンプルな要望 → 画面操作モック（プロトタイプ） → KPT振り返り → 仕様化（spec.md）」という反復サイクルを、GitHub Copilot Coding Agentのカスタムエージェントで運用するための構成です。

## 全体プロセス

```
ラウンドN:
  prototype-agent → モック更新
  人間 → KPTをスプレッドシートで管理しCSVを取り込み (kpt-log/roundN.csv コミット or 02_kpt-feedback に貼付け)
  kpt-agent → 集約・(曖昧点があれば質問して中断 | 改善提案+ループ判定)

  判定が「継続プロトタイピング」の場合:
    → 改善提案をラウンドN+1のprototype-agentへの入力とする
    → spec.mdは生成しない（kpt-summary.mdの蓄積のみ）

  判定が「仕様化」の場合:
    → 人間が承認した上でspec-writerを手動起動
    → spec.md生成後もプロトタイプは残り、ループは継続する
    → 以降の変更差分もspec.mdに追記していく
```

spec.md確定は「ゴール」ではなく「その時点のスナップショット」。確定後もprototype-agent/kpt-agentのループは続きます。

## エージェントの責任分界（重要: 越境しない）

| エージェント | 役割 | 起動タイミング |
|---|---|---|
| `prototype-agent` | 要望または改善提案からモックを生成・更新する | 要望Issue作成時、またはkpt-agentが「継続プロトタイピング」と判定した直後 |
| `kpt-agent` | KPTを集約し、曖昧点があれば質問、なければ改善提案とループ判定を行う | KPT Issue/コメントが記入された時 |
| `spec-writer` | 仕様化が承認された場合のみ、spec.mdを生成・更新する | 人間が明示的にメンション・アサインした時のみ（自動連鎖させない） |

**厳守事項:**
- `kpt-agent`は仕様化の可否を「提案」するだけで、spec.mdの生成は絶対に自動で行わない。必ず人間の承認アクションを挟む。
- `kpt-agent`は曖昧点の質問を1セッション内でまとめて出し切る。複数ラウンドに分割しない（非同期のIssue往復コストを最小化するため）。
- KPTの生ログ（`kpt-log/roundN.csv`）は要約後も削除・改変しない。

## KPTの表形式・ID規約

`kpt-agent`はKPTを`kpt-summary.md`上で表形式に集約する。表カラムは `ID | 種別 | 機能カテゴリ | 内容 | 状態` とする。

- **ID**: `<機能カテゴリ略号>-<連番>`（例 `nav-1`）。機能カテゴリ略号は英小文字スラッグで、`request.md`・`prototype-notes.md`の機能領域から導出し、`kpt-summary.md`先頭の凡例で正式名に対応づける。連番は機能カテゴリごと・feature-name内で一意。
- **同一性**: 一度振ったIDは同一論点が続く限り再利用し、新規論点のみ新IDを採番する。解消された項目は削除せず状態を「解消済み(RoundN)」に更新する。
- **人間入力**: 人間はKPTをスプレッドシートで管理し、`種別,機能カテゴリ,内容,状態`のヘッダを持つCSVとして提供する。IDは集約時に付与するもので、入力CSVや生ログ（`kpt-log/roundN.csv`）には振らない。

## 仕様化の判断材料（readiness-check スキル）

以下を満たす場合に「仕様化」を提案する（最終判断は常に人間）:
- Problem/Tryの内容が直近2ラウンド以上「同じ論点の再確認」になっていない（収束している）
- 未解決の曖昧点がない
- Keepの割合がProblemを上回っている

## KPTの取り込みとラウンド番号

- **feature-name**: 成果物の集約先ディレクトリ名。要望Issue（`01_feature-request.yml`）の「機能名（feature-name）」で指定し、`prototype-agent`が最初のセッションで`docs/specs/<feature-name>/request.md`に記録する。KPT Issue（`02_kpt-feedback.yml`）は同じfeature-nameを指定して対応づける。
- **KPTの取り込み**: 人間はKPTをスプレッドシートで管理し、CSV（`種別,機能カテゴリ,内容,状態`）を成果物に取り込む。優先方法は `docs/specs/<feature-name>/kpt-log/roundN.csv` へのコミット。将来的にIssueフォームへのCSV貼り付けに切り替えられるよう、取得元に依らずCSV本体を入力として扱う。
- **ラウンド番号 N**: `docs/specs/<feature-name>/kpt-log/`内の`roundN.csv`の番号で決まる。人間が`roundN.csv`をコミットした場合はその最新ファイルの番号が当該ラウンド。Issueフォームへの貼り付け方式では`kpt-agent`が既存最大+1の`roundN.csv`を新規作成する。ファイルが無ければRound1。
- **生ログの保存責務**: `kpt-agent`は集約に先立ち、取り込んだKPTのCSVを`kpt-log/roundN.csv`として逐字保存（既に人間が同パスにコミット済みならそのファイルを生ログとする）してから集約する。生ログは要約後も削除・改変しない。

## ディレクトリ規約

成果物は必ず `docs/specs/<feature-name>/` 配下に集約する。ファイル構成は各エージェント定義ファイルを参照。

## なぜこの構成か

詳細は README.md の「設計判断の記録」を参照。
