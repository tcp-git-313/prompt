# W2F — POST Wave2 Regression Test Pack Port

- 抬頭：POST WAVE2 REGRESSION TEST PORT OWNER
- 模型：MiMo-V2.5
- 預計時間：5–10 分鐘
- 任務類型：Port + Revalidation
- Authoritative Base：`b3cc93de970f4c7562eff3a4d5e17614462ae8bb`
- Candidate Commit：`40ae824`

---

你現在是：POST WAVE2 REGRESSION TEST PORT OWNER

不是重新實作。

## BASE

Exact base：`b3cc93de970f4c7562eff3a4d5e17614462ae8bb`

建立：
- branch: `repair/post-wave2-tests-v2`
- worktree: `F:\00-Ticenpi-SaaS\TicenpiPost-w2-tests`

## CANDIDATE

Exact commit：`40ae824`

忽略它目前掛在哪個 branch ref，只使用 commit object。

## TASK

從 fresh Wave2 base 執行：

`git cherry-pick 40ae824`

若 conflict：STOP。

## OWNERSHIP

只能新增 Wave2 test files。任何 production source modified：STOP。

## RUN TESTS

執行現在能跑的 Wave2 tests。

若 Publish safety tests 因 W2B 尚未整合而失敗，標 `EXPECTED_PENDING_W2B`。

若 Frontend SessionGate tests 因 W2A 尚未整合而失敗，標 `EXPECTED_PENDING_W2A`。

不要修改 test 去配合目前舊 source。

## IMPORTANT

判斷 failure 時區分真正 test bug vs 等待 W2A/W2B implementation。

## REQUIRED OUTPUT

```
TASK=W2F_REGRESSION_TEST_PORT
BASE_COMMIT=
SOURCE_COMMIT=40ae824
BRANCH=
WORKTREE=
COMMIT=
FILES_ADDED=
PRODUCTION_SOURCE_MODIFIED=NO
TESTS_PASSING_NOW=
EXPECTED_PENDING_W2A=
EXPECTED_PENDING_W2B=
OTHER_FAILURES=
CHERRY_PICK_CONFLICT=
READY_FOR_INTEGRATION=YES/NO
```
