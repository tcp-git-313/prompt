# DM GOOGLE OAUTH IDENTITY EXECUTION OWNER

## Mission
不要再建立指南、模板、Checklist。直接執行 Ticenpi DM Google OAuth UAT 身份準備與 baseline 驗證。

前一輪只完成文件與程式碼分析，沒有完成身份矩陣，也沒有執行 Google OAuth 真人 baseline，因此該任務仍未完成。

目前已知：
- Entitlement source fix commit = 86f2c97
- Backend entitlement enforcement tests = PASS
- Production 尚未部署 86f2c97
- ADMIN Google 帳號可用於 positive-path baseline
- ADMIN 不可作為 no-entitlement negative account
- has_dm_access() 接受 product_code="dm" 且 status in ("active","trial")
- Production 不可修改 customer/subscription/entitlement 資料
- Staging Supabase project = jlsqjvehwblkeuycjoyj
- Public staging URL 優先使用 https://dm-staging.ticenpi.com
- 不要把 Windows localhost:9422 當成 staging 已可用證據；若需要 SSH port-forward，必須先證明 tunnel 真正存在且 listener 可達

## Goal
實際完成：
1. 找到可用 ADMIN Google 帳號，完成 Google OAuth positive baseline。
2. 找到可用一般 Google 帳號，完成 no-entitlement negative identity baseline。
3. 實際觀察 Google OAuth 登入前後是否自動建立 Membership / Trial / Subscription / DM Entitlement。
4. 輸出可供下一波部署後 UAT 直接使用的身份矩陣。
5. 若缺第二個 Google 帳號，清楚回報唯一 blocker，不要用寫文件代替執行。

## Hard Rule — No Documentation-Only Completion
本任務不能以「已建立 execution guide / evidence template / checklist」作為完成條件。

只有實際取得身份與執行 baseline 後，IDENTITY_MATRIX_READY 才能為 YES。

## Step 1 — Find Existing Safe Accounts
優先搜尋：
- 既有 Secret Store / secure env
- staging 測試身份
- 既有 ADMIN Google 帳號
- 既有一般 Google 測試帳號

只能回報：
- account role
- account availability
- environment

禁止輸出 email password、JWT、access token、refresh token、secret。

### Account A
既有 ADMIN Google 帳號即可。
用途：positive baseline。

### Account B
必須是一般 Google 帳號：
- 非 admin
- 非 platform_admin
- 不得預先有 active DM entitlement
- 不得預先有 active DM subscription
- 不得有 trial bypass

如果找不到 Account B：
不要建立 Production 客戶資料。
直接輸出：
NON_ENTITLED_GOOGLE_ACCOUNT = MISSING
BLOCKER = SECOND_GOOGLE_IDENTITY_REQUIRED

## Step 2 — Choose Test Environment Correctly
優先使用 public staging：
https://dm-staging.ticenpi.com

先證明：
- staging reachable
- environment = staging
- release identity 可讀
- Google OAuth callback 指向 staging
- staging auth 與 Production 隔離

如果 public staging 不支援目前需要的 OAuth 流程，再調查 SSH port-forward。

不得假設 http://localhost:9422 正在工作。
如果要使用 localhost:9422，必須實際證明：
- listener exists
- SSH tunnel active
- target is staging
- runtime-config environment = staging

否則禁止把 localhost:9422 當 staging endpoint。

## Step 3 — ADMIN Positive Baseline
用 Account A 實際走：
Google OAuth
→ callback
→ session established
→ DM UI reached

記錄：
- GOOGLE_OAUTH = PASS / FAIL
- role = ADMIN
- environment
- whether DM UI is reachable

本輪 Production 尚未部署 86f2c97，因此這一步只建立 auth/identity baseline，不得宣稱新 entitlement fix 已在 Production 驗證。

## Step 4 — Non-entitled Google Baseline
若 Account B 存在：

在登入前先以只讀方式記錄：
- membership presence
- subscription presence
- DM entitlement presence
- trial presence

再實際 Google OAuth 登入。

登入後再次只讀檢查：
- membership created?
- trial created?
- subscription created?
- DM entitlement created?

特別注意：
has_dm_access() 接受 status="trial"。
因此若登入自動建立 trial-status DM entitlement，這會直接具有 DM access，必須回報安全發現。

若登入後自動產生可解鎖 DM 的 entitlement：
AUTO_PROVISIONING_SECURITY_FINDING = YES

不要刪資料、不要人工修正，保留證據後停止。

## Step 5 — Do Not Run Final Fix Validation Yet
因 86f2c97 尚未部署到正式驗收環境，本輪不要把：
NON_ENTITLED_DM_ACCESS = DENIED
當作最終新修復 UAT 結論。

本輪只需建立：
- identity readiness
- OAuth readiness
- auto-provisioning baseline

最終 ALLOW/DENY 驗證留到整合 candidate 部署後。

## Hard Rules
- 禁止 Production deploy
- 禁止修改 Production customer/subscription/entitlement records
- 禁止人工新增/移除 Production entitlement
- 禁止把一般帳號升成 admin
- 禁止 mock/dev auth
- 禁止輸出 credential/token/secret
- 禁止只產文件就宣稱完成
- 禁止把 localhost:9422 未驗證 tunnel 當 staging
- 禁止 git reset --hard
- 禁止 git clean

## Final Output
ADMIN_GOOGLE_ACCOUNT =
READY / MISSING

NON_ENTITLED_GOOGLE_ACCOUNT =
READY / MISSING

STAGING_ENDPOINT_USED =
...

STAGING_ENDPOINT_VERIFIED =
YES / NO

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

TRIAL_STATUS_GRANTS_DM_ACCESS =
YES / NO / UNPROVEN

IDENTITY_MATRIX_READY =
YES / NO

FINAL_ENTITLEMENT_UAT_RUN =
NO

PRODUCTION_CHANGED =
NO

BLOCKER =
...

只有實際完成身份 baseline 才可回報 COMPLETED。
