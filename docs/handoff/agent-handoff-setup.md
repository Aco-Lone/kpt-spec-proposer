# VSCode GitHub Copilot 向け インプット資料：カスタムエージェントの handoff 設定

このドキュメントは、Copilotカスタムエージェントの**宣言的handoff機能**をこのKPTフローに導入し、初心者でも「次に誰を・いつ・どう起動するか」に迷わず1クリックで工程を進められるようにするための作業指示書です。

handoffの設定対象は `.github/agents/*.md` の YAML frontmatter であり、Coding Agent（サンドボックス）からは同ディレクトリを編集できません。よって本資料をVSCodeのCopilot Chat（Agentモード）に添付・参照させ、下記を反映してください。

---

## handoff 機能とは（前提）

- 各エージェント定義（`.github/agents/<name>.md`）の frontmatter に `handoffs:` を書くと、そのエージェントの完了時に次工程エージェントへ委譲する**ボタン**がCopilot UIに表示される。
- ボタンを押すと、指定した次エージェントへ **プロンプトと蓄積コンテキスト（会話・決定・ファイル）ごと** 引き継がれる。
- handoffは**ユーザーがボタンを押す**ユーザー起動が基本。自動連鎖ではない。

> 注: クラウド（GitHub.com のIssueアサイン運用）では handoff が未サポートの可能性がある。handoffが効くのは主にIDE（VS Code 等）/CLI。クラウド運用時は従来どおり手動でエージェントをメンション起動する（後述「運用の二重化」）。

## 対象ファイル

- `.github/agents/prototype-agent.md`
- `.github/agents/kpt-agent.md`
- `.github/agents/spec-writer.md`

（いずれも本Coding Agentからは閲覧できなかったため、実ファイルの現状を確認したうえで反映してください。）

---

## 設定するhandoffチェーン

このフローの工程遷移（AGENTS.md「全体プロセス」）をhandoffボタンで表現する。

```
prototype-agent 完了
  └─[KPTを集約する] → kpt-agent

kpt-agent 完了
  ├─[改善提案でモック更新] → prototype-agent      （継続プロトタイピング / 既存feature改善）
  ├─[新featureとして切り出す] → prototype-agent    （新feature切り出し・新request.md起票）
  ├─[spec仕様化する（承認）] → spec-writer         （specトラック収束時。人間承認ゲート）
  └─[rule確定する（承認）] → spec-writer           （spec確定後、ruleトラックが収束した時。人間承認ゲート）
```

## タスク1：prototype-agent に handoff を追加

`.github/agents/prototype-agent.md` の frontmatter に以下を追加する（実在キー名・ツール名は現状のファイルに合わせる）。

```yaml
handoffs:
  - label: KPTを集約する
    agent: kpt-agent
    prompt: このプロトタイプに対するKPTのCSVを取り込み、kpt-summary.md をワイド形式で集約・更新する
```

## タスク2：kpt-agent に handoff を追加（4系統）

`.github/agents/kpt-agent.md` の frontmatter に以下を追加する。4つのボタンで「継続」「新feature切り出し」「spec仕様化承認」「rule確定承認」の分岐を人間に選ばせる。

```yaml
handoffs:
  - label: 改善提案でモック更新
    agent: prototype-agent
    prompt: kpt-summary.md の改善提案（継続プロトタイピング）をもとに、既存featureのモックを更新する
  - label: 新featureとして切り出す
    agent: prototype-agent
    prompt: kpt-summary.md の「新feature切り出し候補」を、人間が指定する新しい feature-name で新しい request.md として起票し、新featureのモックを作る
  - label: spec仕様化する（承認）
    agent: spec-writer
    prompt: |
      spec仕様化が承認された。kpt-summary.md をもとに spec.md（仕様）を生成・更新する。
      rule が同時収束している場合に限り rule.md も同時に生成・更新してよい。rule が未収束ならrule.mdには一切触れない。
  - label: rule確定する（承認）
    agent: spec-writer
    prompt: |
      rule確定が承認された。spec は確定済み。kpt-summary.md をもとに rule.md（デザインルール）のみを生成・更新する。
      spec.md には触れない（spec の独立した変更根拠がない限り）。
```

