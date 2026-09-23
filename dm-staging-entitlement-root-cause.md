# DM STAGING ENTITLEMENT ROOT CAUSE OWNER

## 任務目標

目前 DM Staging 已確認：

- Google / Supabase 登入成功
- App 內 protected request 有帶 Bearer
- Session propagation 正常
- `GET /api/designs` 實際回：
  `403 PRODUCT_ENTITLEMENT_REQUIRED`

因此本任務不要再查 frontend login、cookie、session hydration、Bearer propagation。

真正要追的是：

User
→ profile
→ customer / organization
→ membership
→ DM entitlement
→ Seat applicability
→ platform_admin exemption
→ /api/designs

目標只有一個：

> 找出目前登入的 Staging 測試帳號為什麼沒有通過 DM entitlement gate，判斷是測試資料缺漏、商用授權鏈設定問題，還是 DM entitlement 判斷 bug，並以最小風險修正到 `/api/designs` 正常通過。

完成後才繼續既有 Shared Engine / YUCT closeout。

---

# 已確認，不要重查

以下已經有 live Staging evidence：

- DM Staging release：20260923-215914
- Staging health：PASS
- App request Authorization：PRESENT
- `/api/designs`：
  - HTTP 403
  - code = PRODUCT_ENTITLEMENT_REQUIRED
- 直接在瀏覽器導覽 `/api/designs` 出現 MISSING_AUTH_TOKEN 不代表 App request 缺 token
- frontend/backend Staging Supabase project 一致
- Local dev 與 Staging auth path 不同
- Session propagation 並非目前 blocker

不要重新把問題導回：

- MISSING_AUTH_TOKEN
- stale cookie
- frontend auth hydration
- Bearer propagation
- Shared Engine
- ExtractionHub
- 591 / Sign cookie isolation

除非新的直接 evidence 推翻上述結論。

---

# 591 / Sign 處理原則

本任務不要修改 591 或 Sign。

目前只知道：

- 591 / Sign 尚未完全收斂到中央統一架構
- Staging cookie / session isolation policy 與 DM/Post 存在差異風險
- 尚未證明實際 session pollution

因此本輪規則：

591 = READ ONLY
Sign = READ ONLY

不要現在 patch 舊架構。

只在報告中留下 migration checklist：

- central auth client
- session storage policy
- Staging cookie isolation
- entitlement / Seat
- platform_admin behavior
- CI contract tests

等中央架構定型後一次收斂。

---

# ExtractionHub 處理原則

本輪不要修改 ExtractionHub。

目前 Shared Engine 主要以 wheel/library 方式被產品使用。

ExtractionHub 未來若變成跨產品 HTTP service，再另做：

- service authentication
- product boundary
- tenant boundary
- caller authorization

這不阻塞目前 DM entitlement root cause。

---

# 零風險平行策略

允許平行 READ-ONLY audit：

TRACK A — Current User / Profile / Membership
TRACK B — DM Entitlement Data Chain
TRACK C — Seat Applicability
TRACK D — platform_admin Exemption / Backend Decision

四個 Track 不得修改任何資料或程式。

完成後：

READ-ONLY PARALLEL AUDIT
↓
ROOT CAUSE SYNTHESIS
↓
SINGLE WRITER / SINGLE DATA CHANGE OWNER
↓
TEST
↓
STAGING RETEST

禁止四個 Track 各自修。

---

# 工作範圍

主要：

F:\00-Ticenpi-SaaS\TicenpiDM

可只讀參考：

F:\00-Ticenpi-SaaS\ticenpi-platform
F:\00-Ticenpi-SaaS\deploy
F:\00-Ticenpi-SaaS\TicenpiPost

Central commercial schema / RPC 可查。

Post 只能作 reference，不要修改。

Production：

OUT OF SCOPE

Production DB：

OUT OF SCOPE

---

# PHASE 0 — PREFLIGHT

記錄：

- DM branch
- DM HEAD
- git status
- git diff
- staged diff
- untracked files
- active writer evidence

保留所有既有 dirty WIP。

禁止：

- reset --hard
- clean
- stash
- rebase

確認 Staging：

- current release
- /api/health
- runtime identity
- current environment
- current Supabase project identity

不要輸出：

- JWT
- cookie
- refresh token
- secrets

輸出：

DM ENTITLEMENT PREFLIGHT PASS

或：

DM ENTITLEMENT PREFLIGHT BLOCKED

---

# TRACK A — CURRENT USER / PROFILE / MEMBERSHIP

只讀查目前登入 Staging 帳號的安全 identity summary。

不要輸出完整敏感資訊。

至少確認：

- authenticated user id
- profile 是否存在
- platform role
- customer / organization memberships
- membership status
- active/inactive
- 是否屬於 individual account
- 是否屬於 business account

建立：

CURRENT USER COMMERCIAL CONTEXT

User
↓
Profile
↓
Membership(s)
↓
Customer / Organization

如果 profile/membership 缺漏，直接標記 evidence。

