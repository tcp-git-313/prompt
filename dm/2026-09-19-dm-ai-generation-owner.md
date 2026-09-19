# DM AI GENERATION OWNER

## Mission
找出並修復 Ticenpi DM Production AI 製圖「Failed to fetch」的真正 Root Cause。

Production:
- URL: https://dm.ticenpi.com
- Release: 20260919-110713

已知：
- /api/ai-draw 存在
- invalid JWT → 401
- CORS preflight 正常
- /api/ai-draw 使用 same-origin relative URL
- CORS 不是目前已證明 root cause
- AGNES public endpoint 可達
- /api/health/detail 沒有檢查 AGNES
- AI_FAILURE_REPRODUCED = NO
- AI_ROOT_CAUSE_PROVEN = NO

不要繼續靜態猜測。

## Goal
真人重現一次 AI 製圖 → Failed to fetch，並同時取得：
- Browser Network
- Browser Console
- Production backend logs
- AGNES request evidence

最後定位唯一 failure boundary。

## Tasks

### 1. Production Browser Reproduction
用 Playwright / Browser 開：
https://dm.ticenpi.com

使用已有合法 DM entitlement 的測試帳號。

操作：
登入 → AI 製圖 → 最小 prompt → 點生成 → 等成功或失敗

必須記錄：
- /api/ai-draw request
- method
- timestamp
- request duration
- response status
- response body
- Browser console
- network error
- 是否完全沒有 HTTP response

禁止輸出 JWT/token。

### 2. 同時間 Backend Logs
對齊相同 timestamp：

REQUEST_REACHED_BACKEND =
YES / NO

AGNES_CALL_STARTED =
YES / NO

AGNES_RESPONSE_RECEIVED =
YES / NO

AGNES_STATUS =
...

BACKEND_EXCEPTION =
...

BACKEND_REQUEST_DURATION =
...

### 3. AGNES Production Runtime
只做 value-blind / safe check：

AGNES_API_KEY present =
YES / NO

不要輸出 key。

從 Production backend 相同執行環境確認：
- DNS
- TLS
- connectivity
- request timeout

若安全執行最小 AGNES probe，不得把 key 印到 console/log。

### 4. Timeout Chain
確認實際：
- Cloudflare
- router nginx
- frontend nginx
- gunicorn
- httpx AGNES

各層 timeout。
只有直接 timeout evidence 才能判 timeout root cause。

### 5. Root Cause Gate
只有 direct evidence 才能：
AI_ROOT_CAUSE_PROVEN = YES

若已證明，允許做最小 Source/config 修復並 isolated/staging verify。
若仍無法證明：停止，不亂改。

### 6. 修復後驗證
若已修復：
Production-equivalent isolated test：
POST /api/ai-draw
→ HTTP success
→ 回傳 image
→ image data URL / URL valid

並準備 Browser E2E Gate。

本輪禁止 Production deploy。

### 7. Health Gap
確認是否應把 AGNES 加入 /api/health/detail：
- configured
- endpoint reachable

不要真的消耗大量生成額度做 health check。

## Final Output
AI_FAILURE_REPRODUCED =
YES / NO

AI_FAILURE_BOUNDARY =
...

REQUEST_REACHED_BACKEND =
YES / NO

AGNES_CALL_STARTED =
YES / NO

AGNES_RESPONSE_RECEIVED =
YES / NO

AI_ROOT_CAUSE_PROVEN =
YES / NO

AI_ROOT_CAUSE =
...

CORS_CAUSE =
YES / NO

JWT_CAUSE =
YES / NO

AGNES_CONFIG_CAUSE =
YES / NO

AGNES_NETWORK_CAUSE =
YES / NO

TIMEOUT_CAUSE =
YES / NO

FIX_APPLIED =
YES / NO

FIX_COMMIT =
...

ISOLATED_AI_GENERATION =
PASS / FAIL / NOT_RUN

AGNES_HEALTH_GAP =
YES / NO

PRODUCTION_CHANGED =
NO

READY_FOR_PRODUCTION_RETRY =
YES / NO

完成後停止。
