# kpt-spec-proposer

プロトタイプ開発をKPTで進め、仕様（spec）を固めるための agent plugin です。

シンプルな要望から画面操作可能なモックを作り、人間が記入したKPT（Keep/Problem/Try）をagentが集約・整理し、曖昧な点をagentが質問しながら改善提案を行い、最終的にシステム非依存の`spec.md`へ収束させます。プロトタイプとKPTは1回で終わらず、ループし続けることを前提としています。

## 全体像

```
要望Issue
   → prototype-agent（モック生成）
   → 人間がKPTをスプレッドシートで管理しCSVを取り込み
   → kpt-agent（集約・曖昧点質問・改善提案・ループ判定）
   → 「継続プロトタイピング」なら prototype-agent に戻る
   → 「仕様化」なら人間承認を経て spec-writer が spec.md を生成
   → spec.md確定後もループは継続
```

## リポジトリ構成

```
AGENTS.md                        リポジトリ全体のベース方針、エージェントの責任分界

.github/
  agents/
    prototype-agent.md           要望/改善提案 → モック生成・更新
    kpt-agent.md                 KPT集約 + 曖昧点抽出 + 改善提案 + ループ判定
    spec-writer.md               仕様化承認後にのみ起動し spec.md を生成
  skills/
    mock-scaffolding/SKILL.md        モック実装時の簡略化ルール
    kpt-parsing/SKILL.md             KPTの抽出・分類・生ログ保存ロジック
    ambiguity-detection/SKILL.md     曖昧点の判定ロジック
    readiness-check/SKILL.md         「継続プロトタイピング」か「仕様化」かの判定材料
    spec-template/SKILL.md           システム非依存のspec.mdフォーマット
    change-history-logging/SKILL.md  ラウンドごとの変更履歴の追記ルール
  ISSUE_TEMPLATE/
    01_feature-request.yml       要望入力（feature-nameを指定）
    02_kpt-feedback.yml          KPT取り込み（スプレッドシートのCSVをコミット・貼付け、feature-nameで対応づけ）

docs/
  process/
    prototype-driven-flow.md     GitHubに依存しないプロセス手順書（可搬性のため）
  specs/
    <feature-name>/
      request.md                 元の要望
      prototype-notes.md         ラウンドごとの変更点・簡略化点（追記蓄積）
      kpt-log/roundN.csv         KPTの生ログ（スプレッドシートから取り込んだCSV。削除・改変しない）
      kpt-summary.md             kpt-agentによる集約結果（Markdown表形式＋ID、毎ラウンド更新）
      clarification-log.md       曖昧点の質問と回答（曖昧点がある場合）
      spec.md                    仕様化承認後にのみ生成
```

## 具体的な進め方

GitHub上での実際の操作手順です。ツールに依存しないプロセスとしての説明は`docs/process/prototype-driven-flow.md`を参照してください。

### Step 0. 準備（最初の1回だけ）

1. このリポジトリ（または`.github/agents/`・`.github/skills/`・`.github/ISSUE_TEMPLATE/`・`AGENTS.md`をコピーしたリポジトリ）を用意する。
2. リポジトリでGitHub Copilot Coding Agentが有効になっていることを確認する。
3. KPT記入用のスプレッドシートを用意し、列を`種別,機能カテゴリ,内容,状態`にする（IDの列は作らない。集約時にagentが付与するため）。

### Step 1. 要望を出す（人間）

1. Issueを新規作成し、テンプレート「機能要望 (Feature Request)」を選ぶ。
2. **機能名（feature-name）** を決めて入力する（例 `color-theme-setting`）。以降の成果物はすべて`docs/specs/<feature-name>/`に集約され、KPT Issueもこの名前で対応づける。
3. やりたいこと・背景・除外してよい範囲を記入してIssueを作成する。
4. Issueを`prototype-agent`にアサイン（またはメンション）する。

### Step 2. モックを作る（prototype-agent）

`prototype-agent`が以下を行い、PRを作成します。

