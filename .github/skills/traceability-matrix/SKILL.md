---
name: traceability-matrix
description: US→REQ→AC→TC→テストコードの対応を traceability.md に記録・検証する手順。test-designer・prototype-agent・tdd-orchestrator が使う。
---

# skill: traceability-matrix

`docs/specs/<feature-name>/traceability.md`の更新と検証の手順。記載内容は[traceability.md](templates/traceability.md)が規定する。

## 更新担当
- `test-designer`: US・REQ・AC・TC列。
- `prototype-agent`: テストコード・実装状況列。
- 他のエージェントは編集しない。

## 手順（test-designer）
1. `requirements.md`から対象REQの元USを、`spec.md`からACを、`test-cases.md`からTCを読む。
2. 1TC=1行で、US・REQ・AC・TCを記入する。
3. TCが無いACは行を作り、TC列に`未紐付け`と記入する。

## 手順（prototype-agent）
1. テストコードを書いたら、該当TC行のテストコード列に`<ファイルパス>::<テスト名>`を記入する。テスト名は`[TC-<連番>]`で始める。
2. 実装状況を`Red`（テスト作成済み・失敗）→`Green`（成功）と更新する。

## 検証手順（tdd-orchestrator）
1. 対象REQの全行について、次を確認する。
   - TC列に`未紐付け`が無い。
   - テストコード列が空でない。
   - 実装状況が`Green`である。
2. テストコード列のファイルが実在し、テスト名に記載のTC-IDが含まれることを確認する。
3. 不備があれば、該当行を挙げて担当エージェントへ差し戻す。
4. 要件・ACが廃止された行は、`廃止`と明記して残す。
