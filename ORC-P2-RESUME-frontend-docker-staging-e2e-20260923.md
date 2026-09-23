# ORC P2-RESUME — Frontend Mapping + Login Profile Wiring + Local Docker + CI + Staging E2E

## OWNER

ORC P2 RESUME OWNER

## MODEL

GPT-5.6 Sol — High

## CANONICAL PATHS

Canonical source:

`F:\00-Ticenpi-SaaS\TicenpiLetter`

Central deploy:

`F:\00-Ticenpi-SaaS\deploy`

Target local Docker project:

`ticenpiletter`

## CONFIRMED BASELINE

DB-2 已完成並 PASS。

Current source HEAD after DB-2:

`f504019db59f3fc1f4522957ed9d6ddefec99474`

Staging ORC logical resources currently map to physical tables:

- records -> `v2_records`
- projects -> `v2_projects`
- user_profiles -> `v2_user_profiles`

Important naming rule:

- `v2_*` 只是目前 DB 實體表名。
- 不代表產品有 V1 / V2 版本。
- 報告與設計統一稱 ORC records / projects / user_profiles。
- 只有寫 SQL / code 時才使用實體 table name。

DB contract 已確認：

- authenticated 對 ORC records / projects 可 CRUD 自己資料。
- RLS 以 `user_id = auth.uid()` 隔離。
- ORC user_profiles 僅允許 authenticated 讀自己的資料，不允許一般前端直接任意改敏感欄位。
- `ensure_orc_profile()` 已存在：
  - 第一次登入 ORC 時若 profile 不存在，自動建立。
  - 已存在則直接返回。
  - idempotent。
  - 不使用 global auth trigger。
- `v2_increment_quota()` 已存在：
  - authenticated 可執行。
  - quota 不由 frontend 直接 UPDATE。
- Production 未修改。
- DB authenticated contract test 已 PASS。
- Browser E2E 尚未重驗。
- 現在可以恢復 ORC P2。

## PRIMARY GOAL

完成這條鏈：

`DB contract PASS → Frontend mapping → Login profile wiring → Local DEV → Local Docker → CI → Staging → Browser OCR E2E`

最終必須證明：

1. Frontend 不再查 legacy `records/projects/user_profiles`。
2. 登入 ORC 後會呼叫 `ensure_orc_profile()`。
3. quota 走 `v2_increment_quota()`，frontend 不直接寫 `quota_used`。
4. Local DEV 不退化。
5. 本機 Docker `ticenpiletter` 從 canonical source 重建且 identity 可證明。
6. Local Docker 真實 login / entitlement / OCR / persistence / render / history 全 PASS。
7. exact commit CI 全綠。
8. Staging 僅部署 exact CI immutable digests。
9. Staging Browser E2E 真正 PASS。
10. 不再出現 PGRST205。
11. Production 不修改。

---

# HARD RULES

禁止：

- 修改 Production Supabase。
- 修改 Production DNS / Cloudflare。
- 部署 Production。
- 修改 Production secret。
- 建 legacy compatibility tables/views。
- 把 ORC DB 改回 legacy tables。
- 再改 DB-2 migration / grants / RLS，除非新證據證明 contract 有錯；若需要，HARD STOP。
- 用 service_role 替代一般 authenticated Browser 驗收。
- 跳過 Local Docker gate 直接部署 Staging。
- 用 branch latest 代替 exact SHA。
- 用 mutable tag 代替 digest。
- 以 health PASS / orc-core PASS 宣稱 Browser E2E PASS。
- `git add -A`。
- reset / restore / clean unrelated WIP。
- 修改其他產品。
- 同時保留兩套本機 ORC Docker project 而不說明。

遇到以下情況 HARD STOP：

1. frontend 所需欄位與 DB-2 contract 不相容。
2. `ensure_orc_profile()` 或 quota RPC 介面與 frontend 實際需求不相容。
3. 需要再改 Staging DB schema 才能完成。
4. Production compatibility 無法安全保留。
5. local Docker project naming 與官方 scripts 有不可安全處理的衝突。
6. 需要改 shared platform auth / entitlement contract。
7. canonical source / CI / Staging identity 與已知 evidence 發生重大不一致。

HARD STOP 只輸出：

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

自行讀取並確認：

- branch
- HEAD
- origin/master
- git status
- tracked dirty
- untracked
- frontend/src/lib/supabase.js
- frontend auth/init flow
- runtimeConfig
- login success path
- quota calls
- records/projects/profile calls
- local DEV scripts
- local Docker scripts
- docker-compose.yml
- tests
- Staging compose

