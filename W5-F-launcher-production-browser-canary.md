# W5-F — Launcher Production Browser Canary（platform_admin / Central Product Seats）

## ROLE
Production Browser Canary Verifier

## 推薦模型
GPT-5.6 Luna High
如果瀏覽器診斷遇到複雜前端狀態問題，再升級 GPT-5.6 Sol High。

## 目的

目前已經完成 Launcher source reconciliation。

已確認：

- Production Launcher source = b7f85bf9dc0c81ca3c555c44ebaa952ce2826df7
- Cloudflare Pages Production deployment = 5cb1ad90
- Production live assets 與 b7f85bf byte-identical
- Production Supabase ref = zfpkxsulehkbyuqkkcml
- 沒有 Staging ref crossover
- Central Commercial Core / Seat RPC 已存在於 Production
- 不需要重新 deploy Launcher
- 不需要重做 Platform / Post / DM / OCR
- 不要再做 source reconciliation

目前唯一未證明的是：

**真實 platform_admin 登入 Production 後，Customer detail 裡的 Central Seat Workbench 是否真的 render 並可用。**

本任務只做 Production Browser Canary。

---

# HARD BOUNDARY

禁止：

- 修改 source code
- 修改 build
- 修改 Launcher
- commit
- push
- deploy
- 重新 build
- 修改 Cloudflare
- 修改 Supabase schema
- migration
- 修改 Post / DM / OCR
- 刪除 Production Customer / Member / Seat
- 為了測試而繞過 auth
- 使用 test_access bypass
- 使用 synthetic JWT 取代真人瀏覽器驗證

允許：

- Production 真實瀏覽器驗證
- 唯讀查詢 Production RPC / DB 狀態
- 必要時使用既有正式 admin UI 做最小可逆測試
- 若測試需要建立/指派 canary Seat，必須先確認對既有 Production 資料沒有破壞，並記錄建立與回復步驟

---

# 1. 開啟正確 Production 入口

使用：

https://app.ticenpi.com/?route=admin

不要從 Staging 網域進入。

確認：

CURRENT_URL =
PRODUCTION_SUPABASE_REF = zfpkxsulehkbyuqkkcml
STAGING_REF_PRESENT = NO

若發現 jlsqjvehwblkeuycjoyj：
立即 HARD STOP。

---

# 2. 確認真人 session 是 platform_admin

使用目前真人登入 session。

不要只看畫面。

從實際 client state / profile / RPC response 確認：

AUTHENTICATED = YES
CURRENT_USER_EMAIL =
CURRENT_USER_ID =
PLATFORM_ROLE = platform_admin

如果不是 platform_admin：

CANARY = BLOCKED
原因寫清楚。

不要假設。

---

# 3. 確認 admin route 真正載入 Workbench

在 Production：

/?route=admin

確認實際載入：

- app.js
- workbench.js
- root.CommercialWorkbench / WB equivalent
- admin route 沒有被 hidden
- 沒有 JS exception 阻止 render

檢查 browser console / network。

輸出：

WORKBENCH_JS_LOADED =
WORKBENCH_RUNTIME_OBJECT_PRESENT =
ADMIN_ROUTE_ACTIVE =
JAVASCRIPT_ERRORS =

如果 workbench.js 已載入但 UI 沒 render：
繼續往 role / RPC / state 診斷。

---

# 4. Customer list 不作為新版/舊版判斷依據

注意：

Customer list 首頁與舊版可見 chrome 可能相同。

不要因為首頁長得舊就判定 Launcher 是舊版。

真正判斷點是：

Customer detail
→ Members
→ Entitlements
→ Central Product Seats

---

# 5. 打開一個既有 business Customer

從 Production admin Customer list 選一個既有 business Customer。

優先選：

- 已知有 active member
- 已知有 entitlement
- 不需要建立新的正式 Customer 就能測

不要先新增 Customer。

記錄：

