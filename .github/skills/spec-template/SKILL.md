---
name: spec-template
description: requirements.md の各REQから振る舞い仕様と受け入れ基準（AC）を導出し spec.md を作成・更新する手順。spec-writer が使う。
---

# skill: spec-template

`spec.md`（振る舞い仕様＝WHAT）を作成・更新する手順。記載内容は[spec.md](templates/spec.md)が規定する。特定のSDDツール（Jira記法、OpenAPI専用構文等）に依存しない。

## 対象範囲
`spec.md`は各REQの振る舞い・契約（Design by Contract）・異常系・受け入れ基準を記録する。実装方法（HOW）や見た目・操作感（`docs/knowledge/`の`design-rule`）、エラー文言は書かない。要件そのものは`requirements.md`が正本で、複製しない。

## 参照する資料と観点

| 資料 | 参照する観点 |
|---|---|
| `requirements.md` | 対象REQ・元US・関連CC・依存REQ（仕様化の対象と範囲） |
| `docs/knowledge/`の`concept-class`・`user-story` | 振る舞いで使う概念・用語の一貫性 |
| `docs/knowledge/`の`glossary-term` | 用語の統一 |
| `docs/knowledge/`の`system-proposal` | 非機能要件・制約の参考メモ（確定仕様ではない点に留意） |
| `docs/kpt/kpt-summary.md`・`roundN-detail.md`（存在すれば） | 論点ごとの合意済みStatus、背景の原文（仕様更新の起点） |
| `clarification-log.md`（存在すれば） | 曖昧点への回答（仕様を確定させる根拠） |

`design-token`・`design-rule`は見た目・操作感の制約として参照のみ行い、`spec.md`に書かない。

## 手順
1. `requirements.md`を読み、状態が`有効`のREQを仕様化の対象にする。
2. 上記の資料を参照する。曖昧点は`ambiguity-detection`に従い一問ずつ確認する。
3. [spec.md](templates/spec.md)に従い、REQごとに振る舞いを書く。
4. 振る舞いから契約（事前条件・事後条件・不変条件）を導出する。呼び出し側の責任（事前条件）と提供側の保証（事後条件・不変条件）を混同しない。
5. 全ての事前条件について、違反時の異常系を導出する。さらに、提供側で失敗しうる場合と、事後条件・不変条件が破られうる場合も異常系に加える。失敗後も不変条件が保たれることを確認する。
6. 網羅観点（入力値・状態・順序・同時・競合・外部依存・権限）を一つずつ確認し、対応するIDを記入する。該当しない観点は理由を添えて「該当なし」とする。理由が不明なものは`ambiguity-detection`で確認する。
7. 受け入れ基準は`AC-<REQ番号>-<連番>`で採番し、REQ内で単調増加させる。一度採番したIDは再利用しない。契約IDと異常IDは、いずれかのACの「検証対象」に記載する。
8. カバレッジを確認する。次のいずれかがあれば補ってから進む。
   - ACが無い有効REQ。
   - どのACからも参照されない契約ID・異常ID。
   - 異常系の無い事前条件。
   - 空欄の網羅観点。
9. 「適切に」「わかりやすく」等の曖昧語を使わず、真偽を判定できる文で書く。
10. 更新時は`change-history-logging`に従い変更履歴を追記する。廃止した契約・異常系・ACは削除せず、状態を`廃止`にする。
11. 作成・変更したREQ-ID・契約ID・異常ID・AC-IDを報告する。
