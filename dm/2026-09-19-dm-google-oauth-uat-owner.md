# DM GOOGLE OAUTH UAT OWNER

## Mission
建立並驗證 Ticenpi DM 商用授權的 Google OAuth 真人 UAT 身份矩陣。

目前已知：
- BACKEND_ENTITLEMENT_ENFORCEMENT = PASS
- FIX_COMMIT = 86f2c97
- PRE_COMMIT_TESTS = PASS
- POST_COMMIT_TESTS = PASS
- PRODUCTION_CHANGED = NO
- ADMIN_ACCOUNT_USABLE_FOR_POSITIVE_PATH = YES
- ADMIN_ACCOUNT_USABLE_FOR_NEGATIVE_PATH = NO
- ENTITLED_GOOGLE_ACCOUNT = MISSING
- NON_ENTITLED_GOOGLE_ACCOUNT = MISSING
- NEGATIVE_UAT = BLOCKED_BY_TEST_IDENTITY

本任務只處理 Google OAuth UAT 身份與授權驗證，不重新修改 entitlement source。

## Goal
建立或確認至少兩種 Google OAuth 真人身份：

### Account A — Positive Path
- Google OAuth 正常登入
- 可使用既有 ADMIN Google 帳號
- 不得使用 mock/dev auth
- 預期 protected DM API = ALLOW

### Account B — Negative Path
- Google OAuth 正常登入
- 必須是一般使用者
- 不得是 platform_admin / admin
- 不得有 active DM Subscription
- 不得有 DM Entitlement
- 不得有 Trial bypass
- 預期 protected DM API = 403 DENY

若已有 Account C / D 可安全取得，再驗：
- Expired / inactive subscription → 403 DENY
- Wrong product entitlement → 403 DENY

## Important Rule
如果 Google OAuth 登入本身會自動建立下列任一項：
- Membership
- Trial Subscription
- DM Entitlement
- 其他可直接解鎖 DM 的權限

立即停止修改資料並回報。

這是商用安全行為，不能被當成測試便利流程略過。

不得為了讓測試通過而：
- 人工新增 Production Subscription
- 人工新增 Production Entitlement
- 把一般帳號升成 admin
- 使用 dev bypass
- 使用 mock auth
- 直接改 Production customer data

## Step 1 — 找既有安全身份
先只讀調查：
- 既有 Google OAuth 測試帳號
- Secret Store / 安全環境變數
- Staging 測試身份
- Production 既有 ADMIN 帳號

禁止輸出：
- Google password
- JWT
- refresh token
- access token
- secret

只回報帳號角色與可用性。

## Step 2 — Positive Path
使用 Account A：

Google OAuth
→ 登入 DM
→ 確認 session
→ 呼叫至少一個 protected DM API

記錄：
AUTH = PASS / FAIL
ROLE = ADMIN / USER
ENTITLEMENT_PATH = ALLOW / DENY
HTTP_STATUS = ...

ADMIN 帳號只能證明 positive path。
不能拿來證明 negative path。

## Step 3 — Negative Path
使用 Account B：

Google OAuth
→ 登入成功
→ 確認沒有 admin
→ 確認沒有 active DM entitlement
→ 呼叫 protected DM API

至少測一個：
- /api/designs
- /api/scrape
- /api/ai-draw
- images / removebg 等受保護 endpoint

預期：
403 DENY

若 Account B 登入後被自動授權：
停止並回報：

AUTO_PROVISIONING_SECURITY_FINDING = YES

並說明自動建立了什麼。

## Step 4 — Optional Negative Matrix
若已有安全身份，不修改 Production 資料前提下可驗：

### Expired Subscription
預期：403 DENY

### Wrong Product Entitlement
預期：403 DENY

沒有安全身份就標 NOT_TESTED，不要造資料。

## Step 5 — Evidence
保存：
- timestamp
- endpoint
- HTTP status
- role type
- membership/subscription/entitlement presence summary

禁止保存：
- JWT/token
- password
- secret value

## Hard Rules
- 禁止 Production deploy
- 禁止修改 entitlement source
- 禁止修改 Production customer records
- 禁止人工新增/移除 Production Subscription / Entitlement
- 禁止把一般帳號升成 admin
- 禁止 mock/dev bypass
- 禁止輸出 credential/token/secret
- 禁止 git reset --hard
- 禁止 git clean

## Final Output
GOOGLE_OAUTH_POSITIVE_ACCOUNT =
READY / MISSING

GOOGLE_OAUTH_NEGATIVE_ACCOUNT =
READY / MISSING

ADMIN_POSITIVE_PATH =
PASS / FAIL / NOT_TESTED

NON_ENTITLED_GOOGLE_LOGIN =
PASS / FAIL / NOT_TESTED

NON_ENTITLED_DM_ACCESS =
DENIED / ALLOWED / NOT_TESTED

NEGATIVE_HTTP_STATUS =
...

AUTO_PROVISIONING_SECURITY_FINDING =
YES / NO / UNPROVEN

AUTO_CREATED_MEMBERSHIP =
YES / NO / UNPROVEN

AUTO_CREATED_SUBSCRIPTION =
YES / NO / UNPROVEN

AUTO_CREATED_DM_ENTITLEMENT =
YES / NO / UNPROVEN

EXPIRED_SUBSCRIPTION =
DENIED / ALLOWED / NOT_TESTED

WRONG_PRODUCT =
DENIED / ALLOWED / NOT_TESTED

PRODUCTION_CHANGED =
NO

NEGATIVE_UAT =
PASS / FAIL / BLOCKED_BY_TEST_IDENTITY

READY_FOR_DEPLOYMENT_REVIEW =
YES / NO

完成後停止，不 Deploy Production。
