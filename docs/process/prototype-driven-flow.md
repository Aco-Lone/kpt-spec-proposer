# プロトタイプ駆動KPTフローの手順書（ツール非依存）

この文書は、GitHub Copilot Coding Agentという実装手段に依存しない、プロセス自体の説明です。将来的に別のツールでこのプロセスを再現する場合はこの文書を参照してください。

具体的なファイル形式・判定式・列挙値は各skillファイル（`kpt-parsing` / `ambiguity-detection` / `readiness-check`）とAGENTS.mdを正本とし、ここでは重複定義しません。本書は「何のためにこの手順があるか」と「複数ラウンド・複数featureを扱うときにどの順で何を呼ぶか」に焦点を当てます。

## サイクル

1. **要望の受付**: シンプルな要望を受け付ける。
2. **プロトタイピング**: 要望または前回の改善提案をもとに、画面操作可能なモックを作る。実データ・永続化・認証等は簡略化してよい。簡略化した点は必ず記録する。
3. **KPTレビュー**: プロトタイプに触れた人間が、Keep（良い点）/Problem（課題）/Try（次に試すこと）を、種別・機能カテゴリ・内容・状態の列を持つ表（スプレッドシート等）で記入する（種別ごとに1行）。
4. **入力検証**: 取り込んだKPTのCSVは、取得元（コミット済みパス／貼り付け）の少なくとも一方が揃っているか、ヘッダ列とカラム順、種別ごとの状態の許容値を集約前に検証する。検証エラーは値を推測・補正せず、入力元・行・列・値を示して処理を中断する。
5. **KPT取り込みと集約**: 検証を通過したKPTを構造化データ（CSV）として取り込み、機能カテゴリ別の論点IDを付けたワイド形式（1論点=1行、Keep/Problem/Tryを各列に束ねる）の表に集約して重複や矛盾を整理する。論点IDは取り込み時に付与し、ラウンドをまたいで同一論点を追跡できるよう再利用する。あわせて各論点が「仕様（WHAT）」の話か「機能独自のデザインルール（見た目・操作感）」の話かの区別（対象）を付与し、さらに各論点が既存feature改善か新featureとして切り出すべきかを判別する。区別・判別も取り込み時に付与し、迷う場合は仕様／既存feature改善を既定とする。集約結果は論点ごとの最新スナップショットとして保持し、ラウンドごとの観測は別途、削除・改変しない履歴として残す。
6. **曖昧点の解消**: 集約結果に曖昧な点（範囲・優先度・対象者・成功基準の不明確さ、仕様/デザインの区別不能、既存feature改善/新featureの判別不能、既存デザインルールとの矛盾）があれば、件数で打ち切らず必要な質問をすべて1回のセッションでまとめて選択式で確認する。曖昧点がなくなるまで改善提案には進まない。
7. **改善提案とループ判定**: Try起点で改善案を提示し、あわせて当該ラウンドの収束度・未解決の曖昧点・Keep/Problem比率をもとに、「まだプロトタイプで検証すべきか」「仕様として確定してよいか」を、仕様/デザインの区別（トラック）ごとに判定材料とともに提示する。既存featureのスコープを超える論点があれば新feature切り出し候補もあわせて提示する（実行は人間の承認後）。
8. **分岐**:
   - 継続プロトタイピングの場合: 改善提案を次のプロトタイピング（手順2）への入力とし、サイクルを繰り返す。プロトタイピングは確定済みのデザインルールがあればそれに準拠する。
   - 新feature切り出しの場合: 人間が承認したうえで、新しいfeature-nameで要望を起票し、別の成果物ディレクトリとして手順1から新たなサイクルを開始する。元featureのサイクルはそのまま継続する。
   - 仕様化の場合: 仕様（WHAT）とデザインルール（見た目・操作感）はトラックごとに収束度が異なるため、必ずしも同時には確定しない。仕様トラックが収束すれば、デザインルールが未収束でも人間が承認したうえでシステム非依存の仕様書（spec.md相当）を確定できる。デザインルールは、収束した時点で別途人間が承認したうえで確定する（仕様確定より後追いになってよい）。仕様書に見た目・操作感のルールを混ぜない（可搬性のため）。
