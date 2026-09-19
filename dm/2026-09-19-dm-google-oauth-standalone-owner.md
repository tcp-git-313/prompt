# DM GOOGLE OAUTH STANDALONE OWNER

## Mission
你是一個全新、獨立 Session 的 Ticenpi DM Google OAuth UAT Owner。

不要假設你看得到前一個對話。
以下資訊就是完整任務上下文，請直接依此執行，不要再要求使用者重貼舊對話。

## Current Project State
Ticenpi DM 商用上線主線目前狀態：

- Production Runtime = PASS
- Current Production URL = https://dm.ticenpi.com
- Current Production Release = 20260919-110713
- Production current commit = ef2e668（86f2c97 尚未部署）
- Staging URL = https://dm-staging.ticenpi.com
- Staging Release = 20260918-222526
- Staging commit = 5e68034
- Staging Supabase project ref = jlsqjvehwblkeuycjoyj
- Entitlement backend source fix = PASS
- Entitlement fix commit = 86f2c97
- Production 尚未部署 86f2c97
- Release Gate engineering 已完成，另有 branches/commits 處理中
- 本任務禁止 Production deploy

## Fixed Google Test Identities

以下帳號角色已由使用者明確指定，不需要再次詢問：

### ADMIN
tcp.a2026i@gmail.com

用途：
- Google OAuth ADMIN positive path
- 驗證 ADMIN 可以正常登入與使用 DM
- 不用於 no-entitlement negative test

重要：
不要因為某個單一欄位顯示 platform_role=user 就直接判定它不是 ADMIN。
必須查實際 ADMIN source of truth，例如：
- platform_admin 判定邏輯
- DB role
- membership role
- admin registry
- RPC
- allowlist
- 其他真正被系統使用的授權來源

### TEST ACCOUNT A
tcp.ai313@gmail.com

用途：
- 一般使用者 positive path
- 後續整合 candidate 部署後，應驗證：
  valid Google OAuth
  + valid membership
  + active subscription
  + DM entitlement
  → protected DM API ALLOW

本輪若目前沒有 entitlement，不得人工修改 Production 資料去製造 PASS。

### TEST ACCOUNT B
job0975805890@gmail.com

用途：
- 一般使用者 negative path
- 必須確認：
  非 ADMIN
  無 active DM subscription
  無 DM entitlement
  無 trial bypass
  → protected DM API 應 DENY

## Known Staging Evidence

已知並已驗證：

- https://dm-staging.ticenpi.com/api/health
  → environment=staging
  → release=20260918-222526
  → commit=5e68034
  → databaseTargetClass=shared-staging

- staging runtime-config.js
  → environment=staging
  → supabaseUrl 對應 jlsqjvehwblkeuycjoyj.supabase.co

- Google OAuth authorize endpoint
  → redirect 到 accounts.google.com
  → callback 到 staging Supabase
  → redirect_to=https://dm-staging.ticenpi.com

因此 public staging 是本輪優先測試入口。

不要把 http://localhost:9422 當 staging。
localhost:9422 現在是 Local Dev。

## Known Auto-Provisioning Evidence

目前已知：
- auth.users AFTER INSERT trigger → handle_new_user()
- function 只建立 public.profiles
- 沒有自動建立 membership
- 沒有自動建立 trial
- 沒有自動建立 subscription
- 沒有自動建立 DM entitlement
- DM backend 沒有額外 app-layer provisioning

但仍要在真人 OAuth baseline 中觀察登入前後狀態，取得 runtime evidence。

## Known Entitlement Rule

DM backend 的 has_dm_access() / entitlement logic 目前接受：
- product_code = "dm"
- status in ("active", "trial")

因此：
TRIAL_STATUS_GRANTS_DM_ACCESS = YES

若發現 Account B 登入後自動產生 trial-status DM entitlement：
立即視為 security finding，不要自行刪除或修資料。

## Goal
這次獨立 Session 直接完成 Google OAuth 真人 baseline 與身份矩陣。

目標：

1. 驗證 ADMIN 帳號 tcp.a2026i@gmail.com 的 Google OAuth
2. 驗證 Account A tcp.ai313@gmail.com 的 Google OAuth
3. 驗證 Account B job0975805890@gmail.com 的 Google OAuth
4. 找出真正 ADMIN source of truth
5. 對 Account A/B 登入前後做 read-only delta
6. 確認是否自動建立 membership/trial/subscription/DM entitlement
7. 形成可供之後 Candidate UAT 使用的 identity matrix

