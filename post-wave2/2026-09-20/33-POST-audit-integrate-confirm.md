# POST-AUDIT-INTEGRATE — Full Local/Auth/Git/Deployment Reconciliation

- 抬頭：POST CANONICAL AUDIT + INTEGRATION OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：大型
- 任務性質：先調查 → 建立唯一事實 → 安全整合 → 真實驗證
- 禁止：先猜、先改、先部署

---

你現在是：

POST CANONICAL AUDIT + INTEGRATION OWNER

你的任務不是再做一輪零碎修補。

你要把最近幾天 TicenpiPost 的所有 Git / worktree / Local / Extension / Launcher / Docker / CI / Staging 架構重新收斂成一條「唯一可信主線」，並回答：

1. 這幾天到底改了什麼？
2. 哪些改動真的存在 Git commit？
3. 哪些只是報告/提示詞/文字紀錄？
4. 哪些 commit 已 push？哪些只在 local？
5. 哪些變更真的整合進目前候選 HEAD？
6. 哪些變更造成 regression / 做壞？
7. 哪些 regression 已修回？
8. Local 真實流程目前到底能不能用？
9. Local FB Cookies 應由 Chrome Extension 擷取的 contract 是否仍完整？
10. 商用 Staging / Production FB Cookies 應由 Launcher 擷取的 contract 是否仍完整？
11. Launcher 與 Extension 的責任有沒有被錯誤混合？
12. Google / Supabase / Membership / Tenant / Entitlement 的 Web 認證鏈現在到底怎麼跑？
13. Local Dev / Local Docker / VPS Staging / VPS Production 的真實 port 是多少？
14. 從 Local 到正式 Production 的 immutable release 流程是否閉環？
15. 最終 canonical source 應該是哪個 branch / HEAD？

不要依照聊天摘要直接當真。

必須以：

- Git commit
- git diff
- actual source
- compose
- env contract
- tests
- running local/VPS evidence
- 現有 SSOT 文件

為準。

---

# 0. MUST READ FIRST

先讀：

`F:\00-Ticenpi-SaaS\POST-LOCAL-RECOVERY-2026-09-19.md`

並把它視為「線索」，不是自動視為真實 SSOT。

同時檢查：

`F:\00-Ticenpi-SaaS\TicenpiPost`

以及所有與 TicenpiPost 有關的 git worktrees。

列出：

`git worktree list --porcelain`

不要先修改任何東西。

---

# 1. KNOWN RECENT LINEAGE / EVIDENCE TO VERIFY

下面這些是最近幾天曾回報的 commit / candidate。

你必須逐一用 git object / patch-id / ancestry 驗證，不要直接相信。

## Wave1 integration

Authoritative historical integration base曾為：

`b3cc93de970f4c7562eff3a4d5e17614462ae8bb`

Wave1內容曾包含：

- Auth product seam
- Account lifecycle
- Studio informational state
- FB selector fallback
- Extension direct first-bind
- Local runtime
- Artifact Factory

## Wave2 / integration

曾回報：

W2A recut:
- `d5a109a75142c23f3373a57480012da6ed476a52`
- `f8c053514992741f953e52a8b68da18120b0740f`

W2B Publish Safety:
- `e8e300ba65f9645b7aa23146e5164f961b438a98`

W2D Runtime Config:
- `a5ec9345fe2912fa2a2dd72862ddba7024ce4c04`

W2BD Runtime Wiring:
- `26e6fba85355e5ad3a5f68637dc83bfcde1e8289`

W2E Health/Smoke:
- `ddedd8c41f47d4dff8c99ad3ccd04b273244845a`

W2F:
- `43186fec18577ec52c897eb498cad9af15d3c51c`
- `65ecfaa75fcabb0c5c97cbdf8c931ad859844975`

Integration HEAD曾為：
- `e53162ff4a361b8a4608835bfc1b4e10672370d8`

Typecheck regression fix：
- `53fce622caae69a6b0fa476483347b17af7dc4fb`

W2G CI evidence wiring：
- `c2a0086`

你必須確認：

- commit是否存在
- parent
- changed files
- patch-id
- 是否在目前候選 branch ancestry
- 是否已 push到 remote
- 是否只有 local
- 是否有更晚版本取代

