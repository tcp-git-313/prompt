# DM GO-LIVE ORCHESTRATOR OWNER

## Mission
你是 Ticenpi DM 商用上線總控 Owner。目標是把目前所有已知剩餘工作一次推進到「可安全部署 Production」為止。

不要再把工作拆成需要使用者逐條確認的小任務。能平行就平行，只有真正的硬依賴才等待。

本任務允許：
- 本機 source / tests / Docker 修改
- 獨立 branch/worktree
- Staging deploy / staging verification
- Browser MCP 真人 UAT（Browser MCP 已連線時）
- Hub source 修復
- Central deploy/release-gate source 修復
- SSOT 文件更新（只在技術狀態穩定後）

本任務禁止：
- 未經使用者額外明確批准的 Production deploy
- 修改 Production customer/subscription/entitlement data 來造測試結果
- 輸出 password / JWT / token / secret
- git reset --hard
- git clean

最終目標：
READY_FOR_PRODUCTION_DEPLOY = YES
或精確輸出唯一真實 blocker。

---

# 0. Current Known State

## Production
- URL: https://dm.ticenpi.com
- release: 20260919-110713
- commit: ef2e668
- Runtime = PASS
- Production 尚未部署 entitlement fix 86f2c97

## Staging
- URL: https://dm-staging.ticenpi.com
- release: 20260918-222526
- commit: 5e68034
- Supabase project ref: jlsqjvehwblkeuycjoyj

## Entitlement
- backend source fix = PASS
- commit = 86f2c97
- protected endpoints 已套用 entitlement gate
- tests = PASS
- Production 尚未部署
- Google OAuth final UAT 尚未完成

## Release Gates
- DM commit = 2325cca
- deploy commit = f8d4f0b
- auto release-blocking:
  - Capture Gate A = YES
  - Capture Gate B = YES
  - Auth invalid JWT = YES
  - Auth no entitlement = YES
  - Auth entitled = YES
- Capture Gate C hook = READY / credentials pending
- AI browser E2E hook = WAITING_FOR_AI_FIX
- Human UAT acceptance = DEFINED
- fail-closed = YES
- Production unchanged

## Local Runtime
- localhost:9422 = Local Dev, currently restored
- localhost:9421 = stale Local Docker image
- 9421 stale bundle does not match current source
- current root TicenpiDM/docker-compose.yml has drifted to Production profile
- DO NOT directly run docker compose up from that root until the local/production runtime contract is corrected
- required canonical flow:
  latest Source
  → latest Local Docker :9421
  → Docker verification
  → immutable artifact
  → Staging
  → UAT
  → Production

## Google OAuth Test Identities
ADMIN:
- tcp.a2026i@gmail.com
- Production platform_admin
- Staging currently user
- use for ADMIN/positive-path baseline only

ACCOUNT A:
- tcp.ai313@gmail.com
- general user
- use for ordinary-user positive path after entitlement is available in the candidate/staging test setup

ACCOUNT B:
- job0975805890@gmail.com
- general-user negative-path identity
- currently may not yet exist in auth.users
- intended to prove Google login does not automatically grant DM access

Known source behavior:
- handle_new_user() inserts only public.profiles
- no source-level auto membership/trial/subscription/entitlement provisioning
- DM access accepts entitlement status active OR trial

## AI
- /api/ai-draw exists
- invalid JWT returns 401
- CORS is not the proven root cause
- AGNES public endpoint reachable
- /api/health/detail does not currently check AGNES
- AI_FAILURE_REPRODUCED = NO
- AI_ROOT_CAUSE_PROVEN = NO
- Production user previously observed "Failed to fetch"

## Hub
Known stale state from audit:
- DM card exists
- launch URL pointed to Tailscale/local URL instead of https://dm.ticenpi.com
- release/status stale
- need distinguish development-workflow Hub vs customer-product Hub before changing entitlement wiring

---

# 1. First Action — Establish Workspace Truth

Before editing anything:

1. Inspect all relevant repos/worktrees/branches:
   - TicenpiDM
   - central deploy repo
   - Hub repo
2. Record:
   - branch
   - HEAD
   - clean/dirty
   - untracked files
3. Do not overwrite another Agent's work.
4. Reuse already completed commits where valid:
   - 86f2c97
   - 2325cca
   - f8d4f0b
5. If an expected commit cannot be found, HARD STOP and report exactly which one is missing.

Use separate integration branches/worktrees where needed.

---

# 2. Run Remaining Workstreams in Parallel

Start all independent workstreams concurrently when tooling allows.

## Stream A — Google OAuth Baseline

Prerequisite:
Browser MCP connected to a browser tab.

Use public staging:
https://dm-staging.ticenpi.com

Do not use localhost:9422 as staging.

