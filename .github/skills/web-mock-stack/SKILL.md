---
name: web-mock-stack
description: Webモックの技術スタック（Vite + React + TypeScript）・配置・プレビュー手順。prototype-agent がWebモックを実装するときに mock-scaffolding と併せて使う。
---

# skill: web-mock-stack

Webでモックを作る際の技術スタックと環境ルール。簡略化ルールは `mock-scaffolding` に従い、本スキルはその「Web版の実装手段」を定義する。

## 技術方針
- ビルドツールは **Vite**、フレームワークは **React + TypeScript**（`react-ts` テンプレート）。
- 静的資産＋クライアント側の状態のみで完結させる（サーバサイド・DB・外部APIの本接続は追加しない）。
- 依存追加は最小限に留め、追加した理由を `prototype-notes.md` に記録する。

## 配置ディレクトリ
- モック一式は `mocks/<feature-name>/`。`<feature-name>` は `docs/specs/<feature-name>/` と一致させる。

## 初回セットアップ（`mocks/<feature-name>/` が無い場合）
```bash
npm create vite@latest mocks/<feature-name> -- --template react-ts
cd mocks/<feature-name> && npm install
```
その後 `mock-scaffolding` の原則に従い、ダミー状態・ダミーデータで画面と操作フローを実装する。

## 更新手順（既存モックがある場合）
1. `mocks/<feature-name>/` の既存コードを読み、画面構成と状態管理を把握する。
2. 既存の `docs/specs/<feature-name>/rule.md` があれば読み、変更はそのルールに準拠する。
3. `kpt-summary.md` の改善提案に沿って画面・操作フローを追加・変更する。
4. 既存の画面や操作を壊していないかプレビューで確認する。

## プレビュー
```bash
cd mocks/<feature-name> && npm run dev
```
表示された `http://localhost:5173/`（既定ポート）で操作感を確認する。

## prototype-notes.md への追記項目
`mock-scaffolding` の記載項目に加え、以下も追記する。
- 追加・変更した依存パッケージと追加理由（なければ「依存追加なし」と明記）
- 起動コマンドと確認したポート（既定と異なる場合のみ）
