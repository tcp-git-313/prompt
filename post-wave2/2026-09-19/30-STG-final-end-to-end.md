# STG-FINAL — POST Staging Completion End-to-End

- 抬頭：POST STAGING COMPLETION OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：大型（跨 Post + Central Deploy + CI/GHCR + VPS Staging）
- 目標：一次執行到 `STAGING_COMPLETE=YES`
- Production：嚴禁觸碰

---

你現在是：

POST STAGING COMPLETION OWNER

你的唯一目標：

在不觸碰 Production、不中斷其他產品、不中斷 DM/ORC active work、不中斷現有 Post runtime-staging stack 的前提下，把 POST 從目前狀態一路完成到：

`STAGING_COMPLETE=YES`

這是一個 end-to-end execution task。

不要每完成一小段就回來等確認。

只要沒有命中 HARD STOP，就持續往下一階段執行，直到 Staging 完整完成。

---

# 0. AUTHORITATIVE CURRENT STATE

## POST repo

Repo：

`F:\00-Ticenpi-SaaS\TicenpiPost`

Integration worktree：

`F:\00-Ticenpi-SaaS\TicenpiPost-wave2-integration-v2`

Branch：

`repair/post-v2-wave2-integration-v2`

已完成 W2INT-v2 + W2F-FIX2：

Current expected HEAD：

`53fce622caae69a6b0fa476483347b17af7dc4fb`

狀態：

- backend full：642 passed / 4 pre-existing failures / 0 new
- frontend full：PASS
- typecheck：PASS
- build：PASS
- access matrix：PASS
- publish safety：PASS
- runtime config：PASS
- config drift：PASS
- health/smoke：PASS
- local Docker：PASS
- worktree clean
- Production untouched
- VPS untouched

## W2G CI evidence fix

已完成但尚未整合：

`c2a0086`

預期唯一修改：

`.github/workflows/ci.yml`

內容：

將 package_extension 已產生的 metadata JSON 傳入：

`generate_release_evidence.py --extension-metadata ...`

本地 evidence generation + validation 已 PASS。

## Central Deploy frozen lineage

Clean isolated worktree：

`F:\00-Ticenpi-SaaS\deploy-w2-post`

Branch：

`repair/post-immutable-deploy`

已完成：

1. immutable delivery  
   `429898662fb487eb4bafe222dd11e60453a8a204`

2. G6 evidence + GHCR auth contract  
   `808fe81e082602c50a4c3770487b1d67b9755612`

3. staging service identity + immutable source pinning  
   `39ecc94fdaed5cd2950b277cc15d1b39da28e6c1`

39ecc94 已驗：

- post → evidence service post
- postruntimestaging → evidence service post
- unrelated service fail closed
- deploy.ps1 從 evidence.git_sha 建 detached clean source snapshot
- mutable working tree不進 immutable package
- exact digest preserved
- legacy source preserved

## Central Deploy main

Repo：

`F:\00-Ticenpi-SaaS\deploy`

先前 main 有 DM/ORC active dirty changes，曾涉及：

- server/deploy.sh
- server/audit.sh
- server/critical-smoke.sh
- services.yaml
- DM_RELEASE_GATES.md
- server/warp-proxy-relay-dmruntimestaging.service

本任務開始時必須重新檢查，不得假設仍相同。

## VPS / runtime staging preflight

VPS：

`147.224.255.234`

已驗：

- reachable
- arch = aarch64
- Docker = available
- docker compose = available
- CI image platform = linux/arm64
- platform match = YES
- port 19419 = intended Post runtime-staging
- current stack：
  `ticenpi-post-runtime-staging`
- current symlink有效
- legacy source stack currently running
- release root exists

但：

- VPS `/opt/ticenpi/deploy/deploy.sh` 仍是舊版
- VPS 沒有 immutable_image.py
- VPS deployer尚未具備 immutable-image/GHCR flow
- GHCR token雖存在 Central Secret Store：
  `F:\Secrets\ticenpi.env`
  但先前沒有實際安全 injection path 到 remote deploy process

## Staging target

Service：

`postruntimestaging`

Compose project：

`ticenpi-post-runtime-staging`

Loopback binding：