**重要（承認ゲートとの整合）:**
- 「spec仕様化する（承認）」ボタンの押下＝AGENTS.md が要求する「人間の明示的な承認アクション」とみなす。`kpt-agent` はボタンを提示するだけで、押されない限り `spec-writer` へは渡らない。
- 「rule確定する（承認）」は、spec確定後にruleトラックが独立して収束した時点で改めて提示される、別の承認ボタンである。同じ`spec-writer`をrule確定専用に再起動できるが、specの変更根拠がない限りspec.mdには触れない。
- 「新featureとして切り出す」も同様に、ボタン押下（人間の承認）後にのみ新featureを起票する。`kpt-agent`/`prototype-agent` が自動で新ディレクトリを作らない。

## タスク3：spec-writer 側の承認ゲート確認

- `.github/agents/spec-writer.md` の `disable-model-invocation: true`（別資料 `agent-config-followup.md` タスク2で設定済み想定）は維持する。
- handoffボタン経由の起動が「モデルによる自動起動」ではなく「ユーザー起動」として扱われ、`disable-model-invocation: true` と両立して実行できることを**実機で検証**する。両立しない（ボタンでも起動できない）場合は、spec-writer へのhandoffボタンは付けず、従来どおり人間の手動メンション起動を維持する。
- `spec-writer` からの handoff は不要（仕様化はチェーンの終端。以降はループ先頭の prototype-agent に人間が戻る）。

## タスク4：運用の二重化への注意書き

handoffがIDE中心でしか効かない可能性があるため、READMEまたは本ディレクトリのガイドに以下を明記する。

- IDE（VS Code 等）: handoffボタンで工程を進める。
- クラウド（GitHub.com のIssue運用）: 従来どおり対象エージェントを手動でメンション/アサインして起動する。
- どちらの経路でも、`spec-writer` の起動には人間の承認が必須である点は変わらない。

## タスク5：kpt-agent 本文の集約表フォーマット更新

`.github/agents/kpt-agent.md` 本文が旧ロング形式の表（`ID | 種別 | 機能カテゴリ | 対象 | 内容 | 状態`）を記述している場合、ワイド形式（1論点=1行）へ更新する。

- 置換後の表カラム: `論点ID | 機能カテゴリ | 対象 | Keep | Problem | Try | Status`
- あわせて「同一論点のKeep/Problem/Tryを1行に束ねる」「Statusは論点ごとに1つに集約する」旨を反映する（詳細は `.github/skills/kpt-parsing/SKILL.md`）。
- 「重複統合・矛盾の両論併記・解消項目の状態更新」の記述は、論点ID単位・Status列に読み替えて維持する。

---

## 完了条件（Definition of Done）

- [ ] `prototype-agent` に `kpt-agent` への handoff が1つある。
- [ ] `kpt-agent` に prototype-agent（継続）/ prototype-agent（新feature切り出し）/ spec-writer（spec仕様化承認）/ spec-writer（rule確定承認）への handoff が4つある。
- [ ] `spec-writer` の `disable-model-invocation: true` が維持され、handoffボタン起動との両立が検証されている（両立しない場合はspec-writerへのボタンを外し手動起動に）。
- [ ] `kpt-agent` 本文の集約表がワイド形式（`論点ID | 機能カテゴリ | 対象 | Keep | Problem | Try | Status`）になっている。
- [ ] AGENTS.md の責任分界（越境禁止・人間承認必須）と矛盾しない。
- [ ] specトラックとruleトラックを別々に承認・確定できる（同時確定を必須としない）ことが明記されている。
- [ ] handoff未サポート環境向けに、手動起動フォールバックが明記されている。

## 参考

- ベース方針: `AGENTS.md`（全体プロセス／責任分界／新feature判別）
- プロセス全体像: `docs/process/prototype-driven-flow.md`
- 既存の残タスク資料: `docs/handoff/agent-config-followup.md`
