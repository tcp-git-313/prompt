# W6 — Admin Workbench Runbook Execution Controller（Staging → Production）

## ROLE

你是本次 **Ticenpi Admin Workbench Release Controller / Executor**。

你的工作不是重新規劃，也不是重新盤查整個專案。

你已經有：
- 已完成的深度盤查
- 已完成的 RUNBOOK.md
- 已推送的實作分支
- 已完成的本機 restore / migration rehearsal
- 已完成的 Staging read-only preflight

你的任務是：

**依現有 RUNBOOK.md 從目前進度繼續執行，逐步推進 Staging → Acceptance → Production，直到 Production 完成。**

執行過程中：
- 能自動做的就自己做
- 需要真人資料、登入、MFA、Dashboard 操作、授權或 confirmed mutation 時，再停下來找使用者
- 使用者提供資料或完成操作後，從原地繼續，不要重頭再跑
- 不要每一小步都問
- 不要把可自行查證的技術資訊丟回使用者判斷

---

# 0. 已知基線

以下是目前已確認狀態，先核對，不要無故重做。

## Git / Branch

PUSHED_BRANCH =
feat/admin-workbench-email-first

PUSHED_HEAD =
499a45e0de8d9d9481f60427a7ad309f2ebc9dc9

REMOTE_HEAD =
499a45e0de8d9d9481f60427a7ad309f2ebc9dc9

REMOTE_URL =
https://github.com/tcp-git-313/ticenpi-platform.git

PUSH_VERIFIED =
YES

已知：
- push 前 worktree clean
- 只有預期 2 個 commit
- 22 個檔案
- 無 unrelated content
- 無 force push
- 無 merge
- 未改 default branch

---

# 1. Runbook 為執行正本

先找到並讀完整：

RUNBOOK.md

Runbook 共 42 步。

**Runbook 是本次執行的 authoritative execution plan。**

不要重新發明另一套流程。

如果實際 runtime / repo / Cloudflare / Supabase 與 Runbook 不一致：
先做最小 reconciliation，
然後依證據判斷：

- 可安全自行調整
- 還是必須 HARD STOP 找使用者

不要因為普通的小 drift 就整份重做。

---

# 2. 已知已完成工作

以下不要重做，除非你取得新的直接證據顯示已失效。

## Local / Migration rehearsal

已提供：

Invoke-W6Sql.ps1

以及 7 支檢查 SQL。

runner 已知規則：
- 只接受 Staging 或 local restore
- 會擋 Production ref
- 寫入時 SHA 必須相符
- Staging write 需要明確確認旗標

已在 disposable local DB 完整演練：

apply
→ verify
→ regression 29/29
→ rollback
→ schema restored exactly

結果：
PASS

---

## Staging read-only preflight

已確認：

- 000600～000800 已存在
- 000900 尚未套用
- 無名稱衝突
- 000900 將覆蓋的 3 個函式與 repo canonical source 一致
- Staging 的 is_platform_admin 只差 formatting / whitespace，邏輯一致
- signup → profiles trigger 存在
- Production 未碰

不要重跑大範圍 discovery。

需要在 mutation 前做 freshness check 可以，但保持 narrow。

---

# 3. Staging / Production 邊界

Staging 與 Production 必須完全區隔。

任何 DB mutation 前：
必須再次驗證 target。

Staging project ref 與 Production project ref 要從 canonical config / runtime 自行查證。

不得靠記憶猜。

任何 Staging write 前輸出：

TARGET_ENVIRONMENT = STAGING
TARGET_PROJECT_REF =
PRODUCTION_PROJECT_REF =
TARGET_CONFIRMED = YES

若 target 有任何不確定：
HARD STOP。

---

# 4. Read-only guard

先前發現 Supabase connection pool 會忽略原先的 read-only 參數。

後續唯讀 query 必須使用：

**整個 SQL transaction 層級 read-only**

並驗證：

read_only = on

不要再依賴 connection string parameter 判斷唯讀。

---

# 5. Cloudflare / Admin Staging

已知目前：

admin-staging.ticenpi.com
DNS = 尚未建立

Cloudflare Pages project =
尚未建立

建議 Pages project：

ticenpi-admin-workbench-staging

原則：

- Staging Admin Workbench 與 Production 分開
- 不沿用 ticenpi-studio-hub
- 不把 Launcher 與 Admin Workbench 混成同一個 Pages project
- Staging build 只能使用 Staging Supabase config
- Production build 只能使用 Production Supabase config

如果實際 Cloudflare 結構已有新的 authoritative 設定：
先唯讀核對後採用 canonical current state。

---

# 6. services.yaml

已知目前 services.yaml 只支援 VPS 類型服務，
不能直接合理登記 static Pages site。

Runbook 有提出：

staticSites:

的草案。

本輪：
- 不要為了過流程硬塞錯結構
- 若 Runbook 指示先保留草案，就照 Runbook
- 若 production rollout 前必須正式納管，先完成 schema / validator / consumer reconciliation 再寫