---

# 2. FIRST DELIVERABLE — GIT REALITY MAP

先做完整 READ-ONLY Git 盤點。

至少輸出：

## Repositories

- TicenpiPost repo root
- remote URL
- default branch
- current main HEAD
- all local branches relevant to recent work
- remote branches
- all worktrees

## For every relevant branch/worktree

記錄：

- branch
- HEAD
- clean/dirty
- ahead/behind remote
- upstream
- latest 15 commits
- whether commit is pushed
- whether worktree has unique uncommitted changes

## Build a commit table

欄位：

```
COMMIT
TITLE
PARENT
FILES
PURPOSE
BRANCHES_CONTAINING
PUSHED_REMOTE=YES/NO
STATUS=ACTIVE/SUPERSEDED/QUARANTINED
REGRESSION=YES/NO
FIXED_BY=
```

特別要找：

- 哪些 commit實際改了 auth
- 哪些改了 Extension
- 哪些改了 Launcher/Studio integration
- 哪些改了 compose/env
- 哪些改了 FB cookie/account flow
- 哪些只改 test
- 哪些只改 CI/release tooling

不要改任何 branch。

---

# 3. AUTH ARCHITECTURE — MUST RECONCILE CORRECTLY

這一段是最高優先級。

## Web Identity

預期架構是：

`Google OAuth → Supabase Session/JWT → Membership → Tenant → Entitlement`

但你必須從 source驗證。

回答：

- Google login code在哪
- Supabase client在哪
- backend JWT verification在哪
- membership resolve在哪
- tenant resolve在哪
- entitlement gate在哪
- Local 是否 bypass entitlement
- Production/Staging 是否 enforce entitlement

## Local FB Authentication / Cookies

Local 開發模式預期：

`Chrome Extension → 擷取 Facebook Cookies/Session → Local Backend`

Extension 負責：

- first bind
- FB account identity
- FB cookies/session capture
- recurring device-token sync

驗證：

- Extension source path
- manifest.json
- build path
- first-bind endpoint
- recurring sync endpoint
- device token lifecycle
- duplicate FB account protection
- cookie storage/encryption
- local-only branch condition

不要把 Launcher當 Local first-bind prerequisite。

## Commercial FB Authentication / Cookies

Staging / Production 商用模式預期：

`Launcher → 擷取 Facebook Cookies/Session → VPS Backend`

Launcher負責：

- 商用使用者 Facebook browser/session能力
- FB cookie acquisition/sync
- desktop device identity
- heartbeat / Studio capability

驗證：

- Launcher如何取得 Google/Supabase user identity
- Launcher如何綁 Customer/Tenant
- Launcher如何取得/送出 FB cookies
- Launcher如何更新 cookies
- VPS endpoint如何驗證 Launcher/device
- backend如何把 Launcher送來的 FB account/cookies綁到 User/Tenant

如果 source目前根本沒有完整 Launcher cookie path：

明確標：

`COMMERCIAL_LAUNCHER_FB_COOKIE_FLOW=NOT_IMPLEMENTED/PARTIAL`

不要用 Extension flow假裝商用流程存在。

## IMPORTANT

正確概念不是：

`Extension auth → handoff → Launcher auth`

而是：

Local:
`Extension → Local Backend`

Commercial:
`Launcher → VPS Backend`

Web identity共同使用：

`Google/Supabase User + Tenant`

請確認現在 source是否真的符合。

---

# 4. PORT SSOT — DO NOT GUESS

不要用聊天中任何曾猜過的 port。

直接從：

- source startup commands
- docker-compose.yml
- deploy/staging/compose.yml
- deploy/runtime-staging/compose.yml
- env
- Docker labels
- current listeners
- deployment manifests

建立唯一表：

```
LOCAL_DEV_HOST_PORT=
LOCAL_DEV_INTERNAL_PORT=

LOCAL_DOCKER_HOST_PORT=
LOCAL_DOCKER_CONTAINER_PORT=

VPS_STAGING_HOST_PORT=
VPS_STAGING_CONTAINER_PORT=

VPS_PRODUCTION_HOST_PORT=
VPS_PRODUCTION_CONTAINER_PORT=
```

如果 Local Dev 與 Local Docker不能同時跑：

