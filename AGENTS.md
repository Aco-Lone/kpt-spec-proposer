# AGENTS.md

このリポジトリは「シンプルな要望 → 画面操作モック（プロトタイプ） → KPT振り返り → 仕様化（spec.md）」という反復サイクルを、GitHub Copilot Coding Agentのカスタムエージェントで運用するための構成です。

## 全体プロセス

```
ラウンドN:
  prototype-agent → モック更新
  人間 → KPT記入 (Issueテンプレート 02_kpt-feedback)
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
- KPTの生ログ（`kpt-log/roundN.md`）は要約後も削除・改変しない。

## 規模フラグ（light / full）

要望Issue（`01_feature-request.yml`）作成時に規模を選択する。

- **light**: 小さな要望（UI微修正など）。`kpt-summary.md`までで完結し、`clarification-log.md`・`spec.md`は生成しない。
- **full**: 本フローすべてを実施する。

`prototype-agent`が最初のセッションで`docs/specs/<feature-name>/request.md`にこのフラグを記録し、以降の全エージェントはこれを参照して生成物の量を調整する。

## 仕様化の判断材料（readiness-check スキル）

以下を満たす場合に「仕様化」を提案する（最終判断は常に人間）:
- Problem/Tryの内容が直近2ラウンド以上「同じ論点の再確認」になっていない（収束している）
- 未解決の曖昧点がない
- Keepの割合がProblemを上回っている

## ディレクトリ規約

成果物は必ず `docs/specs/<feature-name>/` 配下に集約する。ファイル構成は各エージェント定義ファイルを参照。

## なぜこの構成か

詳細は README.md の「設計判断の記録」を参照。
