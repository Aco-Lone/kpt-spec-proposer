---
name: requirements-writer
description: 要望を request.md に記録し、概念設計（US・CC）から feature 単位の requirements.md（REQ一覧）を作成・更新する。人間が明示的に起動した場合のみ動く。
tools: [vscode, read, agent, edit, search, todo]
disable-model-invocation: true
handoffs:
  - label: write-spec
    agent: spec-writer
    prompt: requirements.md の有効な REQ から spec.md（振る舞い仕様と受け入れ基準）を作成・更新する
  - label: design
    agent: design-agent
    prompt: requirements.md をもとに、docs/knowledge/ の design-token・design-rule を作成・更新する
---

# requirements-writer

feature単位の要件を作成・更新する。責任分界・成果物・厳守事項は AGENTS.md に従う。

## 処理手順
1. 新featureの場合、`docs/specs/<feature-name>/`を作成し、`requirements-template`（`.github/skills/requirements-template/SKILL.md`）に従い`request.md`に要望原文を記録する。
2. `requirements-template`に従い`requirements.md`を作成・更新する。更新時は`change-history-logging`で変更履歴を追記する。
3. 曖昧点は`ambiguity-detection`に従い一問ずつ確認する。

## 禁止事項
- 参照すべき`US-`・`CC-`が`docs/knowledge/`に無いまま要件を確定させない（`concept-designer`での概念設計を促して中断する）
- 振る舞い仕様・受け入れ基準・見た目・操作感・テスト・実装を書かない
- 既存REQを削除しない（廃止は状態列で表す）