`127.0.0.1:19419:9418`

Health：

`http://127.0.0.1:19419/api/health`

目前 public staging domain可能未 route：

`post-runtime-staging.ticenpi.com`

不得假設 public route已存在。

---

# 1. GLOBAL HARD RULES

## NEVER TOUCH PRODUCTION

禁止任何：

- Production deploy
- Production restart
- Production database mutation
- Production Supabase mutation
- Production publish gate enable
- Production traffic cutover

Production service `post` 不得部署。

只允許：

`postruntimestaging`

## NEVER EXPOSE SECRETS

禁止輸出：

- GHCR_READ_TOKEN
- GHCR_USERNAME value
- Supabase secret
- OAuth secret
- FB token/cookie
- JWT
- passwords

不得輸出 secret：

- value
- length
- prefix
- suffix
- hash

只允許：

`PRESENT=YES/NO`

## NO REAL FACEBOOK WRITE

本任務沒有授權真實：

- Facebook post
- comment
- delete
- relist
- marketplace publish

Publish E2E只做到：

- dry-run
- queue/gate
- fail-closed
- controlled non-writing smoke

`REAL_FB_ACTION=NO`

## NEVER DESTROY OTHER WORK

禁止：

- git reset --hard
- git clean
- git stash別人的工作
- overwrite dirty main
- commit別人的未提交變更
- docker system prune
- docker volume prune
- stop unrelated containers
- delete unrelated worktrees

---

# 2. HARD STOP CONDITIONS

只有下列情況才停止整個任務：

1. Post expected branch/HEAD/worktree state無法安全辨識
2. W2G cherry-pick conflict
3. Central Deploy shared files仍有 active uncommitted writer，且沒有可證明安全的 committed integration point
4. Central Deploy frozen W2C commits無法 cleanly replay到目前 committed main
5. shared-file bridge出現 semantic conflict且無法由現有 contract唯一決定
6. 無法建立安全 GHCR credential transport而必須把 token放 argv/log/persistent plaintext
7. control-plane rollout無 rollback path
8. CI failure不是已知 baseline / infrastructure transient
9. release evidence與 final Post HEAD / digest不一致
10. GHCR immutable ref不是 exact `@sha256`
11. Staging deploy需要 VPS build
12. health / critical smoke / audit FAIL且 rollback後仍無法恢復
13. staging deploy會碰 Production或其他產品
14. OAuth/Entitlement E2E若被定義為本次必須，卻因無 public route完全無法驗證且沒有既有安全替代路徑

命中 HARD STOP：

- 停止
- 保留 evidence
- 若已進行 staging cutover且健康失敗，先 rollback到前一個 staging release
- 不碰 Production
- 輸出唯一 blocker

其餘非致命問題自行處理後繼續。

---

# 3. PHASE A — FINALIZE POST CANDIDATE

## A1. Verify W2INT final state

Worktree：

`F:\00-Ticenpi-SaaS\TicenpiPost-wave2-integration-v2`

確認：

- branch = repair/post-v2-wave2-integration-v2
- HEAD = 53fce622caae69a6b0fa476483347b17af7dc4fb
- worktree clean
- no merge/rebase/cherry-pick operation

若 HEAD不同：

先調查是否是已知 W2G merge或其他本任務允許變更。

不可猜。

## A2. Merge W2G

先：

`git show --name-only c2a0086`

必須只有：

`.github/workflows/ci.yml`

然後：

cherry-pick exact：

`c2a0086`

若 conflict：
HARD STOP。

## A3. Validate W2G closure

至少：

- YAML parse
- extension metadata output path存在
- CI generate_release_evidence實際傳入 --extension-metadata
- local package extension metadata
- local evidence generation PASS
- local evidence validation PASS
- `npx tsc --noEmit` PASS
- `git diff --check` PASS
- worktree clean

不需要重跑 full backend 642 tests。

## A4. Define final Post candidate

記錄：

`POST_FINAL_HEAD=<new HEAD>`

這是唯一可進 Staging的 source SHA。

## A5. Push candidate branch

只 push：

`repair/post-v2-wave2-integration-v2`

不得 push/merge main。

Push前確認：

