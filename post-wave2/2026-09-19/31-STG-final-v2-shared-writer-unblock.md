# STG-FINAL-v2 — Stabilize Shared Writer and Complete POST Staging

- 抬頭：POST STAGING COMPLETION + SHARED WRITER RECOVERY OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：大型
- 目標：一次執行到 `STAGING_COMPLETE=YES`
- Production：嚴禁觸碰

---

你現在是：

POST STAGING COMPLETION + SHARED WRITER RECOVERY OWNER

上一輪 STG-FINAL 唯一 HARD STOP：

Central Deploy main 有 DM/ORC 未提交 shared-file work，沒有完整 committed integration point。

本任務被授權：

1. 先安全判斷這批 dirty work 是否已停止變動
2. 若已停止，將「原樣 dirty state」封存成 dedicated stabilization commit
3. 不改寫 DM/ORC 語意
4. 以該 committed point 作為 Central Deploy integration base
5. 然後一路完成 Post final candidate → CI/GHCR → Central Deploy bridge → VPS control-plane → immutable Staging → E2E
6. 直到 `STAGING_COMPLETE=YES`

除 HARD STOP 外，不要每一階段回來等待確認。

---

# 0. CURRENT AUTHORITATIVE STATE

## POST

Worktree：

`F:\00-Ticenpi-SaaS\TicenpiPost-wave2-integration-v2`

Branch：

`repair/post-v2-wave2-integration-v2`

Expected HEAD：

`53fce622caae69a6b0fa476483347b17af7dc4fb`

Status：

- W2INT-v2 complete
- W2F-FIX2 complete
- backend 642 pass / 4 pre-existing / 0 new
- frontend PASS
- typecheck PASS
- build PASS
- publish safety PASS
- runtime config PASS
- health/smoke PASS
- local Docker PASS
- worktree clean

W2G CI evidence fix尚未整合：

`c2a0086`

唯一預期檔：

`.github/workflows/ci.yml`

## CENTRAL DEPLOY FROZEN W2C

Clean isolated worktree：

`F:\00-Ticenpi-SaaS\deploy-w2-post`

Branch：

`repair/post-immutable-deploy`

Frozen lineage：

1. `429898662fb487eb4bafe222dd11e60453a8a204`
2. `808fe81e082602c50a4c3770487b1d67b9755612`
3. `39ecc94fdaed5cd2950b277cc15d1b39da28e6c1`

Validated：

- immutable exact-digest deploy
- G6 evidence contract
- GHCR auth contract
- post/postruntimestaging evidence service mapping
- clean detached source snapshot bound to evidence.git_sha
- mutable worktree not used in immutable mode
- legacy source mode preserved

## CENTRAL DEPLOY MAIN DIRTY STATE FROM LAST STOP

Repo：

`F:\00-Ticenpi-SaaS\deploy`

Branch：

`main`

Last known HEAD：

`216f709cde8ce0d95b60f120997e13c650029a1a`

Last known dirty shared files：

- server/deploy.sh
- server/audit.sh
- server/critical-smoke.sh
- services.yaml

Known untracked DM files：

- DM_RELEASE_GATES.md
- server/warp-proxy-relay-dmruntimestaging.service

Known content：

- dmruntimestaging
- DM gate profiles
- ORC port 9422→9423
- DM/ORC shared deploy framework work

Sibling branch：

`dm/release-gates-wave1` @ `f8d4f0b`

was proven incomplete relative to dirty worktree.

Do NOT use f8d4f0b as base unless new evidence proves worktree state has since been fully committed elsewhere.

## VPS

Host：

`147.224.255.234`

Target service：

`postruntimestaging`

Port：

`19419`

Compose project：

`ticenpi-post-runtime-staging`

Known ready：

- VPS reachable
- aarch64
- Docker + compose available
- CI target linux/arm64 matches
- port 19419 belongs to intended Post staging stack
- current release/symlink valid
- current stack is legacy-source

Known not ready：

- VPS deployer is old legacy control-plane
- immutable_image.py not deployed
- GHCR credential transport to remote process not yet proven

---

# 1. GLOBAL SAFETY

## NEVER PRODUCTION

Do not deploy/restart/mutate:

`post`

Only:

`postruntimestaging`

Production Supabase / DB / traffic / publish gates untouched.

## NO REAL FB WRITE

No real:

- post
- comment
- delete
- relist
- marketplace publish

## SECRET RULE

Never print:

- token value
- username value if treated secret
- password
- Supabase key
- JWT
- cookies

