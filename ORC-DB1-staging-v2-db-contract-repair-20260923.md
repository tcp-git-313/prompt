# ORC DB-1 — Staging v2 DB Contract Repair

ROLE
你是 Ticenpi ORC / Letter 的 Staging DB Contract Owner。

Canonical source:
F:\00-Ticenpi-SaaS\TicenpiLetter

Central deploy:
F:\00-Ticenpi-SaaS\deploy

CURRENT CONFIRMED STATE
- ORC P2 已在 Phase 2 HARD STOP。
- frontend 仍使用 legacy tables；Staging 使用 v2_*。
- Live Staging catalog 顯示 authenticated 對 v2_records / v2_projects / v2_user_profiles 的 CRUD table privileges 不成立。
- v2 RLS policy 存在，但 table privilege + RLS 必須同時成立。
- Staging v2_user_profiles = 0；目前有 Auth users，因此 quota/profile lifecycle 尚未成立。
- Production legacy tables 存在；本任務不得碰 Production。
- 本任務不是 frontend 修復，也不是 ORC deploy 任務。

GOAL
只處理 Staging v2 DB contract，讓後續 ORC P2 能安全繼續。

最終必須確認：
1. authenticated 對 v2_records / v2_projects / v2_user_profiles 的預期 CRUD contract。
2. RLS 與 table privileges 一致。
3. 新/既有 Auth user 的 v2_user_profiles provisioning 規則明確且可運作。
4. quota RPC 的 v2 contract 明確且可測。
5. Production 完全未改。
6. 不修改 frontend、不重建 Docker、不跑 ORC deploy。

HARD RULES
- Production Supabase：READ ONLY metadata；不得 DDL/DML。
- Staging 只有在完成盤查、產出最小 migration 並確認影響範圍後，才可套用 migration。
- 不得建立 legacy compatibility views。
- 不得把 Staging 改回 legacy tables。
- 不得修改 frontend source。
- 不得修改 Docker / compose / nginx / Cloudflare / DNS。
- 不得 deploy ORC。
- 不得建立測試後門或繞過 RLS。
- 不得使用 service_role 來「證明」authenticated 可用；驗收必須用 authenticated 身分/等價 JWT contract。
- 不得刪除任何既有資料。

PHASE 0 — PREFLIGHT
從 F:\00-Ticenpi-SaaS\TicenpiLetter 自行找到並閱讀：
- db/v2_schema.sql
- db/README.md
- 所有 migration / SQL / RPC 定義
- frontend/src/lib/supabase.js
- auth/profile/quota 相關程式
- tests
- 任何 Supabase setup scripts

確認 git branch / HEAD / status。
本任務若需要修改 repo SQL migration，只能改 ORC repo 中與 v2 contract 直接相關的 SQL/測試文件。

PHASE 1 — READ-ONLY LIVE CONTRACT AUDIT
對 Staging Supabase 做唯讀 catalog / metadata 查詢，整理：

A. tables
- v2_records
- v2_projects
- v2_user_profiles

B. privileges
針對 authenticated / anon / service_role，列出實際 SELECT/INSERT/UPDATE/DELETE privileges。

C. RLS
列出每張表：
- RLS enabled?
- policies
- command
- roles
- USING
- WITH CHECK

D. schema contract
列出前端實際需要的 columns 與 live table columns 差異。

E. profile lifecycle
找出是否存在：
- auth.users trigger
- signup trigger
- first-login RPC
- application upsert
- seed/admin provisioning

F. quota
找出實際 RPC 名稱、參數、return shape、依賴的 profile/table。

不要猜。
所有結論要有 repo 或 live catalog 證據。

PHASE 2 — DEFINE EXPECTED CONTRACT
產出清楚的 expected contract：

authenticated user 應可：
- 自己的 v2_records：需要哪些 CRUD
- 自己的 v2_projects：需要哪些 CRUD
- 自己的 v2_user_profiles：需要哪些 CRUD / 是否只允許特定欄位
- quota RPC：需要哪些 execute privilege / row access

如果前端需求與安全最小權限衝突，HARD STOP，提出最小選項，不自行放寬到全表。

