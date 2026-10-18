# kpt-spec-proposer

概念設計から要件・Spec・デザインを固め、REQ単位のTDDでプロトタイプを作り、KPTで振り返って上流へフィードバックし続けるための agent plugin です。

機能横断の概念設計（概念クラス・ユーザーストーリー）を起点に、要件（`requirements.md`）→ Spec（`spec.md`）・デザイン検討へ進みます。Specからテストケースを作り、`tdd-orchestrator`がREQ単位でTDD実装を管理します。人間が記入したKPT（Keep/Problem/Try）はagentが集約・整理し、曖昧な点を質問しながら、反映先の工程（概念・要件・Spec・デザイン・実装）を提案します。`US → REQ → AC → TC → テストコード`のトレーサビリティを`traceability.md`で追跡します。

## ルール
- **プロンプトは簡潔に保つこと**
  - 括弧で補足はしない
  - 同じことを複数の箇所で書かない
  - 余分なバリデーションや説明は書かない
- **スキルは手順（`SKILL.md`）と記載内容の規定（`templates/`）で構成する**

## 全体像

```
concept-designer（概念クラス・横断US → docs/knowledge/）
   → requirements-writer（request.md・requirements.md）
   → spec-writer（spec.md）‖ design-agent（design-token・design-rule）
   → tdd-orchestrator（REQごとに繰り返す）
        test-designer（test-cases.md・traceability.md）
        prototype-agent（Red→Green→Refactor・モック・テストコード）
   → 人間がKPTを取り込み（CSVコミット / Issue貼付け / 自由文記入）
   → kpt-agent（集約・曖昧点質問・改善提案を反映先エージェント別に整理）
   → 人間が反映先のエージェントを起動してループ継続
```

プロセスの正本は [AGENTS.md](AGENTS.md)（責任分界・厳守事項・詳細ルールの所在）と各skillファイルです。ツール非依存の手順は [docs/process/prototype-driven-flow.md](docs/process/prototype-driven-flow.md) を参照してください。

## エージェントの起動方法（運用の二重化）

各エージェント定義（`.github/agents/*.md`）の frontmatter には、工程遷移を1クリックで進められる宣言的 `handoffs`（ボタン）を設定しています。起動経路は環境によって異なります。

- **IDE（VS Code 等）**: 完了時に表示される handoff ボタンで、蓄積コンテキストごと次工程へ引き継げます。
- **クラウド（GitHub.com のIssue運用）**: handoff は未サポートの可能性があるため、対象エージェントを手動でメンション・アサインして起動します。
- どちらの経路でも、`concept-designer`・`requirements-writer`・`spec-writer`・`design-agent`・`tdd-orchestrator`の起動には**人間の明示的な起動**が必須です。`test-designer`・`prototype-agent`は`tdd-orchestrator`の配下のサブエージェントで、直接起動しません。

## リポジトリ構成

```
AGENTS.md                        ベース方針・責任分界・詳細ルールの正本一覧

.github/
  agents/                        concept-designer / requirements-writer / spec-writer / design-agent /
                                 tdd-orchestrator / test-designer / prototype-agent / kpt-agent
  skills/                        <skill>/SKILL.md（手順）+ templates/（記載内容の規定）
  ISSUE_TEMPLATE/                01_feature-request.yml / 02_kpt-feedback.yml

mocks/<feature-name>/            モック本体（Vite + React + TypeScript）とテストコード（Vitest / Playwright）

docs/
  knowledge/                      OKF(Open Knowledge Format)ナレッジバンドル（concept-class/user-story/design-token/design-rule/glossary-term/system-proposal、リポジトリ横断、常に最新）
  process/prototype-driven-flow.md  ツール非依存のプロセス手順書
  kpt/                            kpt-log/roundN.csv・roundN-detail.md / kpt-summary.md
  specs/<feature-name>/          request.md / requirements.md / spec.md / test-cases.md /
                                 traceability.md / prototype-notes.md / clarification-log.md
```

## 入力仕様（要点）

詳細は各skillファイルを正本とし、ここでは入力時に知っておくべき要点のみを示す。