No length/prefix/hash.

Only PRESENT=YES/NO.

## NO DESTRUCTIVE GIT

Never:

- reset --hard
- clean
- stash other work
- overwrite dirty work
- force push

---

# 2. PHASE Z — STABILIZE CENTRAL DEPLOY SHARED WRITER

This is the new unblock phase.

## Z1. Re-inspect current state

Read:

`F:\00-Ticenpi-SaaS\deploy`

Record:

- branch
- HEAD
- status
- exact dirty files
- diffs of:
  - server/deploy.sh
  - server/audit.sh
  - server/critical-smoke.sh
  - services.yaml
  - DM_RELEASE_GATES.md
  - server/warp-proxy-relay-dmruntimestaging.service

Also inspect:

- git index lock
- merge/rebase/cherry-pick state
- running git processes
- file mtimes

## Z2. Active-writer stability check

Take hash snapshot of all dirty/shared files.

Wait at least 60 seconds.

Take second hash snapshot.

Proceed ONLY if:

- both snapshots identical
- no git operation in progress
- no index.lock
- no relevant file modified during observation
- last write is not actively changing
- no evidence another agent/process is writing them

If files change during observation:

HARD STOP:

`SHARED_WRITER_STILL_ACTIVE=YES`

Do not commit.

## Z3. Create recovery/stabilization branch

If stable, you are authorized to capture the exact dirty state.

Do NOT edit file contents first.

From current committed HEAD, create/switch to dedicated branch while preserving worktree changes:

`integration/dm-orc-shared-stabilize-20260919`

Do not use stash.

Stage ONLY the known DM/ORC shared-state files after re-validating each is related:

- server/deploy.sh
- server/audit.sh
- server/critical-smoke.sh
- services.yaml
- DM_RELEASE_GATES.md
- server/warp-proxy-relay-dmruntimestaging.service

If additional dirty files exist:

classify.

If clearly same DM/ORC task, include and report.

If unrelated/unknown:

leave unstaged and HARD STOP before commit unless it can remain outside branch safely.

## Z4. Checkpoint exact dirty state

Commit exact captured state without semantic edits:

`chore(deploy): checkpoint dm orc shared deploy state`

Record:

`CENTRAL_STABILIZATION_COMMIT=`

This checkpoint may be intermediate; it exists to guarantee no work loss.

## Z5. Validate checkpoint before using as staging base

Without changing product semantics, run:

- bash -n server/deploy.sh
- bash -n server/audit.sh
- bash -n server/critical-smoke.sh
- YAML parse services.yaml
- manifest validation fixtures
- profile whitelist inspection
- service registry inspection
- any existing Central Deploy unit tests relevant to DM/ORC/shared framework
- git diff --check

If checkpoint has syntax/schema/test failures that indicate incomplete DM/ORC work:

DO NOT silently fix their product behavior.

HARD STOP with exact failures.

If checkpoint is internally consistent enough to serve as integration base:

`CENTRAL_STABILIZED=YES`

Use this commit as new Central committed base.

Do not merge it into main.

---

# 3. PHASE A — FINALIZE POST CANDIDATE

## A1. Verify Post state

Expected:

branch:
`repair/post-v2-wave2-integration-v2`

HEAD:
`53fce622caae69a6b0fa476483347b17af7dc4fb`

clean.

If already contains c2a0086 patch-id and is otherwise expected, accept and continue.

## A2. Merge W2G

Verify:

`git show --name-only c2a0086`

must only touch:

`.github/workflows/ci.yml`

Cherry-pick:

`c2a0086`

Conflict → HARD STOP.

Validate:

- YAML parse
- package extension metadata path
- `--extension-metadata` wiring
- local evidence generation
- local evidence validation
- `npx tsc --noEmit`
- git diff --check
- clean

Set:

`POST_FINAL_HEAD=<HEAD>`

## A3. Push Post candidate

Push only:

`repair/post-v2-wave2-integration-v2`

No main merge.

---

# 4. PHASE B — REAL CI / GHCR / EVIDENCE

Wait for actual CI from POST_FINAL_HEAD.

Must pass required jobs.

Retrieve authoritative:

- post-release JSON
- exact `image@sha256`
- CI run id/url
- extension metadata

Validate:

- evidence.git_sha == POST_FINAL_HEAD
- source_state == clean
- service == post
- platform == linux/arm64
- artifact_digest == immutable_reference digest
- exact `@sha256:<64hex>`

No fabricated digest.

Set:

