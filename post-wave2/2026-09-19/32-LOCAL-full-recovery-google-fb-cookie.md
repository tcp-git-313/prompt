# LOCAL-FINAL — Restore Local Google Auth + FB Cookie End-to-End

- 抬頭：POST LOCAL FULL RECOVERY OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：大型
- 目標：一次執行到 `LOCAL_COMPLETE=YES`
- Staging / Production：禁止觸碰

---

你現在是：

POST LOCAL FULL RECOVERY OWNER

唯一目標：

把 TicenpiPost 本機真實使用流程完整恢復並驗證到：

`LOCAL_COMPLETE=YES`

不要再只驗 health / UI / unit tests。

這次必須把真正的：

Google OAuth
→ Supabase session
→ Membership / tenant
→ Local Web
→ Chrome Extension
→ Facebook first bind
→ FB cookies/session sync
→ recurring device-token sync
→ Facebook groups/read-only data fetch

整條鏈跑通。

只要沒有命中 HARD STOP，就持續執行，不要每一小步回來等待確認。

---

# 0. TARGET / CURRENT STATE

Repo：

`F:\00-Ticenpi-SaaS\TicenpiPost`

優先使用目前正式 Wave2 integration worktree：

`F:\00-Ticenpi-SaaS\TicenpiPost-wave2-integration-v2`

branch：

`repair/post-v2-wave2-integration-v2`

已知正式整合 HEAD 至少包含：

`53fce622caae69a6b0fa476483347b17af7dc4fb`

如果 HEAD 已前進，先辨識是否只是已核准 W2G CI evidence fix或其他已知 integration commit。

不要 reset / rollback 已完成成果。

已知目前真實問題：

Local UI 顯示：

`Google 登入尚未設定（NEXT_PUBLIC_SUPABASE_*）`

所以本任務第一優先不是 Staging，而是 Local 真實身份與 FB cookie 流程。

---

# 1. GLOBAL HARD RULES

## 禁止碰 Staging / Production

禁止：

- VPS mutation
- Staging deploy
- Production deploy
- Production Supabase mutation
- Central Deploy repo mutation
- GHCR deploy
- DNS / Cloudflare production change

## 禁止 real Facebook write

本任務只允許 Facebook read / bind / cookie/session validation。

禁止：

- 真實發文
- comment
- delete
- relist
- marketplace publish

`REAL_FB_ACTION=NO`

## 不得猜 Supabase

尤其禁止把 legacy Supabase URL/key「先塞進去試」。

必須先確認正確 Local / Staging-compatible Supabase identity SSOT。

若只能找到 legacy / ambiguous credentials：

HARD STOP，回報候選來源與差異。

## Secrets

不得輸出：

- Supabase secret values
- anon/publishable key value
- service role key
- OAuth client secret
- FB cookie/token
- JWT
- passwords

只允許 presence / source path / variable name。

---

# 2. PHASE A — IDENTIFY THE CORRECT SUPABASE SSOT

先只讀全面盤點，不要先改 env。

檢查：

