# POST RELEASE — CONTINUE FROM 19418 ACCEPTANCE GATE

## ROLE
POST RELEASE OWNER

## MODE
STRICT EXECUTOR MODE — PLAN IS FROZEN

你只能執行既定計畫，不得擅自重新規劃、改架構、擴大 scope、跳過 gate、重排硬依賴。

若實際狀態與本計畫衝突：
STOP，僅回報 CURRENT / EXPECTED / CONFLICT / MINIMUM_OPTIONS。
不得自行選新方案。

---

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
- Facebook C / Extension unavailable = PASS
- DEV 9458 → Extension → 19458 = PASS
- DEV 與 Acceptance DB isolation = PASS
- unknown/empty environment + seat_policy=skip = fail closed
- Production Supabase + seat_policy=skip = fail closed
- Extension version 0.4.8 已完成 Reload 驗證
- 目前已有 4 個 local commits
- 尚未 push
- 尚未跑 CI
- 尚未進 Staging
- Production 未修改
- Central Seat 未修改

不要重跑已 PASS 項目，除非後續證據顯示 regression。

---

## 1. FIXED EXTENSION TOPOLOGY

不得混淆 frontend injection 與 backend target。

DEV：

Browser page:
`http://localhost:9458`

Extension injected into:
`9458`

One-shot backend target:
`http://localhost:19458`

ACCEPTANCE：

Browser page:
`http://localhost:19418`

Extension injected into:
`19418`

One-shot backend target:
`http://localhost:19418`

PRODUCTION：

Browser page:
`https://post.ticenpi.com`

Extension injected into:
`post.ticenpi.com`

One-shot backend target:
`https://post.ticenpi.com`

`19458` 只是 DEV backend。
不得把 Extension content script 改成注入 19458。

---

## 2. REMAINING LOCAL GATES ONLY

現在只補：

A. bootstrap direct evidence

B. tenant ID stability evidence

C. Facebook A：
Launcher online → Launcher primary

D. Facebook B：
Launcher offline + Extension installed → Extension fallback

不要重新規劃。

---

## 3. BOOTSTRAP DIRECT EVIDENCE

不得為取得證據修改 production source。

優先使用：

- browser network evidence
- existing API logs
- 或目前登入 session / JWT 直接呼叫正式 endpoint

直接驗證：

`POST /api/session/bootstrap`

要求：

- HTTP success
- Commercial authorization PASS
- resolved tenant 與 current authenticated user 一致
- endpoint idempotent

同一 User 呼叫兩次：

- 不建立第二 Tenant
- tenant_id 不變

記錄：

BOOTSTRAP_HTTP =
BOOTSTRAP_TENANT_ID =
BOOTSTRAP_IDEMPOTENT = PASS/FAIL

---

## 4. TENANT STABILITY

取得目前 authenticated User 的 tenant_id。

優先：

`GET /api/session/state`

或 repo 既有正式 endpoint。

記錄：

TENANT_BEFORE_REFRESH =

完整 reload 19418。

重新取得：

TENANT_AFTER_REFRESH =

再呼叫一次 bootstrap：

TENANT_AFTER_BOOTSTRAP =

要求：

TENANT_BEFORE_REFRESH
=
TENANT_AFTER_REFRESH
=
TENANT_AFTER_BOOTSTRAP

不得只用「帳號清單相同」當 tenant evidence。

---

## 5. FACEBOOK A — LAUNCHER ONLINE

19418 正式規則：

Launcher / Ticenpi Studio = primary
Extension = fallback

先依既有 contract 確認 Studio online。

不要重寫 Launcher。

如果目前 Studio offline：

STOP WITH USER ACTION

只要求使用者啟動既有 Ticenpi Launcher / Studio。

不得要求安裝新軟體。
不得修改架構。

Studio online 後：

19418
→ 點 Facebook binding
→ Launcher / Studio primary

要求：

- Extension 不得搶 primary
- 使用既有 bind-request contract
- 不建立第二套 cookie/account model

記錄：

STUDIO_ONLINE =
LAUNCHER_PRIMARY_USED =
EXTENSION_PRIMARY_TRIGGERED = YES/NO

PASS：

STUDIO_ONLINE = YES
LAUNCHER_PRIMARY_USED = YES
EXTENSION_PRIMARY_TRIGGERED = NO

---

## 6. FACEBOOK B — EXTENSION FALLBACK

接著驗：

Launcher offline
+
Extension 0.4.8 installed

先唯讀確認：

`extension/manifest.json`

19418 是否存在於必要的：

- content_scripts.matches
- externally_connectable.matches
- host_permissions

只核對實際需要的 permission。
不得無必要擴權。

若 manifest 已正確支援 19418，但頁面沒偵測到 Extension：

STOP WITH USER ACTION

要求：

1. chrome://extensions
2. TicenpiPost Extension
3. 確認版本 0.4.8
4. Reload
5. 回到 `http://localhost:19418`
6. 完整重新整理

不要改程式掩蓋瀏覽器 extension reload 問題。