CUSTOMER_ID =
CUSTOMER_NAME =
CUSTOMER_TYPE =
CUSTOMER_STATUS =

---

# 6. 驗證 Customer Detail 新 Workbench

進入 Customer detail 後確認以下區塊是否存在：

1. Members
2. Entitlements
3. Central Product Seats

要求：

MEMBERS_SECTION_PRESENT =
ENTITLEMENTS_SECTION_PRESENT =
CENTRAL_PRODUCT_SEATS_PRESENT =

Members 必須能看到 stable user_id。

輸出：

MEMBER_ROWS =
STABLE_USER_ID_VISIBLE = YES/NO

---

# 7. 驗證 Production RPC 真實回應

透過 browser/network 或唯讀 RPC 驗證 Workbench 使用的 Production RPC。

至少確認：

admin_list_customer_members
effective_entitlement_details
admin_list_product_seats

如果 UI 有 lifecycle action，也確認相關 RPC 存在：

assign_product_seat
release_product_seat
admin_suspend_customer
admin_reactivate_customer
admin_suspend_customer_member
admin_reactivate_customer_member

本階段先以 read/list 為主。

輸出：

RPC_ADMIN_LIST_CUSTOMER_MEMBERS =
RPC_EFFECTIVE_ENTITLEMENT_DETAILS =
RPC_ADMIN_LIST_PRODUCT_SEATS =

NETWORK_ERRORS =
RPC_ERRORS =

---

# 8. 查 job0975805890@gmail.com 為什麼沒有出現在首頁

不要期待 email 直接出現在 Customer list。

唯讀查 Production：

job0975805890@gmail.com

確認：

AUTH_USER_EXISTS =
USER_ID =
CUSTOMER_MEMBERSHIP_EXISTS =
MEMBER_CUSTOMER_ID =
MEMBER_STATUS =
ENTITLEMENTS =
SEAT_ASSIGNMENTS =

如果沒有 membership：

結論必須寫：

JOB_USER_NOT_VISIBLE_REASON = NO_PRODUCTION_CUSTOMER_MEMBERSHIP

這不是 Launcher 舊版證據。

如果有 membership：
確認該 email / user_id 是否會在對應 Customer → Members 顯示。

---

# 9. 驗證 Central Product Seats 列表

在 Customer detail 的 Central Product Seats：

確認 UI 能列出該 Customer 的產品 Seat 狀態。

首發產品應以 Production canonical product codes 為準：

post
dm
ocr

不要把 orc 當成 product code。

如果該 Customer 沒有某產品 entitlement：
UI 應合理顯示無 entitlement / 不可 assign，
而不是前端 crash。

輸出：

POST_SEAT_UI =
DM_SEAT_UI =
OCR_SEAT_UI =

UI_RENDER_ERRORS =

---

# 10. Seat assign/release 可用性驗證

先判斷是否已有安全的普通 Production canary member。

如果存在：
可對 canary member 做一次最小可逆測試：

BEFORE
→ assign seat
→ UI refresh
→ verify assigned
→ release seat
→ verify restored to BEFORE

只能測一個明確 canary member。

不得動正式客戶正在使用的 Seat。

如果沒有安全 canary member：
不要建立正式資料來硬測。

輸出：

SAFE_CANARY_MEMBER_AVAILABLE =
SEAT_MUTATION_TEST =
SEAT_ASSIGN_RESULT =
SEAT_RELEASE_RESULT =
STATE_RESTORED =

如果因沒有安全 canary member 而不能做：
標 MANUAL_CANARY_DATA_REQUIRED，不算 UI rendering failure。

---

# 11. 判斷使用者看到「舊介面」的真正原因

只能根據實測選一項：

A. 首頁 chrome 本來相同；新版 Workbench 在 Customer detail，實際正常
B. platform_admin role 沒有正確載入
C. Workbench JS 載入但 runtime state / RPC 失敗
D. Customer detail route/render bug
E. Production data 缺少 membership / entitlement / Seat，所以看不到預期資料
F. 其他有直接證據的原因