9. **仕様確定後も継続**: 仕様書・デザインルールの確定はプロセスの終了ではなく、その時点のスナップショットである。以降もプロトタイピングとKPTは続き、各成果物との差分は変更履歴として記録し続ける。

## 設計思想

- **曖昧なまま次工程に進めない**: 人間の記述は省略されがちなため、解釈違いによる手戻りを防ぐ目的で、曖昧点の解消を仕様化の前工程に必ず挟む。
- **仕様化は人間の承認を必須とする**: 自動化された提案をそのまま仕様として確定させない。仕様確定とデザインルール確定は別々に承認してよい。
- **仕様確定をゴールにしない**: プロトタイプとKPTのループは仕様確定後も継続し、仕様書は都度更新されるスナップショットとして扱う。

---

## サイクル全体像（フローチャート）

登場する「役割」は実装手段に依存しません（Coding Agentでは`prototype-agent`/`kpt-agent`/`spec-writer`という具体的なagentに対応します）。

```mermaid
flowchart TD
    A[要望の受付] --> B["プロトタイピング\n(モック生成・更新)"]
    B --> C["人間: プロトタイプを操作し\nKeep/Problem/Tryを記入"]
    C --> D["入力検証\n(取得元・ヘッダ・状態値)"]
    D -- 検証エラー --> C
    D -- 検証OK --> E["KPT取り込みと集約\n(論点ID・対象・新feature判別を付与)"]
    E --> F{曖昧点あり?}
    F -- あり --> G["選択式の質問を1セッションで出し切る\n(clarification-log.md に記録)"]
    G --> H["人間が回答"]
    H --> E
    F -- なし --> I["改善提案 + トラック別ループ判定\n(readiness-check)"]
    I --> J{総合アクション}
    J -- 継続プロトタイピング --> B
    J -- 新feature切り出し候補あり --> K["人間が承認"]
    K --> L["新しいfeature-nameで要望を起票\n(別ディレクトリ・Round1から独立)"]
    L --> B
    J -- "spec仕様化 or rule確定\n(人間承認待ち)" --> M["人間が承認"]
    M --> N["spec.md / rule.md をトラック別に生成・更新\n(change-history-loggingで差分追記)"]
    N --> B
```

ポイント:
- `J`（ループ判定）から`B`（プロトタイピング）へ戻る矢印が3本あるとおり、**「継続」「新feature切り出し」「仕様化」のいずれの分岐でも、元のfeatureおよび新featureのプロトタイピング/KPTループは止まらない**。
- 仕様化（`N`）はループから抜ける終端ではなく、単に`spec.md`/`rule.md`という成果物が更新されるだけで、その後も`B`に戻って回り続ける。

---

## ラウンドごとのagent呼び出し（シーケンス図）

1つのfeatureについて、Round1〜Round3の間にどの役割がいつ呼ばれるかを示します（Coding Agent運用ではそれぞれ`prototype-agent`/`kpt-agent`/`spec-writer`に対応）。

```mermaid
sequenceDiagram
    actor H as 人間
    participant P as prototype-agent役
    participant K as kpt-agent役
    participant S as spec-writer役

    Note over H,P: Round1
    H->>P: 要望Issue（feature-name指定）
    P->>P: request.md作成 + モック初版生成
    P-->>H: KPT記入を依頼
    H->>H: round1.csv 作成・コミット（または貼り付け）
    H->>K: KPT取り込みを依頼（feature-name指定）
    K->>K: 入力検証 → kpt-log/round1.csv 保存（生ログ）
    K->>K: 論点ID・対象・新feature判別を付与 → kpt-summary.md 集約
    alt 曖昧点あり
        K-->>H: 質問を1セッションでまとめて投稿
        H-->>K: 回答をコメント
        K->>K: clarification-log.md 更新 → 再集約
    end
    K-->>H: 改善提案 + 総合アクション=継続プロトタイピング

    Note over H,P: Round2
    H->>P: 改善提案をもとにモック更新を依頼
    P->>P: prototype-notes.md に Round2 追記
    H->>H: round2.csv 作成・コミット
    H->>K: KPT取り込みを依頼
    K->>K: kpt-log/round2.csv 保存 → kpt-summary.md 更新（KPT一覧は上書き、ラウンド履歴は追記）
    K-->>H: 総合アクション=spec仕様化（人間承認待ち）

    Note over H,S: Round2終盤: spec確定（ruleは未収束）
    H->>S: spec仕様化を承認（specアクション）
    S->>S: spec.md 新規作成
    S-->>H: 「rule は継続プロトタイピング継続。rule.mdは未作成」と明記

    Note over H,P: Round3（spec確定後もループは継続）
    H->>P: 追加の改善提案でモック更新を依頼
    P->>P: prototype-notes.md に Round3 追記（rule準拠状況も記録）
    H->>H: round3.csv 作成・コミット
    H->>K: KPT取り込みを依頼
    K->>K: kpt-log/round3.csv 保存 → kpt-summary.md 更新
    K-->>H: 総合アクション=rule確定（人間承認待ち）（specは確定済みのため変更なし）
    H->>S: rule確定を承認（ruleアクション）
    S->>S: rule.md 新規作成（spec.mdには触れない）
```