- clean
- final HEAD exact
- no secret files
- no unexpected untracked files

---

# 4. PHASE B — WAIT FOR REAL CI / GHCR ARTIFACT

Post CI on push 必須完成。

不要 fabricated local digest。

## B1. CI gates

要求：

- backend job PASS（已知 baseline若 CI contract允許）
- frontend job PASS
- arm64 image job PASS
- source clean gate PASS
- extension package PASS
- metadata generation PASS
- image push PASS
- buildx digest capture PASS
- release evidence generation PASS
- release evidence validation PASS
- artifact upload PASS

如果是 transient runner/network failure：

可 retry一次既有 workflow。

如果是 source/test failure：

HARD STOP。

## B2. Fetch authoritative artifact

取得：

- `post-release-<POST_FINAL_HEAD>.json`
- exact immutable reference：
  `ghcr.io/tcp-git-313/ticenpi-post@sha256:<64hex>`
- extension metadata/artifact evidence

驗：

- evidence.git_sha == POST_FINAL_HEAD
- source_commit == POST_FINAL_HEAD（若 schema要求）
- source_state == clean
- artifact_digest == immutable_reference digest
- service == post
- platform == linux/arm64
- evidence schema PASS

記錄：

`POST_IMAGE_DIGEST=`
`POST_IMMUTABLE_REF=`

---

# 5. PHASE C — CENTRAL DEPLOY WRITER / BASE RECONCILIATION

## C1. Re-check main now

Repo：

`F:\00-Ticenpi-SaaS\deploy`

只讀確認：

- current branch
- current HEAD
- worktree status
- worktrees
- active task evidence

特別檢查：

- server/deploy.sh
- server/audit.sh
- server/critical-smoke.sh
- services.yaml

## C2. Active writer gate

如果 shared files仍有「未提交、正在進行中的 DM/ORC writer」：

HARD STOP。

不要：

- steal dirty diff
- commit它
- stash它
- copy它到自己 branch後假裝整合

如果 changes已經 commit到 current main / committed branch，且 main clean：

繼續。

如果 main有 unrelated untracked files但 shared files clean且可證明無 active writer：

只忽略 unrelated file，不刪。

## C3. Fresh Central Deploy integration worktree

不要直接在 dirty/main施工。

從「最新安全 committed Central Deploy main HEAD」建立 fresh worktree，例如：

`F:\00-Ticenpi-SaaS\deploy-post-staging-final`

branch：

`repair/post-staging-final`

記錄：

`CENTRAL_BASE_HEAD=`

## C4. Replay frozen W2C lineage

依序 cherry-pick：

1. 429898662fb487eb4bafe222dd11e60453a8a204
2. 808fe81e082602c50a4c3770487b1d67b9755612
3. 39ecc94fdaed5cd2950b277cc15d1b39da28e6c1

若任一 conflict：
HARD STOP。

禁止 ours/theirs盲解。

---

# 6. PHASE D — GHCR CREDENTIAL TRANSPORT

目前 source contract讀：

- GHCR_READ_TOKEN
- GHCR_USERNAME

Central Secret Store：

`F:\Secrets\ticenpi.env`

## D1. Build safe transport

建立：

Central Secret Store
→ deploy.ps1 local process
→ secure stdin / ephemeral remote process env
→ remote deploy process
→ `docker login ghcr.io --password-stdin`

要求：

- GHCR_READ_TOKEN required
- GHCR_USERNAME optional，沿用既有 safe default
- token不在 argv
- token不在 remote command string
- token不在 log
- token不進 evidence
- token不進 manifest
- token不進普通 temp files
- 不需要 sshd AcceptEnv
- 不需要 sudoers env_keep
- remote process exit後不持久化

優先只改 deploy.ps1 / helper。

如果現有 remote execution模型確實要求 server/deploy.sh協作，此階段已在 fresh Central integration branch，可在不破壞 shared current committed changes的前提下做最小修改。

## D2. Test credential transport

fixture only：

- present → plan PASS
- missing → fail before SSH mutation
- token not in argv
- token not in logs
- username absent → safe default
- malformed Central Secret Store → fail closed
- legacy-source unaffected

不要 real docker login。

Commit：

