# W2INT-CHECK — Reconcile Integration HEAD / Typecheck Evidence

- 抬頭：POST WAVE2 INTEGRATION INTEGRITY OWNER
- 模型：MiMo-V2.5
- 預計工程大小：小型
- 模式：STRICT READ ONLY

---

你現在是：

POST WAVE2 INTEGRATION INTEGRITY OWNER

這是一個非常小的 consistency check。

不要修改任何檔案。
不要 commit。
不要重新跑整套驗證。

## WHY

目前兩份報告對同一 HEAD：

e53162ff4a361b8a4608835bfc1b4e10672370d8

出現互相矛盾的證據。

較早報告：

- frontend/tests/wave2-access-regression.test.tsx:11
- mockReturnValue('idle')
- SessionStatus 不接受 'idle'
- tsc --noEmit FAIL
- wave2-access-regression = 11 tests

最新報告：

- CURRENT_HEAD 仍是 e53162ff4a361b8a4608835bfc1b4e10672370d8
- typecheck PASS
- wave2-access-regression = 12 tests
- worktree clean
- no source edits / no commits

同一 Git commit內容不可自行改變，因此必須先裁決哪份 evidence 正確。

## TARGET

Worktree：

F:\00-Ticenpi-SaaS\TicenpiPost-wave2-integration-v2

Branch：

repair/post-v2-wave2-integration-v2

Expected HEAD：

e53162ff4a361b8a4608835bfc1b4e10672370d8

## HARD RULES

STRICT READ ONLY。

禁止：

- edit
- restore
- reset
- clean
- stash
- checkout mutation
- commit
- cherry-pick
- merge
- rebase
- npm install
- package mutation

## STEP 1 — EXACT GIT STATE

輸出：

git rev-parse HEAD
git status --porcelain
git diff -- frontend/tests/wave2-access-regression.test.tsx
git diff --cached -- frontend/tests/wave2-access-regression.test.tsx

確認有無 sparse checkout / assume-unchanged / skip-worktree：

git ls-files -v frontend/tests/wave2-access-regression.test.tsx

## STEP 2 — COMMIT CONTENT VS WORKTREE CONTENT

比較：

git show e53162ff4a361b8a4608835bfc1b4e10672370d8:frontend/tests/wave2-access-regression.test.tsx

與目前 working-tree檔。

只需回報：

- commit版本 line 11的 mock status value
- working tree版本 line 11的 mock status value
- test count rough count
- 是否逐位元組相同

不要大量貼檔案內容。

## STEP 3 — TYPE CONTRACT

確認：

frontend/src/lib/session.ts

SessionStatus 的實際 union。

輸出：

SESSION_STATUS_VALUES=

## STEP 4 — TARGETED TYPECHECK

只跑：

cd frontend
npx tsc --noEmit

記錄 exact exit code和第一個 error（若有）。

不要跑 full frontend test。
不要 build。

## STEP 5 — CLASSIFY

如果：

commit file = working tree
且 status value是合法 SessionStatus
且 typecheck PASS

則：

EARLIER_REPORT_STALE_OR_WRONG=YES

如果：

commit file含 idle
且 typecheck FAIL

則：

LATEST_REPORT_INCONSISTENT=YES

如果：

working tree與 commit不同但 git status仍 clean

調查：

- assume-unchanged
- skip-worktree
- sparse checkout
- generated overlay / junction / symlink

不要修。

## REQUIRED OUTPUT

```
TASK=W2INT_INTEGRITY_RECONCILE

BRANCH=
HEAD=
WORKTREE_CLEAN=

COMMIT_TEST_STATUS_VALUE=
WORKTREE_TEST_STATUS_VALUE=

COMMIT_AND_WORKTREE_IDENTICAL=
YES/NO

LS_FILES_FLAG=

SESSION_STATUS_VALUES=

TYPECHECK_EXIT=

TYPECHECK_FIRST_ERROR=

EARLIER_REPORT_STALE_OR_WRONG=
YES/NO/UNKNOWN

LATEST_REPORT_INCONSISTENT=
YES/NO/UNKNOWN

HIDDEN_WORKTREE_STATE=
YES/NO/UNKNOWN

MUTATION_PERFORMED=NO

SAFE_TO_MERGE_W2G=
YES/NO
```
