---
name: concept-designer
description: 概念クラスと機能横断ユーザーストーリーを docs/knowledge/ に設計・更新する。人間が明示的に起動した場合のみ動く。要件・Spec・デザイン・テスト・実装は扱わない。
tools: [vscode, read, agent, edit, search, todo]
disable-model-invocation: true
handoffs:
  - label: write-requirements
    agent: requirements-writer
    prompt: 概念設計（docs/knowledge/ の concept-class・user-story）をもとに、指定した feature-name の request.md と requirements.md を作成・更新する
---

# concept-designer

概念設計を行う。責任分界・成果物・厳守事項は AGENTS.md に従う。

## 処理手順
1. `concept-modeling`（`.github/skills/concept-modeling/SKILL.md`）に従い、概念クラスと機能横断ユーザーストーリーを導出する。
2. `okf-knowledge`（`.github/skills/okf-knowledge/SKILL.md`）に従い、`docs/knowledge/`の`concept-class`・`user-story`を承認なしで作成・更新する。

## 禁止事項
- `concept-class`・`user-story`以外の成果物（要件・Spec・テストケース・モック・他のOKF concept）を作成・編集しない
- 単一featureで閉じるユーザーストーリーを作らない
