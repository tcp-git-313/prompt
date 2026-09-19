# W2A-RECUT — Rebuild W2A on the Authoritative Wave2 Base

- 抬頭：POST WAVE2 FRONTEND ACCESS RECUT OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：小型
- Authoritative Base：b3cc93de970f4c7562eff3a4d5e17614462ae8bb
- Original W2A commits：33034c18 + 0030b44243e301fd89325d187264ab21df12929a

---

你現在是：

POST WAVE2 FRONTEND ACCESS RECUT OWNER

這不是重新設計 W2A。

目標是把已完成的 W2A 兩段實作，重新建立在唯一 authoritative Wave2 base 上，產生可安全 Integration 的 base-relative lineage。

## KNOWN PROBLEM

W2INT 在 cherry-pick：

0030b44243e301fd89325d187264ab21df12929a

時衝突。

原因：

0030b44 的 parent 是：

33034c18

而 W2INT base：

b3cc93de970f4c7562eff3a4d5e17614462ae8bb

沒有包含 33034c18。

33034c18 是 W2A 第一段：

fix(post): make studio state informational in web access

0030b44 是第二段：

entitlement/backend_unavailable correction + verification

因此 0030b44 不能單獨當 W2A candidate。

## AUTHORITATIVE BASE

Exact base：

b3cc93de970f4c7562eff3a4d5e17614462ae8bb

不得使用 movable branch 猜 base。

## OLD CONFLICTED INTEGRATION

目前：

F:\00-Ticenpi-SaaS\TicenpiPost-wave2-integration

處於 unresolved cherry-pick conflict。

它是 evidence。

禁止修改它。

不要：

- cherry-pick --abort
- reset
- restore
- clean
- checkout
- resolve conflict

本 task 使用全新 worktree。

## CREATE FRESH RECUT WORKTREE

建立：

branch:
repair/post-frontend-access-state-recut

worktree:
F:\00-Ticenpi-SaaS\TicenpiPost-w2-frontend-recut

直接從 exact base：

b3cc93de970f4c7562eff3a4d5e17614462ae8bb

建立。

確認：

HEAD == exact base
git status --porcelain == empty

否則 STOP。

## STEP 1 — VERIFY ORIGINAL COMMIT OWNERSHIP

只讀確認：

git show --name-only 33034c18
git show --name-only 0030b44243e301fd89325d187264ab21df12929a

兩個 commits 必須只涉及 W2A ownership：

- frontend/src/components/SessionGate.tsx
- dedicated SessionGate test files

如果發現 backend / deploy / extension / docker / AccountStatusList / session.ts：

STOP。

## STEP 2 — VERIFY LINEAGE

確認：

33034c18 的 parent 是否能 cleanly relate to authoritative base。

不要假設。

記錄：

ORIGINAL_W2A_FIRST_PARENT=
ORIGINAL_W2A_SECOND_PARENT=

## STEP 3 — APPLY W2A IN CORRECT ORDER

從 fresh authoritative base：

先 cherry-pick：

33034c18

再 cherry-pick：

0030b44243e301fd89325d187264ab21df12929a

任何 conflict：

立即 STOP。

禁止：

- ours
- theirs
- manual resolution
- rebase
- reset

## STEP 4 — VERIFY FINAL BEHAVIOR

必須符合：

signed_out → BLOCK
no_tenant → BLOCK
not_entitled → BLOCK
real auth failure → BLOCK

studio_offline → ALLOW WEB
account_mismatch → ALLOW WEB

backend_unavailable → distinct service unavailable state

unknown reason → fail-safe block

Google OAuth / JWT / membership / entitlement architecture不變。

Launcher / Studio不成為 Web shell prerequisite。

## STEP 5 — TEST

至少：

- targeted SessionGate tests
- full frontend tests
- tsc --noEmit
- next build

不得改 production source來迎合 test，除非 cherry-picked commits本身已包含。

## STEP 6 — FINAL CLEANNESS

git status --porcelain 必須 empty。

不要 squash。

保留兩個 atomic commits。

不要 push。

## REQUIRED OUTPUT

```
TASK=W2A_RECUT

AUTHORITATIVE_BASE=b3cc93de970f4c7562eff3a4d5e17614462ae8bb

BRANCH=
WORKTREE=

ORIGINAL_W2A_FIRST=33034c18
ORIGINAL_W2A_SECOND=0030b44243e301fd89325d187264ab21df12929a

ORIGINAL_W2A_FIRST_PARENT=
ORIGINAL_W2A_SECOND_PARENT=

OWNERSHIP_AUDIT=
PASS/FAIL

CHERRY_PICK_FIRST=
PASS/FAIL

CHERRY_PICK_SECOND=
PASS/FAIL

RECUT_FIRST_COMMIT=
RECUT_SECOND_COMMIT=
RECUT_HEAD=

FILES_CHANGED=

ACCESS_GATE_MATRIX=
PASS/FAIL

TESTS=
TYPECHECK=
BUILD=

OLD_CONFLICTED_INTEGRATION_TOUCHED=
NO

FINAL_WORKTREE_CLEAN=
YES/NO

READY_FOR_W2INT_RETRY=
YES/NO
```