指出衝突。

如果 source沒有 canonical Local Dev port：

標 UNKNOWN/CONFIGURABLE，不要猜。

---

# 5. LOCAL SUPABASE / GOOGLE OAUTH — FIND THE ACTUAL BREAK

目前使用者真實看到過：

`Google 登入尚未設定（NEXT_PUBLIC_SUPABASE_*）`

所以必須找 root cause。

檢查：

- frontend讀哪些 NEXT_PUBLIC_SUPABASE_* 變數
- backend讀哪些 SUPABASE_* 變數
- docker-compose是否有 wiring
- local dev env是否有 wiring
- .env.example
- current secure local env source
- Central Secret Store
- 是否指向 legacy Supabase
- 正確 project ID / URL source
- Google provider是否 enabled
- callback URL contract

輸出：

```
LOCAL_GOOGLE_LOGIN_CURRENT=
PASS/FAIL

ROOT_CAUSE=

FRONTEND_ENV_WIRING=
PASS/FAIL

BACKEND_ENV_WIRING=
PASS/FAIL

CORRECT_SUPABASE_PROJECT=

LEGACY_SUPABASE_REFERENCE_FOUND=
YES/NO

SAFE_FIX_REQUIRED=
```

若要修，先提出最小 fix plan。

只有在 root cause唯一且安全時才修改。

---

# 6. LOCAL FUNCTIONAL RECOVERY — REAL E2E, NOT JUST TESTS

只有 Local真實跑通後才能往後。

## Local Dev

驗證 actual canonical Local Dev mode。

必須：

- Google Login
- Supabase session
- Membership/Tenant
- Web不用 Launcher也能進
- Extension current build
- FB first bind
- FB cookie capture
- second cookie sync
- no duplicate account
- FB group/read-only fetch
- restart persistence

## Local Docker

重建 canonical Local Docker。

必須：

- 正確 host/container port
- exact current source
- Google Login
- Supabase session
- Extension first bind
- FB cookie sync
- FB read-only
- health
- UI
- restart persistence

只有真實 E2E才可：

`LOCAL_ACCEPTED=YES`

不要用：

- UI 200
- /api/health 200
- unit tests PASS

取代真實 login/cookie驗證。

---

# 7. LAUNCHER COMMERCIAL FLOW — VERIFY BEFORE STAGING

這幾天最可能被忽略的就是這條。

在任何 Staging deploy前，先從 source/code path回答：

```
LAUNCHER_AUTH_IMPLEMENTED=
YES/NO/PARTIAL

LAUNCHER_GOOGLE_SUPABASE_IDENTITY=
PASS/FAIL/NOT_FOUND

LAUNCHER_TENANT_BINDING=
PASS/FAIL/NOT_FOUND

LAUNCHER_FB_COOKIE_CAPTURE=
PASS/FAIL/NOT_FOUND

LAUNCHER_FB_COOKIE_SYNC=
PASS/FAIL/NOT_FOUND

VPS_DEVICE_AUTH=
PASS/FAIL/NOT_FOUND

COMMERCIAL_FLOW_READY_FOR_STAGING=
YES/NO
```

如果 Launcher商用 FB cookie flow不完整：

這就是正式 blocker。

不要先部署 Staging假裝完成。

---

# 8. RELEASE / CI / IMMUTABLE DELIVERY

確認目前 source是否已具備：

## Post side

- W2G `--extension-metadata` wiring
- clean source gate
- extension package
- ARM64 build
- GHCR push
- digest capture
- release evidence
- exact immutable_reference

## Central Deploy

曾有 local lineage：

- 4298986
- 808fe81
- 39ecc94

你要確認：

- 這些 commit在哪個 repo
- 是否只有 local
- 是否已 push
- 是否需要整合
- immutable deploy是否 exact digest
- VPS build是否 disabled
- release source snapshot是否 bound to git SHA
- GHCR credential transport是否完成
- VPS control-plane目前是否仍舊版

不要管 DM產品功能本身。

只檢查 POST需要的 isolated Central Deploy path。

---

# 9. CANONICAL RELEASE FLOW TO CONFIRM

最終流程必須整理成下列格式，並填入「實際 port / branch / SHA / service」。