`POST_IMAGE_DIGEST=`
`POST_IMMUTABLE_REF=`

---

# 5. PHASE C — BUILD FRESH CENTRAL POST-STAGING INTEGRATION

Use:

`CENTRAL_STABILIZATION_COMMIT`

as the base.

Create fresh worktree:

`F:\00-Ticenpi-SaaS\deploy-post-staging-final`

Branch:

`repair/post-staging-final`

Do not work directly in original checkout.

Replay frozen W2C lineage in order:

1. 429898662fb487eb4bafe222dd11e60453a8a204
2. 808fe81e082602c50a4c3770487b1d67b9755612
3. 39ecc94fdaed5cd2950b277cc15d1b39da28e6c1

If conflict:

do not blindly ours/theirs.

First determine whether conflict is mechanical due DM/ORC evolved shared code or semantic.

If any frozen W2C commit touches the same shared semantics in a way that cannot be uniquely preserved:

HARD STOP.

Non-overlap helper/test conflicts may be resolved only if patch intent is unambiguous and all tests prove equivalence.

Record:

`CENTRAL_W2C_HEAD=`

---

# 6. PHASE D — GHCR SECURE CREDENTIAL TRANSPORT

Implement secure transport on Central integration branch.

Goal:

`F:\Secrets\ticenpi.env`
→ deploy.ps1
→ secure stdin / ephemeral remote process env
→ remote immutable deploy process
→ docker login --password-stdin

Requirements:

- GHCR_READ_TOKEN required
- GHCR_USERNAME optional per existing contract
- token not argv
- token not remote command string
- token not logs
- token not evidence
- token not manifest
- token not persistent VPS file
- no sshd AcceptEnv requirement
- no sudoers env_keep requirement
- missing token fails before SSH/deploy mutation

Prefer deploy.ps1/helper only.

If current final shared framework requires minimal server/deploy.sh support, now you may modify it because DM/ORC state has been stabilized into committed base. Preserve all DM/ORC behavior.

Run fixtures; no real auth yet.

Commit:

`fix(deploy): securely transport ghcr credentials`

---

# 7. PHASE E — FINAL POST SHARED-FILE BRIDGE

On Central integration branch, preserve all stabilized DM/ORC changes.

## services.yaml

Register Post smoke profile for:

- post
- postruntimestaging

Expected profile:

`post-core`

unless current repo canonical naming dictates another exact name.

## server/critical-smoke.sh

Add Post adapter preserving DM/ORC profiles.

For runtime staging map:

- base URL = http://127.0.0.1:19419
- compose project = ticenpi-post-runtime-staging

Use approved Post W2E smoke implementation.

Do not invent a second smoke implementation.

## server/deploy.sh / server/audit.sh

Make valid smoke-profile contracts consistent.

Preserve every existing committed DM/ORC/591/Sign profile.

Unknown profile fail closed.

Only change required_services if existing architecture clearly requires it.

## validation

Run all relevant:

- bash -n
- YAML parse
- PowerShell parse
- W2C unit tests
- immutable tests
- manifest fixtures
- current DM/ORC smoke/profile tests
- Post profile tests
- legacy mode
- immutable dry-run
- rollback compatibility
- secret scan
- git diff --check

Commit:

`feat(deploy): register post immutable staging delivery`

Set:

`CENTRAL_FINAL_HEAD=`

Push:

`repair/post-staging-final`

No main merge.

---

# 8. PHASE F — CONTROL-PLANE VPS ROLLOUT

Deploy control-plane only.

Do not deploy product yet.

First inspect current /opt/ticenpi/deploy install/update mechanism.

Use safest available:

1. versioned/side-by-side candidate
2. validation
3. atomic cutover
4. rollback-ready previous deployer

If no versioned mechanism, create timestamped backup + candidate + atomic replacement.

Install required control-plane files:

- deploy.sh
- audit.sh
- critical-smoke.sh
- immutable_image.py
- services registry/helper as required

Validate on VPS without product cutover:

- bash -n
- helper validation
- legacy dry-run
- immutable dry-run
- missing GHCR credential fail closed
- postruntimestaging→post evidence mapping
- exact digest parsing
- no product container ID/uplink changed

Failure → rollback control-plane + HARD STOP.

---

# 9. PHASE G — REAL GHCR AUTH PRECHECK

Presence-only:

`GHCR_READ_TOKEN_PRESENT=YES/NO`

Use secure transport.

Perform:

`docker login ghcr.io --password-stdin`

Do not print token.

Failure → HARD STOP.

---