`fix(deploy): securely transport ghcr credentials`

---

# 7. PHASE E — FINAL SHARED-FILE BRIDGE

基於「目前最新 committed Central Deploy main內容」建立 Post registration。

不得覆蓋 DM/ORC已提交功能。

## E1. services.yaml

為：

- post
- postruntimestaging

加入正確：

`criticalSmokeProfile`

預期 profile：

`post-core`

若 repo已有 canonical命名，優先沿用 canonical。

不要刪/改其他產品 profile。

## E2. server/critical-smoke.sh

加入 Post adapter。

Central caller contract：

- --service
- --profile
- --release

Post W2E contract需要：

- base-url
- compose-project

建立明確 mapping：

### postruntimestaging
- base URL = http://127.0.0.1:19419
- compose project = ticenpi-post-runtime-staging

### post
不得在本任務實際執行 Production smoke，但 registration可保持既有 production manifest需要的合法值。

不要猜 production port；從 current services.yaml/contract取得。

Adapter必須呼叫 release中/approved Post smoke implementation，不得硬編另一套功能邏輯。

## E3. server/deploy.sh

同步：

`valid_smoke_profiles`

加入 Post profile。

保留：

DM
591
ORC
以及 current committed main新增的所有 profiles。

Unknown profile仍 fail closed。

## E4. server/audit.sh

同步相同 profile whitelist。

deploy.sh / audit.sh profile whitelist不得無故不一致。

required_services：

只在現有 manifest architecture確實要求 postruntimestaging成為 required service時才加。

不要為了過 test強行擴 required_services。

## E5. Central validation

至少：

- bash -n
- PowerShell parse
- immutable_image tests
- W2C tests
- manifest validation
- all known service/profile fixtures
- post / postruntimestaging registration
- existing DM/ORC/Sign/591 profiles不 regression
- legacy mode
- immutable dry-run
- rollback path
- git diff --check
- secret scan

Commit：

`feat(deploy): register post immutable staging delivery`

記錄：

`CENTRAL_FINAL_HEAD=`

Push：

`repair/post-staging-final`

不得 merge main。

---

# 8. PHASE F — CONTROL-PLANE VPS ROLLOUT

現在才允許更新：

`/opt/ticenpi/deploy`

但只更新 deployment control plane，不部署任何 product。

## F1. Inspect existing install mechanism

先確認 VPS目前：

- deploy root
- ownership/mode
- backup/version scheme
- symlinks
- current hashes

## F2. Safe rollout mode

優先順序：

1. versioned/side-by-side candidate directory
2. validated candidate
3. atomic cutover
4. old deployer可立即 rollback

如果現有平台沒有 versioned mechanism：

建立最低限度：

- timestamped backup
- temporary candidate path
- syntax/fixture validation
- atomic replace

但不能破壞 legacy deployment。

## F3. Files

只 rollout正式 control-plane所需檔案，例如：

- server/deploy.sh
- server/audit.sh
- server/critical-smoke.sh
- server/immutable_image.py
- services.yaml
- necessary helper

deploy.ps1留在 local operator side，不需要安裝到 Linux VPS，除非 repo既有架構明確如此。

## F4. Validate control plane before product deploy

VPS上只做 non-mutating validation：

- bash -n
- python helper --help/schema fixture
- manifest validation
- legacy dry-run
- immutable dry-run
- GHCR missing-auth fail closed
- service mapping：
  postruntimestaging → evidence post
- exact digest parsing
- no product container changed

驗：

現有 `ticenpi-post-runtime-staging` container IDs / uptime沒有因 control-plane rollout改變。

若 control plane validation FAIL：

rollback control plane。

HARD STOP。

---

# 9. PHASE G — GHCR AUTH REAL PRECHECK

現在允許對 GHCR做 authentication precheck，但不要 pull/run image以外的多餘操作。

## G1. Presence only

確認：

GHCR_READ_TOKEN_PRESENT=YES

GHCR_USERNAME_PRESENT=YES/NO

不要輸出值。

## G2. Secure auth

透過剛實作的 secure transport：

- remote deploy process收到 ephemeral credential
- `docker login ghcr.io --password-stdin`

不得在 output中顯示 token。

