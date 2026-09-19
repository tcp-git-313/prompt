# DM CAPTURE E2E OWNER

## Mission
完成 Ticenpi DM Capture Gate C 的 Production Browser E2E。

目前：
- Capture CORS Root Cause = PROVEN
- CORS Fix = DEPLOYED
- Production Release = 20260919-110713
- Capture Gate A = PASS
- Capture Gate B = PASS
- Capture Gate C = PENDING

本輪只完成真正 Browser E2E。

## Goal
使用合法、有 DM entitlement 的帳號：

登入
→ 開 DM
→ 輸入 Capture fixture URL
→ 點擷取
→ /api/scrape 成功
→ 結果圖片真正在 UI 出現

## Credentials
只能從安全來源讀：
- DM_TEST_EMAIL
- DM_TEST_PASSWORD
或既有 Secret Store

禁止：
- 印出 password
- 印出 JWT
- 寫入 Git
- 寫入 MD
- 寫入 test source

若沒有可用 credential：
不要建立 Production 客戶。
不要人工開 entitlement。

直接輸出：
CAPTURE_GATE_C = BLOCKED_BY_CREDENTIALS

## Tasks

### 1. 帳號確認
登入後確認：
- Authentication = valid
- Membership = valid
- Subscription = active
- DM Entitlement = enabled

不得使用：
- Admin bypass
- Trial bypass
- Dev bypass
- Mock auth

### 2. Browser E2E
開：
https://dm.ticenpi.com

進 DM Editor。
輸入既有 Capture fixture URL。
按「擷取」。

取得：
- /api/scrape request
- status
- duration
- OPTIONS（若有）
- response

### 3. UI 真實結果
不只驗 backend JSON。

必須確認：
- result element visible
- 至少一張 image visible
- naturalWidth > 0
- naturalHeight > 0
- image request HTTP success

### 4. Browser Errors
必須：
- Failed to fetch = NO
- CORS error = NO
- blocking console error = NO

### 5. Production Identity
確認：
- /api/health
- /runtime-config.js

必須：
- environment = production
- release = 20260919-110713

## Final Output
TEST_ACCOUNT_AUTH =
PASS / FAIL

MEMBERSHIP =
PASS / FAIL

SUBSCRIPTION =
ACTIVE / INVALID

DM_ENTITLEMENT =
ENABLED / DISABLED

TRIAL_OR_ADMIN_BYPASS_USED =
NO / YES

CAPTURE_REQUEST =
PASS / FAIL

CORS =
PASS / FAIL

FAILED_TO_FETCH =
NO / YES

RESULT_UI_VISIBLE =
YES / NO

RESULT_IMAGE_LOADED =
YES / NO

PRODUCTION_RELEASE =
...

CAPTURE_BROWSER_E2E =
PASS / FAIL / BLOCKED

PRODUCTION_CAPTURE =
PASS / FAIL / BLOCKED

CAPTURE_INCIDENT =
CLOSED / BLOCKED

PRODUCTION_CHANGED =
NO

完成後停止。
