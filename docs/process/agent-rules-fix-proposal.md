# 修正案: kpt-agent運用ルールの見直し

## 対象課題
1. kpt-agentはプロトタイプを絶対に実装しない
2. spec化はroundの最後、spec化したら本開発へハンドオフ
3. KPTは機能横断のため、機能のフォルダには入れない
4. rule.mdは短文で意図は記載しない
5. kptのログはワイド型のcsv形式

## 修正方針

### 1. kpt-agentはプロトタイプを絶対に実装しない
- `.github/agents/kpt-agent.md` の禁止事項に、モック本体・`docs/design.md`・`rule.md`・`prototype-notes.md`・`request.md` など prototype-agent の生成物を一切作成・編集しないことを明記する。
- handoffs（`update-mock-improve` / `extract-new-feature`）はあくまで提案の受け渡しであり、kpt-agent自身が `mocks/` 配下やモック関連ファイルに触れないことを明文化する。

### 2. spec化はroundの最後、spec化したら本開発へハンドオフ
- `AGENTS.md` の全体プロセス図を、「spec化はラウンドとは独立、いつでも可」から「spec化を行う場合は当該ラウンドの最終ステップとして行う」に変更する。
- `AGENTS.md` の厳守事項も同様に更新する。
- `.github/agents/spec-writer.md` の完了時コメント文言を、「spec.md生成後は本開発（実装）チームへハンドオフし、当該feature（スコープ）のプロトタイピング/KPTループはここで終了する」という趣旨に変更する。

### 3. KPTは機能横断のため、機能のフォルダには入れない
- `AGENTS.md` の「ディレクトリ規約」を変更し、`kpt-log/roundN.csv` と `kpt-summary.md` を `docs/specs/<feature-name>/` から外し、リポジトリ横断の置き場所（例: `docs/kpt/kpt-log/roundN.csv`、`docs/kpt/kpt-summary.md`）に移動する。
- `.github/skills/kpt-parsing/SKILL.md` 内のパス参照を横断パスに置き換える。
- `.github/agents/kpt-agent.md`・`.github/ISSUE_TEMPLATE/02_kpt-feedback.yml` のパス例も新パスに更新する。
- `AGENTS.md` の責任分界表・正本表も新パスに合わせて更新する。

### 4. rule.mdは短文で意図は記載しない
- `.github/skills/design-rule-template/SKILL.md` のテンプレートと「記載の原則」を改訂し、各ルール項目は短い箇条書き（一文）のみとし、「なぜそうするか」という意図・理由の文章を書かないことを明記する。
- 「基本ルールからの逸脱・例外」の説明文言から「理由」を削除し、逸脱内容のみを短く記載する形式に変更する。
- 論点ID（例: `theme-2`）の付記は出典・トレーサビリティ目的として残し、意図説明とは区別することを明記する。
- `.github/agents/prototype-agent.md` の rule.md 更新手順に「短文・意図説明なし」の制約を追記する。

### 5. kptのログはワイド型のcsv形式
- `.github/skills/kpt-parsing/SKILL.md` の「KPT入力CSVの形式」を、現行の長形式（種別ごとに1行）からワイド形式（1行=1論点、列に `Keep`/`Problem`/`Try` とそれぞれの状態を持つ）に変更する。
- 入力バリデーション章のヘッダ検証・値検証をワイド形式の列構成に合わせて更新する。
- 処理章のStep 1〜3（種別ごとの行を論点へ束ねるロジック）を簡略化し、人間側が既に論点単位で入力するため、agentは列マッピング・正規化・論点ID付与に専念する形に更新する。
- `.github/ISSUE_TEMPLATE/02_kpt-feedback.yml` のCSVカラム説明文言を新しいワイド形式の列構成に更新する。
- `kpt-summary.md` の「入力状態」列の定義説明を、状態がKeep/Problem/Try別の列で直接入力される新形式に合わせて更新する。

## 横断的な整合作業
- 上記変更後、`AGENTS.md` の詳細ルール正本表・ディレクトリ規約・責任分界表の記述に矛盾がないか確認する。
- 各ドキュメントのパス例・列名例が一貫しているか目視レビューする。