- **KPT CSVのヘッダ**: `機能カテゴリ,Keep,Keep状態,Problem,Problem状態,Try,Try状態`（この順で7列固定、1行=1論点）。状態は内容が非空の列のみ必須。→ [kpt-parsing](.github/skills/kpt-parsing/SKILL.md)
- **KPTの取得元**: 「コミット済みCSVパス」「Issue本文への貼り付けCSV」「自由文記入」のいずれか。自由文の場合はagentがCSV形式に整形して`roundN.csv`に保存する。CSVパスと貼り付けを両方指定した場合は内容が一致しないと処理を中断する。→ 同上
- **ID体系**: `US-<NNN>`（横断ユーザーストーリー）/ `CC-<名前>`（概念クラス）/ `REQ-<NNN>` / `AC-<REQ番号>-<n>` / `TC-<NNN>`。→ [traceability-matrix](.github/skills/traceability-matrix/SKILL.md)
- **TDDの進め方**: REQ単位。`tdd-orchestrator`が`requirements.md`の記載順（依存REQ先行）に全REQを自走し、REQごとに完了報告する。検証に失敗したREQで停止する。→ [tdd-workflow](.github/skills/tdd-workflow/SKILL.md)
- **docs/knowledge/（OKF）の更新**: `concept-designer`（`concept-class`・`user-story`）・`design-agent`（`design-token`・`design-rule`）・`kpt-agent`（`glossary-term`・`system-proposal`）が承認なしで適宜更新する。常に最新の内容に上書きし、変更履歴・ログは持たない。`design-rule`の各項目は一文の短い箇条書きのみとし、意図・理由は書かない。→ [okf-knowledge](.github/skills/okf-knowledge/SKILL.md)

## 設計判断の記録

この構成に至るまでに検討し、採用しなかった選択肢も含めて記録します。

