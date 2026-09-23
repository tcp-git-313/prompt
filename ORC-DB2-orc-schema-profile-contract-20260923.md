# ORC DB-2 — ORC 正式資料合約 + 自動 Profile 建立

## OWNER

ORC DATABASE CONTRACT OWNER

## MODEL

GPT-5.6 Sol — High

## CONTEXT

Canonical source：

`F:\00-Ticenpi-SaaS\TicenpiLetter`

Central deploy：

`F:\00-Ticenpi-SaaS\deploy`

上一輪已確認：

- ORC Staging 的 OCR backend / health / routing 本身正常。
- Browser E2E 目前卡在前端資料存取。
- 前端還在使用舊資料表名稱：
  - `records`
  - `projects`
  - `user_profiles`
- Staging 目前真正存在的 ORC 正式資料表，底層實體名稱暫時是：
  - `v2_records`
  - `v2_projects`
  - `v2_user_profiles`
- 這些 `v2_*` 只是歷史技術命名，不代表產品有 V1 / V2 版本。
- 從這一輪開始，文件與報告統一稱：
  - ORC records
  - ORC projects
  - ORC user_profiles
- 除非真的在寫 SQL / code，需要指出實體 table name，否則不要再把產品稱為 V2。
- live Staging 已確認三張 ORC table 有 RLS，但 authenticated 目前缺少實際 CRUD table privileges。
- ORC user_profiles 目前沒有「登入後自動建立」的正式流程。
- quota RPC 已存在，但缺 profile 時會 PROFILE_NOT_FOUND。
- Production 不得修改。
- ORC P2 frontend / Docker / CI / Staging deploy 目前全部暫停，先把 DB contract 做正確。

---

# USER DECISION — FROZEN

以下產品規則已由使用者決定，不再討論 A/B/C：

## ORC Profile 建立規則

```
使用者登入 ORC
↓
Supabase Auth 成功
↓
如果 ORC user_profile 不存在
→ 自動建立
↓
如果已存在
→ 直接使用
```

要求：

- 第一次登入建立。
- 之後登入不能重複建立。
- 必須 idempotent。
- 不需要 admin 預先 seed。
- 不需要人工建立。
- 不需要因目前 Staging 既有 Auth users 做一次性「救帳號」。
- profile existence 與 ORC entitlement 是兩回事：
  - profile 可以因登入而建立；
  - 是否能使用商用 ORC 功能仍由既有 entitlement / backend auth contract 判定；
  - 不得因 profile 存在就自動取得商用權限。

---

# GOAL

把 ORC 正式資料層做成一個完整、可驗證的 contract：

1. ORC records：登入使用者只能 CRUD 自己的資料。
2. ORC projects：登入使用者只能 CRUD 自己的資料。
3. ORC user_profiles：
   - 登入後若不存在，自動建立。
   - 可讀自己的 profile。
   - 不允許前端直接任意改：
     - role
     - plan
     - quota_limit
     - quota_used
4. quota：
   - 由既有安全 RPC / DB-controlled path 更新。
   - frontend 不可直接改 quota_used。
5. RLS + GRANT 必須同時正確。
6. 用 authenticated 身分做真實 contract test。
7. Production 完全不變。
8. 完成後 ORC P2 才能恢復。

---

# HARD RULES

## 禁止

- 不得修改 Production Supabase。
- 不得修改 Production schema/data/RLS/grants/RPC。
- 不得改 Cloudflare / DNS。
- 不得 deploy ORC。
- 不得重建 Docker。
- 不得修改 frontend table mapping；那是 ORC P2 下一階段。
- 不得建立 legacy compatibility tables/views。
- 不得把正式 ORC schema 改回 legacy tables。
- 不得 DISABLE RLS。
- 不得用 `USING (true)` / `WITH CHECK (true)` 直接全放行。
- 不得用 service_role 當 authenticated 驗收替代品。
- 不得讓 frontend 可直接修改 role / plan / quota_limit / quota_used。
- 不得刪除任何既有資料。
- 不得為了方便而讓 authenticated 擁有超過實際需求的 privilege。

## HARD STOP

以下任一發生立即停止：

1. 需要修改 Production 才能完成。
2. ORC user_profiles schema 缺必要欄位，必須做超出本任務的產品資料模型重構。
3. 既有 quota RPC 與「profile 自動建立」設計無法安全共存。
4. authenticated 最小權限無法在現有 RLS 下成立。
5. 需要修改 shared platform auth / membership / entitlement schema。
6. 發現 Staging target project 不是預期的 Staging project。
7. 需要改 frontend 才能證明 DB contract。

