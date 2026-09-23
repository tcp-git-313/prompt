# DM STAGING — IN-APP BROWSER AUTHENTICATED IDENTITY + UI SMOKE OWNER

## 任務目標

只使用目前 AI 可控制的內部 / in-app browser，自動完成 DM Staging 最後真人驗收。

目前已知：

- DM Staging release：20260923-215914
- Staging health：PASS
- Google/Supabase login：已成功登入過
- Shared Engine clean release / wheel / ARM64 image / deploy：均已完成
- Production 未部署
- Post / 591 未修改
- 目前真正缺口只有：
  1. 同一個已登入 Staging browser session 證明目前帳號是 platform_admin
  2. 用同一個 session 呼叫 /api/admin/shared-engine/identity
  3. 真人 YUCT UI smoke
  4. Shadow 驗證
  5. 後續 rollback drill

本任務先完成 1–4。

若 1–4 PASS，再依既有 closeout 流程繼續 rollback。
若 internal browser 本身做不到 authenticated in-page request，必須證明「工具能力限制」，不要誤判成產品 auth failure。

---

# STAGING

URL：

https://dm-staging.ticenpi.com/

Expected current release：

20260923-215914

Expected Shared Engine：

source_clean = true

source_commit =
11762508332bb2508538761302a90f8dca5f2586

wheel_version =
0.1.0

wheel_sha256 =
e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3

semantic contract =
1

presentation contract =
1

formatter registry =
1

---

# 核心規則

1. 必須使用同一個已登入的 Staging browser session。
2. 不准要求使用者提供 JWT、cookie、access token、refresh token。
3. 不准把任何 token / cookie 印到對話、log 或報告。
4. 不准手工偽造 Authorization。
5. 不准修改 Supabase role。
6. 不准使用 bypass header。
7. 不准用 loopback-only admin shortcut 當 live Staging 證據。
8. 若 session 已失效，正常重新走 Google/Supabase login。
9. 不要開新的匿名 session 去替代已登入 session。
10. 不要把「內部瀏覽器工具無法存取 session」誤判成「帳號不是 platform_admin」。

---

# PHASE 0 — BROWSER PREFLIGHT

自動開啟或切回：

https://dm-staging.ticenpi.com/

確認：

- 頁面載入成功
- api = ok
- environment = staging
- release = 20260923-215914
- frontend/backend version 與 candidate 一致
- 頁面目前為 authenticated 狀態

如果未登入：

正常執行 Google/Supabase login flow。

如果需要使用者完成 Google OAuth：

停在登入畫面等待使用者完成。

完成後必須回到同一個 Staging tab / session 繼續。

---

# PHASE 1 — VERIFY CURRENT AUTHENTICATED USER

優先使用產品現有 UI / authenticated endpoint / current-user mechanism。

目標：

證明：

authenticated = true

並取得 role / platform role。

可接受證據優先順序：

1. 產品已存在 current-user / profile / admin session API
2. 產品已存在 platform_admin 專屬頁面，且 server-side gate 成功
3. 產品前端 runtime 已取得 user metadata / profile role，且資料來源可追到正式 auth contract
4. 同一 page context 的 authenticated API request

不能只因畫面上有「Admin」字樣就判 platform_admin。

如果能直接證明：

role = platform_admin

輸出：

PLATFORM_ADMIN VERIFIED

進 Phase 2。

如果證明：

authenticated = true
但 role 無法由現有 UI / API 證明：

不要先判 FAIL。

進 Phase 1B。

---

# PHASE 1B — USE SAME PAGE CONTEXT

嘗試使用 internal browser 提供的「同頁面執行 / page evaluation / browser context request」能力。

要求：

- request 必須從目前 dm-staging.ticenpi.com 已登入 page context 發出
- 讓產品現有 session / auth client 自己附帶憑證
- 不讀取、不輸出 token
- 不手工建立 Bearer token

優先嘗試：

GET /api/admin/shared-engine/identity

如果同源 fetch 能自動使用 cookie/session：

使用 credentials include。

如果產品使用 Supabase Bearer token，優先呼叫「前端現有 authenticated API helper / client wrapper」而不是直接讀 token。

可透過：

- app 已載入的 API client
- existing frontend service
- existing authenticated request helper

但不得把 token內容輸出。

如果 internal browser 可以執行 page-side JS，但無法直接取得 app helper：

允許在 page context 中安全地使用現有 session object 發送請求，條件是：

- token 不被列印
- token 不離開 browser context
- 最終只輸出 HTTP status + response JSON

若工具連 page-context execution 都不支援：

輸出：

INTERNAL_BROWSER_AUTH_CONTEXT_LIMITATION

並明確區分：

Product auth failure = NOT PROVEN

Tool limitation = PROVEN

不要要求使用者貼 JWT。

---

# PHASE 2 — SHARED ENGINE IDENTITY

在同一 authenticated platform_admin session 呼叫：

GET /api/admin/shared-engine/identity

成功條件：

HTTP 200

並核對：

source_clean = true

source_commit =
11762508332bb2508538761302a90f8dca5f2586

wheel_version =
0.1.0

wheel_sha256 =
e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3

semantic contract =
1

presentation contract =
1

formatter registry =
1

如果 API field name 略有不同，以實際 schema 對應。

如果 HTTP：

401
→ session/auth 沒被帶進 request，先判斷是不是 tool context 問題

403
→ authenticated but authorization denied，需確認 current role evidence

200
→ PASS

不要把 401/403 一律解讀成帳號錯誤。

輸出：

SHARED ENGINE IDENTITY VERIFIED

或：

SHARED ENGINE IDENTITY BLOCKED

並附根因分類：

