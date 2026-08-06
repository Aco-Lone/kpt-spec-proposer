---
name: web-mock-stack
description: Webモックの技術スタック（Vite + React + TypeScript）・配置・プレビュー手順。prototype-agent がWebモックを実装するときに mock-scaffolding と併せて使う。
---

# skill: web-mock-stack

Webでモックを作る際の技術スタックと環境ルール。UIの操作感を再現する簡略化ルールは `mock-scaffolding` スキルに従い、本スキルはその「Web版の実装手段」を定義する。

## 技術方針
- ビルドツールは **Vite**、フレームワークは **React + TypeScript**（`react-ts` テンプレート）を使う。
- モックは静的資産＋クライアント側の状態のみで完結させる。サーバサイド・DB・外部APIの本接続は追加しない（`mock-scaffolding` の原則を継承）。
- 依存追加は最小限に留める。UI操作感の再現に必要なものだけを入れ、理由を `prototype-notes.md` に記録する。

## 配置ディレクトリ
- モック一式はリポジトリ直下の `mocks/<feature-name>/` に配置する。`<feature-name>` は `docs/specs/<feature-name>/` と一致させる。
- 成果物ドキュメント（`request.md` / `prototype-notes.md` / `kpt-*`）は従来どおり `docs/specs/<feature-name>/` に置く。モックのコードとは配置先を分ける。

## 初回セットアップ手順（`mocks/<feature-name>/` が無い場合）
1. `mocks/<feature-name>/` を作成する。
2. Vite の React + TypeScript テンプレートで雛形を生成する。
   ```bash
   npm create vite@latest mocks/<feature-name> -- --template react-ts
   ```
3. 依存をインストールする。
   ```bash
   cd mocks/<feature-name>
   npm install
   ```
4. `mock-scaffolding` の原則に従い、ダミー状態・ダミーデータで画面と操作フローを実装する。

## 更新手順（既存モックがある場合）
1. `mocks/<feature-name>/` の既存コードを読み、現在の画面構成と状態管理を把握する。
2. 既存の `docs/specs/<feature-name>/rule.md`（機能独自のデザインルール）があれば読み、以降の変更はそのルールに準拠する。
3. 改善提案（`kpt-summary.md`）に沿って画面・操作フローを追加・変更する。
4. 既存の画面や操作を壊していないかをプレビューで確認する。

## プレビュー（動作確認）
- ローカル開発サーバを起動して確認する。
  ```bash
  cd mocks/<feature-name>
  npm run dev
  ```
- 起動後に表示される `http://localhost:5173/`（既定ポート）をブラウザで開いて操作感を確認する。

## 出力時の記載必須項目（prototype-notes.md）
`mock-scaffolding` の記載項目に加え、以下も追記する。
- 追加・変更した依存パッケージと、その追加理由（なければ「依存追加なし」と明記）
- 起動コマンドと確認したポート（既定と異なる場合のみ）