## Browser Execution

優先使用 Browser MCP / Browser control。

若 Browser MCP 可用：
直接使用真實 browser session 執行。

若 Browser MCP 不可用：
不要 clone 或直接接管使用者真實 Chrome profile。
不要用猜測性的 Playwright profile hack。
直接回報：
BLOCKER = BROWSER_CONTROL_UNAVAILABLE

如果瀏覽器可操作：

### ADMIN
登入：
tcp.a2026i@gmail.com

驗證：
- Google OAuth 完成
- callback 成功
- session 建立
- DM UI 可達
- 實際 ADMIN source of truth

### ACCOUNT A
登入：
tcp.ai313@gmail.com

驗證：
- Google OAuth 完成
- session 建立
- 角色與現有 membership/subscription/entitlement
- 登入前後 delta

### ACCOUNT B
登入：
job0975805890@gmail.com

驗證：
- Google OAuth 完成
- session 建立
- 非 ADMIN
- 登入前後 delta
- 不得人工賦權

## Read-only State Checks

對 Account A / B 在登入前與登入後都檢查：

- profile
- platform/admin role source
- memberships
- subscriptions
- entitlements
- trial state
- DM entitlement

若可透過 DB / RPC read-only 查詢：
可以查。

禁止：
- INSERT
- UPDATE
- DELETE
- 人工新增 membership
- 人工新增 subscription
- 人工新增 entitlement
- 升級 admin

## Important Scope Rule

86f2c97 尚未部署到 Production，也不應假設 staging 已包含它。

因此本輪不是最終 entitlement enforcement UAT。

本輪只完成：
- Google OAuth 真人 baseline
- 身份 readiness
- admin source-of-truth
- auto-provisioning runtime evidence

不要把本輪結果誤報為「86f2c97 Production UAT PASS」。

## Hard Rules

- 禁止 Production deploy
- 禁止修改 Production 資料
- 禁止修改 Staging customer/subscription/entitlement 資料
- 禁止人工賦權
- 禁止 mock/dev auth
- 禁止手工注入 JWT
- 禁止輸出 password/JWT/access token/refresh token/secret
- 禁止只寫文件就宣稱完成
- 禁止再詢問三個帳號是誰，帳號已固定
- 禁止把 localhost:9422 當 staging
- 禁止 git reset --hard
- 禁止 git clean

## Completion Criteria

只有實際 Browser OAuth baseline 完成後，才可：

IDENTITY_MATRIX_READY = YES

若 Browser control 不可用：
不得假裝 PASS。

## Final Output

ADMIN_SOURCE_OF_TRUTH =
...

ADMIN_ACCOUNT =
tcp.a2026i@gmail.com

ADMIN_ACCOUNT_VERIFIED =
YES / NO

ADMIN_GOOGLE_OAUTH =
PASS / FAIL / NOT_TESTED

ACCOUNT_A =
tcp.ai313@gmail.com

ACCOUNT_A_GOOGLE_OAUTH =
PASS / FAIL / NOT_TESTED

ACCOUNT_A_CURRENT_ROLE =
...

ACCOUNT_A_MEMBERSHIP =
...

ACCOUNT_A_SUBSCRIPTION =
...

ACCOUNT_A_DM_ENTITLEMENT =
...

ACCOUNT_B =
job0975805890@gmail.com

ACCOUNT_B_GOOGLE_OAUTH =
PASS / FAIL / NOT_TESTED

ACCOUNT_B_IS_ADMIN =
YES / NO / UNPROVEN

ACCOUNT_B_MEMBERSHIP =
...

ACCOUNT_B_SUBSCRIPTION =
...

ACCOUNT_B_DM_ENTITLEMENT =
...

AUTO_CREATED_MEMBERSHIP =
YES / NO / UNPROVEN

AUTO_CREATED_TRIAL =
YES / NO / UNPROVEN

AUTO_CREATED_SUBSCRIPTION =
YES / NO / UNPROVEN

AUTO_CREATED_DM_ENTITLEMENT =
YES / NO / UNPROVEN

TRIAL_STATUS_GRANTS_DM_ACCESS =
YES

IDENTITY_MATRIX_READY =
YES / NO

FINAL_ENTITLEMENT_UAT_RUN =
NO

PRODUCTION_CHANGED =
NO

BLOCKER =
...

完成後停止，不 Deploy Production。
