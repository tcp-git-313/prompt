# DM DUAL AUTH SESSION / 403 ROOT CAUSE + LOCAL/STAGING AUTH PARITY OWNER

## 任務目標

目前 DM Staging 已部署 Shared Engine candidate，但真人登入驗收時發現：

- Staging 頁面已登入
- `GET /api/designs` 回 `403 Forbidden`
- 同一個 request evidence 顯示：
  - Authorization Bearer 來自一個登入身份 / Supabase project
  - Browser Cookie / stored session 來自另一個登入身份 / Supabase project
- 因此目前高度懷疑：
  1. stale session
  2. localStorage / cookie 雙 session 混用
  3. frontend auth client 與 API Authorization injector 使用不同 session source
  4. Staging / local Supabase project wiring 不一致
  5. local dev auth bypass 讓正式 Staging auth path 長期沒有被完整驗證

本任務唯一目標：

> 找出 DM Staging 403 的真正 root cause，修正「登入 session / Authorization / Supabase project / role / entitlement」一致性，並建立 Local 與 Staging auth parity tests，讓這類錯誤之後不能再默默進入 Staging。

本任務完成前：

- 不做 rollback
- 不部署 Production
- 不切 Extraction Core Primary
- 不修改 Central Seat 邏輯
- 不修改 Post / 591
- 不開始其他 Adapter
- 不把 403 直接歸因為 Seat

---

# 安全前提

先把目前已暴露過的 Staging token / cookie 視為不再可信。

不要：

- 在 log 印 JWT
- 在 report 印 Cookie
- 在測試 fixture 寫真 token
- 把 access_token / refresh_token commit
- 複製使用者曾貼出的舊 token 進測試
- 直接 decode 後把完整 payload 貼到報告
- 透過 DB 手工改 platform_admin
- 關閉 JWT 驗證
- 使用 auth bypass 來「讓測試過」

如需測試：

只記錄安全 identity summary：

- issuer host / project alias
- subject hash 或 user id 前 8 碼
- email masked
- role
- token source
- expiry status

禁止輸出完整 token。

---

# 零風險平行策略

本任務允許平行加速，但只允許：

```
PARALLEL READ-ONLY AUDITS
        ↓
ROOT CAUSE SYNTHESIS
        ↓
SINGLE WRITER FIX
        ↓
PARALLEL TESTS
        ↓
SINGLE RELEASE DECISION
```

## 可平行 Track

### TRACK A — Frontend Auth Source Audit
只讀 DM frontend。

### TRACK B — Backend Auth / JWT / Role Audit
只讀 DM backend。

### TRACK C — Supabase / Runtime Config Audit
只讀 Staging config、runtime-config、env wiring。

### TRACK D — Local vs Staging Auth Parity Audit
只讀 local dev / staging routing / bypass logic / tests。

以上四個 Track：

- 不改 code
- 不改 env
- 不改 Supabase
- 不 deploy
- 不 commit
- 不 reset / stash / clean

完成四份 audit 後才允許單一 Writer 修改。

如果任何 Track 發現其他 active writer 正在碰相同 auth files：

HARD STOP

輸出：

DM AUTH WRITE COLLISION

---

# 工作目錄

主要：

F:\00-Ticenpi-SaaS\TicenpiDM

只讀參考：

F:\00-Ticenpi-SaaS\ExtractionHub
F:\00-Ticenpi-SaaS\ticenpi-platform
F:\00-Ticenpi-SaaS\deploy

Post / 591：

READ ONLY

Production：

OUT OF SCOPE

---

# PHASE 0 — PREFLIGHT

記錄：

DM branch
DM HEAD
git status
git diff
git diff --cached
untracked files

記錄 Staging：

- release
- backend digest
- frontend digest
- /api/health
- runtime-config identity
- current Supabase project/config identity
- auth-related env names（只列 PRESENT/MISSING，不列 secret value）

禁止：

git reset --hard
git clean
git stash
git rebase

確認：

- 沒有 active deploy
- 沒有 active rollback
- 沒有 active writer 修改 auth source
- current Staging candidate 未 drift

輸出：

DM AUTH PREFLIGHT PASS

或：

DM AUTH PREFLIGHT BLOCKED

---

# TRACK A — FRONTEND AUTH SOURCE AUDIT

只讀找出所有登入與 token source。

至少盤點：

- Supabase client creation
- session restore
- onAuthStateChange
- localStorage key
- cookie key
- sessionStorage
- API client
- fetch wrapper
- Axios interceptor（若有）
- Authorization injection
- logout
- login callback
- runtime-config loading
- environment project selection
- platform_admin UI gating

建立：

## FRONTEND AUTH SOURCE MAP

```
Google Login
↓
Supabase client
↓
Session storage location
↓
Frontend current user state
↓
API client
↓
Authorization header
```