HARD STOP 時只輸出：

```
CURRENT
EXPECTED
CONFLICT
EVIDENCE
MINIMUM OPTIONS
```

---

# PHASE 0 — PREFLIGHT

從：

`F:\00-Ticenpi-SaaS\TicenpiLetter`

自行搜尋並閱讀實際檔案，不要求使用者提供 MD。

至少找：

- `db/v2_schema.sql`
- `db/README.md`
- 所有 ORC migration / SQL
- quota RPC 定義
- profile 相關 SQL / trigger / RPC
- `frontend/src/lib/supabase.js`
- auth 初始化流程
- tests
- Supabase setup scripts

確認：

- branch
- HEAD
- git status
- tracked dirty
- untracked

不要修改既有 unrelated WIP。

---

# PHASE 1 — DEFINE THE ORC LOGICAL CONTRACT

先把產品層與實體表名分開。

輸出：

```
ORC LOGICAL RESOURCE         CURRENT PHYSICAL TABLE
records                      v2_records
projects                     v2_projects
user_profiles                v2_user_profiles
```

注意：

- `v2_*` 只是目前 DB 實體名。
- 本輪不做 rename。
- 不要新增任何 V1/V2 產品概念。

然後確認每張表真正欄位、PK、user_id、FK、timestamps、quota 欄位。

---

# PHASE 2 — READ-ONLY LIVE STAGING AUDIT

對 Staging 做唯讀 catalog / metadata audit。

確認：

## ORC records
- table exists
- columns
- RLS enabled
- policies
- authenticated grants

## ORC projects
同上。

## ORC user_profiles
同上。

## quota RPC
確認：
- function name
- args
- return
- SECURITY DEFINER / INVOKER
- execute grants
- 如何鎖定 auth.uid()
- profile missing 時現在的行為

## Auth / Profile
確認：
- 是否已有任何 trigger / RPC 自動建 profile
- 若沒有，證明沒有
- 不要因目前 Auth users 存在就做 backfill

---

# PHASE 3 — DESIGN THE PROFILE ENSURE CONTRACT

使用者已決定：

`登入 ORC → 沒 profile 就自動建立`

請實作為單一、明確、可重複呼叫的 DB contract。

優先設計：

`ensure_orc_profile()`

實際 function name 可依 repo 命名規則調整，但語意必須一致。

必要行為：

```
auth.uid() IS NULL
→ reject

profile exists for auth.uid()
→ return existing profile
→ no duplicate

profile missing
→ INSERT one row
→ user_id = auth.uid()
→ quota_used = 0
→ 其他欄位使用安全預設值
→ return created profile
```

必要安全條件：

- caller 只能建立自己的 profile。
- caller 不可指定別人的 user_id。
- caller 不可傳入 role。
- caller 不可傳入 plan。
- caller 不可傳入 quota_limit。
- caller 不可傳入 quota_used。
- concurrent calls 不得建立 duplicate。
- 使用 unique constraint / conflict handling 保證 idempotent。

不要用 auth.users global trigger，除非 repo 已明確依賴這種設計且不會讓所有平台產品註冊事件自動建立 ORC profile。

這一輪優先採：
「ORC app 登入後呼叫 ensure profile RPC」
而不是「所有 Supabase Auth signup 全域 trigger」。

---

# PHASE 4 — MINIMUM GRANT + RLS CONTRACT

建立最小權限。

## ORC records

authenticated：

- SELECT own rows
- INSERT own rows
- UPDATE own rows
- DELETE own rows

RLS 必須限制：

`user_id = auth.uid()`

## ORC projects

同 records。

## ORC user_profiles

authenticated：

- SELECT own profile
- 不給一般 unrestricted table UPDATE
- profile 建立走 ensure RPC
- quota 修改走 quota RPC
- role / plan / quota_limit 由受控後台或未來正式管理流程負責

如果前端未來確實需要修改非敏感 profile 欄位：
不要因此直接 GRANT 全 row UPDATE。
先列出欄位與安全做法；超出本輪則 HARD STOP 或留待後續。

## quota

authenticated：

- EXECUTE quota RPC
- 不直接 UPDATE quota_used

quota RPC 必須：

- 只操作 auth.uid() 自己的 profile
- 不接受任意 user_id
- 不允許跨 user
- 缺 profile 時的行為與 ensure contract 一致

若最安全做法是 quota RPC 先 internal ensure profile，再 increment：
可以評估，但必須避免隱性建立造成設計混亂。
優先保持：
登入初始化 ensure profile → 後續 quota RPC。

---

