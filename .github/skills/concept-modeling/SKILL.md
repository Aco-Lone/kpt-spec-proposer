---
name: concept-modeling
description: 概念クラスと機能横断ユーザーストーリーを導出し docs/knowledge/ に記録する手順。concept-designer が概念設計を行うときに使う。
---

# skill: concept-modeling

概念設計（概念クラス・機能横断ユーザーストーリー）を導出する手順。記載内容は`okf-knowledge`の`concept-class`・`user-story`テンプレートが規定する。

## 手順
1. 人間の入力（目的・業務の説明・既存の`docs/knowledge/`）を読む。
2. 業務上の名詞から概念クラス候補を挙げる。実装手段（DB・画面・API）に依存する要素は候補にしない。
3. 属性・関連・多重度を整理する。曖昧な点は`ambiguity-detection`に従い一問ずつ確認する。
4. 複数featureにまたがるユーザーストーリーを導出し、関与する概念クラスを紐付ける。単一featureで閉じるストーリーは`requirements-writer`の責務とし、ここでは作らない。
5. `okf-knowledge`の手順に従い、テンプレート[concept-class.md](../okf-knowledge/templates/concept-class.md)・[user-story.md](../okf-knowledge/templates/user-story.md)でconceptを作成・更新し、`index.md`を更新する。
6. 既存conceptと重複・矛盾しないか確認する。矛盾する場合は人間に確認する。
7. 変更したconceptの`CC-`・`US-`を一覧で報告する。