- **なぜ概念設計を最初の工程に置くか**: 機能ごとに要件を起こすと、同じ概念（用語・関連）が機能間で食い違う。概念クラスと機能横断のユーザーストーリーを先に固め、要件は`US-`・`CC-`を参照して起こすことで、機能をまたいだ一貫性とトレーサビリティの起点を確保する。概念設計は`docs/knowledge/`のOKF conceptとし、常に最新のみ保持する。
- **なぜ要件（`requirements.md`）とSpec（`spec.md`）を分離するか**: 要件は「何を満たすか」でTDDの処理単位（REQ）の正本、Specは「どう振る舞うか」と受け入れ基準（AC）の正本。混在させるとREQの追加・変更とACの追加・変更が絡み、影響範囲を追えなくなる。
- **なぜSpecをDesign by Contractで記述し、異常系を網羅するか**: 「呼び出し側が満たす条件（事前条件）」「提供側が保証する結果（事後条件）」「常に成立する条件（不変条件）」を分けると、責任の所在と契約違反時の振る舞いが明確になる。正常系だけのSpecではテストケースも正常系に偏るため、全ての事前条件に異常系を定め、網羅観点（入力値・状態・順序・競合・外部依存・権限）で漏れを確認する。契約と異常系の全項目をACで参照することで、異常系のテストをTDDに確実に乗せる。
- **なぜSpecをプロトタイプの前に置くか**: 以前はプロトタイプとKPTを経てからspec化していたが、テストを先に書くTDDにはACが先に必要。KPTの結果は要件・Specへフィードバックし、変更履歴に差分を残してから下流（テスト・実装）を更新する。
- **なぜTDDをREQ単位のオーケストレーターで管理するか**: テストケース作成と実装を別々に呼び出すと、どこまで検証済みかが曖昧になる。`tdd-orchestrator`がREQごとに`test-designer`→`prototype-agent`を呼び、全AC・TCが緑で`traceability.md`に未紐付けがないことを確認して初めて完了とする。オーケストレーター自身は成果物を編集せず、検証に専念する。
- **なぜREQ完了で人間の確認を待たないか**: 検証（テスト実行・トレーサビリティ確認）をオーケストレーターが担うため。ただし検証に失敗したREQでは停止し、後続REQには進まない（依存が崩れるため）。
- **なぜ`test-designer`・`prototype-agent`をサブエージェントにするか**: テストケースの期待値を実装側が都合よく変えることを防ぐ。`prototype-agent`はTCの期待値を変えられず、誤りは`test-designer`へ差し戻す。
- **なぜデザイン検討を`design-agent`に分離したか**: デザインは機能の仕様と独立して固まり、`spec-writer`と並行して進められる。`prototype-agent`はTDD実装に専念できる。
- **なぜスキルを手順とテンプレートに分けるか**: 手順（いつ何をするか）と記載内容（何を書くか）は変更理由が異なる。分けることで、書式の変更が手順に影響せず、同じ規定の重複記述も避けられる。
- **なぜ曖昧点の質問を一問ずつ対話的に確認するか**: 複数の曖昧点を一度に列挙すると人間が読み飛ばし・雑な一括回答をしがちで、解釈違いによる手戻りにつながる。`ambiguity-detection`に従い1つの論点を推奨回答付きで掘り下げてから次に進むことで回答の精度を上げる。往復は同一セッション内の対話で完結させ、複数セッションには分割しない。
- **なぜKPTを表形式＋機能カテゴリ論点IDで管理するか**: 文章のまま蓄積するとラウンドをまたいだ同一論点の追跡が文面照合に依存して不安定になる。1論点=1行のワイド形式に安定した論点IDを振ることで「今どう評価され、何が課題で、次に何を試すか」を横断で読める。IDの接頭辞は種別ではなく機能カテゴリとし、同じ機能領域のフィードバックをまとめられるようにした。論点IDと対象は集約時にagentが付与し、人間の記述負荷を増やさない。
- **なぜKPTをスプレッドシート管理＋CSV取り込みにしたか**: KPTを既存のスプレッドシートで管理しているチームが多いため。取得元に依らずCSV本体を入力として扱い、現在の優先方法は`docs/kpt/kpt-log/roundN.csv`へのコミット（このファイルがそのまま生ログになる）。一方で集約結果はGitHub上で表として読めるMarkdown表にしている。KPTは機能横断のフィードバックのため、特定featureのフォルダには入れずリポジトリ横断の`docs/kpt/`に置く。
- **なぜKPT入力にCSVだけでなく自由文も許すか**: CSV作成自体が心理的ハードルになりKPT記入が滞ることを避けるため。自由文で受け取ってもagentがCSV形式に整形してから`roundN.csv`に保存するため、生ログの形式・不可変性は変わらない。
- **なぜKPTの改善提案に反映先エージェントを付けるか**: KPTの論点は概念・要件・Spec・デザイン・実装のどこに影響するかが異なる。反映先を明示し、人間が該当エージェントを起動することで、上流から順に更新する運用を守れる。`kpt-agent`は反映先の成果物を編集しない。
- **なぜspec.mdとdesign-ruleを分離したか**: spec.mdに配色・余白・遷移を混ぜると実装スタックへの依存が仕様に染み出し、可搬性という設計意図に反する。またデザインルールは複数の仕様から参照されうる共有資産で、複製するとドリフトを招く。確定の仕方も非対称（spec=変更履歴を追記、design-rule=承認なしで最新に上書き）にしている。
- **なぜdesign-tokenをデザイン検討の初回に確定するか**: デザインルールは3層（spec / design-rule / design-token）。`design-token`は独自に育てる共有ルールではなく、Material Design のような**確立済み外部標準への準拠宣言と基本トークンの記録**。外部標準を参照するだけなので空箱にならない。横断共有が本質なので`docs/knowledge/design/tokens/`に置く。
- **なぜ「対象」をagentが付与し安定化させるか**: 人間に書かせると記述負荷が増える（IDと同じ方針）。ただし判定がラウンドごとに揺れると成果物の振り分けが不安定になるため、一度確定した対象は再利用し、迷ったら`spec`を既定とし、真に判別不能なものだけを質問化する。
- **なぜdesign.md/rule.md/KPT用語/システム構成案/概念設計をOKFへ統合したか**: これらは性質は異なるが「リポジトリ横断で常に最新を保つ知識」という共通点を持つ。個別ファイルのまま増やすと参照場所がバラつくため、Google Cloudの Open Knowledge Format（concept=1ファイル+YAML frontmatter）に倣った単一バンドル`docs/knowledge/`に統合した。`system-proposal`は確定仕様ではなく参考メモという位置づけを崩さないため`status: reference-only`を必須にしている。
- **なぜdocs/knowledge/に変更履歴を持たせないか**: requirements.mdやspec.mdのような「合意形成の跡」を追う必要がある成果物とは異なり、OKFは常に最新の状態が分かればよい知識であるため、履歴を持たせず本文を都度上書きする。ラウンドをまたいだ経緯はprototype-notes.mdやkpt-summary.mdの方に残る。