輸出：

OLD_UI_OBSERVATION_ROOT_CAUSE_CLASS =
OLD_UI_OBSERVATION_ROOT_CAUSE =

不要再用 source mismatch / stale Cloudflare cache 當根因，除非本次取得新的直接反證。

---

# 12. 最終 Launcher Canary 判定

只有以下全部成立才能 PASS：

- Production Supabase 正確
- platform_admin 真人 session 正確
- admin route 正確
- Customer detail 正常
- Members 正常
- Entitlements 正常
- Central Product Seats 正常 render
- Production RPC 無 blocker
- 沒有 Staging/Production crossover
- 沒有 critical JS error

如果 Seat mutation 因缺安全 canary member 沒做，
可以把：

LAUNCHER_UI_CANARY = PASS
SEAT_MUTATION_CANARY = PENDING

但不能把整個 Seat 功能宣告完整 E2E PASS。

---

# 13. 不要重新部署

若 Launcher Browser Canary PASS：

REDEPLOY_LAUNCHER = NO

不要重建或重部署。

若 FAIL：
先回報精確 root cause。

只有證明 source/runtime code 本身真的需要修，
才提出最小修正方案。

本任務不實作修正。

---

# 最終輸出

只回：

PRODUCTION_URL =
PRODUCTION_SUPABASE_REF =
ENVIRONMENT_CROSSOVER =

AUTHENTICATED =
CURRENT_USER_EMAIL =
CURRENT_USER_ID =
PLATFORM_ROLE =

WORKBENCH_JS_LOADED =
WORKBENCH_RUNTIME_OBJECT_PRESENT =
ADMIN_ROUTE_ACTIVE =
JAVASCRIPT_ERRORS =

CUSTOMER_ID =
CUSTOMER_NAME =
CUSTOMER_TYPE =
CUSTOMER_STATUS =

MEMBERS_SECTION_PRESENT =
ENTITLEMENTS_SECTION_PRESENT =
CENTRAL_PRODUCT_SEATS_PRESENT =
STABLE_USER_ID_VISIBLE =

RPC_ADMIN_LIST_CUSTOMER_MEMBERS =
RPC_EFFECTIVE_ENTITLEMENT_DETAILS =
RPC_ADMIN_LIST_PRODUCT_SEATS =
NETWORK_ERRORS =
RPC_ERRORS =

JOB0975805890_AUTH_USER_EXISTS =
JOB0975805890_USER_ID =
JOB0975805890_CUSTOMER_MEMBERSHIP_EXISTS =
JOB0975805890_MEMBER_CUSTOMER_ID =
JOB0975805890_MEMBER_STATUS =
JOB0975805890_ENTITLEMENTS =
JOB0975805890_SEAT_ASSIGNMENTS =
JOB_USER_NOT_VISIBLE_REASON =

POST_SEAT_UI =
DM_SEAT_UI =
OCR_SEAT_UI =
UI_RENDER_ERRORS =

SAFE_CANARY_MEMBER_AVAILABLE =
SEAT_MUTATION_TEST =
SEAT_ASSIGN_RESULT =
SEAT_RELEASE_RESULT =
STATE_RESTORED =

OLD_UI_OBSERVATION_ROOT_CAUSE_CLASS =
OLD_UI_OBSERVATION_ROOT_CAUSE =

LAUNCHER_UI_CANARY = PASS/FAIL
SEAT_MUTATION_CANARY = PASS/PENDING/FAIL
LAUNCHER_PRODUCTION_ACCEPTED = YES/NO
GO_LIVE_BLOCKED_BY_LAUNCHER = YES/NO
REDEPLOY_LAUNCHER = YES/NO

BLOCKER =
SMALLEST_NEXT_ACTION =

不要修改任何 source / deploy / Production schema。
本任務只完成真實 Production platform_admin Browser Canary。