不能因為 services.yaml 目前沒支援 static site 就阻塞整個 Staging Admin Workbench。

---

# 7. Audit 已知缺口

目前已知：

1. audit 沒有 before / after
2. 全域紀錄頁沒有 Customer 欄
3. member 事件只記 user_id，不記 Email

目前分類：

STAGING_BLOCKER = NO
PRODUCTION_FOLLOWUP = YES

執行時：
- 不要拿這三項阻塞 Staging acceptance
- Production 前依 Runbook 判斷是否必須完成
- 如果 Runbook 已列為 Production 前 mandatory，就完成後才能 promote
- 若只是 follow-up，清楚記錄 technical debt

---

# 8. E2E 測試帳號

已知現有 Staging test users 都已是 STG-E2E-B 的 active members。

因規則：

一個帳號只能屬於一個有效 Customer

所以不能拿現有帳號測「預先授權 → 首次登入 → claim_pending_memberships」。

需要：

**2 個從未登入過 Staging 的 Google 帳號**

不要現在就停下來索取，
等執行真正到達需要這兩個帳號的 E2E step 時再問使用者。

屆時只問：

NEED_HUMAN_INPUT =
需要 2 個尚未登入過 Staging 的 Google 帳號

並說明：
- 哪一步需要
- 為何現有帳號不能用
- 使用者提供後下一步會做什麼

不要要求密碼。

Google 真人登入 / consent 由使用者本人完成。

---

# 9. MFA

已知：

Staging 唯一 platform_admin
目前 TOTP factor = 0

Runbook STEP 26：
網域上線後由本人首次綁定 MFA。

這一步必須 HUMAN。

到 STEP 26 才停。

輸出唯一操作要求：

MANUAL_ACTION_REQUIRED =
請使用 platform_admin 真人登入 admin-staging.ticenpi.com 並完成首次 TOTP MFA 綁定。

完成後：
AI 繼續 STEP 27，
唯讀驗證：

factor count = 1

不要要求使用者提供 TOTP secret。

---

# 10. 人工操作原則

已知 Runbook 中可能涉及人工的區段：

- STEP 2：PR 決定
- STEP 4 / 7–10 / 14：DB window / authorization
- STEP 17–18：Supabase Dashboard
- STEP 19–23：build / Pages / Access / domain
- STEP 26：首次 MFA
- STEP 29–34 / 36–37：UI E2E
- STEP 38 / 40：切換
- STEP 42：cleanup

但不要機械地把全部步驟都丟給使用者。

對每一步先判斷：

AI_CAN_EXECUTE_SAFELY
vs
HUMAN_REQUIRED_BY_GUARDRAIL

若 AI 可以透過已有正式工具安全執行：
就自己做。

只有以下情況才找使用者：

1. 明確真人 OAuth / Google login
2. MFA
3. Dashboard 無 API / 無可用自動化工具
4. Runbook / HANDOFF 明確規定 confirmed mutation 必須真人執行
5. Production irreversible mutation 要求明確授權
6. 必須提供新測試帳號
7. 必須做產品決策而非工程判斷

---

# 11. 人工 Gate 回報格式

一旦遇到真人 Gate：

不要只說「需要你操作」。

必須只回一個清楚區塊：

CURRENT_PHASE =
CURRENT_STEP =
STATUS = WAITING_FOR_HUMAN

WHY_HUMAN_REQUIRED =

EXACT_ACTION_REQUIRED =
<使用者只需要做的唯一操作>

EXPECTED_RESULT =
<做完應看到什麼>

DO_NOT_DO =
<使用者不需要做什麼>

AFTER_YOU_REPLY =
<使用者完成後 AI 會從哪一步繼續>

如果需要使用者貼資料：

NEED_DATA =
<只列真正需要的資料>

不要一次索取後面幾個 Phase 才會用的資料。

---

# 12. 使用者回覆後的續跑規則

使用者完成真人操作或提供資料後：

1. 驗證結果
2. 更新 current step state
3. 從該 step 下一個 checkpoint 繼續
4. 不重新跑已 PASS 的 Phase
5. 不重新要求相同資料
6. 一路繼續到下一個真正 Human Gate

---

# 13. Staging 執行目標

依 Runbook 完成：

DB readiness
→ 000900 migration
→ verification
→ regression
→ Staging Admin Workbench build
→ Cloudflare Pages
→ Access / domain
→ platform_admin MFA
→ UI E2E
→ pending-membership claim E2E
→ commercial / seat / admin regression
→ audit validation
→ Staging acceptance

Runbook L 節 19 項 acceptance gate：

**全部 PASS 才能：**

STAGING_ACCEPTANCE_GATE = YES

缺一項 mandatory：
不得宣告 ACCEPTED。

---

# 14. Staging 失敗處理

普通 failure：

investigate
→ root cause
→ minimal fix
→ focused test
→ mandatory regression
→ resume

不要遇到普通 CI / frontend / test failure 就停下來問使用者。

