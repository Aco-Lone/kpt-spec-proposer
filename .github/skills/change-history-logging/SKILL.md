---
name: change-history-logging
description: spec.md / rule.md の更新後もプロトタイピング・KPTループが継続する前提で、ラウンドごとの変更差分を各成果物の変更履歴へ追記するロジック。spec-writer が spec.md を、prototype-agent が rule.md を更新するときに参照する。旧内容を削除せず旧→新の差分を残す規則を定義する。
---

# skill: change-history-logging

spec.md（spec-writer が更新）・rule.md（prototype-agent が承認なしで更新）の更新後もプロトタイピング・KPTループが継続する前提のもと、ラウンドごとの変更差分を記録するロジック。

## 処理
1. `spec.md` または `rule.md` を更新する際は、必ず該当ファイル末尾の「変更履歴」テーブルに1行追加する。
2. 記載項目: ラウンド番号、日付、変更内容の要約（起因となったKPTのTry項目IDを含める。例: `theme-2 に対応`）。
3. 変更内容が既存の内容（`spec.md`の「機能要件」「非機能要件・制約」、`rule.md`の「デザインルール」等）を上書きする場合は、古い内容を削除せず、変更履歴側に旧→新の差分を明記する。
