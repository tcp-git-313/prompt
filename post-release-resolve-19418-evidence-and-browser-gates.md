# POST RELEASE — RESOLVE 19418 EVIDENCE + BROWSER GATES

## ROLE
POST RELEASE OWNER

## MODEL
LUNA Medium

## MODE
STRICT EXECUTOR MODE — PLAN IS FROZEN

你只能執行本 Prompt。
不得擅自改架構、擴 Scope、替換既定方案、跳過 Gate、重排硬依賴。

若實際狀態與本 Prompt 衝突：
STOP，只回報 CURRENT / EXPECTED / CONFLICT / MINIMUM_OPTIONS。
不得自行選新方案。

## CURRENT VERIFIED STATE

Project:
`F:\00-Ticenpi-SaaS\TicenpiPost`

Branch:
`feat/post-tenant-provisioning`

目前已確認：

- OAuth callback = PASS
- Supabase session = PASS
- Commercial Auth = PASS
- SessionGate fix #2 = PASS
- Main UI = PASS
- Facebook C / Extension unavailable UI = PASS
- 19418 API health = 200
- 9458 → Extension → 19458 = PASS
- DEV / Acceptance DB isolation = PASS
- seat skip fail-closed protections = PASS
- Extension manifest 已包含 19418
- Extension version 0.4.8 已建立
- 4 個 local commits 已建立
- 尚未 push
- 尚未跑 CI
- 尚未進 Staging
- Central Seat 未修改
- Production 未修改

目前剩餘阻塞：

1. bootstrap direct evidence 不足
2. tenant ID stability direct evidence 不足
3. Studio 目前 offline
4. 19418 頁面沒有 Extension handshake

不要重跑已 PASS 項目，除非新證據顯示 regression。

## 1. IMPORTANT PLAN CLARIFICATION

本次解除兩個「證據取得方式」上的假性阻塞，但不降低驗收標準。

### Bootstrap

`POST /api/session/bootstrap` response 目前只回：

- tenant: boolean
- tenant_created

不回 tenant_id。

因此：

不得為了「讓測試容易」修改 production API response 加 tenant_id。

可以用：

- 正式 endpoint HTTP evidence
- backend integration test
- local Acceptance DB direct read
- existing application logs

組合證明 bootstrap 成功與冪等。

### Tenant stability

tenant_id 不必從 bootstrap response 取得。

可以從 Local Acceptance DB 中既有正式 mapping 直接唯讀取得。

不得因此新增 debug endpoint。

## 2. BOOTSTRAP ENDPOINT — VERIFY CONTRACT FIRST

先唯讀：

`backend/app/api/studio.py`

及 bootstrap 直接 dependencies。

確認：

- endpoint path
- auth dependency
- commercial dependency
- tenant ensure function
- existing tenant path
- new tenant path
- response schema
- 是否設計成「只有缺 tenant 時 UI 才呼叫」
- 已有 tenant 的 user 是否正常不會在每次 page load 呼叫 bootstrap

如果「既有 tenant 不會自動 POST bootstrap」是目前正式設計：

記錄：

EXISTING_TENANT_UI_BOOTSTRAP_CALL_REQUIRED = NO

不要把「API log 沒看到 POST」
判定為產品失敗。

## 3. BOOTSTRAP DIRECT EVIDENCE — NO JWT HACKS

不得繞過 Auth。
不得 hardcode JWT。
不得使用 DEV X-Tenant-Key 驗 19418 formal bootstrap。

優先順序：

### Method A — Existing integration test / authenticated test harness

搜尋既有 backend tests 是否已能：

- 建 authenticated user
- 通過 commercial auth
- 呼叫 `POST /api/session/bootstrap`

若已有：
直接執行，不修改 production source。

若沒有，但 repo 既有 test fixtures 已能建立正式 authenticated principal：
允許新增「test-only」coverage，前提是：

- 不改 production behavior
- 不新增 debug endpoint
- 不繞 commercial auth
- 測同一 User 呼叫 bootstrap 兩次
- 驗證只存在一個 tenant mapping

若需新增測試：
獨立 test-only commit，不得混入既有 4 commits。

### Method B — Existing logged-in browser Network evidence

若正常 UI 流程能自然觸發 bootstrap：
可使用 Network / backend access log 證明 HTTP success。

不得為取得證據修改 UI 讓它強制呼叫 bootstrap。

