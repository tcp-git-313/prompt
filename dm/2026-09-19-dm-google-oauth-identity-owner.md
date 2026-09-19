# DM GOOGLE OAUTH IDENTITY OWNER

## Mission
為 Ticenpi DM 商用授權 UAT 準備 Google OAuth 真人測試身份與 baseline。

這是 WAVE 1 平行任務。
不要把目前尚未部署到 Production 的 entitlement 修復 commit `86f2c97` 當成已在 Production 生效。

目前已知：
- Entitlement source fix = PASS
- FIX_COMMIT = 86f2c97
- Production 尚未部署這個 fix
- ADMIN Google 帳號可用於 positive path
- ADMIN 不可用於 no-entitlement negative path
- Negative UAT 目前 BLOCKED_BY_TEST_IDENTITY

## Goal
找出並確認：
1. Account A：既有 ADMIN Google 帳號，供後續 positive path。
2. Account B：一般 Google 帳號，非 admin、無 DM entitlement，供後續 negative path。
3. 是否存在 Google OAuth 自動建立 Membership / Trial / Subscription / DM Entitlement 的行為。
4. 為後續部署後 UAT 準備可重複使用的身份矩陣。

本輪不是最終 entitlement UAT。
不要因為目前 Production 還沒部署 86f2c97，就判定新修復 PASS/FAIL。

## Step 1 — Read-only Identity Discovery
優先從既有安全來源找：
- Secret Store
- 安全環境變數
- Staging 測試身份
- Production 既有 ADMIN 帳號
- 已存在的一般 Google 測試帳號

只回報角色與可用性。
禁止輸出 email 密碼、JWT、access token、refresh token、secret。

## Step 2 — Account A
確認既有 ADMIN Google 帳號：
- 可正常 Google OAuth
- 非 mock/dev auth
- 可作為後續 positive path 帳號

本輪可做登入確認，但不要把它拿來證明 negative entitlement path。

## Step 3 — Account B
尋找既有一般 Google 帳號：
- 非 platform_admin
- 非 admin
- 無 active DM entitlement
- 無 active DM subscription
- 無 trial bypass

若不存在：
回報 NON_ENTITLED_GOOGLE_ACCOUNT = MISSING
不要自行建立 Production 客戶或人工改權限。

## Step 4 — Auto-Provisioning Baseline
若已有安全的一般 Google 帳號可登入，觀察登入前後是否自動建立：
- Membership
- Trial Subscription
- Subscription
- DM Entitlement
- 其他可直接解鎖 DM 的權限

若登入會造成上述變化：
記錄為 SECURITY_FINDING。
不要自行修資料。

注意：
這一步是在建立 baseline，不是在驗證 86f2c97 的最終 enforcement。

## Hard Rules
- 禁止 Production deploy
- 禁止修改 entitlement source
- 禁止修改 Production customer data
- 禁止人工新增/移除 Subscription / Entitlement
- 禁止把一般帳號升成 admin
- 禁止 mock/dev bypass
- 禁止輸出 credential/token/secret
- 禁止 git reset --hard
- 禁止 git clean
- 不需要等待使用者再次確認，直接調查

## Final Output
ADMIN_GOOGLE_ACCOUNT =
READY / MISSING

NON_ENTITLED_GOOGLE_ACCOUNT =
READY / MISSING

ADMIN_GOOGLE_OAUTH =
PASS / FAIL / NOT_TESTED

NON_ENTITLED_GOOGLE_OAUTH =
PASS / FAIL / NOT_TESTED

AUTO_PROVISIONING_SECURITY_FINDING =
YES / NO / UNPROVEN

AUTO_CREATED_MEMBERSHIP =
YES / NO / UNPROVEN

AUTO_CREATED_TRIAL =
YES / NO / UNPROVEN

AUTO_CREATED_SUBSCRIPTION =
YES / NO / UNPROVEN

AUTO_CREATED_DM_ENTITLEMENT =
YES / NO / UNPROVEN

IDENTITY_MATRIX_READY =
YES / NO

FINAL_ENTITLEMENT_UAT_RUN =
NO

PRODUCTION_CHANGED =
NO

BLOCKER =
...

完成後停止。