- root .env / .env.local / .env.example
- frontend env files
- backend env/config
- docker-compose.yml
- deploy/runtime-staging/*
- deploy/staging/*
- Central Secret Store（只做 key presence，不輸出值）
- documented Supabase project IDs / URLs
- Google OAuth config references
- current Supabase project lineage
- legacy project references

必須回答：

```
LOCAL_SUPABASE_SOURCE=
LOCAL_SUPABASE_PROJECT_ID=
LOCAL_SUPABASE_URL_SOURCE=
LOCAL_ANON_OR_PUBLISHABLE_KEY_SOURCE=
BACKEND_SUPABASE_URL_SOURCE=
BACKEND_SUPABASE_ANON_KEY_SOURCE=
GOOGLE_OAUTH_ENABLED_FOR_THIS_PROJECT=
LEGACY_SUPABASE_FOUND=
LEGACY_SUPABASE_EXCLUDED=
```

如果有多個候選 project：

比對：

- organizations
- memberships
- devices
- profiles
- customer/subscription/entitlement schema
- OAuth provider enabled state
- known staging test users

不可用猜測決定。

---

# 3. PHASE B — FIX LOCAL ENV CONTRACT

目標：

Local frontend 和 backend 使用同一個正確 Supabase project。

## Frontend required contract

確認實際程式讀哪些變數。

至少處理目前 UI 明確缺少的：

- `NEXT_PUBLIC_SUPABASE_URL`
- 對應的 public anon / publishable key variable

不要假設名稱，先讀 source。

## Backend required contract

確認 backend identity layer實際讀：

- SUPABASE_URL
- anon/public key
- JWT verification related config

不要使用 service-role key作一般 local login。

## Docker / Local runtime

確保：

`docker-compose.yml`

會把正確 Local env傳到：

- frontend
- backend/api
- worker（若需要）

不要硬編 secret literal。

使用 env interpolation / approved local env source。

保持：

- TICENPI_LOCAL_DEVELOPMENT=1
- TICENPI_REQUIRE_STUDIO=0
- commercial entitlement local bypass as frozen architecture
- publish master/action gates OFF

## Env safety

更新必要的：

- .env.example
- local env contract docs

但真實 secret不要 commit。

若真實 local env已存在於安全 user-local檔：

沿用。

---

# 4. PHASE C — GOOGLE OAUTH LOCAL E2E

啟動 isolated Local Post stack。

優先使用既有 Local port：

`127.0.0.1:19418`

如果已有 intended Local stack，就安全更新它。

不要碰其他 unrelated stack。

確認：

`GET /api/health = 200`
`GET / = 200`

然後做真正 Google login。

## 必須驗證

1. Login button不再顯示「NEXT_PUBLIC_SUPABASE_* 未設定」
2. Google OAuth provider正常開啟
3. OAuth callback回 Local app
4. Supabase session建立
5. `useSession` / backend JWT identity成功
6. user email / principal正確
7. local membership bootstrap成功
8. tenant resolved
9. Web shell可進
10. Launcher / Studio未啟動時仍可進 Web

如果 Google OAuth需要一次人工 browser consent：

只在這個點要求使用者做最小互動：

`請在已開啟的 Google 頁面完成登入/同意，完成後回到本機頁面`

不要要求使用者手動改設定。

完成後繼續自動驗。

---

# 5. PHASE D — LOCAL SESSION / MEMBERSHIP CONTRACT

登入後驗：

- authenticated = true
- email正確
- tenant存在
- local bootstrap path成立
- no production commercial gate block
- Studio offline只是 informational
- account mismatch只是 informational
- Web access不依賴 Launcher

確認：

`/api/session/state`

仍回報真實 Studio capability，不得 fake online。

---

# 6. PHASE E — CHROME EXTENSION CURRENT BUILD

找出 authoritative Extension source / build output。

確認不是舊 extension。

必須記錄：

- manifest version
- extension version
- git SHA / build source
- Extension ID（只記 ID，不記任何 cookie/token）

如果需要重新 build：

從 current Local source build。

不要從舊 release資料夾拿 artifact。

如果 Chrome currently loaded extension不是 current build：

更新/load unpacked current build。

若需要一次人工點選 Chrome「Reload extension」：

只要求這一個動作。

---

# 7. PHASE F — FACEBOOK FIRST BIND E2E

前提：

Chrome 已登入一個測試用 Facebook account。

只驗 read/bind，不做 write。

目標鏈：

Local Web
→ Extension
→ 取得目前 Facebook session/cookies
→ Local backend first-bind endpoint
→ FB account identity mapping
→ encrypted/persisted session/cookie state
→ UI顯示綁定帳號

## 必須驗

- 不需要 Launcher先 online
- Extension direct first bind path實際被走到
- FB account id / display identity符合目前 Chrome登入帳號
- 不建立錯 tenant
- duplicate protection正常
- response無 cookie/token洩漏
- log無 cookie/token

如果 Extension 無法擷取 cookies：

調查：

- permissions
- host_permissions
- Chrome cookies API
- extension runtime messaging
- DOMContentLoaded / race
- origin allowlist
- local endpoint
- CORS / auth
- device token bootstrap

只修真正 root cause。

不要先亂改 CORS/JWT/Proxy。

---

# 8. PHASE G — RECURRING COOKIE SYNC E2E

first bind成功後，不刪帳號。

再次觸發 cookie/session sync。

必須驗：

- 使用 existing account
- 不新增 duplicate FB account
- recurring path走 device token
- cookie/session timestamp或version有合理更新
- same tenant
- same FB account mapping
- idempotent behavior
- backend可重新讀取有效 session

如果 device token需要 bootstrap：

驗證 bootstrap來源與保存位置。

不得把 token印出。

---

# 9. PHASE H — FACEBOOK READ-ONLY VALIDATION

使用已同步 cookie/session做安全 read-only FB驗證。

優先：

- account/session validity check
- group list fetch
- existing read-only Facebook data endpoint

至少證明：

```
FB_SESSION_VALID=YES
FB_ACCOUNT_MATCH=YES
FB_GROUP_READ=PASS
```

如果 current product沒有專門 session-check endpoint，可用既有「抓社團列表」read-only path證明。

禁止：

- publish
- comment
- delete
- relist
- marketplace write

Publish gates保持全部 OFF。

---

# 10. PHASE I — RESTART / PERSISTENCE TEST

驗證不是只在單次 process有效。

安全重啟 Local Post stack。

重新登入 Web（若 session仍有效可沿用）。

驗：

- Google config仍存在
- Supabase session流程正常
- FB account binding仍存在
- recurring sync仍可執行
- group/read-only fetch仍 PASS
- 不產生 duplicate account

---

# 11. PHASE J — REGRESSION

至少跑：

## Backend targeted

- auth
- identity
- membership
- account lifecycle
- device token
- FB bind/sync
- relevant extension/backend contract
- publish safety gates

## Frontend

- SessionGate
- access regression
- account UI tests
- typecheck
- build

## Extension

- static tests
- DOM ready/race regression
- messaging
- first-bind contract

## Docker

- compose config
- health
- UI

不要因 full historical 4 backend baseline failures阻塞，只要證明沒有新增 failure。

---

# 12. SOURCE CHANGE RULE

如果需要修 source：

只修 Local auth / extension / FB cookie root cause。

不要順手重構。

每一個 fix：

- 最小 diff
- dedicated test
- commit
- 不 push除非 final local verification全部 PASS

如果只是 env wiring：

真實 credential不 commit。

---

# 13. FINAL HUMAN-OBSERVABLE ACCEPTANCE

最終必須真的達成：

1. 使用者打開 Local POST
2. Google登入按鈕可用
3. Google真實登入成功
4. 進 Web 工作台
5. 不開 Launcher仍可用
6. Chrome已登入 FB
7. Extension第一次 bind成功
8. UI顯示正確 FB account
9. 第二次 cookie sync成功
10. 沒新增 duplicate account
11. FB group/read-only data可抓
12. Local restart後仍正常

只有全部完成才可：

`LOCAL_COMPLETE=YES`

---

# 14. HARD STOP

只在以下停止：

1. 正確 Supabase project無法唯一識別
2. Google OAuth provider / callback需要外部 dashboard mutation但目前沒有安全權限
3. 需要 production secret / production mutation
4. current Chrome沒有可用 FB login且必須使用者本人登入
5. 必須做 real FB write才能驗證（不允許）
6. source root cause涉及另一 active dirty writer，無法安全隔離
7. credential/security可能外洩

如果只是需要使用者完成 Google/FB登入頁：

這不是 HARD STOP。

請只要求一次最小 browser interaction，然後繼續。

---

# 15. REQUIRED OUTPUT

```
TASK=POST_LOCAL_FULL_RECOVERY

BRANCH=
START_HEAD=
FINAL_HEAD=

LOCAL_SUPABASE_PROJECT_ID=
LOCAL_SUPABASE_SOURCE=

LEGACY_SUPABASE_FOUND=
YES/NO

LEGACY_SUPABASE_EXCLUDED=
YES/NO

FRONTEND_SUPABASE_ENV=
PASS/FAIL

BACKEND_SUPABASE_ENV=
PASS/FAIL

DOCKER_ENV_WIRING=
PASS/FAIL

GOOGLE_OAUTH_CONFIG=
PASS/FAIL

GOOGLE_LOGIN_E2E=
PASS/FAIL

SUPABASE_SESSION=
PASS/FAIL

MEMBERSHIP_BOOTSTRAP=
PASS/FAIL

TENANT_RESOLUTION=
PASS/FAIL

WEB_WITHOUT_LAUNCHER=
PASS/FAIL

SESSION_STATE_REAL_STUDIO_STATE=
PASS/FAIL

EXTENSION_BUILD_CURRENT=
PASS/FAIL

EXTENSION_FIRST_BIND=
PASS/FAIL

FB_ACCOUNT_MATCH=
PASS/FAIL

FB_COOKIE_CAPTURE=
PASS/FAIL

FB_COOKIE_PERSIST=
PASS/FAIL

DEVICE_TOKEN_SYNC=
PASS/FAIL

SECOND_COOKIE_SYNC=
PASS/FAIL

DUPLICATE_FB_ACCOUNT_CREATED=
YES/NO

FB_SESSION_VALID=
YES/NO

FB_GROUP_READ=
PASS/FAIL

LOCAL_RESTART_PERSISTENCE=
PASS/FAIL

BACKEND_TESTS=

FRONTEND_TESTS=

TYPECHECK=
PASS/FAIL

BUILD=
PASS/FAIL

EXTENSION_TESTS=

LOCAL_HEALTH=
PASS/FAIL

LOCAL_UI=
PASS/FAIL

REAL_FB_ACTION=NO

STAGING_TOUCHED=NO
PRODUCTION_TOUCHED=NO
VPS_TOUCHED=NO

BLOCKERS=

LOCAL_COMPLETE=
YES/NO
```

成功最低標準：

```
GOOGLE_LOGIN_E2E=PASS
SUPABASE_SESSION=PASS
MEMBERSHIP_BOOTSTRAP=PASS
WEB_WITHOUT_LAUNCHER=PASS
EXTENSION_FIRST_BIND=PASS
FB_COOKIE_CAPTURE=PASS
DEVICE_TOKEN_SYNC=PASS
SECOND_COOKIE_SYNC=PASS
DUPLICATE_FB_ACCOUNT_CREATED=NO
FB_SESSION_VALID=YES
FB_GROUP_READ=PASS
LOCAL_RESTART_PERSISTENCE=PASS
REAL_FB_ACTION=NO
LOCAL_COMPLETE=YES
```

完成後停止。

不要進 Staging。
不要進 Production。
