---
name: change-history-logging
description: spec.md / rule.md の更新差分を変更履歴に追記するロジック。spec-writer / prototype-agent が各成果物を更新するときに使う。
---

# skill: change-history-logging

`spec.md` / `rule.md` の更新差分を記録するロジック。

## 処理
1. `spec.md` または `rule.md` を更新する際は、必ず該当ファイル末尾の「変更履歴」テーブルに1行追加する。
2. 記載項目: ラウンド番号、日付、変更内容の要約（起因となった論点IDを含める。例: `theme-2 に対応`）。
3. 既存の内容を上書きする場合は、古い内容を削除せず、変更履歴側に旧→新の差分を明記する。