PHASE 3 — PROFILE PROVISIONING DECISION
判斷現有設計應採哪一種：
A. auth.users trigger 自動建立 v2_user_profiles
B. 第一次登入由 RPC/application upsert
C. admin/seed 建立

優先沿用 repo 已有設計，不要發明第二套。

若 repo 沒有明確設計：
HARD STOP，回報 A/B/C 的風險與建議，不直接實作。

既有 Auth users 若缺 profile：
只能提出「可重複、可稽核、只補缺漏」的 backfill。
不得覆寫現有 profile。

PHASE 4 — MINIMUM STAGING MIGRATION
只有 Phase 1~3 已確認 contract 才建立最小 migration。

允許的修改範圍僅限：
- 必要 table grants/revokes
- 必要 RLS policy 修正
- 已確認的 profile provisioning
- 已確認的 quota RPC execute/contract
- 對應 regression test / migration test

禁止：
- DISABLE RLS
- USING (true) / WITH CHECK (true) 這類無條件放行，除非 repo 明確要求且有證據
- broad grants 超過前端實際需求
- service_role 當一般使用者路徑
- legacy compatibility tables/views

在套用前先輸出：
CURRENT
EXPECTED
SQL DIFF SUMMARY
SECURITY IMPACT
ROLLBACK PLAN

如果 migration 會影響 Production 共用物件，HARD STOP。

PHASE 5 — APPLY TO STAGING ONLY
確認 target project 是 Staging 後才可執行。

套用後驗證：
- privileges
- RLS
- policies
- profile provisioning
- quota RPC

任何一步失敗就停止，不要繼續 ORC P2。

PHASE 6 — AUTHENTICATED CONTRACT TEST
用 Staging 的 authenticated test identity / 等價 JWT contract 驗：

v2_records
- create own row
- read own row
- update own row
- delete own row
- 不能讀/改其他 user row

v2_projects
- 同上，依實際產品需求

v2_user_profiles
- profile 存在
- 可讀自己
- 只能修改允許欄位
- 不能讀/改他人敏感資料

quota RPC
- 可呼叫
- profile 存在時正常
- 不會跨 user
- 無 profile 的 fallback/provisioning 行為符合設計

測試資料完成後清除，只清本任務新建測試資料。

PHASE 7 — EXISTING USERS
確認現有 Staging Auth users：
- 是否都有 v2_user_profiles
- 若補建，列出補建數量，不輸出敏感資料
- quota contract 是否全部可用

PHASE 8 — REGRESSION / REPO EVIDENCE
如果 repo 有 migration/test framework：
- 加入 deterministic test，確保 grants/RLS/profile/quota contract 不再漂移。
- 只提交本任務 SQL/test/documentation。
- 不碰 frontend。

如果沒有合適 test framework，明確寫「未自動化」與可重現唯讀驗證命令。

PHASE 9 — FINAL REPORT
嚴格輸出：

ORC STAGING V2 DB CONTRACT REPORT

1. SOURCE
Path:
Branch:
HEAD:
Dirty:

2. LIVE STAGING BEFORE
Tables:
Privileges:
RLS:
Profiles:
Quota RPC:

3. EXPECTED CONTRACT
records:
projects:
user_profiles:
quota:

4. ROOT CAUSE
Confirmed:

5. MIGRATION
Files changed:
SQL scope:
Security impact:
Rollback:

6. STAGING APPLY
Applied = YES / NO
Target:
Result:

7. AUTHENTICATED TEST
records:
projects:
user_profiles:
cross-user isolation:
quota:
Result:

8. EXISTING USERS
Auth users:
Profiles before:
Profiles after:
Missing:

9. PRODUCTION
Changed = NO
Only metadata read = YES / NO

10. ORC P2 STATUS
Can resume Phase 2 = YES / NO

11. NEXT STEP
如果 YES：回到 ORC P2，從 frontend runtime table mapping 開始。
如果 NO：列出唯一剩餘 blocker。

FINAL STATUS = PASS / BLOCKED / FAIL

PASS 條件：
- authenticated CRUD contract 符合預期
- RLS isolation PASS
- profile provisioning PASS
- quota RPC PASS
- existing Staging users profile contract PASS
- Production unchanged

若任一條件未證明，不得寫 PASS。