読み取り方:
- `K`（kpt-agent役）は**毎ラウンド必ず呼ばれる**（KPTが記入されるたび）。
- `S`（spec-writer役）は**人間の明示的な承認があったときだけ**、かつ**承認されたトラック（spec/rule）のファイルだけ**を生成・更新する。
- spec確定後もRound3のように`K`と`P`の呼び出しは続き、`S`は「rule確定」など別トラックの承認があったときに再び呼ばれる。

---

## トラック別収束状態（状態遷移図）

`対象=spec`と`対象=rule`は独立したトラックとして収束・確定する。1つのfeature内で両トラックが別々の状態遷移を持つことを示す。

```mermaid
stateDiagram-v2
    [*] --> 継続プロトタイピング
    継続プロトタイピング --> 収束傾向: 収束度OK & 未回答質問なし & Keep比率良好（当該ラウンド）
    収束傾向 --> 継続プロトタイピング: 新たなProblemの再発 or 新規曖昧点発生
    収束傾向 --> 確定済み: 人間が承認（spec-writer実行）
    確定済み --> 継続プロトタイピング: 確定後の新ラウンドでProblemが再発（spec.md/rule.mdは変更せず次の収束を待つ）
    継続プロトタイピング --> 確定済み: 確定後さらにProblemが解消し再承認（change-history-loggingで差分追記）
```

specトラックとruleトラックはそれぞれ独立にこの状態機械を持つため、「specは確定済み、ruleは継続プロトタイピング」という組み合わせが常態として起こり得る。

---

## 複数ラウンド・複数featureを扱うときの運用

### ラウンド番号とファイルの対応関係

- ラウンド番号は `docs/specs/<feature-name>/kpt-log/` 内の `roundN.csv` の連番で決まる（feature-nameごとに独立したカウンタ）。詳細はAGENTS.md「KPTの取り込みとラウンド番号」を参照。
- 1ラウンド = 1回のKPT記入 → 1回の`kpt-agent`役呼び出し、が基本単位。
- `kpt-summary.md`は「KPT一覧」（論点ごとの最新スナップショット、上書き）と「ラウンド履歴」（ラウンドごとの観測、追記専用）の2つを持つ。ラウンドを何度回しても**過去のラウンド履歴行は消えない**。

### ケースA: 同一featureでプロトタイピングを繰り返す（最も基本的な反復）

1. 人間がプロトタイプを試し、`roundN.csv`を作成・コミット（または貼り付け）。
2. `kpt-agent`役を**同じfeature-name**で起動する。
3. 曖昧点があれば質問への回答を待ってから、改善提案とループ判定を得る。
4. 「継続プロトタイピング」なら、改善提案を入力として`prototype-agent`役を起動しモックを更新する。
5. 1に戻り、`round(N+1).csv`として次のラウンドを記入する。

呼び出すagentは常に「prototype-agent → kpt-agent」の順で、feature-nameは変えない。

### ケースB: 新機能を追加する（新feature切り出し）

既存featureのKPTの中に、既存`request.md`のスコープを超える要望が混ざっていた場合の流れ。