如果 auth FAIL：
HARD STOP。

---

# 10. PHASE H — IMMUTABLE STAGING DEPLOY

只部署：

`postruntimestaging`

使用：

`POST_IMMUTABLE_REF`

與 authoritative：

`post-release.json`

## H1. Pre-deploy identity

必須確認：

- target service = postruntimestaging
- evidence service = post
- mapping合法
- evidence.git_sha = POST_FINAL_HEAD
- source snapshot HEAD = POST_FINAL_HEAD
- source snapshot clean
- artifact digest match
- image exact @sha256
- platform arm64
- deploy mode = immutable-image
- VPS build = NO

## H2. Deploy

使用 Central Deploy正式入口：

`deploy.ps1`

等價目標：

`-Service postruntimestaging`
`-DeployMode immutable-image`
`-ImageRef <POST_IMMUTABLE_REF>`
`-EvidencePath <post-release.json>`

不要手工繞過 deployment framework。

## H3. Required remote execution

Immutable path：

- validate manifest
- validate evidence binding
- validate GHCR auth
- preflight
- extract exact release snapshot
- render compose:
  - remove build:
  - pin api/worker to exact image@sha256
- pull exact digest
- `docker compose up -d --no-build`
- health
- critical smoke
- commit manifest/release identity
- post-deploy audit

任何 FAIL：

自動 rollback至前一 staging release。

Rollback後驗舊 staging health。

若舊 release也不健康：
HARD STOP並報告。

---

# 11. PHASE I — STAGING HEALTH / SMOKE / AUDIT GATE

這是 E2E前唯一 serial gate。

必須 PASS：

## Deployment identity

- running api/worker image digest == POST_IMAGE_DIGEST
- no build occurred on VPS
- release identity git SHA == POST_FINAL_HEAD
- evidence stored/linked correctly
- current symlink points new release

## Health

- GET http://127.0.0.1:19419/api/health = 200
- status ok
- container readiness
- Redis
- Postgres
- worker
- config drift

## Critical smoke

`post-core`

必須 PASS。

## Audit

Central：

`-Service postruntimestaging -Audit`

必須 PASS。

若任一 FAIL：
rollback + HARD STOP。

---

# 12. PHASE J — STAGING E2E MATRIX

Health Gate PASS後，可平行執行以下驗證。

不要讓不同 E2E writer修改 source。

## Track J1 — Auth / Membership / Entitlement

使用既有 staging/test identities或已配置環境。

不要建立 production customer。

驗：

- Google/Supabase identity path
- JWT principal
- membership
- entitled access
- not_entitled blocked
- no_tenant blocked
- tenant isolation
- production-like X-Tenant-Key disabled behavior

如果 Staging沒有 public OAuth callback route：

先檢查 repo/infra是否已有 staging route機制。

若能安全建立「僅 staging」route且不碰 Production：
建立並驗證。

若沒有權限/機制：
不要亂改 DNS；標記 blocker。

但至少完成 backend auth contract smoke。

## Track J2 — Studio informational state

驗：

- online
- studio_offline
- account_mismatch
- /api/session/state回報真實 capability
- offline/mismatch不阻擋 Web product access

## Track J3 — Mobile/Web

若 public route可用：

- mobile viewport
- Web shell
- auth return
- core navigation

若只能 loopback：

完成可自動驗的 responsive/build/runtime contract，不偽造外部 OAuth成功。

## Track J4 — Extension / Bind contract

不做 real FB write。

驗：

- extension package metadata matches POST_FINAL_HEAD
- first-bind path contract
- DOMContentLoaded/race regression
- recurring device token sync contract
- backend endpoint/auth contract

只有既有安全 test account/session可用時才做登入/綁定；不得輸出 cookie/token。

## Track J5 — Publish Safety

Staging environment：

驗：

- master OFF → no write
- master ON + all action OFF → no write
- individual action gates isolation
- action ON + master OFF → no write
- unknown action fail closed
- dry-run queue/executor path

不得打開 real external write。

## Track J6 — Marketplace / Relist

只做：

- queue
- payload
- dry-run
- safety gate
- no real FB write

## E2E PASS rule