### Method C — Existing backend formal auth helper

只有 repo 既有正式 tooling 能安全取得 current session token 時才可用。
不得自行從瀏覽器秘密儲存區暴力抽 token。

## 4. BOOTSTRAP IDEMPOTENCY EVIDENCE

PASS 必須證明：

同一 authenticated User：

第一次 bootstrap
→ success

第二次 bootstrap
→ success

且：

- tenant mapping count 不增加
- tenant_id 不變
- 不建立第二 tenant
- 不產生 duplicate tenant_members

記錄：

BOOTSTRAP_TEST_METHOD =
BOOTSTRAP_FIRST =
BOOTSTRAP_SECOND =
TENANT_MAPPING_COUNT_BEFORE =
TENANT_MAPPING_COUNT_AFTER =
BOOTSTRAP_IDEMPOTENT = PASS/FAIL

## 5. TENANT ID — USE LOCAL ACCEPTANCE DB DIRECTLY

不要修改 API。

唯讀確認 Local Acceptance 實際 DB container / volume。

從既有 schema 中找到：

`tenant_members`

或 ADR-002 定義的正式 mapping table。

用目前登入 user 的 stable user_id 查：

tenant_id

如果 email → user_id mapping 可從現有：

- profiles
- commercial context
- logs
- known local identity mapping

唯一且明確取得，
可以使用。

如果無法唯一判斷當前登入 user_id：

STOP。

不得猜 user。

記錄：

AUTH_USER_ID =
TENANT_ID_BEFORE_RELOAD =

完整 reload `http://localhost:19418`

再唯讀 DB：

TENANT_ID_AFTER_RELOAD =

再完成 bootstrap idempotency test 後：

TENANT_ID_AFTER_BOOTSTRAP =

PASS：

三者完全一致。

## 6. DO NOT BLOCK BOOTSTRAP/TENANT ON STUDIO OR EXTENSION

Bootstrap / Tenant verification
與 Launcher / Extension browser gate 是兩條獨立 evidence path。

先完成能自動完成的：

- bootstrap contract verification
- bootstrap idempotency
- tenant direct DB stability

不得因 Studio offline
或 content script 未載入
而跳過這些可獨立完成的工作。

## 7. LAUNCHER GATE — USER ACTION ONLY WHEN NEEDED

19418 正式規則：

Launcher / Ticenpi Studio = primary

如果：

`/api/session/state`

仍顯示：

studio_online = false

不要修改 Post source。

先唯讀確認 Studio online detection contract：

- health URL / local port
- timeout
- expected Launcher process
- deep-link / handshake mechanism

如果 detection contract 正常，
且單純是 Launcher 未啟動：

STOP WITH USER ACTION

只要求：

「啟動既有 Ticenpi Launcher / Studio」。

不得：

- 安裝新程式
- 改 detection timeout
- fake studio_online
- bypass Launcher gate

使用者啟動後再測：

studio_online = true

點 Facebook binding：

→ Launcher primary

PASS：

LAUNCHER_PRIMARY_USED = YES
EXTENSION_PRIMARY_TRIGGERED = NO

## 8. EXTENSION 19418 — DIAGNOSE BEFORE SOURCE CHANGE

已知：

MANIFEST_MATCH = YES
CONTENT_SCRIPT_LOADED = NO
EXTENSION_ID_VISIBLE = NO
PAGE_HANDSHAKE = 未成功

不要直接改 manifest。

先做只讀差異診斷：

### A. Same-browser control test

在「同一個 Chrome profile / 同一個 browser instance」：

1. 開 `http://localhost:9458`
2. 檢查 Extension ID/version handshake
3. 開 `http://localhost:19418`
4. 檢查同一 handshake

記錄：

9458_EXTENSION_DETECTED =
19418_EXTENSION_DETECTED =

### B. Interpret

如果：

9458 = NO
19418 = NO

優先判定：

BROWSER_PROFILE_OR_EXTENSION_RUNTIME_ISSUE

不要改 Post source。

如果：

9458 = YES
19418 = NO

才繼續查：

- content script match
- runtime site access
- extension content.js initialization condition
- page origin checks
- externally_connectable handshake
- console/runtime errors

仍然先只讀。

如果：

9458 = YES
19418 = YES

直接進 Extension fallback E2E。

## 9. MANUAL CHROME EXTENSION GATE

