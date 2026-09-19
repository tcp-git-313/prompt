# DM ENTITLEMENT OWNER

## Mission
修復 Ticenpi DM 商用授權安全。Production Runtime 已通過，但 Commercial Go-Live 仍 BLOCKED。

已知審計：
- TRIAL_VISIBLE_IN_PRODUCTION = YES
- TRIAL_CREATES_SUBSCRIPTION = NO
- TRIAL_CREATES_ENTITLEMENT = NO
- TRIAL_BYPASSES_ENTITLEMENT = UNPROVEN
- BACKEND_ENFORCES_DM_ENTITLEMENT = NO
- INVALID_JWT = DENIED

## Goal
建立真正的 Backend 商用授權 Gate：

Google JWT
→ User
→ Membership
→ Active Subscription
→ DM Entitlement
→ Protected DM API

只有以下全部成立才 ALLOW：
- valid JWT
- valid Membership
- active Subscription
- DM Entitlement enabled

缺任一條件：
- DENY
- HTTP 403
- 使用既有或明確的 PRODUCT_ENTITLEMENT_REQUIRED 類錯誤碼

## Tasks

### 1. 先定位既有授權架構
調查：
- entitlement.py
- CurrentUserDep
- my_commercial_context()
- has_dm_access()
- membership
- subscription
- entitlement RPC / DB path
- protected routers

列出 endpoint 分類：
A. 只驗 JWT
B. 已驗 entitlement
C. 明確 public

不要直接全域亂改。

### 2. 建立最小共用 Dependency
優先重用既有 my_commercial_context() / has_dm_access()。

建立或整理共用 dependency，例如 require_dm_entitlement()。

必須 fail closed。
禁止 frontend-only authorization。

### 3. Protected API 全面套用
至少涵蓋：
- designs
- scrape
- ai-draw
- upload / image operations
- save/update/delete
- 其他真正 DM 商用功能 API

Health / login callback 等明確 public endpoint 不要誤鎖。

### 4. Trial
不要只隱藏 Trial 按鈕。

先確認 Trial code 是否真的能給 API access。
若 Production 不應提供 Trial，可提出 UI 關閉方案，但 Backend enforcement 才是核心。

### 5. 必做測試
CASE A
valid JWT + active subscription + DM entitlement
→ ALLOW

CASE B
valid JWT + no DM entitlement
→ 403 DENY

CASE C
valid JWT + expired/inactive subscription
→ 403 DENY

CASE D
valid JWT + wrong product entitlement
→ 403 DENY

CASE E
invalid JWT
→ 401 DENY

CASE F
直接知道 dm.ticenpi.com API URL，但沒有 entitlement
→ DENY

不得修改 Production 客戶資料測試。
優先使用 existing fixtures / isolated tests / staging-safe identities。

### 6. 驗證
執行 targeted auth tests、entitlement tests、relevant backend tests。
確認 existing entitled user path 沒被破壞。

## Hard Rules
禁止：
- Production deploy
- 修改 Production DB customer records
- 人工開 Subscription / Entitlement
- 修改 Cloudflare / Tailscale / Docker daemon
- git reset --hard
- git clean

本輪只修 Source + Tests。

## Final Output
BACKEND_ENTITLEMENT_ENFORCEMENT =
PASS / FAIL

PROTECTED_ENDPOINTS_COVERED =
...

VALID_ENTITLED_USER =
PASS / FAIL

VALID_NO_ENTITLEMENT =
DENIED / ALLOWED

EXPIRED_SUBSCRIPTION =
DENIED / ALLOWED

WRONG_PRODUCT =
DENIED / ALLOWED

INVALID_JWT =
DENIED / ALLOWED

TRIAL_BACKEND_BYPASS =
YES / NO / UNPROVEN

TESTS =
...

FIX_COMMIT =
...

WORKING_TREE_CLEAN =
YES / NO

PRODUCTION_CHANGED =
NO

READY_FOR_COMMERCIAL_UAT =
YES / NO

完成後停止，不 Deploy Production。