AUTH_SESSION_NOT_PROPAGATED
ROLE_NOT_PLATFORM_ADMIN
TOOL_CONTEXT_LIMITATION
IDENTITY_MISMATCH
OTHER

---

# PHASE 3 — YUCT HUMAN UI SMOKE

只有 Shared Engine Identity VERIFIED 後才繼續。

在同一 Staging browser session：

1. 進 DM 正常擷取 UI
2. 使用已知可用、合法公開的 YUCT URL
3. 從 UI 正常提交
4. 等待正式 Legacy 結果
5. 確認 user-visible result 正常
6. 不要直接呼叫 fixture API 取代真人 UI

驗證：

- Legacy YUCT primary path 正常
- Shadow 不阻擋 Legacy
- Core/Shadow failure 不影響 Legacy result
- Admin Shadow Console 可看到該 run
- comparator / presentation evidence 可讀

---

# PHASE 4 — SHADOW SEMANTIC + PRESENTATION

在 Shadow Console 找最新真人 run。

至少確認：

- Legacy raw
- Core raw
- comparison status
- Canonical
- DM final display

如果剛好有：

Legacy：
3房2廳2衛

Core：
3房(室)2廳2衛

則應：

NORMALIZED_MATCH

Canonical：
3 / 2 / 2

DM compact：
3房2廳2衛

如果沒有這個實際案例：

不要偽造。

只驗目前真人 run 的真實欄位。

另外確認：

Presentation formatter 改變
不得改變 semantic comparison status。

如果 UI 可安全切換 preview：

測一次 compact / slash preview。

---

# PHASE 5 — LEGACY PRIMARY CONFIRMATION

確認：

Legacy remains user-visible primary = YES

Extraction Core Primary = NO

Legacy scraper deleted = NO

如果 Shadow / Shared Engine 出錯：

Legacy user response should remain unaffected。

如果無法證明：

LEGACY PRIMARY NOT VERIFIED

停止，不進 rollback。

---

# PHASE 6 — DECIDE WHETHER TO CONTINUE ROLLBACK

只有以下全部 PASS：

- platform_admin VERIFIED
- shared-engine identity VERIFIED
- YUCT human UI smoke PASS
- Shadow PASS
- Legacy Primary CONFIRMED

才允許接續既有 rollback closeout。

如果全部 PASS：

輸出：

AUTHENTICATED STAGING CLOSEOUT GATE PASS

然後依既有正式流程繼續：

candidate
20260923-215914
↓
rollback
20260922-131327
↓
health
↓
restore candidate
↓
identity
↓
final READY

如果任一項 BLOCKED：

不要 rollback。

---

# IMPORTANT — INTERNAL BROWSER LIMITATION FALLBACK

如果唯一 blocker 是：

內部瀏覽器無法在同一已登入 page context 發 authenticated request

則不要再重複嘗試匿名 endpoint。

請檢查 DM frontend source（只讀優先）找出：

- Supabase client
- session restore
- API client wrapper
- Authorization injection path
- admin API helper

然後用 internal browser 可用能力，從頁面現有 client 發 request。

如果仍然無法：

最後輸出：

INTERNAL_BROWSER_AUTH_CONTEXT_LIMITATION

並提供「最小產品修正方案」：

在 platform_admin admin page 加一個只讀：

Shared Engine Identity

元件。

要求：

- 使用既有 authenticated API client
- platform_admin only
- 不顯示 token
- 直接顯示 identity JSON / verified status
- 不改 Shared Engine
- 不改 Central Seat
- 不改 Production

但本任務預設先不要寫這個 UI；只有工具能力確定不足時才提出。

---

# FINAL REPORT FORMAT

# DM IN-APP BROWSER AUTHENTICATED VALIDATION REPORT

## 1. Browser Session

Staging URL:
Authenticated:
Google login:
Session preserved:
PASS / FAIL

## 2. Platform Admin

Role evidence source:
Role:
platform_admin verified:
YES / NO

## 3. Shared Engine Identity

HTTP:
source_clean:
source_commit:
wheel_version:
wheel_sha256:
semantic:
presentation:
formatter registry:

Expected match:
PASS / FAIL / BLOCKED

## 4. Internal Browser Capability

Same-page authenticated request:
PASS / FAIL

Tool context limitation:
YES / NO

## 5. YUCT Human UI

URL:
Legacy result:
Shadow run:
Admin console:

Result:
PASS / FAIL / NOT_RUN

## 6. Semantic / Presentation

Status:
Canonical:
DM display:
Formatter switch:
Comparator unchanged:

Result:
PASS / FAIL / NOT_AVAILABLE

## 7. Legacy Primary

Legacy primary:
YES / NO / NOT_VERIFIED

Core primary:
NO / YES

## 8. Rollback Gate

All prerequisites:
PASS / BLOCKED

Rollback allowed:
YES / NO

## 9. Final Result

只能：

AUTHENTICATED STAGING CLOSEOUT GATE PASS

或

DM STAGING AUTHENTICATED VALIDATION BLOCKED

如果 BLOCKED：

Blocker:
AUTH_SESSION_NOT_PROPAGATED
ROLE_NOT_PLATFORM_ADMIN
TOOL_CONTEXT_LIMITATION
IDENTITY_MISMATCH
YUCT_UI_FAILURE
LEGACY_PRIMARY_NOT_VERIFIED
OTHER

---

# HARD STOP

不要：

- 要求使用者貼 JWT
- 要求使用者貼 cookie
- 修改 Central Seat
- 修改 Production
- 修改 Production DB
- 切 Core Primary
- 刪 Legacy scraper

如果 Gate PASS，再接續既有 rollback closeout。