如果自動 browser harness
無法控制 / 觀察真正使用者 Chrome profile 的 Extension：

不要把它誤判成產品 FAIL。

回報：

MANUAL_BROWSER_REQUIRED = YES

並 STOP WITH USER ACTION。

使用者需在實際 Chrome：

1. `chrome://extensions`
2. 確認 TicenpiPost = 0.4.8
3. Enabled
4. Reload
5. 開 `http://localhost:9458`
6. 確認 DEV 頁能偵測 Extension
7. 開 `http://localhost:19418`
8. Ctrl+Shift+R
9. 確認 19418 頁能偵測 Extension

如果 Chrome 顯示 Site access 設定，
只要求使用者確認 localhost:19418 被允許。

不得自行擴大權限到不必要網站。

## 10. EXTENSION FALLBACK E2E

只有在：

19418_EXTENSION_DETECTED = YES

後執行。

讓 Studio offline。

19418：
點 Facebook bind

預期：

Studio offline
→ Extension detected
→ Extension fallback
→ formal bind token
→ one-shot backend = 19418

必須證明：

BIND_REQUEST_AUTH = Bearer JWT / formal authenticated flow
BIND_TOKEN = tenant-bound
ONE_SHOT_TARGET = http://localhost:19418
DEV_X_TENANT_KEY_USED = NO
BACKGROUND_TARGET_CHANGED = NO

不得因 fallback 永久改 background auto-sync target。

## 11. LOCAL ACCEPTANCE PASS GATE

以下全部 PASS 才允許 Push：

BOOTSTRAP_CONTRACT = PASS
BOOTSTRAP_IDEMPOTENT = PASS
TENANT_ID_STABLE = PASS
LAUNCHER_PRIMARY = PASS
EXTENSION_FALLBACK = PASS
FORMAL_BIND_TOKEN = PASS
ONE_SHOT_TARGET_19418 = PASS
BACKGROUND_TARGET_NOT_POLLUTED = PASS
EXTENSION_UNAVAILABLE_UI = PASS

若只剩需要使用者操作的 Launcher / Chrome Extension：

先把所有可自動驗證 evidence 做完，
然後一次性回報 USER ACTION，
不要一項一項打斷。

## 12. AFTER LOCAL PASS

全部 PASS 後，回到既有 frozen POST RELEASE plan：

Push
→ CI
→ immutable ARM64 image
→ Staging deploy
→ Post Staging self-gate
→ 等 CENTRAL_SEAT_STAGING_READY

不得進 Production，
直到 Central Seat Session 明確回報：

`CENTRAL_SEAT_STAGING_READY = PASS`

## 13. GIT RULES

目前 4 commits 不得：

- squash
- rebase
- amend
- force push

除非本次為 bootstrap coverage
新增純 test-only commit。

若新增 test-only commit：

commit message 建議：

`test(post): prove session bootstrap idempotency`

只准包含直接 bootstrap / tenant evidence tests。

不得混 production source。

## 14. NO PLAN CHANGE

你認為有更好的方案時：

不得直接做。

只回：

PROPOSED_DEVIATION =
WHY =
IMPACT =

然後 STOP。

## 15. REPORT FORMAT

完成自動調查後只回：

PHASE = 19418 REMAINING ACCEPTANCE
RESULT = PASS / USER_ACTION / HARD_STOP

BOOTSTRAP_CONTRACT =
BOOTSTRAP_TEST_METHOD =
BOOTSTRAP_IDEMPOTENT =

AUTH_USER_ID =
TENANT_ID_BEFORE_RELOAD =
TENANT_ID_AFTER_RELOAD =
TENANT_ID_AFTER_BOOTSTRAP =
TENANT_ID_STABLE =

STUDIO_ONLINE =
LAUNCHER_PRIMARY =

9458_EXTENSION_DETECTED =
19418_EXTENSION_DETECTED =
MANUAL_BROWSER_REQUIRED =

EXTENSION_FALLBACK =
FORMAL_BIND_TOKEN =
ONE_SHOT_TARGET_19418 =
BACKGROUND_TARGET_NOT_POLLUTED =

COMMITS =
GIT_STATUS =

NEXT =

如果 RESULT = USER_ACTION：
一次列出所有需要使用者做的動作。

如果 RESULT = PASS：
直接繼續 Push → CI → Staging，
不需要再次詢問。
