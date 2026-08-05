# VSCode GitHub Copilot 向け インプット資料：エージェント定義の残タスク

このドキュメントは、Coding Agent（サンドボックス）では `.github/agents/` にアクセスできず対応できなかった残タスクを、**VSCode の GitHub Copilot で対応する**ための作業指示書です。前段のPRで、スキルの `SKILL.md` 化・light/full 差分の明記・ラウンド採番／生ログ保存責務・feature-name 追加などは対応済みです。本資料はその続きにあたります。

VSCode の Copilot Chat（Agent モード）にこのファイルを添付・参照させ、下記タスクを実行させてください。

---

## 対象ファイル

- `.github/agents/prototype-agent.md`
- `.github/agents/kpt-agent.md`
- `.github/agents/spec-writer.md`

（いずれも本Coding Agentからは閲覧できなかったため、実ファイルの現状を確認したうえで反映してください。）

## 背景（なぜこの変更が必要か）

- スキルは `.github/skills/<name>.md`（フラット）から `.github/skills/<name>/SKILL.md`（サブフォルダ + YAML frontmatter）へ移行済み。**旧フラットファイルは削除済み**のため、旧パス参照は壊れている。
- AGENTS.md は「specを起こすのは人間の判断であり、kpt-agent は仕様化を提案・自動連鎖させない」と明記しているが、これがプロンプト本文の記述だけに依存していると破られやすい。frontmatter 設定でも担保する必要がある。

---

## タスク1：スキル参照パスの更新（3ファイル）

各エージェント定義本文が旧スキルパスを参照している場合、新パスへ置換する。

- 置換前（例）: `.github/skills/mock-scaffolding.md`
- 置換後（例）: `.github/skills/mock-scaffolding/SKILL.md`

対象スキル名（すべて `.github/skills/<name>/SKILL.md`）:

| スキル | 主な参照元エージェント |
|---|---|
| `mock-scaffolding` | prototype-agent |
| `design-rule-template` | prototype-agent |
| `kpt-parsing` | kpt-agent |
| `ambiguity-detection` | kpt-agent |
| `spec-template` | spec-writer |
| `change-history-logging` | prototype-agent（rule.md）/ spec-writer（spec.md） |

チェック手順:
1. 各エージェントファイル内を `.github/skills/` で全文検索する。
2. `<name>.md` 形式の参照を `<name>/SKILL.md` に修正する。
3. 参照が本文に無い場合は、担当スキルを明示的に参照する記述（「◯◯の際は `.github/skills/<name>/SKILL.md` に従う」等）を追記することを検討する。

## タスク2：承認ゲートを frontmatter で担保（spec-writer）

`.github/agents/spec-writer.md` の YAML frontmatter に以下を設定し、モデルによる自動起動を禁止する（人間の明示的な起動のみ許可）。

```yaml
disable-model-invocation: true
```

- 目的: AGENTS.md「specを起こすのは人間の判断であり、kpt-agent は spec化を提案・自動連鎖させず、spec.md 生成は必ず人間の明示起動を経る」を設定レベルで担保する。
- `user-invocable` は既定 `true` のままでよい（人間が手動で呼べる必要があるため）。

## タスク3：ツール権限の最小化（3ファイル）

各エージェントの frontmatter `tools` を責務に応じて最小化する。現状の設定を確認し、過剰な権限があれば削る。

- `prototype-agent`: モック（コード）生成・更新に必要なツール（読み取り・編集・検索等）。
- `kpt-agent`: KPT集約・文書更新中心。コード編集が不要なら編集系ツールは最小限に。
- `spec-writer`: `spec.md`（文書）生成のみ。コード実行系ツールは付与しない。

> 注: `tools` に指定できる値は環境で利用可能なツール名に依存する。現状の各ファイルの記述に合わせ、実在するツール名のみを使うこと。

## タスク4：frontmatter 必須項目の確認（3ファイル）

各エージェントに以下があるか確認し、無ければ追加する。

- `description`（必須）: エージェントの役割・境界を1〜2文で。
- `name`（任意・未指定ならファイル名が使われる）。

---

## 完了条件（Definition of Done）

- [x] 3エージェントのスキル参照がすべて `.github/skills/<name>/SKILL.md` になっている（壊れた旧パス参照が残っていない）。
- [x] `spec-writer` に `disable-model-invocation: true` が設定されている。
- [x] 各エージェントの `tools` が責務に対して過不足ない。
- [x] 各エージェントに `description` が存在する。
- [x] AGENTS.md の責任分界（prototype/kpt/spec-writer の越境禁止、specは人間の判断で起動、ruleは承認なしで更新）と矛盾しない。

## 参考

- 変更済みスキルの実体: `.github/skills/*/SKILL.md`
- ベース方針: `AGENTS.md`
- プロセス全体像: `README.md` / `docs/process/prototype-driven-flow.md`