確認 DB-2 commit `f504019...` 在目前 branch 上且沒有被覆蓋。

---

# PHASE 1 — FRONTEND LOGICAL RESOURCE MAPPING

目標：frontend 不再散落使用 legacy physical names。

建立集中 mapping。

Logical resources:

```
records
projects
user_profiles
```

Staging / Local Docker Staging-like physical mapping:

```
records       -> v2_records
projects      -> v2_projects
user_profiles -> v2_user_profiles
```

Production/default：

- 保持目前既有 legacy contract。
- 不得因本次改動被自動切到 `v2_*`。

優先使用 runtime config / environment-aware mapping。

要求：

- table mapping 只有單一來源。
- 不可在多個 component 硬編碼。
- 搜尋所有 legacy table call sites，不只修第一個 PGRST205。
- DEV offline mode 不退化。

---

# PHASE 2 — LOGIN PROFILE WIRING

在「登入 ORC 成功、auth session 可用」後接：

`ensure_orc_profile()`

規則：

```
login success
↓
authenticated session ready
↓
ensure_orc_profile()
↓
profile ready
↓
load ORC user data
```

注意：

- profile existence 不等於 entitlement。
- 不得因 profile 建立就給產品權限。
- entitlement 仍走既有 commercial auth contract。
- ensure profile 必須在讀 history / quota / projects 前完成。
- 重複登入不得建立 duplicate。
- 失敗要有明確 error state，不得 silently continue。

---

# PHASE 3 — QUOTA WIRING

搜尋 frontend 所有 quota write path。

要求：

- 禁止 frontend 直接 UPDATE `quota_used`。
- 使用 DB-2 已驗證的 `v2_increment_quota()`。
- 確認 RPC args / return shape。
- quota RPC 失敗不能被吞掉。
- 不得允許 caller 指定別的 user_id。

若目前 frontend 有直接改：
- role
- plan
- quota_limit
- quota_used

敏感欄位路徑要移除或改成 read-only。

---

# PHASE 4 — REGRESSION TESTS

至少補：

1. Staging runtime -> ORC `v2_*` physical mapping。
2. Production/default -> legacy mapping。
3. login success -> ensure profile before ORC data load。
4. repeated ensure does not create duplicate behavior at frontend contract level。
5. quota write only via RPC。
6. frontend 不再有未受控 legacy table hardcode。
7. PGRST205 相關 mapping regression。

使用現有 test framework。

---

# PHASE 5 — LOCAL DEV

依 repo 官方啟動方式。

驗：

- frontend
- backend
- health
- direct OCR
- proxy OCR
- existing tests
- new tests
- frontend build

Local DEV 若使用 offline trial，這層主要做 regression gate。

全部 PASS 才繼續。

---

# PHASE 6 — LOCAL DOCKER PROJECT ALIGNMENT

Target local project：

`ticenpiletter`

先讀：

- `scripts/start_local_docker.ps1`
- `docker-compose.yml`
- verify/fingerprint scripts
- Docker labels
- 現有 running `ticenpiletter`

如果 official script 現在使用別的 project name：

先判斷能否安全統一到 `ticenpiletter`。

安全條件：

- 不影響 Staging。
- 不影響中央 manifest。
- 不影響其他產品。
- verify scripts 可同步。
- 不留下兩套 ORC local project。

若不能證明安全：

HARD STOP。

不得默默保留：
`ticenpiletter` + 另一套 official local project。

---

# PHASE 7 — REBUILD LOCAL DOCKER FROM CANONICAL SOURCE

只有前面 PASS 才重建。

必須使用 official local Docker flow。

新的 images 必須可證明：

- canonical source path
- exact SHA
- build purpose
- environment = staging
- frontend/backend identity
- fingerprint / labels

最終不得再出現：

`source SHA = UNKNOWN`

輸出：

```
LOCAL DOCKER IDENTITY
Project:
Source path:
Source SHA:
Frontend image:
Backend image:
Frontend port:
Backend port:
Environment:
Table mapping:
Supabase project hostname:
```

不要輸出 secret。

---

# PHASE 8 — LOCAL DOCKER TRUE E2E

必須實際測：

```
Google/Supabase login
→ authenticated session
→ ensure_orc_profile()
→ commercial entitlement
→ /ocr
→ upload fixture
→ parse
→ extract-address
→ quota RPC
→ persistence
→ result render
→ history reload
```

