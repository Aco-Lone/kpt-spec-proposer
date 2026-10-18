---
name: okf-knowledge
description: Open Knowledge Format(OKF)に倣った docs/knowledge/ ナレッジバンドルの更新手順。concept-class・user-story・design-token・design-rule・glossary-term・system-proposal の6種を記録する。concept-designer・design-agent・kpt-agent が docs/knowledge/ を更新するときに使う。
---

# skill: okf-knowledge

`docs/knowledge/`配下のconceptを、Markdown + YAML frontmatterで作成・更新する手順。記載内容は`templates/`が規定する。1concept=1ファイル、常に最新のみ保持し、変更履歴は持たない。

## concept種別・担当・配置・テンプレート

| type | 担当エージェント | 配置 | テンプレート |
|---|---|---|---|
| `concept-class` | concept-designer | `concepts/CC-<名前>.md` | [concept-class.md](templates/concept-class.md) |
| `user-story` | concept-designer | `stories/US-<NNN>.md` | [user-story.md](templates/user-story.md) |
| `design-token` | design-agent | `design/tokens/<token-name>.md` | [design-token.md](templates/design-token.md) |
| `design-rule` | design-agent | `design/rules/<feature-name>/<rule-name>.md` | [design-rule.md](templates/design-rule.md) |
| `glossary-term` | kpt-agent | `glossary/<term>.md` | [glossary-term.md](templates/glossary-term.md) |
| `system-proposal` | kpt-agent | `proposals/<proposal-name>.md` | [system-proposal.md](templates/system-proposal.md) |

バンドル全体の入口は`docs/knowledge/index.md`（[index.md](templates/index.md)）。

## 手順
1. 更新対象のtypeを決め、担当エージェントが自身の担当typeのみ更新する。
2. 既存の同名ファイルがあれば読み、なければ該当テンプレートから作成する。
3. テンプレートの必須項目を埋める。内容が変わる場合は本文を上書きし、旧内容の併記・追記型の履歴は残さない。
4. 1ファイルに複数conceptを混在させない。
5. `docs/knowledge/index.md`に追加・更新したconceptへのリンクを反映する。
6. `concept-class`・`user-story`のIDは次の規則で採番する。一度採番したIDは再利用しない。
   - `US-<NNN>`: リポジトリ全体で単調増加の3桁連番。
   - `CC-<名前>`: PascalCaseの概念名。
7. 概念クラスと横断ユーザーストーリーの関与関係は双方向で整合させる（`CC-`の「関与するユーザーストーリー」と`US-`の「関与する概念クラス」）。