1. `kpt-agent`役が集約時に、その論点を「新feature候補」と判別し、`kpt-summary.md`末尾の「次のアクション」に論点ID・想定feature-name案・根拠を提示する（自動では何も作成しない）。
2. 人間が新feature切り出しを承認し、**新しいfeature-name**を決めて要望Issue（または同等の入力）を起票する。
3. `prototype-agent`役を**新しいfeature-name**で起動する。既存featureとは完全に独立した`docs/specs/<new-feature-name>/`が作られ、ラウンド番号もRound1から独立して始まる。
4. 既存feature側のループ（ケースA）はそのまま並行して続く。既存featureの`round(N).csv`と新featureの`round(1).csv`は別ディレクトリ・別カウンタなので混ざらない。

以降、2つのfeatureは完全に独立したサイクルとして、それぞれ別々にラウンドを重ねる。

### ケースC: 仕様化（spec/rule確定）後もKPTを継続する

1. あるfeatureのspecトラックが収束し、人間が承認して`spec-writer`役（specアクション）が`spec.md`を生成する。
2. その後もプロトタイピングとKPT記入は止まらない。`round(N+1).csv`を作成し、`kpt-agent`役を同じfeature-nameで起動する（ケースAと同じ手順）。
3. ruleトラックが別ラウンドで収束すれば、人間が承認して`spec-writer`役（ruleアクション）が`rule.md`を生成する。`spec.md`には触れない。
4. **確定後のfeatureでProblemが再発した場合**（例: 一度確定した仕様に新たな課題が見つかった）も、`kpt-summary.md`のラウンド履歴に新しい観測が追記されるだけで、既存の`spec.md`/`rule.md`はすぐには変更しない。再度そのトラックが収束し、人間が承認した時点で`spec-writer`役を**再起動**し、`change-history-logging`スキルに従って**旧内容を消さずに変更差分を追記**する。

### ケースD: 新機能追加後に、以前の機能を修正する（並行運用時の切り替え）

複数のfeatureが並行して存在する状態で、直近作業していたfeature以外を修正するケース。呼び出しの起点は常に「どのfeature-nameを対象にするか」を明示することです。

1. feature-nameを**必ず指定**して`prototype-agent`役 / `kpt-agent`役を起動する。feature-nameはそのfeatureの`docs/specs/<feature-name>/`・`kpt-log/roundN.csv`・`kpt-summary.md`のみを読み書きする一意のキーであり、ディレクトリを跨いだ状態は共有されない。
2. 対象feature側で新しい`roundN.csv`（そのfeatureのカウンタの続き）を作成する。他featureのラウンド番号には影響しない。
3. `kpt-agent`役は指定されたfeature-nameの`kpt-summary.md`のみを更新する。指定を誤ると別featureのディレクトリを誤って読み書きするため、起動のたびにfeature-nameを確認する。
4. 対象featureが仕様確定済みであれば、ケースCの「確定後のfeatureでProblemが再発した場合」に従う（新規に丸ごと書き直すのではなく差分追記）。

**具体例（時系列）:**

| 時点 | 対象feature-name | 呼び出すagent役 | 操作 |
|---|---|---|---|
| 1 | `color-theme` | prototype-agent | 要望Issueからモック初版生成 |
| 2 | `color-theme` | kpt-agent | round1.csv集約 → 継続プロトタイピング |
| 3 | `color-theme` | prototype-agent → kpt-agent | round2.csv集約 → spec仕様化承認 → spec.md確定 |
| 4 | `color-theme` | spec-writer | specアクションでspec.md生成 |
| 5 | `export-feature`（新規） | prototype-agent | 新feature切り出し承認後、round1から独立して開始 |
| 6 | `export-feature` | kpt-agent | round1.csv集約 → 継続プロトタイピング |
| 7 | `color-theme`（以前の機能に戻る） | prototype-agent → kpt-agent | round3.csv集約（specは確定済みのため維持、rule側で新たなProblemを検知） |
| 8 | `color-theme` | spec-writer（ruleアクション） | rule.md新規生成（spec.mdは変更なし） |
| 9 | `color-theme` | prototype-agent → kpt-agent | round4.csv集約 → 確定済みrule.mdに対する新たなProblemを検知 |
| 10 | `color-theme` | spec-writer（ruleアクション・再確定） | change-history-loggingでrule.mdに差分追記（旧内容は削除しない） |

このように、**feature-nameが分岐点であり、ラウンド番号・生ログ・集約結果はすべてfeature-nameごとに独立**しています。同時に複数featureを運用していても、agentの呼び出し手順そのもの（ケースA〜C）は変わりません。

