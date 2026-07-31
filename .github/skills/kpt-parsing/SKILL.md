---
name: kpt-parsing
description: 人間が記入したKPT（Keep/Problem/Try）の生記述を構造化・集約するロジック。kpt-agent がKPTのIssue/コメントを集約し kpt-summary.md を更新するときに参照する。生ログの保存、重複統合、矛盾の両論併記、過去ラウンドとのマージ規則を定義する。
---

# skill: kpt-parsing

KPT（Keep/Problem/Try）の生記述を構造化データに変換するロジック。

## 処理
0. 集約に先立ち、記入されたKPTの生テキストを`docs/specs/<feature-name>/kpt-log/roundN.md`として保存する。ラウンド番号 N の採番は AGENTS.md「ラウンド番号とKPT生ログ」に従う。生ログは要約後も削除・改変しない。
1. 記入されたテキストをKeep/Problem/Tryの3カテゴリに分類する（フォームで既に分かれている場合はそのまま使用）。
2. カテゴリ内の項目について、意味的に重複するものを統合する（元の文言は失わず、統合元を注記する）。
3. 矛盾する記述（例: 同じ機能についてKeepとProblemが両方ある）がある場合は、削除・統合せず両論併記し「矛盾あり」と明示する。
4. 過去ラウンドの`kpt-summary.md`がある場合は、今回分をマージし、解消済み項目には取り消し線または「解消済み（Round N）」を付与する。