# PHASE 5 — CREATE MINIMUM MIGRATION

完成 Phase 1~4 後才建立 migration。

允許修改範圍：

- ORC repo 的 SQL migration
- 必要 regression tests
- 必要 DB contract documentation

Migration 僅處理：

1. authenticated 最小 grants
2. 必要 revokes
3. RLS 修正（若需要）
4. ensure profile RPC
5. quota execute/contract 修正（若需要）
6. unique / constraint（只有 idempotency 必要時）

套用前先輸出：

```
CURRENT
EXPECTED
SQL DIFF SUMMARY
SECURITY IMPACT
ROLLBACK PLAN
```

確認沒有 Production impact 才可繼續。

---

# PHASE 6 — APPLY TO STAGING ONLY

明確確認 target：

Staging Supabase project。

Production 不得連線做 write。

套用 migration 後重新查：

- grants
- RLS
- functions
- execute privileges
- constraints

確定 live DB 與 migration 相符。

---

# PHASE 7 — AUTHENTICATED CONTRACT TEST

這是必要驗收。

不能用 service_role 代替。

使用 Staging authenticated test identity / 等價 user JWT。

## Profile

第一次：

`ensure profile`

預期：
- 建 1 row
- user_id = auth.uid()
- safe defaults

第二次再呼叫：

預期：
- 不新增
- 回同一 profile

## records

- create own
- read own
- update own
- delete own

## projects

同上。

## cross-user isolation

使用另一 authenticated 身分驗：

- 不能讀別人的 records
- 不能改別人的 records
- 不能刪別人的 records
- projects 同樣隔離
- 不能讀/改別人的 profile

## sensitive profile fields

確認一般 authenticated path 不能直接改：

- role
- plan
- quota_limit
- quota_used

## quota RPC

確認：

- 可執行
- 只增加自己的 quota
- 不可指定別人
- 不跨 user
- profile 已存在時正常

完成後只清本任務新增的測試 records/projects。
不要刪除測試 identity。

---

# PHASE 8 — EXISTING STAGING AUTH USERS

不要把現有 Auth user 當正式客戶。

只報：

```
Auth identities currently present: N
ORC profiles currently present: N
```

本輪不做全量 backfill。

規則是：

`使用者下次進 ORC → ensure profile → 自動建立`

因此不需要為目前既有 Auth identities 預先建立 ORC profile。

除非 E2E 測試所用 identity 在測試過程自然建立 profile，這是正常行為。

---

# PHASE 9 — REGRESSION TEST

建立 deterministic DB contract test，至少防止：

- RLS 開著但忘記 GRANT
- records/projects authenticated CRUD privilege 遺失
- user_profiles 被誤給 unrestricted UPDATE
- ensure profile 可建立別人的 user_id
- ensure profile 重複建立
- quota RPC 可跨 user
- sensitive fields 可被一般 user 直接更新

若 repo 沒有 DB test framework：
建立最小可重現 contract verification script。

---

# PHASE 10 — COMMIT

只有 Staging contract test 全 PASS 才 commit。

不得 `git add -A`。

只加入本任務相關：

- migration
- DB tests
- DB contract docs（若必要）

不要碰 frontend。

不要 deploy ORC。

---

# REQUIRED FINAL REPORT

```
ORC DATABASE CONTRACT FINAL

1. SOURCE
Path:
Branch:
HEAD before:
HEAD after:
Files changed:

2. ORC LOGICAL SCHEMA
records -> physical:
projects -> physical:
user_profiles -> physical:

3. PROFILE RULE
Login:
Missing profile:
Existing profile:
Idempotent:
Global auth trigger used = NO / YES

4. PERMISSIONS
records:
projects:
user_profiles:
quota:

5. RLS
records:
projects:
user_profiles:
cross-user isolation:

6. STAGING MIGRATION
Applied:
Migration file:
Target:
Result:

7. AUTHENTICATED TEST
profile create:
profile second call:
records CRUD:
projects CRUD:
profile read:
sensitive fields protected:
quota RPC:
cross-user isolation:

8. EXISTING AUTH IDENTITIES
Auth identities:
Profiles before:
Profiles after:
Backfill performed = NO

9. PRODUCTION
Changed = NO

10. COMMIT
SHA:

11. ORC P2
Can resume = YES / NO

12. NEXT STEP
If YES:
Resume ORC P2 from frontend ORC table mapping → Local DEV → Local Docker → CI → Staging E2E.

FINAL STATUS = PASS / BLOCKED / FAIL
```

只有 authenticated contract test 全部 PASS，且 Production unchanged，才能寫 PASS。