Execute real Google OAuth for:
- tcp.a2026i@gmail.com
- tcp.ai313@gmail.com
- job0975805890@gmail.com

For each:
- complete OAuth
- confirm callback/session
- read-only inspect profile/admin source/membership/subscription/entitlement before/after
- do not print tokens

Confirm runtime auto-provisioning delta:
- membership
- trial
- subscription
- DM entitlement

If Browser MCP is unavailable:
do not replace this with docs.
Set:
OAUTH_BASELINE = BLOCKED_BY_BROWSER
and continue other parallel streams.

Do not treat current staging as final validation of 86f2c97 unless the candidate containing 86f2c97 has actually been deployed there.

## Stream B — AI Root Cause + Fix

Use real Production browser flow only for root-cause reproduction; no Production mutation.

Allowed test account:
tcp.a2026i@gmail.com

Collect same-timestamp evidence:
- Browser Network
- Browser Console
- active Production backend logs
- AGNES call evidence

Determine:
- request reached backend?
- AGNES call started?
- AGNES response received?
- actual HTTP status / exception / duration
- key present? value-blind only
- DNS/TLS/connectivity
- timeout chain

AI_ROOT_CAUSE_PROVEN = YES only with direct evidence.

Only after proof:
- apply smallest source/config fix in isolated branch/worktree
- base on a branch that includes 86f2c97
- add tests
- isolated/staging-equivalent verification
- no Production deploy

If AGNES health coverage is clearly missing, add a low-cost configured/reachable health check without consuming meaningful generation quota.

## Stream C — Local Docker Canonical Flow Repair

Goal:
restore the intended development/release chain.

Required end state:
- Local Dev remains separate
- Local Docker :9421 is rebuilt from current integrated source
- Local Docker uses local/disposable runtime contract
- Production runtime config is not accidentally used by local docker
- same application source/artifact can progress to staging/production with environment-specific runtime injection

Investigate why root TicenpiDM/docker-compose.yml became production-profile and identify the intended responsibility split among:
- root compose
- local/disposable profile
- generated/release compose
- central deploy
- staging runtime compose
- production runtime compose

Do not blindly revert.
Repair architecture with the smallest coherent change.

After repair:
- rebuild latest Local Docker on 9421
- verify UI/API/runtime identity
- verify source SHA / image identity
- ensure no Production Supabase/origin/targets are used locally

## Stream D — Hub Sync Fix

Confirm Hub type first:
- development workflow Hub
- customer product Hub
- other

If development-workflow Hub:
fix only launch/status/release truth.

Production DM launch URL must be:
https://dm.ticenpi.com

Do not add customer entitlement logic to the wrong Hub.

Add invariant/test preventing Production DM card from using:
- localhost
- 127.0.0.1
- Tailscale local URL
- staging URL

Prefer machine-readable current release/status source if already supported.
Do not create a large new architecture unless required.

## Stream E — Release Gates Integration Readiness

Reuse completed work:
- TicenpiDM 2325cca
- deploy f8d4f0b

Do not rewrite it from scratch.

Check compatibility with:
- entitlement 86f2c97
- AI fix
- Local Docker contract fix
- Hub changes

Capture Gate C and AI Browser E2E must remain non-PASS until real browser validation happens.

No hardcoded JWT secret/password/token may enter Git.

---

# 3. Integration Candidate

When Streams B/C/D/E have completed source work, create a single coherent integration candidate.

Integration requirements:
- contains entitlement fix 86f2c97
- contains release-gate work 2325cca / f8d4f0b or equivalent merged descendants
- contains proven AI fix if one was required
- contains Local Docker/release-contract correction
- contains Hub fix in its own repo as appropriate
- no unrelated changes
- clean tracked worktrees after commit
- preserve unrelated untracked incident/test artifacts without deleting them

Run:
- backend targeted/full tests as appropriate
- frontend tests/typecheck/build
- auth/entitlement tests
- release gates
- Docker build/config validation
- local docker :9421 validation
- secret scan on changed/tracked files

If new regression appears:
stop integration and identify owning commit/stream.

---

# 4. Build Immutable Artifact

From the integrated DM candidate:

1. Build Local Docker from the integrated source.
2. Verify:
   - :9421 UI
   - /api/health
   - runtime identity
   - expected local/disposable contract
3. Build/package immutable release artifact.
4. Record:
   - Git SHA
   - image digest
   - artifact identity
5. Ensure the artifact that will go to staging is the same application artifact that passed Local Docker validation.

No Production runtime secrets embedded into image.

---

# 5. Deploy Candidate to Staging

Staging deploy is allowed by this task.

Deploy the integrated immutable candidate to:
https://dm-staging.ticenpi.com

Requirements:
- staging environment/runtime contract
- staging Supabase project jlsqjvehwblkeuycjoyj
- candidate release identity visible
- health PASS
- central auto gates PASS

