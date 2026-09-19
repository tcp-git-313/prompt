# DM AI GENERATION OWNER

## Mission
找出並修復 Ticenpi DM Production AI 製圖「Failed to fetch」的真正 Root Cause。

這是 WAVE 1 平行任務，可與 Google Identity、Local Runtime、Hub Sync、Release Gate 工程同時執行。

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

## Test Account Rule
可以使用既有 ADMIN Google 帳號作為本次 AI Production 重現帳號。

理由：
本任務目標是定位 AI request path 的 failure boundary，不是驗證 entitlement negative path。

要求：
- 必須走真實 Production Google OAuth / Browser 流程
- 不得使用 mock/dev auth
- 不得手工注入 JWT
- 不得輸出 password/JWT/token
- 記錄是否存在 admin bypass，但本次不得用結果證明一般無訂閱使用者權限正確

## Goal
真人重現一次 AI 製圖 → Failed to fetch，並同時取得：
- Browser Network
- Browser Console
- Production backend logs
- AGNES request evidence

最後定位唯一 failure boundary。

## Step 1 — Production Browser Reproduction
不要先建立猜 selector 的 Playwright 腳本。

先直接用真實 Production Browser 重現：
https://dm.ticenpi.com

流程：
Google OAuth 登入
→ AI 製圖
→ 最小 prompt
→ 點生成
→ 等成功或失敗

記錄：
- /api/ai-draw request
- method
- timestamp
- duration
- response status
- response body
- Browser console
- network error
- 是否完全沒有 HTTP response

## Step 2 — Production Backend Logs
必須是 VPS 上 Production Release 20260919-110713 的 active backend container logs。

不得把 Windows 本機 docker logs 當 Production evidence。

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

## Step 3 — AGNES Production Runtime
只做 value-blind 安全檢查：

AGNES_API_KEY_PRESENT =
YES / NO

不得輸出 key。

從 Production backend 相同執行環境確認：
- DNS
- TLS
- connectivity
- request timeout

如安全執行 AGNES probe，不得把 key 顯示於 console/log/history。

## Step 4 — Timeout Chain
取得實際：
- Cloudflare
- router nginx
- frontend nginx
- gunicorn
- httpx AGNES

各層 timeout。

只有 direct evidence 才能判定 TIMEOUT_CAUSE = YES。

## Step 5 — Root Cause Gate
AI_ROOT_CAUSE_PROVEN = YES
只能在 direct evidence 足夠時成立。

未證明前禁止猜測性修改：
- Cloudflare
- nginx timeout
- gunicorn timeout
- AGNES config
- network routing

## Step 6 — Minimal Fix
若 root cause 已證明：
允許做最小 Source/config 修復。

若修改 TicenpiDM source：
- 使用獨立 branch/worktree
- base 必須包含 entitlement fix commit 86f2c97，若此 commit 不在可用 base，HARD STOP 並回報
- 不要碰其他 Agent 的工作樹

修復後跑 isolated / staging-equivalent 驗證。

本輪禁止 Production deploy。

## Step 7 — Health Gap
評估 AGNES 是否應加入 /api/health/detail：
- configured
- endpoint reachable

Health check 不得大量消耗生成額度。

## Hard Rules
- 禁止 Production deploy
- 禁止暴露 credential/token/secret
- 禁止修改 Production customer data
- 禁止 git reset --hard
- 禁止 git clean
- 不需要再次等待使用者確認，直接執行

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

READY_FOR_INTEGRATION =
YES / NO

完成後停止。