# 10. PHASE H — IMMUTABLE POST RUNTIME-STAGING DEPLOY

Deploy ONLY:

`postruntimestaging`

Inputs:

- POST_FINAL_HEAD
- authoritative post-release JSON
- POST_IMMUTABLE_REF

Before mutation verify:

- target service=postruntimestaging
- evidence service=post
- mapping valid
- evidence.git_sha=POST_FINAL_HEAD
- source snapshot SHA=POST_FINAL_HEAD
- snapshot clean
- exact digest match
- arm64
- immutable mode
- VPS build disabled

Execute through official Central Deploy entry point.

Required path:

- validate manifest
- validate evidence
- GHCR auth
- preflight
- release snapshot
- render compose removing build
- pin api/worker exact image@sha256
- docker pull exact digest
- `up -d --no-build`
- health
- critical smoke
- commit manifest
- audit

Failure → automatic rollback to previous staging release, verify old health, HARD STOP.

---

# 11. PHASE I — STAGING SERIAL HEALTH GATE

Must PASS before E2E:

- running api/worker image digest == POST_IMAGE_DIGEST
- release git SHA == POST_FINAL_HEAD
- no VPS build
- current symlink points new release
- /api/health 200
- container readiness
- Redis
- Postgres
- worker
- config drift
- post-core critical smoke
- Central audit

Any fail → rollback + HARD STOP.

---

# 12. PHASE J — STAGING E2E

After health gate, run parallel where safe.

## J1 Auth / Membership / Entitlement

- identity/JWT
- membership
- entitled
- not_entitled
- no_tenant
- tenant isolation
- X-Tenant-Key disabled production-like behavior

## J2 Studio State

- online
- studio_offline
- account_mismatch
- real /api/session/state
- Web remains usable offline/mismatch

## J3 Mobile/Web

If public staging route exists:

- responsive/mobile
- Google OAuth callback
- shell/navigation

If no route:

do not fabricate success.

Use only safe existing mechanism.

## J4 Extension / Bind Contract

No real FB write.

Verify:

- extension artifact metadata matches POST_FINAL_HEAD
- first-bind contract
- DOM ready/race regression
- recurring device-token contract
- backend auth contract

## J5 Publish Safety

Staging:

- master OFF = no write
- master ON + actions OFF = no write
- action isolation
- action ON + master OFF = no write
- unknown action fail closed
- dry-run executor path

## J6 Marketplace / Relist

- payload
- queue
- dry-run
- safety gates
- no real write

---

# 13. SUCCESS RULE

Only declare:

`STAGING_COMPLETE=YES`

when mandatory staging acceptance truly passes.

Never lower standards merely to obtain YES.

Production remains untouched.

No automatic Production promotion.

---

# 14. FINAL OUTPUT

```
TASK=POST_STAGING_COMPLETION_V2

SHARED_WRITER_STILL_ACTIVE=
YES/NO

CENTRAL_STABILIZATION_BRANCH=
CENTRAL_STABILIZATION_COMMIT=

CENTRAL_STABILIZED=
YES/NO

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
CENTRAL_W2C_HEAD=
CENTRAL_FINAL_HEAD=

W2C_IMMUTABLE_LINEAGE=
PASS/FAIL

GHCR_CREDENTIAL_TRANSPORT=
PASS/FAIL

TOKEN_EXPOSED=
NO

SHARED_FILE_BRIDGE=
PASS/FAIL

CENTRAL_REMOTE_PUSH=
PASS/FAIL

VPS_CONTROL_PLANE_ROLLOUT=
PASS/FAIL

VPS_CONTROL_PLANE_ROLLBACK_READY=
YES/NO

GHCR_REAL_AUTH=
PASS/FAIL

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

BLOCKERS=

STAGING_TECHNICAL_COMPLETE=
YES/NO

STAGING_COMPLETE=
YES/NO
```

成功時最少必須：

```
SHARED_WRITER_STILL_ACTIVE=NO
CENTRAL_STABILIZED=YES
W2G_MERGED=YES
CI_STATUS=PASS
RELEASE_EVIDENCE=PASS
SOURCE_EVIDENCE_MATCH=PASS
IMMUTABLE_DIGEST_MATCH=PASS
VPS_BUILD_USED=NO
HEALTH=PASS
CRITICAL_SMOKE=PASS
AUDIT=PASS
REAL_FB_ACTION=NO
PRODUCTION_TOUCHED=NO
STAGING_COMPLETE=YES
```

完成後停止，不進 Production。