同步檢查：

- Browser Network
- Browser Console
- frontend logs
- backend logs

必要 PASS：

- no PGRST205
- ensure profile success
- no duplicate profile
- entitlement behavior correct
- OCR POST 200
- persistence success
- result renders
- history reload works
- badge 不停在「檢查中」
- quota RPC success
- no direct quota update
- no unexpected 401/403

FAIL 不得進 CI。

---

# PHASE 9 — COMMIT / PUSH / CI

只有 Local DEV + Local Docker E2E PASS 才做。

確認 tracked diff 只包含本任務。

禁止 `git add -A`。

commit / push exact SHA。

等待 exact commit GitHub CI：

`completed + success`

取得：

- CI run ID
- frontend digest
- backend digest
- linux/arm64 evidence

任一 required job FAIL：
停止，不部署。

---

# PHASE 10 — PIN STAGING DIGESTS

CI 全綠後才更新：

`F:\00-Ticenpi-SaaS\TicenpiLetter\deploy\runtime-staging\compose.yml`

只 pin exact immutable digests。

依 repo 現有正式 release flow 處理 source commit / pin commit，不自行發明流程。

---

# PHASE 11 — STAGING DEPLOY

使用：

`F:\00-Ticenpi-SaaS\deploy\deploy.ps1`

依現有流程：

1. DryRun
2. Preflight
3. Deploy

必須使用 clean worktree / exact commit。

不得從 dirty canonical working tree 打包。

不得動其他產品。

任一步 FAIL：
停止，不繞過 gate。

---

# PHASE 12 — STAGING TRUE BROWSER E2E

部署後必須真 Browser 測：

`https://letter-staging.ticenpi.com`

驗：

1. runtime config = staging
2. exact release / commit
3. Google login
4. ensure_orc_profile success
5. entitlement allow
6. OCR upload
7. parse
8. extract-address
9. quota RPC
10. persistence
11. result render
12. history
13. reload
14. badge
15. logout/login
16. no PGRST205
17. console 無本次相關 uncaught error

必須能證明：

```
Browser click = YES
Request created = YES
Frontend nginx reached = YES
Backend reached = YES
OCR completed = YES
Response 200 = YES
Profile ready = YES
Persistence = YES
UI rendered = YES
History reload = YES
Quota RPC = YES
```

---

# PHASE 13 — AUTHORIZATION MATRIX

如果既有 Staging 固定 test identities 可用：

- platform_admin -> PASS
- platform_test_allow -> PASS
- platform_test_deny -> BLOCKED
- no entitlement -> BLOCKED

若缺 test identity / credentials：
標記 BLOCKED，不得假裝 PASS。

---

# FINAL REPORT

```
ORC P2 RESUME FINAL

1. SOURCE
Path:
HEAD before:
HEAD after:
Files changed:

2. FRONTEND MAPPING
records:
projects:
user_profiles:
Production default:
Legacy hardcodes remaining:

3. PROFILE WIRING
Login hook:
ensure_orc_profile:
Ordering:
Idempotent behavior:

4. QUOTA
RPC:
Direct quota update remaining:
Result:

5. LOCAL DEV
Tests:
OCR:
Build:
Result:

6. LOCAL DOCKER
Project:
Source SHA:
Frontend image:
Backend image:
Ports:
Environment:
Login:
Profile:
Entitlement:
OCR:
Persistence:
Render:
History:
Quota:
Result:

7. CI
Exact SHA:
Run ID:
Frontend digest:
Backend digest:
Result:

8. STAGING
Release:
Source SHA:
Frontend digest:
Backend digest:
Login:
Profile:
Entitlement:
OCR:
Persistence:
Render:
History:
Quota:
PGRST205:
Result:

9. ALIGNMENT
Canonical Source:
Local Docker:
CI:
VPS:
ALL ALIGNED = YES / NO

10. AUTHORIZATION MATRIX
admin:
allow:
deny:
no entitlement:

11. PRODUCTION
Changed = NO
Legacy default preserved = YES / NO

12. COMMITS
source fix:
staging pin:
other:

13. REMAINING RISKS

14. FINAL STATUS
PASS / BLOCKED / FAIL
```

只有以下全部成立才能 PASS：

- frontend mapping 正確
- ensure profile 正確
- quota 正確
- Local Docker true E2E PASS
- exact CI PASS
- Staging Browser true E2E PASS
- no PGRST205
- result render PASS
- history PASS
- identity aligned
- Production unchanged