- `docs/specs/<feature-name>/request.md`に要望原文を記録
- 画面操作可能なモックを実装（実データ連携・永続化・認証は行わない）
- `prototype-notes.md`に`## Round N`見出しで実装内容と簡略化した点を追記
- KPT記入を促すコメント（KPT Issueテンプレートへのリンク付き）を残す

人間はPRをレビューし、問題なければマージしてモックを触れる状態にします。

### Step 3. KPTを記入する（人間）

1. モックを実際に操作し、スプレッドシートにKeep/Problem/Tryを記入する。
2. CSVとして書き出し、`docs/specs/<feature-name>/kpt-log/roundN.csv`にコミットする（Nはラウンド番号。初回は`round1.csv`）。
   - コミットが難しい場合は、KPT Issueの「KPT CSV（任意・貼り付け）」欄に貼り付けてもよい。この場合`kpt-agent`が`roundN.csv`を新規作成する。
3. Issueを新規作成し、テンプレート「KPTフィードバック」を選ぶ。feature-name・対象プロトタイプ・CSVのパスを記入する。
4. Issueを`kpt-agent`にアサイン（またはメンション）する。

### Step 4. KPTを集約してもらう（kpt-agent）

`kpt-agent`はCSVを生ログとして保存したうえで、`kpt-summary.md`に`ID | 種別 | 機能カテゴリ | 内容 | 状態`の表として集約します。その後、結果は次の2つに分岐します。

**A. 曖昧点がある場合**

- 選択式の質問（最大5件）が1回のIssueコメントにまとめて投稿され、`clarification-log.md`に記録される
- `kpt-summary.md`の末尾は「次のアクション: 回答待ち」になる
- 人間は**すべての質問にまとめて1回で回答**し、再度`kpt-agent`をメンションする → Step 4をやり直す

**B. 曖昧点がない場合**

- 改善提案がIssueコメントに投稿される
- `kpt-summary.md`の末尾に「次のアクション: 継続プロトタイピング」または「次のアクション: 仕様化（人間承認待ち）」が明記される

### Step 5. 分岐して進む（人間が判断）

**「継続プロトタイピング」の場合**

改善提案を入力として、再び`prototype-agent`をメンションする → Step 2に戻る（ラウンド番号が1つ進む）。spec.mdはまだ作りません。

**「仕様化（人間承認待ち）」の場合**

判定材料（収束度・曖昧点の有無・Keep/Problem比率）を人間が確認し、納得できた場合のみ`spec-writer`を明示的にメンション・アサインして起動する。`spec-writer`はモデルからの自動起動が禁止されているため、人間が起動しない限りspec.mdは生成されません。納得できなければ承認せず、継続プロトタイピングとして進めても構いません。

### Step 6. 仕様化後も続ける

`spec.md`は「その時点のスナップショット」です。確定後もモックの改善とKPT記入は続け、Step 2〜5を繰り返します。以降の差分は`spec-writer`が変更履歴として`spec.md`に追記します。

### 進め方の要点

- 人間が明示的にagentを起動するタイミングは、要望作成時・KPT提出時・仕様化承認時の3つ。
- 曖昧点への回答は1回でまとめて行う（往復回数を減らすため）。
- `kpt-log/roundN.csv`は生ログなので、集約後も削除・編集しない。

## 設計判断の記録

この構成に至るまでに検討し、採用しなかった選択肢も含めて記録します。

### なぜエージェントを2〜3体に絞ったか

当初は「モック生成」「KPT集約」「曖昧点質問」「仕様化」を4つの独立したカスタムエージェントに分ける案を検討しました。しかし、GitHub Copilot Coding Agentは基本的にIssue/PRへのメンションで起動する非同期モデルであり、フェーズが変わるたびに人間が「次はこのagentを呼ぶ」と手動で切り替える必要がありました。これは呼び忘れや手戻りの原因になるため、性質が近い「KPT集約」と「曖昧点抽出・質問」を`kpt-agent`という1つのエージェントに統合し、内部で条件分岐（曖昧点があれば質問して中断、なければ改善提案へ進む）させる設計にしました。

