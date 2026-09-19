# DM RELEASE GATES OWNER

## Mission
把已存在但未真正 blocking 的 DM 核心驗證，工程化接入中央 Release Pipeline。

這是 WAVE 1 平行任務。

已知審計：
- current critical-smoke 只涵蓋 YUCT data scrape + RemoveBG
- Capture Gate A script = EXISTS，但未接中央 pipeline
- Capture Gate B script = EXISTS，但未接中央 pipeline
- Capture Gate C script = EXISTS，但從未 PASS、需要真人/credential
- Auth invalid JWT / entitlement tests = EXISTS，但未接中央 deploy
- AI Browser E2E = MISSING
- Human UAT executable gate = MISSING
- CURRENT_RELEASE_GATE_SUFFICIENT = NO
- gate_a/gate_b/generate_test_token 曾發現 hardcoded JWT-like secret literal，不能進 Git

## Goal
本輪只做「Gate 工程」：
1. 把可以自動、穩定、無真人依賴的 Gate 接成 release-blocking。
2. 為需要真人/Browser/credential 的 Gate 建立明確 hook/stage，不偽造 PASS。
3. 移除 hardcoded secret 設計，改為安全 secret/env loader。
4. 不執行 Production deploy。

## Scope
中央：
F:\00-Ticenpi-SaaS\deploy

DM：
F:\00-Ticenpi-SaaS\TicenpiDM

先調查現有 branch/worktree 與 86f2c97 的關係。

若需修改 TicenpiDM tracked source：
使用獨立 branch/worktree，base 必須包含 86f2c97。
若無法安全建立，HARD STOP，不碰其他 Agent 工作樹。

## Step 1 — Gate Classification
把 Gate 分三類：

### AUTO BLOCKING NOW
應可在無真人介入下穩定執行，例如：
- Capture fixture Gate A
- Capture external Gate B（若 deterministic 且成本/穩定性可接受）
- Auth invalid JWT
- Auth valid no entitlement（使用 isolated fixture/test）
- Auth valid entitlement（使用 isolated fixture/test）

### BROWSER / CREDENTIAL BLOCKING
需要正式安全身份與 Browser：
- Capture Gate C
- AI Browser E2E

本輪建立 pipeline hook/contract，但沒有合法 credential 或尚未部署對應 fix 時，不得偽造 PASS。

### HUMAN ACCEPTANCE
Human UAT 不應假裝成自動測試。
建立 release checklist / explicit acceptance state 即可。

## Step 2 — Safe Secret Handling
檢查：
- gate_a_capture_fixture.py
- gate_b_external_capture.py
- gate_c_browser_e2e.py
- generate_test_token.py
- 其他相關 untracked scripts

禁止任何 hardcoded JWT secret / password / token 進 Git。

改成：
- env
- Secret Store
- existing secure loader

缺 secret 時：
fail closed / BLOCKED_BY_CREDENTIALS。

## Step 3 — Integrate Auto Gates
把安全且 deterministic 的自動 Gate 接進中央 deploy flow。

要求：
- 有清楚 runner
- 非只有文件
- exit code 非 0 會阻擋 release acceptance / 觸發既定 rollback
- log 不洩漏 secrets
- 可在 dry-run / CI 或 isolated runner 驗證

不要破壞其他產品 deploy。

## Step 4 — Capture Gate C Hook
建立正式 Gate C contract：
- DM_TEST_EMAIL / DM_TEST_PASSWORD 或安全 Google OAuth 測試身份來源
- Browser runner
- production/staging identity check
- image naturalWidth/naturalHeight
- Failed to fetch = NO

若現在 credential 未備妥：
Gate 狀態必須是 BLOCKED，不是 PASS。

## Step 5 — AI Browser E2E Hook
AI Agent 可能正在平行找 root cause。

本任務不要搶改 AI feature implementation。

只建立可接入的 Gate interface / stage：
Browser → /api/ai-draw → image render

若 AI root cause/fix 尚未完成：
標記 WAITING_FOR_AI_FIX，不偽造 PASS。

## Step 6 — Human UAT
建立明確 acceptance requirement：
- entitled user ALLOW
- non-entitled user DENY
- Capture browser PASS
- AI browser PASS
- Hub launch PASS

Human UAT 未完成時：
COMMERCIAL_GO_LIVE 不得標 READY。

## Step 7 — Validation
至少證明：
- auto gate fail 時 pipeline 真的 fail/block
- unrelated services 不受影響
- no hardcoded secret in tracked diff
- no Production deploy performed

## Hard Rules
- 禁止 Production deploy
- 禁止修改 Production customer data
- 禁止人工建立 Production entitlement 測試資料
- 禁止暴露 secret/token/password
- 禁止把 pending Browser Gate 寫成 PASS
- 禁止 git reset --hard
- 禁止 git clean
- 不需要再次等待使用者確認，直接開始

## Final Output
CAPTURE_GATE_A_RELEASE_BLOCKING =
YES / NO

CAPTURE_GATE_B_RELEASE_BLOCKING =
YES / NO

AUTH_INVALID_JWT_RELEASE_BLOCKING =
YES / NO

AUTH_NO_ENTITLEMENT_RELEASE_BLOCKING =
YES / NO

AUTH_ENTITLED_RELEASE_BLOCKING =
YES / NO

CAPTURE_GATE_C_HOOK =
READY / BLOCKED

AI_BROWSER_E2E_HOOK =
READY / WAITING_FOR_AI_FIX / BLOCKED

HUMAN_UAT_ACCEPTANCE =
DEFINED / MISSING

HARDCODED_SECRET_REMOVED =
YES / NO / NOT_APPLICABLE

PIPELINE_FAIL_CLOSED =
YES / NO

TESTS =
...

FIX_COMMIT =
...

PRODUCTION_CHANGED =
NO

READY_FOR_INTEGRATION =
YES / NO

完成後停止。