---

# TRACK B — DM ENTITLEMENT DATA CHAIN

只讀追目前 DM entitlement 的正式 source of truth。

找出：

- entitlement table / RPC
- product key
- subscription / plan mapping
- active status
- start/end date
- customer linkage
- organization linkage
- individual linkage
- Staging-specific seed / fixture

建立：

DM ENTITLEMENT CHAIN

Customer / Organization
↓
Subscription
↓
Plan
↓
Product
↓
Entitlement
↓
DM allow/deny

回答：

1. 目前登入使用者所屬 commercial entity 是否存在 DM entitlement？
2. entitlement 是否 active？
3. product key 是否正確？
4. 是否掛錯 customer / organization？
5. 是否存在但時間已失效？
6. 是否 entitlement RPC 沒讀到？
7. 是否 Staging 測試資料根本未建立？

若資料缺漏：

分類：

STAGING_DATA_MISSING

不要立刻改 DB，等四個 Track 合併後再決定。

---

# TRACK C — SEAT APPLICABILITY

只讀確認 DM 對目前 account type 是否需要 Seat。

不要假設所有 entitlement 都需要 Seat。

建立：

SEAT DECISION

Account type
↓
Entitlement present?
↓
Seat model applicable?
↓
Seat assigned?
↓
Allow / deny

明確回答：

- individual account 是否需要 Seat
- business account 是否需要 Seat
- platform_admin 是否 Seat exempt
- DM 現行 route 是否已接 Central Seat authoritative gate
- `PRODUCT_ENTITLEMENT_REQUIRED` 是否可能在 Seat gate 之前就已返回

若目前根本沒有 entitlement：

Seat = NOT YET APPLICABLE

不要把問題誤判為 Seat。

---

# TRACK D — PLATFORM_ADMIN EXEMPTION / BACKEND DECISION

只讀追：

`GET /api/designs`

實際 dependency chain。

建立：

Bearer
↓
JWT validation
↓
current user
↓
platform_admin?
↓
commercial context
↓
DM entitlement
↓
Seat if applicable
↓
allow / 403

確認：

1. platform_admin 在現行 DM contract 是否應豁免 entitlement？
2. 若應豁免，實作是否真的先於 entitlement gate？
3. 若不豁免，這是既有設計還是 bug？
4. `PRODUCT_ENTITLEMENT_REQUIRED` 由哪一個 dependency / RPC 產生？
5. 目前登入帳號 backend 實際辨識的 platform role 是什麼？

如果：

platform_admin 應豁免
但 backend 仍走 entitlement deny

分類：

PLATFORM_ADMIN_EXEMPTION_BUG

如果：

platform_admin 本來就不豁免

不要自行改 policy，需依中央既有 contract。

---

# PHASE 1 — ROOT CAUSE SYNTHESIS

四個 Track 完成後，根因只能從 evidence 判定。

分類：

A. STAGING_DATA_MISSING
B. MEMBERSHIP_MISSING
C. WRONG_CUSTOMER_OR_ORG_LINK
D. ENTITLEMENT_MISSING
E. ENTITLEMENT_INACTIVE
F. PRODUCT_KEY_MISMATCH
G. ENTITLEMENT_RPC_BUG
H. PLATFORM_ADMIN_EXEMPTION_BUG
I. SEAT_REQUIRED_NOT_ASSIGNED
J. SEAT_GATE_ORDER_BUG
K. OTHER
L. MULTIPLE_CAUSES

輸出：

DM ENTITLEMENT ROOT CAUSE

Evidence:
Current user commercial context:
Expected:
Actual:
Minimal correction:

如果證據不足：

HARD STOP

DM ENTITLEMENT ROOT CAUSE INCONCLUSIVE

---

# PHASE 2 — DECIDE DATA FIX VS CODE FIX

## Case A — Staging 測試資料缺漏

例如：

- membership 不存在
- DM entitlement 不存在
- entitlement inactive
- 測試 account 沒有正確 customer/org linkage
- business account 需要 Seat 但未 assigned

這種情況：

優先使用既有正式 admin / RPC / seed flow 補資料。

不要直接手改任意 table，除非現有正式流程不存在。

任何 Staging data change 必須記錄：

- before
- intended contract
- operation
- after
- rollback/revert method

禁止碰 Production。

## Case B — 程式 bug

例如：

- entitlement RPC 明明應 allow 卻 deny
- product key mapping 錯誤
- platform_admin exemption implementation 與中央 contract 不一致
- Seat gate order 錯誤

才允許修改 code。

採：

single writer

最小修正

補 regression

走正式 CI / Staging deploy

---

# PHASE 3 — TEST MATRIX

至少驗：

## Identity / Membership

- current user resolves
- correct profile
- correct membership
- inactive membership denied

## Entitlement

- no DM entitlement → deny
- active DM entitlement → allow
- inactive entitlement → deny
- wrong product → deny

## Account Type

- individual
- business

