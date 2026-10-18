---
name: requirements-template
description: request.md を記録し、横断US・概念クラスから requirements.md（REQ一覧）を導出・更新する手順。requirements-writer が使う。
---

# skill: requirements-template

feature単位の要件（REQ）を導出・更新する手順。記載内容は`templates/`が規定する。

## テンプレート
- [request.md](templates/request.md)
- [requirements.md](templates/requirements.md)
- 変更履歴: [change-history.md](../change-history-logging/templates/change-history.md)

## 手順（新規）
1. 要望原文を[request.md](templates/request.md)に従い`docs/specs/<feature-name>/request.md`へ記録する。
2. `docs/knowledge/`の`user-story`・`concept-class`を読み、要望に関与する`US-`・`CC-`を特定する。該当がなければ`concept-designer`での概念設計を促して中断する。
3. 曖昧点は`ambiguity-detection`に従い一問ずつ確認する。
4. [requirements.md](templates/requirements.md)に従い、REQを導出する。各REQに元`US-`・関連`CC-`を必ず付ける。
5. REQの記載順を処理順とする。依存するREQは先に書く。依存関係は「依存REQ」列に明記する。
6. 各REQは検証可能な一文で書く（曖昧語を避ける）。実装手段・見た目・操作感は書かない。

## 手順（更新）
1. 更新の起点（KPT論点ID・概念設計の変更・人間の指示）を特定する。
2. 既存REQを削除せず、変更・廃止はREQの状態列で表し、`change-history-logging`に従い変更履歴へ追記する。
3. REQのIDは再利用しない。新規REQは末尾の連番で採番する。
4. 変更したREQ-IDの一覧を報告する（下流の`spec-writer`・`tdd-orchestrator`への入力となる）。
