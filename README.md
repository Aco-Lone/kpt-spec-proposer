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