對每一層列：

- source
- key name
- Supabase project identity
- refresh behavior
- cleanup behavior
- whether stale value can survive logout
- whether cookie and localStorage can diverge

特別回答：

1. Authorization Bearer 是從哪裡拿？
2. Cookie `sb-ticenpi-auth*` 是誰寫的？
3. 是否存在兩個 Supabase client？
4. 是否存在兩個 project URL / anon key source？
5. runtime-config 與 build-time env 是否可能指向不同 project？
6. logout 是否只清其中一種 storage？
7. API client 是否會持有 stale in-memory token？
8. route reload 後 token source 是否改變？

輸出：

FRONTEND DUAL SESSION RISK MATRIX

---

# TRACK B — BACKEND AUTH / JWT / ROLE AUDIT

只讀盤點：

- JWT validation dependency
- Supabase issuer validation
- audience validation
- JWKS source
- project URL / issuer allowlist
- role resolution
- profile lookup
- platform_admin resolution
- entitlement dependency
- /api/designs auth chain
- /api/admin/shared-engine/identity auth chain
- local bypass
- staging bypass disable

建立：

## BACKEND AUTH CHAIN

```
Authorization Bearer
↓
JWT verify
↓
issuer / audience
↓
user identity
↓
profile / role
↓
entitlement
↓
route allow/deny
```

回答：

1. `/api/designs` 的 403 是哪一層丟的？
2. token valid but wrong project 時是 401 還是 403？
3. valid authenticated user but missing entitlement 時是 403 嗎？
4. valid platform_admin 是否 bypass entitlement？
5. role 是從 JWT claim 還是 DB profile 查？
6. backend 是否可能接受 project A token，再去 project B profile table 查 user？
7. 是否有 issuer/project mismatch guard？
8. error response 是否足以區分：
   - invalid token
   - wrong project
   - valid user / no entitlement
   - not platform_admin

不要改行為。

輸出：

BACKEND 403 DECISION MATRIX

---

# TRACK C — SUPABASE / RUNTIME CONFIG AUDIT

只讀比較：

## Local

- Supabase URL
- anon/public key source
- auth bypass flag
- runtime-config
- frontend build-time env
- backend env
- cookie namespace

## Staging

同上。

## Production

只讀 config identity，不讀 secret value。

建立：

SUPABASE PROJECT MATRIX

| Environment | Frontend Project | Backend Project | Cookie Namespace | Runtime Config | Build-Time Config | Auth Bypass |

要求：

同一 environment 的 frontend/backend project 必須一致。

特別檢查：

- old Staging Supabase project references
- Production project accidentally bundled into Staging frontend
- stale runtime-config.js
- build-time VITE/SUPABASE env baked into image
- cookie name reused across incompatible project/session
- localhost cookie/domain behavior與 Staging 不同

如果找到：

FRONTEND_PROJECT != BACKEND_PROJECT

直接標：

ROOT CAUSE CANDIDATE — PROJECT MISMATCH

但仍需 Track A/B 證據確認。

---

# TRACK D — LOCAL VS STAGING AUTH PARITY AUDIT

只讀確認：

本機是否：

- 自動 platform_admin
- dev bypass
- loopback bypass
- local auth disabled
- fake test identity
- mock entitlement

Staging 是否：

- 真 JWT
- 真 Supabase
- 真 profile role
- 真 entitlement

建立：

## AUTH PARITY MATRIX

| Capability | Local | Staging | Same Path? | Risk |

至少：

Login
Session restore
Authorization injection
JWT validation
Issuer validation
Role lookup
Entitlement
platform_admin
/api/designs
/admin identity
logout/session cleanup

回答：

> 本機「不用登入」是否讓 Staging auth bug 無法在 local 被發現？

若是：

標記：

LOCAL_AUTH_PARITY GAP

但不要直接刪 local dev convenience。

要設計 automated parity tests。

---

# PHASE 1 — SYNTHESIZE ROOT CAUSE

四個 Track 完成後，單一 Planner 統整。

根因只能使用證據分類：

A. STALE_BROWSER_SESSION
B. MULTIPLE_SUPABASE_CLIENTS
C. FRONTEND_BACKEND_PROJECT_MISMATCH
D. BUILD_RUNTIME_CONFIG_MISMATCH
E. STALE_IN_MEMORY_BEARER
F. COOKIE_NAMESPACE_COLLISION
G. LOGOUT_CLEANUP_INCOMPLETE
H. VALID_USER_MISSING_ENTITLEMENT
I. ROLE_RESOLUTION_BUG
J. ISSUER_VALIDATION_BUG
K. OTHER
L. MULTIPLE_CAUSES

每個 root cause 必須附：