依現行正式 contract 驗證。

## Seat

只有 Seat applicable 才測：

- assigned → allow
- unassigned → deny
- release/reassign behavior if existing tests support it

不要為了本任務重新設計 Central Seat。

## platform_admin

依中央既有正式 contract 驗：

- expected exemption behavior
- DM implementation matches central policy

---

# PHASE 4 — STAGING RETEST

使用同一正式 App flow。

不要直接導覽 API URL 當 auth proof。

重新登入 / reload 後：

`GET /api/designs`

要求：

- Authorization present
- 不再回 PRODUCT_ENTITLEMENT_REQUIRED

若 current test account 應有 DM 使用權：

Expected:

HTTP 200

如果改成另一個 typed denial：

記錄新的 code，不要掩蓋。

---

# PHASE 5 — SHARED ENGINE IDENTITY

只有 `/api/designs` entitlement gate 通過後才繼續。

用具 platform_admin 權限的正式 Staging session 驗：

`GET /api/admin/shared-engine/identity`

Expected identity：

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

若 200 + identity match：

SHARED ENGINE IDENTITY VERIFIED

---

# PHASE 6 — NEXT CLOSEOUT GATE

若以下全部 PASS：

- authenticated session
- /api/designs authorized
- platform_admin identity endpoint verified
- Shared Engine identity match

輸出：

DM COMMERCIAL AUTH GATE PASS

接續既有：

YUCT UI smoke
↓
Shadow
↓
Legacy primary
↓
rollback
↓
restore candidate
↓
READY

本提示詞可以在 entitlement gate PASS 後停止，不必自行擴大任務。

---

# 591 / SIGN MIGRATION CHECKLIST

只記錄，不修改。

未來兩產品正式接中央統一架構時必須一次處理：

- authoritative Supabase/client config
- central auth client pattern
- auth/session lifecycle
- Production vs Staging cookie isolation
- centralized protected API client
- entitlement contract
- Seat applicability
- platform_admin policy
- tenant isolation
- CI auth contract tests
- Staging E2E
- Production cutover

不要現在為舊架構做局部 patch。

---

# EXTRACTIONHUB FUTURE CHECKLIST

只記錄，不修改。

若未來改為共享 HTTP service，需先完成：

- service authentication
- calling product identity
- tenant boundary
- authorization
- request audit
- rate/abuse controls
- internal/private network policy
- no cross-tenant result exposure

目前 wheel/library 使用模式不阻塞 DM。

---

# FINAL REPORT FORMAT

# DM STAGING ENTITLEMENT ROOT CAUSE REPORT

## 1. Baseline

Branch:
HEAD:
Staging release:
Health:
Collision:

## 2. Current User Context

Authenticated:
Profile:
Platform role:
Membership:
Account type:
Customer / organization:

## 3. DM Entitlement

Product:
Entitlement exists:
Active:
Linked entity:
Expected:
Actual:

## 4. Seat

Applicable:
YES / NO

Assigned:
YES / NO / N/A

Seat blocker:
YES / NO

## 5. platform_admin

Expected policy:
Backend actual:
Exemption correct:
YES / NO / N/A

## 6. Root Cause

Classification:
Evidence:
Minimal correction:

## 7. Change

Data fix:
YES / NO

Code fix:
YES / NO

Files:
DB objects:
Staging only:
YES

Production touched:
NO

## 8. Tests

Membership:
Entitlement:
Account type:
Seat:
platform_admin:
Backend regression:

## 9. Staging Retest

/api/designs:
Authorization present:
YES

HTTP:
Response code:

PRODUCT_ENTITLEMENT_REQUIRED resolved:
YES / NO

## 10. Shared Engine Identity

HTTP:
Expected identity match:
PASS / FAIL / NOT_RUN

## 11. Deferred Architecture Work

591:
DEFERRED TO CENTRAL ARCHITECTURE MIGRATION

Sign:
DEFERRED TO CENTRAL ARCHITECTURE MIGRATION

ExtractionHub HTTP auth boundary:
DEFERRED UNTIL SHARED SERVICE MODE

## 12. Isolation

Central Seat redesign:
NONE

Post:
UNCHANGED

591:
UNCHANGED

Sign:
UNCHANGED

ExtractionHub:
UNCHANGED

Production:
UNCHANGED

Production DB:
UNCHANGED

## 13. Final Result

只能：

DM STAGING ENTITLEMENT ROOT CAUSE FIXED

或

DM STAGING ENTITLEMENT ROOT CAUSE BLOCKED

---

# HARD STOP

不要：

- 再修 frontend auth/session，除非有新的直接 evidence
- 修改 591
- 修改 Sign
- 修改 ExtractionHub
- 重構 Central Seat
- 修改 Production
- 修改 Production DB
- 切 Core Primary
- 刪 Legacy scraper

若 `/api/designs` 200 且 Shared Engine identity 驗證通過，停止並回報，等待 YUCT UI smoke / rollback closeout。