如果 public route / Google OAuth因現有 staging infrastructure缺失而無法做 browser OAuth，但：

- backend auth contract PASS
- entitlement/membership integration可用
- health/smoke/audit PASS
- immutable identity PASS
- no source/runtime blocker

則輸出：

`STAGING_TECHNICAL_COMPLETE=YES`

但：

`STAGING_COMPLETE=YES`

只有在「本專案既定 staging acceptance」不要求 public browser OAuth時才可給 YES。

若既定 acceptance要求 browser OAuth，則必須把 route問題解決後才 YES。

不要為了達成 YES 降低標準。

---

# 13. PHASE K — FINAL FREEZE

若全部要求 PASS：

記錄：

## Post

- final branch
- POST_FINAL_HEAD
- CI run id/url
- image digest
- immutable ref
- evidence artifact

## Central Deploy

- branch
- CENTRAL_BASE_HEAD
- CENTRAL_FINAL_HEAD
- control-plane deployed hash/version

## Staging

- service
- release id
- current symlink
- running image digest
- health
- critical smoke
- audit
- E2E matrix

確認：

- Production untouched
- real FB action = NO
- staging source == evidence git SHA
- staging image == CI exact digest
- VPS build = NO

---

# 14. FINAL OUTPUT

只有所有 mandatory staging gates真的完成，才可輸出：

`STAGING_COMPLETE=YES`

完整輸出：

```
TASK=POST_STAGING_COMPLETION

POST_BRANCH=
POST_FINAL_HEAD=

W2G_MERGED=
YES/NO

POST_REMOTE_PUSH=
PASS/FAIL

CI_RUN=
CI_STATUS=

RELEASE_EVIDENCE=
PASS/FAIL

POST_IMAGE_DIGEST=
POST_IMMUTABLE_REF=

CENTRAL_BASE_HEAD=
CENTRAL_BRANCH=
CENTRAL_FINAL_HEAD=

W2C_IMMUTABLE_LINEAGE=
PASS/FAIL

SHARED_WRITER_CONFLICT=
YES/NO

SHARED_FILE_BRIDGE=
PASS/FAIL

GHCR_CREDENTIAL_TRANSPORT=
PASS/FAIL

TOKEN_EXPOSED=
NO

CENTRAL_REMOTE_PUSH=
PASS/FAIL

VPS_CONTROL_PLANE_ROLLOUT=
PASS/FAIL

VPS_CONTROL_PLANE_ROLLBACK_READY=
YES/NO

VPS_BUILD_USED=
NO

STAGING_SERVICE=postruntimestaging
STAGING_PORT=19419

DEPLOYED_GIT_SHA=
DEPLOYED_IMAGE_DIGEST=

SOURCE_EVIDENCE_MATCH=
PASS/FAIL

IMMUTABLE_DIGEST_MATCH=
PASS/FAIL

HEALTH=
PASS/FAIL

CRITICAL_SMOKE=
PASS/FAIL

AUDIT=
PASS/FAIL

AUTH_ENTITLEMENT_E2E=
PASS/FAIL/PARTIAL

STUDIO_STATE_E2E=
PASS/FAIL

MOBILE_WEB_E2E=
PASS/FAIL/PARTIAL

EXTENSION_BIND_E2E=
PASS/FAIL/PARTIAL

PUBLISH_SAFETY_E2E=
PASS/FAIL

MARKETPLACE_RELIST_E2E=
PASS/FAIL

REAL_FB_ACTION=NO

PRODUCTION_TOUCHED=NO

ROLLBACK_TEST_OR_EVIDENCE=
PASS/FAIL

STAGING_TECHNICAL_COMPLETE=
YES/NO

BLOCKERS=

STAGING_COMPLETE=
YES/NO
```

---

# SUCCESS CONDITION

成功時：

```
STAGING_COMPLETE=YES
PRODUCTION_TOUCHED=NO
REAL_FB_ACTION=NO
VPS_BUILD_USED=NO
SOURCE_EVIDENCE_MATCH=PASS
IMMUTABLE_DIGEST_MATCH=PASS
HEALTH=PASS
CRITICAL_SMOKE=PASS
AUDIT=PASS
```

完成後停止。

不要自動進 Production。