Evidence
Affected path
Why local did/didn't catch it
Why Staging returns 403
Minimal fix

如果證據不足：

HARD STOP

輸出：

DM AUTH ROOT CAUSE INCONCLUSIVE

不要亂修。

---

# PHASE 2 — SAFE BROWSER SESSION RESET TEST

在修改 code 前先判斷是否只是 stale browser state。

使用正常操作：

1. Staging logout
2. 清除 DM Staging site data：
   - cookies
   - localStorage
   - sessionStorage
3. 關閉舊 DM Staging tabs
4. 重新開一個新的 Staging tab
5. 正常 Google login
6. 不使用任何舊 token
7. 觀察第一個 authenticated request

不要：

- 手工貼 JWT
- 手工改 storage
- 手工改 cookie

驗：

- current displayed account
- Authorization identity summary
- cookie/session identity summary
- issuer/project consistency
- /api/designs result

如果清乾淨後：

/api/designs = 200

且所有 identity 一致：

根因至少包含：

STALE_BROWSER_SESSION / LOGOUT_CLEANUP_INCOMPLETE

仍需修 cleanup / regression，不能只叫使用者每次清 site data。

如果仍 403：

繼續 code root cause。

---

# PHASE 3 — SINGLE WRITER FIX

只有 root cause 已有證據才修改。

原則：

- minimal
- generic
- environment-correct
- no auth bypass
- no project hardcode if config system exists
- no token logging
- no Production change

可能修正方向依證據選擇，不預設：

### Session source

確保只有一個 authoritative auth client。

### API Authorization

每次 request 取 current valid session，而不是 cached stale token。

### Project binding

frontend/backend/runtime-config 必須同 project。

### Logout

完整清理：
- Supabase session
- related cookie
- localStorage
- in-memory state

但只清 DM own namespace。

### Cookie namespace

若不同環境 / project 共用 cookie name 造成 collision：

改成明確 environment/project-safe namespace。

不要破壞 Production existing cookies without migration plan。

### Error classification

若目前 403 無法分辨 root cause：

可補 sanitized error code：

INVALID_TOKEN
WRONG_ISSUER
AUTHENTICATED_NOT_ENTITLED
PLATFORM_ADMIN_REQUIRED

但不要回敏感資料。

---

# PHASE 4 — LOCAL AUTH PARITY TEST HARNESS

保留 local dev convenience 可以，但新增正式 parity test path。

目標：

Local CI 必須能測到接近 Staging 的 auth chain。

至少新增：

1. valid Staging-like JWT fixture（測試簽名 / local mock issuer，不可真 token）
2. wrong issuer
3. wrong audience
4. authenticated normal user
5. platform_admin
6. entitled user
7. non-entitled user
8. stale session replacement
9. logout cleanup
10. Authorization source refresh

不要讓 local bypass 覆蓋這些 tests。

正式規則：

```
Local interactive dev
可以 convenience

CI auth contract tests
必須 bypass OFF
```

---

# PHASE 5 — FRONTEND SESSION TESTS

至少驗：

### Login

login success
→ one current session

### Refresh

token refresh
→ API client uses refreshed token

### Account switch

User A logout
→ User B login
→ request must use User B only

### Project switch guard

若 runtime config project 與 stored session issuer 不一致
→ old session must be rejected / cleared safely

### Logout

logout
→ no stale bearer
→ no stale app cookie
→ no stale current user

### Reload

page reload
→ restored session and API authorization same identity

---

# PHASE 6 — BACKEND AUTH TESTS

至少：

/api/designs

Unauthenticated
→ expected deny

Invalid JWT
→ deny

Wrong issuer
→ deny

Valid authenticated but no entitlement
→ expected 403

Valid entitled
→ 200

platform_admin
→ according to current contract

/api/admin/shared-engine/identity

Unauthenticated
→ deny

Normal user
→ deny

platform_admin
→ 200

不要把 expected contract 改成配合 bug。

---

# PHASE 7 — REAL STAGING RETEST

只有 Local + tests PASS 才進 Staging。

如 code changed：

走正式 CI / image / manifest / Staging deploy 流程。

不要直接 hot patch container。

Staging retest：

1. 清除舊 session
2. 正常登入 platform_admin
3. 確認 single identity
4. `GET /api/designs`
5. `GET /api/admin/shared-engine/identity`

要求：

/api/designs
→ 200 或符合 platform_admin 現行產品 contract

/admin/shared-engine/identity
→ 200

並確認兩個 request 使用同一 authenticated identity / project。

不要輸出 token。

---

# PHASE 8 — SHARED ENGINE IDENTITY CONTINUE GATE

如果 `/api/admin/shared-engine/identity` = 200：

重新核對既有 expected：

source_clean = true

