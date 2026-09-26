# W6 — Admin Workbench Push + Staging Human Setup Handoff

## ROLE
Release Preparation / Human Handoff Owner

## 推薦模型
GPT-5.6 Sol High
若只執行 push 與整理 runbook，可用 GPT-5.6 Luna High。

## 現況

已完成本機開發與測試：

WORKTREE =
F:\00-Ticenpi-SaaS\.worktrees\admin-workbench-platform

BRANCH =
feat/admin-workbench-email-first

目前已有 2 個本機 commit。

主要新增內容：

- 20260926_000900_admin_email_first_workbench.sql
- rollback
- Email-first pending membership
- pending Seat reservation
- Seat limit 將 assigned + reserved 一起計算
- claim_pending_memberships()
- 空 Customer 永久刪除
- audit log 讀取
- 中文 Admin Workbench
- 新 Admin RPC 要求 platform_admin + aal2
- 新工作台位於 admin-workbench/
- 舊工作台目前仍保留

本機測試已完成。

Staging / Production 尚未修改。

---

# 本輪目標

只做兩件事：

1. 把目前已完成且已 commit 的 branch 安全 push 到 GitHub。
2. Push 完後，整理一份「可以交給人照著做」的完整 Staging 人工設定與操作流程。

本輪不要實際執行任何 Staging / Production mutation。

---

# Phase 1 — Push branch

先確認：

- worktree 路徑正確
- branch = feat/admin-workbench-email-first
- HEAD 與前述 2 commits 存在
- worktree clean
- 沒有 unrelated WIP 被帶入
- remote repo 正確

然後：

- push 此 branch 到 origin
- 不 merge main
- 不 force push
- 不改 default branch
- 不建立 Production release

Push 後必須驗證：

REMOTE_BRANCH_EXISTS = YES
REMOTE_HEAD_MATCHES_LOCAL = YES

輸出：

PUSHED_BRANCH =
PUSHED_HEAD =
REMOTE_URL =
PUSH_VERIFIED = YES/NO

---

# Phase 2 — Read-only discovery for exact setup instructions

Push 完後，唯讀盤查並取得人工流程需要的實際值。

從 workspace / repo / deploy / Supabase config / Cloudflare config / existing service patterns 自動查：

## Staging

- Staging Supabase ref
- Staging DB migration execution path
- existing backup / restore scripts
- canonical migration deployment method
- admin-staging.ticenpi.com 是否已存在
- Cloudflare Pages project 是否已存在
- DNS / Pages custom domain pattern
- Supabase Redirect URL / Site URL 現況
- TOTP MFA enablement 現況
- services.yaml 現況
- secret loader
- build command
- deploy command
- health / smoke / E2E method

已知 Staging Supabase canonical ref：

jlsqjvehwblkeuycjoyj

Production ref：

zfpkxsulehkbyuqkkcml

任何 Staging 流程都不得寫到 Production ref。

---

# Phase 3 — 產出完整「人工設定 / 操作 Runbook」

最後必須給我一份完整、按順序、可直接照做的人工流程。

不能只寫高階概念。

每一步必須包含：

STEP_ID =
WHO = AI / HUMAN
WHERE =
COMMAND_OR_UI_PATH =
WHAT_TO_DO =
EXPECTED_RESULT =
VERIFY =
IF_FAIL =

---

# Runbook 必須涵蓋

## A. Git / Branch

1. Push branch 已完成
2. remote branch / HEAD 驗證
3. 是否需要 PR
4. 哪些 commit 會進 Staging
5. 不得 merge main 的情況與時機

---

## B. Staging Database

完整順序：

1. fresh logical backup
2. backup sha256
3. isolated restore
4. restore readability / row-count verification
5. migration preflight
6. 套用：
   20260926_000900_admin_email_first_workbench.sql
7. migration verification
8. rollback rehearsal
9. rollback file identity
10. evidence recording

必須使用實際存在的 script / command。

如果需要人執行，給唯一可複製指令。

不要猜。

---

## C. Supabase Staging MFA

完整說明：

1. 到哪個 Supabase project
2. UI 路徑
3. 如何確認 TOTP MFA 是否啟用
4. 是否需要開 enrollment / challenge
5. Redirect URLs 要加入什麼
6. Site URL 是否需要調整
7. 哪些值不能動
8. 如何確認 admin-staging.ticenpi.com 可回跳

必須固定：

Staging project =
jlsqjvehwblkeuycjoyj

不要碰 Production project。

---

## D. admin-staging.ticenpi.com

查清楚並給出：

1. Cloudflare Pages project 建立方式
2. project 名稱
3. production/staging branch 規則
4. custom domain
5. DNS 是否由 Pages 自動建立或需手動 CNAME
6. build output directory
7. build command
8. env / secret injection
9. rollback deployment
10. 如何驗證 live asset identity

若 Cloudflare Pages 已存在，改成 verify / reuse，不要重建。

---

## E. services.yaml

查清楚：

1. admin service 是否已登記
2. 若未登記，應加在哪個 section
3. service id / name
4. domain
5. port / static site semantics
6. health / smoke contract
7. 是否會影響 shared deploy manifest

只提供計畫與 exact change。
本輪不要修改。

---

## F. Secrets / Build

目前已知：

F:\HUB\scripts\load-ticenpi-secrets.ps1

曾建議：

. F:\HUB\scripts\load-ticenpi-secrets.ps1; npm run build:staging

請實際確認：

