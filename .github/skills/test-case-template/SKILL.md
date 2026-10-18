---
name: test-case-template
description: spec.md の受け入れ基準から REQ 単位でテストケース（TC）を導出し test-cases.md に記録する手順。test-designer が使う。
---

# skill: test-case-template

1つのREQを入力として、そのACからテストケースを導出する手順。記載内容は[test-cases.md](templates/test-cases.md)が規定する。

## 手順
1. 入力の対象REQを確認する。複数REQを同時に扱わない。
2. `spec.md`の当該REQの振る舞い・契約・異常系・ACを読む。ACが無い、または検証不能な場合、あるいは契約ID・異常IDのうちACから参照されないものがある場合は、TCを作成せず、`spec-writer`への差し戻しとして報告する。
3. 全ACについて、1つ以上のTCを導出する。異常系のAC（契約違反・失敗）も省略せず、必ずテストケース化する。異常系のTCは、期待結果に振る舞いと失敗後の状態（不変条件が保たれていること）を含める。
4. TCのレベルを決める。画面を介さずロジックで検証できるものは`unit`（Vitest）、画面操作を伴うものは`e2e`（Playwright）。
5. [test-cases.md](templates/test-cases.md)に従い`docs/specs/<feature-name>/test-cases.md`へ追記する。TC-IDは feature内で単調増加の連番で採番し、再利用しない。
6. 既存TCに影響する更新は、該当TCの状態を`更新`にして内容を書き換える。ACの廃止に伴うTCは状態を`廃止`にし、行は削除しない。
7. `traceability-matrix`に従い、`traceability.md`のUS・REQ・AC・TC列を更新する。
8. 当該REQの全ACにTCがあることを確認し、追加・更新したTC-IDを報告する。