source_commit =
11762508332bb2508538761302a90f8dca5f2586

wheel_version =
0.1.0

wheel_sha256 =
e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3

semantic = 1
presentation = 1
formatter registry = 1

PASS 後：

輸出：

DM STAGING AUTH + SHARED ENGINE IDENTITY PASS

接續既有流程：

YUCT UI smoke
↓
Shadow
↓
Legacy primary
↓
rollback
↓
restore
↓
READY

---

# PHASE 9 — PARALLEL TEST EXECUTION

修正完成後，可以安全平行執行：

TRACK T1 — Backend auth tests

TRACK T2 — Frontend session/auth tests

TRACK T3 — Shared Engine / Shadow regression

TRACK T4 — Cookie isolation / tenant tests

條件：

- 測試 Track 只跑 tests
- 不修改 source
- 若測試失敗，只回報
- 只有 single writer 負責修正

不要讓四個測試 Agent 各自修 code。

---

# PHASE 10 — REGRESSION

至少：

DM backend formal tests
frontend Vitest
cookie isolation
auth/RLS
tenant isolation
shared engine identity
Shadow
Legacy YUCT auth regression
frontend build
Docker smoke if packaging changed
git diff --check

如果 auth config / Docker image 改變：

必須補 ARM64 hosted CI。

---

# PHASE 11 — DOCUMENTATION

更新既有：

DM current-state
DM history
auth handoff / architecture docs（如果現有）

記錄：

- root cause
- stale session behavior
- local vs staging parity gap
- exact fix
- tests
- Staging evidence

不要記：

JWT
cookie
email full identity
secret

---

# FINAL REPORT FORMAT

# DM DUAL AUTH SESSION / 403 ROOT CAUSE REPORT

## 1. Baseline

Branch:
HEAD:
Staging release:
Health:
Collision:

## 2. Parallel Audits

Frontend auth:
PASS / BLOCKED

Backend auth:
PASS / BLOCKED

Supabase config:
PASS / BLOCKED

Local/Staging parity:
PASS / BLOCKED

Parallel writes:
0

## 3. Auth Source Map

Frontend session source:
Authorization source:
Cookie source:
Supabase project:
Backend expected project:

Consistent:
YES / NO

## 4. Root Cause

Classification:

Evidence:

Why 403:

Why local did not catch it:

## 5. Browser Reset Test

Old session cleared:
YES / NO

Fresh login:
PASS / FAIL

/api/designs:
HTTP xxx

Identity consistent:
YES / NO

## 6. Fix

Files:
Behavior changed:
Auth bypass added:
NO

Token logging added:
NO

## 7. Local Auth Parity

Interactive local bypass:
PRESERVED / CHANGED

CI bypass:
OFF / FAIL

Wrong issuer:
PASS / FAIL

Normal user:
PASS / FAIL

Entitled user:
PASS / FAIL

platform_admin:
PASS / FAIL

Account switch:
PASS / FAIL

Logout cleanup:
PASS / FAIL

## 8. Frontend Regression

Session restore:
Token refresh:
Account switch:
Project mismatch guard:
Logout:
Reload:

## 9. Backend Regression

/api/designs:
Unauth:
Wrong issuer:
No entitlement:
Entitled:
platform_admin:

/api/admin/shared-engine/identity:
Unauth:
Normal user:
platform_admin:

## 10. Staging Retest

Release:
Authenticated account identity:
SAFE SUMMARY ONLY

/api/designs:
HTTP xxx

/api/admin/shared-engine/identity:
HTTP xxx

Same auth project:
PASS / FAIL

Same authenticated identity:
PASS / FAIL

## 11. Shared Engine Identity

source_clean:
source_commit:
wheel_version:
wheel_sha256:
semantic:
presentation:
formatter:

Expected match:
PASS / FAIL

## 12. Parallel Test Tracks

Backend:
Frontend:
Shadow:
Cookie/Tenant:

## 13. Isolation

Central Seat:
UNCHANGED

Post:
UNCHANGED

591:
UNCHANGED

Production:
UNCHANGED

Production DB:
UNCHANGED

## 14. Next Gate

如果 auth + identity PASS：

READY FOR YUCT UI SMOKE + ROLLBACK CLOSEOUT

如果未通過：

BLOCKED

## 15. Final Result

只能：

DM STAGING AUTH ROOT CAUSE FIXED

或

DM STAGING AUTH ROOT CAUSE BLOCKED

---

# HARD STOP

本輪不要：

- 修改 Central Seat
- 修改 Post
- 修改 591
- 部署 Production
- 修改 Production DB
- 切 Core Primary
- 刪 Legacy scraper
- 開始其他 Adapter

若 Staging auth + shared-engine identity PASS，停止並回報，等待接續 YUCT UI smoke / rollback closeout。
