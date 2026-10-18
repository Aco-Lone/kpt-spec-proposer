---
name: mock-scaffolding
description: モック実装時の簡略化ルールと prototype-notes.md の記録手順。prototype-agent がモックのコードを生成・更新するときに使う。
---

# skill: mock-scaffolding

画面操作ができるモックを実装する際の共通ルール。TDDの進め方は`tdd-workflow`に従う。

## 原則
- 実データ永続化（DB接続等）は行わない。状態はメモリ内やダミーJSONで保持する。
- 認証・外部API連携は行わない。必要な場合はダミーレスポンスで代替する。
- UIの操作感（画面遷移、ボタン、フォーム入力の反映）を最優先で再現する。
- 簡略化してもTCは満たす。TCを満たせない簡略化はしない。

## prototype-notes.md の記録手順
1. 実装したREQごとに、[prototype-notes.md](templates/prototype-notes.md)に従い追記する。
2. 既存の記載は削除・改変しない。