- 正確 workdir
- loader 是否 canonical
- build command
- required env keys
- 哪些 key 只確認存在，不可輸出值
- build output path
- 如何驗證沒有 Production/Staging crossover

最後給可直接複製的 PowerShell 指令。

---

## G. 首次 platform_admin MFA 設定

這一段要非常清楚，因為有安全時序問題。

說明：

1. admin-staging 上線後
2. 哪個 platform_admin 帳號先登入
3. Google login
4. TOTP enrollment
5. 掃 QR / 輸入驗證碼
6. 如何確認 session 進入 aal2
7. 如何避免其他人先綁 verifier
8. 是否要先限制 admin-staging 網域存取
9. 完成後如何驗證 admin RPC 真的要求 aal2

不要輸出 TOTP secret。
不要把 secret 寫進 repo。

---

## H. Google 首次登入自動認領 E2E

必須設計完整 Staging E2E：

### Scenario 1 — 已註冊使用者
- Admin Workbench 先輸入 Email
- 已有 profile
- 直接成為 Member
- Entitlement / Seat 正常

### Scenario 2 — 未註冊使用者
- Admin Workbench 先輸入 Email
- 建 pending membership
- 建 pending Seat reservation
- Pending Seat 計入 seat_limit
- 使用 Google 第一次登入
- 驗證 email_confirmed_at
- 自動 claim
- pending → member
- reserved → assigned

### Scenario 3 — Google 首次登入時 email 尚未 confirmed
- 驗證 claim 沒誤認
- 登入後呼叫補救 claim_pending_memberships()
- 確認最後可認領

### Scenario 4 — 無預先授權
- Google Login 成功
- 不取得產品權限
- 顯示未開通 / 方案 / 聯絡業務
- 不可看 Customer / Member / Seat 資料

### Scenario 5 — Seat limit
- assigned + reserved 一起計算
- 超過 seat_limit 必須拒絕
- downgrade 防護必須包含 reserved

### Scenario 6 — Invitation lifecycle
- cancel
- expiry
- Seat reservation 釋放

### Scenario 7 — Customer delete
- 空 Customer 可刪
- 曾有 Entitlement / Subscription / Seat / Member 的 Customer 不可永久刪
- 應改停用 / 封存

---

## I. 新舊工作台並存

明確寫出：

### Stage 1
新後端 migration 上 Staging
舊工作台仍可用
新 Workbench 開始測

### Stage 2
admin-staging.ticenpi.com 完成所有 E2E
舊工作台加「舊版」提示（若需要）

### Stage 3
啟用「所有 admin 操作 require aal2」
舊工作台自然失效

### Stage 4
再從 Launcher 移除舊 Workbench 入口

重要：

Production 不要有長時間的「MFA 前門 + 舊工作台側門」並存。

---

## J. Legacy Workbench 關閉開關

查清楚這個開關實際如何實作。

輸出：

LEGACY_ADMIN_DISABLE_MECHANISM =
BACKEND_FLAG_OR_RPC =
REVERSIBLE = YES/NO

說明：

- 怎麼開
- 怎麼關
- 開了之後哪些 admin RPC 被 aal2 保護
- 舊工作台會收到什麼錯誤
- rollback 怎麼做

---

## K. Audit Log / 中文化

驗證：

- 所有 Workbench UI 狀態中文化
- roles：
  member → 成員
  admin → 管理員
  owner → 擁有者
- source：
  manual → 手動新增
- errors：
  admin_required
  seat_downgrade_blocked
  user_not_found
  等都映射中文

底層 canonical values 不改。

確認 Audit UI 可讀：

- actor
- action
- customer
- target
- before/after
- timestamp

---

## L. Staging Acceptance Gate

最後必須定義：

STAGING_ACCEPTED = YES

需要哪些 mandatory PASS。

至少：

- migration
- rollback rehearsal
- admin-staging live
- Google login
- MFA enrollment
- aal2 admin RPC enforcement
- pending membership
- Seat reservation
- claim
- no-auth/no-entitlement deny
- seat limit
- Customer lifecycle
- audit
- 中文 UI
- no Production crossover

---

## M. Production 之後怎麼做

只寫 runbook，不執行。

Production 必須等 Staging ACCEPTED。

Production runbook 要說明：

- backup
- restore verification
- migration
- admin.ticenpi.com
- Production MFA
- Production platform_admin enrollment
- smoke
- legacy workbench disable
- Launcher old entry removal
- rollback

Production DNS / Cloudflare / DB mutation 都必須等明確授權。

---

# 最終輸出格式

先回：

PUSHED_BRANCH =
PUSHED_HEAD =
REMOTE_HEAD =
PUSH_VERIFIED =

然後：

# Admin Workbench Staging Human Setup Runbook

依順序列：

STEP 1
STEP 2
STEP 3
...

每一步都必須有：

WHO
WHERE
COMMAND_OR_UI_PATH
WHAT_TO_DO
EXPECTED_RESULT
VERIFY
IF_FAIL

最後再列：

MANUAL_ACTIONS_ONLY =
AI_CAN_DO_LATER =
STAGING_ACCEPTANCE_GATE =
PRODUCTION_NOT_TOUCHED = YES

本輪禁止實際執行：

- Staging DB migration
- Cloudflare / DNS mutation
- Supabase MFA setting mutation
- services.yaml modification
- Staging deploy
- Production mutation

本輪只：
Push branch
→ Verify push
→ Read-only discovery
→ 給完整人工與設定 Runbook。