以下才 HARD STOP：

- target environment 不明
- Production ref 出現在 Staging mutation path
- migration checksum mismatch
- destructive/unexpected schema drift
- rollback proof 失敗
- auth / tenant boundary 無法安全判斷
- 需要使用者產品決策
- 真人 authentication required

---

# 15. Staging ACCEPTED 後自動進 Production Preparation

Staging 19 項全 PASS 後：

不要停在「Staging 完成」等待新提示詞。

直接開始 Production read-only preparation：

- Production current-state verification
- Production config diff
- Production Supabase / Cloudflare target verification
- rollback target
- artifact identity
- migration applicability
- secrets presence
- domain / Access / Pages plan
- production canary identities / requirements
- backup / rollback requirements

只要是 read-only，都自行完成。

---

# 16. Production Mutation Gate

本提示詞的目標是一路做到 Production 完成。

但是：

**Production 真正 mutation 前仍遵守現有 HANDOFF / Runbook guardrail。**

如果 Runbook 或現有規則要求使用者確認 Production window：

在第一個 Production mutation 前只停一次。

輸出：

CURRENT_PHASE = PRODUCTION
STATUS = WAITING_FOR_HUMAN

PRODUCTION_PREFLIGHT = PASS

PLANNED_MUTATIONS =
<精確列出即將執行的 mutation>

ROLLBACK_READY =
YES/NO

BACKUP_REQUIRED =
YES/NO

EXACT_APPROVAL_REQUIRED =
「核准 Production 執行」

使用者回覆核准後：
一路執行 Production，
不要每個小步再問。

只有再次遇到真人 OAuth / MFA / Dashboard-only 操作或新的不可逆超範圍變更，才再停。

---

# 17. Production 執行原則

Production 必須：

- 使用已在 Staging ACCEPTED 的相同 source/artifact
- 不為 Production 重新發明不同邏輯
- 不帶 Staging test data
- 不帶 Staging test_access overlay
- 不帶 Staging Supabase ref
- Production config 使用 Production canonical ref
- runtime identity 可驗證
- migration 有 backup / rollback proof（若適用）
- Admin Workbench Production 與 Staging site 隔離
- 真人 canary 完成後才 ACCEPTED

---

# 18. Production Canary

依 Runbook / 實際功能完成真人 canary。

至少包含適用項：

- platform_admin Production login
- MFA
- Customer list / detail
- member search by Email
- stable user_id
- pending membership claim
- entitlement
- Central Product Seats
- assign / release
- suspend / reactivate
- audit
- no cross-customer access
- no Staging ref
- no Staging data
- fail-closed behavior

真人 login / MFA 時再叫使用者。

其餘可自動驗證的不要叫使用者手動看。

---

# 19. Release Evidence

每個重大階段都要留下 evidence。

至少：

source_commit
deploy/build identity
environment
project ref
migration state
runtime identity
health
auth
commercial
seat
e2e
canary
accepted_at
rollback target

Staging：

DEPLOYED != ACCEPTED

Production：

DEPLOYED != ACCEPTED

真人 canary / mandatory gate 全 PASS 才能：

PRODUCTION_ACCEPTED = YES

---

# 20. 最終完成條件

只有以下全部完成：

STAGING_ACCEPTANCE_GATE = YES
PRODUCTION_DEPLOYED = YES
PRODUCTION_CANARY = PASS
PRODUCTION_ACCEPTED = YES

才輸出：

GO_LIVE = YES

---

# 21. 最終回報格式

完成後回：

PROJECT =
SOURCE_BRANCH =
SOURCE_COMMIT =

STAGING_MIGRATION =
STAGING_ADMIN_SITE =
STAGING_MFA =
STAGING_E2E =
STAGING_ACCEPTANCE_GATE =

PRODUCTION_PRECHECK =
PRODUCTION_MIGRATION =
PRODUCTION_ADMIN_SITE =
PRODUCTION_MFA =
PRODUCTION_CANARY =
PRODUCTION_ACCEPTED =

AUDIT_FOLLOWUPS =
SERVICES_YAML_STATUS =
CLEANUP_STATUS =

RELEASE_EVIDENCE =
ROLLBACK_TARGET =

GO_LIVE = YES/NO

MANUAL_ACTIONS_COMPLETED =
UNRESOLVED_FOLLOWUPS =

---

# 22. 現在開始方式

現在不要重新出計畫。

先：

1. 讀 RUNBOOK.md
2. 對照 42 step 找出目前已完成到哪一步
3. 用現有 evidence 核對
4. 從第一個尚未完成的 step 繼續
5. 能自動做就做
6. 碰到真人 Gate 才停下來找使用者
7. 使用者完成後繼續
8. 一路直到 Production ACCEPTED / GO_LIVE

第一個回報應是：

RUNBOOK_RESUME_STEP =
WHY =
NEXT_AUTOMATED_ACTIONS =

然後直接開始執行，不要停在重新規劃。
