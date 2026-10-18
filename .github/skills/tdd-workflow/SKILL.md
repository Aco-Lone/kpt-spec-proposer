---
name: tdd-workflow
description: REQ単位でテストケース作成からTDD実装までを進める手順（オーケストレーションとRed・Green・Refactor）。tdd-orchestrator と prototype-agent が使う。
---

# skill: tdd-workflow

TDDは要件（REQ）ごとに進める。テストケース作成は`test-designer`、実装は`prototype-agent`がサブエージェントとして担当する。

## 手順（tdd-orchestrator）
1. 前提を確認する。満たさなければ中断して報告する。
   - `requirements.md`・`spec.md`が存在する。
   - `docs/knowledge/design/tokens/`に`design-token`が存在する。
   - `clarification-log.md`に`spec`・`global`の未回答質問がない。
   - 対象REQの全ACが`spec.md`に存在する。
2. `requirements.md`の記載順に、状態が`有効`のREQを処理対象とする。起動時に対象REQの指定があればそれに限定する。
3. 各REQについて次を順に行う。
   1. `test-designer`を呼び出し、当該REQのTC作成と`traceability.md`の更新を依頼する。
   2. `prototype-agent`を呼び出し、当該REQのTCに対するTDD実装を依頼する。
   3. `traceability-matrix`の検証手順で完了を確認する。
   4. 全テスト（当該REQのTCに加え既存テスト）の実行結果が緑であることを確認する。
   5. [req-completion-report.md](templates/req-completion-report.md)に従い完了報告を出し、人間の確認を待たず次のREQへ進む。
4. 検証に不備があれば、担当エージェントへ差し戻して修正させる。解消できない場合は当該REQで停止し、理由を報告する。後続REQには進まない。
5. 全REQの完了後、REQごとの完了報告を集約して最終報告とする。
6. サブエージェントの成果物（`test-cases.md`・テストコード・実装）は、オーケストレーター自身は編集しない。

## 手順（prototype-agent）
1. 入力のREQとそのTCを読む。`spec.md`のACと`docs/knowledge/`の`design-token`・`design-rule`に準拠する。
2. TCごとに次を繰り返す。
   1. **Red**: TCを検証するテストを先に書く。テスト名は`[TC-<連番>]`で始める。`unit`はVitest、`e2e`はPlaywrightで書く。実行して失敗することを確認する。
   2. **Green**: テストを通す最小限の実装を書く。実行して成功することを確認する。
   3. **Refactor**: 緑を保ったまま整理する。
3. TCの期待値を変更しない。TCやACの誤り・矛盾に気づいたら実装を止め、該当TC-IDと内容を呼び出し元へ返す。
4. `mock-scaffolding`の簡略化ルールに従う。簡略化してもTCは満たす。
5. `traceability-matrix`に従い、`traceability.md`のテストコード・実装状況列を更新する。
6. `mock-scaffolding`のテンプレートに従い、`prototype-notes.md`へ当該REQの実装内容を追記する。
7. 実施結果（TC-ID・テスト結果・変更ファイル・簡略化した点）を呼び出し元へ返す。