If staging deploy fails:
use rollback and stop with exact root cause.

---

# 6. Staging Browser UAT — Run in Parallel

Once the integrated candidate is live on staging, run all browser UATs concurrently when possible.

## UAT A — Google OAuth / Entitlement

Use:
ADMIN = tcp.a2026i@gmail.com
Account A = tcp.ai313@gmail.com
Account B = job0975805890@gmail.com

Important:
Staging admin role may differ from Production.
Do not silently mutate roles just to make tests pass.

Required truths:
- Google OAuth works
- new Google login does not auto-create unauthorized DM access
- ordinary non-entitled user can authenticate but protected DM API returns 403
- entitled ordinary user ALLOW test must use an existing safe staging entitlement fixture or isolated test identity; do not create Production data

If staging lacks a safe positive entitlement identity and creating one in staging is allowed by existing test-fixture procedures, use the established staging fixture process only. Do not invent ad-hoc Production-like data mutations.

## UAT B — Capture Gate C

Real browser:
- login
- DM editor
- capture fixture URL
- /api/scrape success
- result UI visible
- image loaded with naturalWidth/naturalHeight > 0
- no Failed to fetch
- no CORS error
- release identity matches candidate

## UAT C — AI Browser E2E

Real browser:
- login
- AI draw
- minimal prompt
- /api/ai-draw success
- returned/rendered image valid
- no Failed to fetch
- no blocking console error
- release identity matches candidate

## UAT D — Hub

Verify:
- launch URL
- release/status
- DM opens correct public endpoint
- no local/Tailscale/staging misrouting on Production card

## UAT E — Release Gate Behavior

Verify:
- successful gates pass
- at least one controlled forced-failure path fails closed
- pending Browser Gate is never reported PASS without evidence

---

# 7. Go-Live Gate

Only when all are true:

- ENTITLEMENT_BACKEND = PASS
- AUTH_NO_ENTITLEMENT = DENIED
- AUTH_ENTITLED = ALLOW
- GOOGLE_OAUTH = PASS
- CAPTURE_BROWSER_E2E = PASS
- AI_BROWSER_E2E = PASS
- HUB_SYNC = PASS
- LOCAL_DOCKER = PASS
- STAGING = PASS
- RELEASE_GATES = PASS
- NO_NEW_REGRESSIONS = YES
- SECRETS_IN_TRACKED_DIFF = NO

then:

READY_FOR_PRODUCTION_DEPLOY = YES

Otherwise:
READY_FOR_PRODUCTION_DEPLOY = NO

Do not deploy Production in this task.

---

# 8. SSOT Update

Only after technical state is stable and staging UAT is complete:

Update the appropriate DM SSOT:
- _charter/current-state.md
- _charter/history.md

Record:
- integrated candidate SHA
- staging release
- entitlement fix
- AI root cause/fix
- Local Docker contract correction
- Hub fix
- release-gate state
- Browser UAT results
- Commercial Go-Live state

Do not modify frozen F:\ProjectMD\TICENPI-ECOSYSTEM.md.

---

# 9. Final Output

Return one concise final block:

DM_GO_LIVE_ORCHESTRATION =
PASS / BLOCKED

INTEGRATED_DM_SHA =
...

LOCAL_DOCKER =
PASS / FAIL

LOCAL_DOCKER_IMAGE_DIGEST =
...

STAGING_RELEASE =
...

STAGING =
PASS / FAIL

GOOGLE_OAUTH =
PASS / FAIL / BLOCKED

ADMIN_OAUTH =
PASS / FAIL / NOT_APPLICABLE

ACCOUNT_A_OAUTH =
PASS / FAIL / BLOCKED

ACCOUNT_B_OAUTH =
PASS / FAIL / BLOCKED

AUTO_PROVISIONING =
SAFE / SECURITY_FINDING / UNPROVEN

AUTH_ENTITLED =
ALLOW / FAIL / NOT_TESTED

AUTH_NO_ENTITLEMENT =
DENIED / FAIL / NOT_TESTED

CAPTURE_BROWSER_E2E =
PASS / FAIL / BLOCKED

AI_ROOT_CAUSE_PROVEN =
YES / NO

AI_FIX_COMMIT =
...

AI_BROWSER_E2E =
PASS / FAIL / BLOCKED

HUB_SYNC =
PASS / FAIL

RELEASE_GATES =
PASS / FAIL

SSOT_UPDATED =
YES / NO

PRODUCTION_CHANGED =
NO

READY_FOR_PRODUCTION_DEPLOY =
YES / NO

BLOCKERS =
...

If READY_FOR_PRODUCTION_DEPLOY = YES:
Stop and wait for explicit Production deploy approval.
Do not deploy Production automatically.