```
① 最新 Canonical Source
   ↓
② Local Dev 真實驗證
   ↓
③ Local Google / Supabase
   ↓
④ Local Chrome Extension FB Cookies First Bind
   ↓
⑤ Local Device Token / Second Cookie Sync
   ↓
⑥ Local FB Read-only
   ↓
⑦ LOCAL_DEV_ACCEPTED
   ↓
⑧ 重建 Local Docker
   ↓
⑨ Local Docker 完整 E2E
   ↓
⑩ LOCAL_DOCKER_ACCEPTED
   ↓
⑪ Freeze Candidate Git SHA
   ↓
⑫ Push GitHub
   ↓
⑬ GitHub Actions ARM64 Build
   ↓
⑭ Extension Package + Metadata
   ↓
⑮ Immutable Release Evidence + Exact Digest
   ↓
⑯ Staging Preflight
   ↓
⑰ SAME Candidate / Exact Digest → VPS Staging
   ↓
⑱ Staging Automated Gates
   ↓
⑲ Commercial Launcher Google/Supabase Auth
   ↓
⑳ Launcher FB Cookie Capture / Sync → VPS
   ↓
㉑ VPS FB Read-only
   ↓
㉒ Staging Browser / Launcher UAT
   ↓
㉓ STAGING_ACCEPTED
   ↓
㉔ Production Preflight
   ↓
㉕ SAME CI ARM64 Digest → VPS Production
   ↓
㉖ Production Gates + Human Smoke
   ↓
㉗ ACCEPT / ROLLBACK
   ↓
㉘ SSOT / Hub / History 更新
```

注意：

Local FB cookie path = Chrome Extension

Commercial Staging/Production FB cookie path = Launcher

不可再混用。

---

# 10. INTEGRATION DECISION

完成調查後，先建立：

`CANONICAL_INTEGRATION_PLAN`

內容：

- chosen canonical branch
- chosen canonical HEAD
- commits to keep
- commits to drop
- commits to cherry-pick
- commits superseded
- local-only changes
- remote-pushed changes
- source fixes still required
- env fixes required
- Extension fixes required
- Launcher fixes required
- CI fixes required
- Central Deploy fixes required

如果目前已經存在 clean branch包含所有正確變更：

不要再重建。

如果需要 integration：

建立 fresh isolated worktree。

禁止直接在 dirty original tree整合。

任何 conflict：

STOP。

不要 ours/theirs盲解。

---

# 11. MUTATION RULE

這不是「只讀報告」任務。

但必須：

先調查完
→ 建立 canonical truth
→ 才能修改

允許的修改只限：

- 明確確認的 regression
- 缺少的 Local auth/env wiring
- Local Extension contract
- Commercial Launcher contract
- CI evidence wiring
- necessary integration commits

不要順手重構。

每個修改都：

- minimal diff
- dedicated test
- commit
-記錄 SHA

不要碰 Production。

不要 real FB write。

---

# 12. FINAL VALIDATION

至少：

## Local

- Google real login
- Supabase session
- Membership/Tenant
- Extension first bind
- cookies captured
- second sync
- duplicate = NO
- FB read-only
- Local Docker
- restart persistence

## Source

- backend targeted
- frontend tests
- typecheck
- build
- extension tests
- compose config
- secret scan

## Commercial architecture

- Launcher identity path
- Launcher FB cookie path
- VPS device auth contract

如果 Launcher path尚不能真 E2E：

不要宣告 staging ready。

---

# 13. REQUIRED OUTPUT