---

## 7. FALLBACK CONTRACT

Launcher / Studio offline 時：

19418
→ 點 Facebook binding
→ Studio offline
→ Extension detected
→ Extension fallback
→ formal bind token
→ one-shot backend = 19418

要求證明：

EXTENSION_DETECTED = YES

BIND_REQUEST_AUTH =
Bearer JWT / formal authenticated flow

BIND_TOKEN =
tenant-bound

ONE_SHOT_TARGET =
http://localhost:19418

DEV_X_TENANT_KEY_USED =
NO

BACKGROUND_TARGET_CHANGED =
NO

不得因這次 fallback
永久覆寫 background auto-sync target。

---

## 8. LOCAL ACCEPTANCE PASS CONDITION

以下全部 PASS 才可 push：

BOOTSTRAP_DIRECT = PASS
BOOTSTRAP_IDEMPOTENT = PASS
TENANT_REFRESH_STABLE = PASS
LAUNCHER_PRIMARY = PASS
EXTENSION_FALLBACK = PASS
FORMAL_BIND_TOKEN = PASS
ONE_SHOT_TARGET_19418 = PASS
BACKGROUND_TARGET_NOT_POLLUTED = PASS
EXTENSION_UNAVAILABLE_UI = PASS

若任一 FAIL：
STOP，不得 push。

---

## 9. PUSH

Local acceptance 全 PASS 後：

確認：

`git status`

必須乾淨。

確認目前 4 個既有 commits scope 正確。

不得：

- squash
- rebase
- force push
- 改寫 commit history

然後：

`git push`

記錄：

PUSHED_BRANCH =
FINAL_LOCAL_COMMIT =

---

## 10. CI

Push 後依既有 Post CI 流程執行。

要求：

backend = PASS
frontend = PASS
migration tests = PASS
ARM64 image build = PASS

不得建立第二套 CI。

取得：

SOURCE_COMMIT =
IMAGE_DIGEST =
CI_RUN =
RELEASE_EVIDENCE =

任何 CI FAIL：

STOP。

先分辨：

NEW_REGRESSION
或
PRE_EXISTING_FAILURE

不得跳過。

---

## 11. IMMUTABLE IMAGE

後續 Staging 只能使用 CI 產生的 immutable ARM64 image。

禁止：

- latest
- local build
- mutable tag

記錄 exact IMAGE_DIGEST。

---

## 12. STAGING DEPLOY

依既有：

`F:\00-Ticenpi-SaaS\deploy`

以及既有 PRODUCT_TO_STAGING / central deployment policy。

不得重新設計 deploy。

先：

- preflight
- dry-run
- manifest isolation check

如果發現：

- unrelated service drift
- central manifest collision
- permission-required VPS operation

STOP。

不得 reset / overwrite 其他產品。

若 VPS deploy 必須由使用者執行：

STOP WITH USER ACTION

只提供唯一精確 command。

---

## 13. POST STAGING SELF-GATE

部署後驗：

- health
- runtime identity
- source commit
- image digest
- Google OAuth
- Supabase session
- Commercial Auth
- SessionGate fix #2
- bootstrap
- tenant stability
- Launcher primary
- Extension fallback
- budget tenant isolation
- tasks
- posting smoke
- foreign tenant spoof

目前 Central Seat 尚未 READY 時：

Post Staging 可以暫時沿用既定：

`TICENPI_SEAT_POLICY=skip`

只驗 Post 自身。

不得因此進 Production。

---

## 14. CENTRAL SEAT HARD STOP

完成 Post 自身 Staging Gate 後：

檢查：

CENTRAL_SEAT_STAGING_READY

若不是 PASS：

STOP。

只回報：

POST_STAGING_READY_FOR_SEAT = YES
CENTRAL_SEAT_STAGING_READY = NO
PRODUCTION_BLOCKED = YES

不得：

- 自己做 Central Seat
- 修改 shared central schema
- 建 Post 專屬 Seat SSOT
- 提前進 Production

---

## 15. WHEN CENTRAL SEAT READY

只有當另一個 Central Seat Session 明確回報：

`CENTRAL_SEAT_STAGING_READY = PASS`

才繼續：

- 讀 Central Seat consumer contract
- Post 接正式 status contract
- Staging seat_policy=require
- Business assigned seat E2E
- Business unassigned deny
- Seat release ≤30 sec deny
- Individual account E2E
- Device token entitlement revalidation closure
- Final Staging Gate
- Production preflight
- Production deploy

不得猜 RPC 名稱或 schema。

---

## 16. NO PLAN CHANGES

你是 Executor。

若你認為有更好的方案：

不得直接改。

只能回報：

PROPOSED_DEVIATION =
WHY =
IMPACT =

然後 STOP。

---

## 17. PROGRESS REPORT

每個大 Phase 只回：

PHASE =
RESULT = PASS/FAIL/BLOCKED
EVIDENCE =
COMMITS =
TESTS =
NEXT =

不要長篇重述整份計畫。
