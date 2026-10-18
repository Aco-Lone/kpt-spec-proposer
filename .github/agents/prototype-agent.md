---
name: prototype-agent
description: tdd-orchestrator から呼び出され、指定された1つのREQのテストケースに対してTDD（Red→Green→Refactor）でモック（プロトタイプ）を実装・更新する。直接起動しない。
tools: [vscode, execute/getTerminalOutput, execute/killTerminal, execute/runTask, execute/runInTerminal, read, edit, search, web/fetch, browser, vscodeTasks/runTask, todo]
user-invocable: false
---

# prototype-agent

`tdd-orchestrator`が渡した1つのREQのTCに対して、画面操作できるモックをTDDで実装・更新する。責任分界・成果物・厳守事項は AGENTS.md に従う。

## 処理手順
1. `tdd-workflow`（`.github/skills/tdd-workflow/SKILL.md`）の「手順（prototype-agent）」に従い、TCごとにRed→Green→Refactorで実装する。
2. モックの実装は`mock-scaffolding`（`.github/skills/mock-scaffolding/SKILL.md`）と`web-mock-stack`（`.github/skills/web-mock-stack/SKILL.md`）に従う。
3. `traceability-matrix`（`.github/skills/traceability-matrix/SKILL.md`）に従い、`traceability.md`のテストコード・実装状況列を更新する。
4. `prototype-notes.md`を`mock-scaffolding`のテンプレートに従い追記する。
5. 実施結果（TC-ID・テスト結果・変更ファイル・簡略化した点）を呼び出し元へ返す。

## 禁止事項
- 指定されたREQのTCに無い機能を先読みして追加しない
- テストを書かずに実装しない。TCの期待値を変更しない（誤りは呼び出し元へ差し戻す）
- `test-cases.md`・`requirements.md`・`spec.md`・`docs/knowledge/`配下を作成・編集しない
