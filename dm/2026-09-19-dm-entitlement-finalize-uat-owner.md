# DM ENTITLEMENT FINALIZE & UAT OWNER

## Mission
完成 Ticenpi DM Backend Entitlement 修復的 Git 收尾與驗證準備。

目前已知功能面結果：
- BACKEND_ENTITLEMENT_ENFORCEMENT = PASS
- PROTECTED_ENDPOINTS_COVERED = designs (CRUD), variants (POST/GET), scrape (POST), ai-draw (POST), images (GET/POST/DELETE), removebg (POST)
- VALID_ENTITLED_USER = PASS
- VALID_NO_ENTITLEMENT = DENIED
- EXPIRED_SUBSCRIPTION = DENIED
- WRONG_PRODUCT = DENIED
- INVALID_JWT = DENIED
- TRIAL_BACKEND_BYPASS = NO
- TESTS = PASS
- PRODUCTION_CHANGED = NO

目前尚未收尾：
- FIX_COMMIT = PENDING
- working tree 仍有 5 個 router 修改 + untracked test files
- 因此 WORKING_TREE_CLEAN 不能判 YES

## Goal
1. 檢查並整理本次 entitlement 修復變更。
2. 把必要 source + tests 一起 commit。
3. commit 後重跑相關測試。
4. 確認 git status --porcelain 為空。
5. 準備後續 Google OAuth 真人驗證矩陣。
6. 本輪禁止 Production deploy。

## Step 1 — Diff Audit
先查看目前 git diff / git status。

確認：
- 只有本次 entitlement 修復相關 source/tests
- 沒有 unrelated changes
- untracked tests 是本次必要驗證資產
- 沒有 secret、token、password、JWT、Production credential

若發現 unrelated changes：
HARD STOP，不要混入同一 commit。

## Step 2 — Tests Before Commit
至少重跑：
- auth tests
- entitlement tests
- RLS / commercial-context 相關 tests
- 受影響 router tests

若任何新 regression：
停止，不 commit。

## Step 3 — Commit
若 diff 與 tests 都正確：

建立單一明確 commit，例如：
fix(dm): enforce commercial entitlement on protected APIs

commit 必須包含：
- entitlement enforcement source changes
- 必要 tests

禁止：
- squash 其他 unrelated history
- amend unrelated commit
- reset --hard
- git clean

## Step 4 — Post-Commit Verification
commit 後再次執行：
- targeted auth/entitlement tests
- git status --porcelain

要求：
WORKING_TREE_CLEAN = YES
只有在 git status --porcelain 完全空白時成立。

## Step 5 — Google OAuth UAT Plan
Google 帳號可以作為真人驗證帳號，而且優先使用真實 Google OAuth 流程。

但帳號角色必須分開：

### Account A — Entitled Google User
- Google OAuth 正常登入
- valid Membership
- active Subscription
- DM Entitlement enabled
- 非 dev/mock auth

預期：
DM protected API = ALLOW

### Account B — Non-entitled Google User
- Google OAuth 正常登入
- 必須是一般使用者
- 不得是 platform_admin / admin
- 不得有 Trial bypass
- 不得有 active DM entitlement

預期：
DM protected API = 403 DENY

### Account C — Expired / inactive subscription
若已有安全測試身份可用：
預期 403 DENY

### Account D — Wrong product entitlement
若已有安全測試身份可用：
預期 403 DENY

重要：
- ADMIN Google 帳號可以測「正常登入 / 功能可用」
- ADMIN 不可以作為 no-entitlement negative test 主帳號，因為可能有 admin bypass
- 不得為了測試去修改 Production customer data
- 不得人工新增/移除 Production Subscription 或 Entitlement

若目前只有 ADMIN Google 帳號：
回報：
NEGATIVE_UAT = BLOCKED_BY_TEST_IDENTITY

不要硬做假測試。

## Hard Rules
- 禁止 Production deploy
- 禁止修改 Production DB customer records
- 禁止人工開/關 Production Subscription / Entitlement
- 禁止輸出 Google 密碼、JWT、token、secret
- 禁止 git reset --hard
- 禁止 git clean
- 禁止把 unrelated changes 一起 commit

## Final Output
ENTITLEMENT_SOURCE_COMPLETE =
YES / NO

PRE_COMMIT_TESTS =
PASS / FAIL

FIX_COMMIT =
<sha>

POST_COMMIT_TESTS =
PASS / FAIL

WORKING_TREE_CLEAN =
YES / NO

GOOGLE_OAUTH_VALIDATION_READY =
YES / NO

ENTITLED_GOOGLE_ACCOUNT =
READY / MISSING

NON_ENTITLED_GOOGLE_ACCOUNT =
READY / MISSING

ADMIN_ACCOUNT_USABLE_FOR_POSITIVE_PATH =
YES / NO

ADMIN_ACCOUNT_USABLE_FOR_NEGATIVE_PATH =
NO

NEGATIVE_UAT =
READY / BLOCKED_BY_TEST_IDENTITY

PRODUCTION_CHANGED =
NO

READY_FOR_DEPLOYMENT_REVIEW =
YES / NO

完成後停止，不 Deploy Production。