一方で「モック生成（コード生成）」と「仕様書生成（文書生成）」は処理の性質が明確に異なるため、`prototype-agent`と`spec-writer`は分離したままにしています。

### なぜ曖昧点の質問を1セッション内で完結させるルールにしたか

Coding Agentは同期対話ではなく、Issueコメント→セッション再起動という非同期・低頻度なやりとりです。質問のたびにセッションを立ち上げて人間の回答を待つ往復を繰り返すと、待ち時間とコストが積み重なります。そのため「曖昧点は1回のセッションでまとめて出し切り、人間は1回でまとめて回答する」という往復回数を最小化するルールをAGENTS.mdに明記しました。

### なぜ仕様化を自動連鎖させず、人間の承認を必須にしたか

KPTからの改善提案は、プロトタイプでさらに検証すべきものと、仕様として確定してよいものが混在します。`kpt-agent`が「仕様化してよい」と判断した内容を自動でそのまま`spec-writer`に渡してspec.mdを確定させてしまうと、検証不足のまま仕様が固まるリスクがあります。そのため`kpt-agent`はあくまで判定結果（readiness-checkの材料）を提示するだけにとどめ、`spec-writer`の起動は必ず人間の明示的なアクションを挟む設計にしています。

### なぜKPTを表形式＋機能カテゴリIDで管理するか

KPTの項目を文章のまま蓄積すると、ラウンドをまたいだ同一論点の追跡が文面の照合に依存し、収束判定（readiness-check）が不安定になります。そこで集約結果を`ID | 種別 | 機能カテゴリ | 内容 | 状態`の表にし、各項目に安定IDを付与しました。IDの接頭辞は「Keep/Problem/Tryの種別」ではなく「機能カテゴリ（プロトタイプの機能領域）」とし、同じ機能領域のフィードバックをラウンド横断でまとめられるようにしています。IDは集約時に付与し、人間の入力（生ログ・KPTのCSV）には振らないことで、人間の記述負荷を増やさずに追跡性を得ています。

### なぜKPTをスプレッドシート管理＋CSV取り込みにしたか

KPTは既存のスプレッドシートで管理しているチームが多く、その運用に合わせて「スプレッドシートで管理→CSVで取り込む」方式を採用しました。入力CSVのカラムは `種別,機能カテゴリ,内容,状態`で、IDは含めません（集約時にagentが付与）。現在の優先方法は `docs/specs/<feature-name>/kpt-log/roundN.csv` へのコミットで、このファイルがそのまま生ログになります。将来的にIssueフォームへのCSV貼り付けに切り替えられるよう、取得元に依らずCSV本体を入力として扱います。一方で集約結果（`kpt-summary.md`）は、GitHub上で表として読めてreadiness-checkの人間レビューに有利なためMarkdown表のままにしています。

### なぜspec.mdをシステム非依存のMarkdown形式にしたか

プロセス自体（Issueテンプレートやカスタムエージェント）はGitHub Copilot Coding Agentに強く依存しますが、成果物であるspec.mdは他のツール（Notion、Jira、他のSDDツールなど）にそのまま転記・移植できるよう、特定システムの記法に依存しない見出し構造のみのフォーマットにしています。プロセスの可搬性を高めるため、`docs/process/prototype-driven-flow.md`にGitHubに依存しない手順書も別途用意しています。

### なぜプロトタイプ→KPTのループがspec.md確定後も終わらない設計にしたか

spec.mdの確定を「開発の終点」として扱うと、確定後に得られた新しい気づき（KPT）を反映する仕組みがなくなってしまいます。そのため、spec.md確定はあくまで「その時点のスナップショット」と位置づけ、以降もプロトタイプの改善とKPTの記入は継続し、spec.mdとの差分は`change-history-logging`スキルを通じて追記していく運用にしています。
