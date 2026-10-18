---
name: web-mock-stack
description: Webモックの技術スタック（Vite + React + TypeScript）・テスト基盤（Vitest + Playwright）・配置・プレビュー手順。prototype-agent がWebモックを実装するときに mock-scaffolding・tdd-workflow と併せて使う。
---

# skill: web-mock-stack

Webでモックを作る際の技術スタックと環境ルール。簡略化ルールは `mock-scaffolding` に従い、本スキルはその「Web版の実装手段」を定義する。

## 技術方針
- ビルドツールは **Vite**、フレームワークは **React + TypeScript**（`react-ts` テンプレート）。
- テストは `unit` レベルを **Vitest + React Testing Library**、`e2e` レベルを **Playwright** で書く。
- 静的資産＋クライアント側の状態のみで完結させる（サーバサイド・DB・外部APIの本接続は追加しない）。
- 依存追加は最小限に留め、追加した理由を `prototype-notes.md` に記録する。

## 配置ディレクトリ
- モック一式は `mocks/<feature-name>/`。`<feature-name>` は `docs/specs/<feature-name>/` と一致させる。
- `unit` テストは `mocks/<feature-name>/src/**/*.test.tsx`、`e2e` テストは `mocks/<feature-name>/e2e/*.spec.ts`。テスト名は `[TC-<連番>]` で始める。

## 初回セットアップ（`mocks/<feature-name>/` が無い場合）
```bash
npm create vite@latest mocks/<feature-name> -- --template react-ts
cd mocks/<feature-name> && npm install
npm install -D vitest jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event @playwright/test
npx playwright install chromium
```
その後 `mock-scaffolding` の原則に従い、ダミー状態・ダミーデータで画面と操作フローを実装する。

## 更新手順（既存モックがある場合）
1. `mocks/<feature-name>/` の既存コードを読み、画面構成と状態管理を把握する。
2. 既存の `docs/knowledge/design/rules/<feature-name>/`（OKF `design-rule`）があれば読み、変更はそのルールに準拠する。
3. `tdd-workflow` に従い、対象REQのTCをテストとして先に追加し、画面・操作フローを追加・変更する。
4. 既存のテストが緑のままであることを確認する。

## テスト実行
```bash
cd mocks/<feature-name> && npx vitest run
cd mocks/<feature-name> && npx playwright test
```

## プレビュー
```bash
cd mocks/<feature-name> && npm run dev
```
表示された `http://localhost:5173/`（既定ポート）で操作感を確認する。

## prototype-notes.md への追記
追加・変更した依存パッケージと追加理由、起動コマンドと確認したポートを、`mock-scaffolding` の `prototype-notes.md` テンプレートに従い記載する。
