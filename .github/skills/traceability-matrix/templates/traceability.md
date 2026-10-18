# Traceability: <feature-name>

<!--
必須。1TC=1行。TCが無いACは TC列を `未紐付け` として1行残す。
- US / REQ / AC / TC: それぞれのID。他ファイルのIDと完全一致させる
- テストコード: `<ファイルパス>::<テスト名>`。テスト名は `[TC-<連番>]` で始める
- 実装状況: `未着手` / `Red` / `Green`
- 廃止した要件・AC・TC の行は削除せず、実装状況に `廃止` と書く
-->
| US | REQ | AC | TC | テストコード | 実装状況 |
|---|---|---|---|---|---|
| US-001 | REQ-001 | AC-001-1 | TC-001 | mocks/<feature-name>/src/xxx.test.tsx::[TC-001] <テスト名> | 未着手 |