```
TASK=POST_CANONICAL_AUDIT_INTEGRATE

=== GIT REALITY ===

POST_REPO=
REMOTE=
DEFAULT_BRANCH=
MAIN_HEAD=

WORKTREES=

CANONICAL_BRANCH=
CANONICAL_HEAD=
CANONICAL_WORKTREE=

PUSHED_COMMITS=
LOCAL_ONLY_COMMITS=
SUPERSEDED_COMMITS=
QUARANTINED_COMMITS=

RECENT_CHANGE_TABLE=

=== PORT SSOT ===

LOCAL_DEV_HOST_PORT=
LOCAL_DEV_INTERNAL_PORT=

LOCAL_DOCKER_HOST_PORT=
LOCAL_DOCKER_CONTAINER_PORT=

VPS_STAGING_HOST_PORT=
VPS_STAGING_CONTAINER_PORT=

VPS_PRODUCTION_HOST_PORT=
VPS_PRODUCTION_CONTAINER_PORT=

=== WEB AUTH ===

GOOGLE_SUPABASE_CHAIN=
PASS/FAIL

LOCAL_GOOGLE_LOGIN=
PASS/FAIL

FRONTEND_SUPABASE_ENV=
PASS/FAIL

BACKEND_SUPABASE_ENV=
PASS/FAIL

MEMBERSHIP_TENANT=
PASS/FAIL

ENTITLEMENT_CONTRACT=
PASS/FAIL

=== LOCAL EXTENSION / FB ===

EXTENSION_SOURCE=
EXTENSION_BUILD=

LOCAL_EXTENSION_FIRST_BIND=
PASS/FAIL

LOCAL_FB_COOKIE_CAPTURE=
PASS/FAIL

LOCAL_DEVICE_TOKEN_SYNC=
PASS/FAIL

SECOND_COOKIE_SYNC=
PASS/FAIL

DUPLICATE_FB_ACCOUNT=
YES/NO

LOCAL_FB_READ=
PASS/FAIL

=== COMMERCIAL LAUNCHER / FB ===

LAUNCHER_AUTH_IMPLEMENTED=
YES/NO/PARTIAL

LAUNCHER_GOOGLE_SUPABASE_IDENTITY=
PASS/FAIL/NOT_FOUND

LAUNCHER_TENANT_BINDING=
PASS/FAIL/NOT_FOUND

LAUNCHER_FB_COOKIE_CAPTURE=
PASS/FAIL/NOT_FOUND

LAUNCHER_FB_COOKIE_SYNC=
PASS/FAIL/NOT_FOUND

VPS_DEVICE_AUTH=
PASS/FAIL/NOT_FOUND

COMMERCIAL_FLOW_READY_FOR_STAGING=
YES/NO

=== REGRESSIONS ===

REGRESSIONS_INTRODUCED=

REGRESSIONS_ALREADY_FIXED=

UNRESOLVED_REGRESSIONS=

=== CI / RELEASE ===

W2G_EVIDENCE_WIRING=
PASS/FAIL

ARM64_CI=
PASS/FAIL/NOT_RUN

IMMUTABLE_RELEASE_CONTRACT=
PASS/FAIL

CENTRAL_DEPLOY_STATUS=

VPS_CONTROL_PLANE_STATUS=

=== LOCAL ACCEPTANCE ===

LOCAL_DEV_ACCEPTED=
YES/NO

LOCAL_DOCKER_ACCEPTED=
YES/NO

=== FINAL INTEGRATION ===

COMMITS_INTEGRATED=
NEW_FIX_COMMITS=
FINAL_HEAD=
FINAL_WORKTREE_CLEAN=
YES/NO

=== NEXT RELEASE FLOW ===

RELEASE_FLOW=
<use the numbered ①→㉘ format with actual values>

BLOCKERS=

READY_FOR_STAGING=
YES/NO
```

---

# 14. SUCCESS STANDARD

不可以只因 unit tests PASS就寫 READY。

最低要求：

```
LOCAL_GOOGLE_LOGIN=PASS
LOCAL_EXTENSION_FIRST_BIND=PASS
LOCAL_FB_COOKIE_CAPTURE=PASS
LOCAL_DEVICE_TOKEN_SYNC=PASS
SECOND_COOKIE_SYNC=PASS
DUPLICATE_FB_ACCOUNT=NO
LOCAL_FB_READ=PASS
LOCAL_DOCKER_ACCEPTED=YES

LAUNCHER_AUTH_IMPLEMENTED=YES
LAUNCHER_FB_COOKIE_CAPTURE=PASS
LAUNCHER_FB_COOKIE_SYNC=PASS
VPS_DEVICE_AUTH=PASS

FINAL_WORKTREE_CLEAN=YES
READY_FOR_STAGING=YES
```

如果 Launcher商用流程未完成：

`READY_FOR_STAGING=NO`

並明確指出最小缺口。

不要用 Extension Local path冒充商用 Launcher path。
