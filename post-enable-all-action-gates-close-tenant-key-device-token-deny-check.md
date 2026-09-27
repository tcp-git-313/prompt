# TICENPIPOST — ENABLE ALL USER ACTION GATES + CLOSE PRODUCTION X-TENANT-KEY + DEVICE-TOKEN DENY CHECK

## ROLE
TicenpiPost Production Safety / Enablement Writer

## RECOMMENDED MODEL
LUNA High

## MODE
SINGLE WRITER — FROZEN PLAN
DO NOT CHANGE SCOPE OR ARCHITECTURE

Project:
F:\00-Ticenpi-SaaS\TicenpiPost

Known runtime truth from latest audit:

Production:
- environment=production
- release=20260925-035353
- commit=27e4efedbd91
- digest=sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0
- Commercial Gate = ON
- Seat policy = require
- Central Seat authoritative by config
- TICENPI_ALLOW_TENANT_KEY = 1
- TICENPI_TENANT_KEYS = SET, one short legacy token
- all current Facebook write gates publish/comment/marketplace/relist/delete = OFF
- join_group is currently NOT covered by the gate system and therefore can still write

Staging:
- environment=staging
- release=20260924-111425
- commit=27e4efedbd91
- same image digest
- Commercial Gate = ON
- Seat policy = require
- TICENPI_ALLOW_TENANT_KEY = 0
- all current Facebook write gates = OFF
- join_group is not gated

User decisions are FROZEN:

1. Product features are meant to be usable.
2. Keep the gate mechanism only as an emergency kill switch.
3. Normal Staging and Production state after this task:
   - publish ON
   - comment ON
   - marketplace_publish ON
   - relist ON
   - delete ON
   - join_group ON
4. Production legacy X-Tenant-Key must be disabled:
   TICENPI_ALLOW_TENANT_KEY=0
5. Do NOT redesign X-Tenant-Key by merely making the key longer.
6. Existing formal browser/Extension/Studio flow should use Google/Supabase + Bind Token + Device Token, not X-Tenant-Key.
7. ADR-003 Device Token:
   - test the real revoked/denied condition in Staging only if safely possible;
   - if old Device Token is already DENIED after entitlement/seat removal, record PASS and do NOT spend effort changing source;
   - if old Device Token still authorizes the affected background route, HARD STOP and report evidence; do not silently redesign auth in this task.

Do not deviate from these decisions.

---

# 0. SAFETY / EXECUTION RULES

This is a WRITE task, but scope is narrow.

Allowed:
- inspect repo/source/tests/docs
- modify gate mapping/config/tests
- modify production/staging deployment manifests as required
- modify minimal code needed to bring join_group under the same gate framework
- update relevant documentation/ADR note only if needed to reflect proven runtime behavior
- commit/push through the existing release workflow
- deploy Staging first
- verify Staging
- then promote the exact same immutable application image/config change through the existing Production policy
- verify Production
- perform a targeted Device Token deny test in Staging using an existing safe test identity if possible

Forbidden:
- feature redesign
- LINE/LIFF work
- Marketplace driver redesign
- Central Seat schema changes
- database cleanup
- customer data cleanup
- production Facebook test posts/comments/group joins
- production destructive E2E
- changing Customer/Seat/Tenant architecture
- changing Bind Token or Device Token format unless a proven blocker appears
- creating new auth bypasses
- force-push
- reset/restore unrelated dirty work
- modifying unrelated WIP
- rebuilding a different app image for Production than the one validated in Staging

If frozen plan conflicts with actual code/runtime:
HARD STOP with exact evidence.

---

# 1. PRE-FLIGHT — PROVE X-TENANT-KEY IS NOT REQUIRED BY CURRENT FORMAL CLIENTS

Before changing Production:

Search current source and historical release source for all use of:

- X-Tenant-Key
- TICENPI_ALLOW_TENANT_KEY
- TICENPI_TENANT_KEYS
- X-Bind-Token
- X-Device-Token
- bind-request
- import-cookies

At minimum inspect:

- frontend
- extension
- desktop / Launcher handoff code inside this repo
- backend binding/auth routes
- current deployed source commit 27e4efe
- relevant tests

Prove the current formal paths:

### Web/Google
Google/Supabase JWT
→ commercial auth
→ tenant resolution

### Initial Facebook binding
authenticated user
→ POST /api/auth/bind-request
→ X-Bind-Token
→ /api/auth/import-cookies

### Existing device/background path
X-Device-Token
→ existing intended device route(s)

Output:

FORMAL_WEB_REQUIRES_X_TENANT_KEY = YES/NO
FORMAL_EXTENSION_BIND_REQUIRES_X_TENANT_KEY = YES/NO
FORMAL_STUDIO_BIND_REQUIRES_X_TENANT_KEY = YES/NO
BACKGROUND_DEVICE_FLOW_REQUIRES_X_TENANT_KEY = YES/NO

Required result before continuing:

all = NO

If any = YES:
HARD STOP.
Do not disable Production X-Tenant-Key until the dependency is understood.

Also search tests for legacy behavior so expected breakage is known.

---

# 2. PRESERVE LEGACY KEY DATA BUT DISABLE THE AUTH PATH

The user explicitly wants:

TICENPI_ALLOW_TENANT_KEY=0

Do NOT spend time rotating or lengthening the old key.

Minimal target:

Production running container:
TICENPI_ALLOW_TENANT_KEY=0

Staging must remain:
TICENPI_ALLOW_TENANT_KEY=0

It is acceptable for TICENPI_TENANT_KEYS secret material to remain present temporarily if unused.
Do not print it.
Do not rotate it in this task.

Verify code behavior proves:

ALLOW_TENANT_KEY=0
→ X-Tenant-Key path cannot authenticate
→ normal Google/Supabase path remains unaffected
→ Bind Token / Device Token routes remain their intended separate mechanisms

Add/update tests so:

- X-Tenant-Key rejected when allow flag=0
- Google/JWT path still works
- bind token path still works
- device token path still works according to current contract

No fabricated Production auth attempts.

---

# 3. DEFINE GATE POLICY — KEEP KILL SWITCH, NORMAL STATE ON

Do NOT remove the gate infrastructure.

The final policy is:

Gate = emergency kill switch
Normal service = ON

Required effective state in both Staging and Production:

TICENPI_AUTO_PUBLISH=1
TICENPI_AUTO_PUBLISH_POST=1
TICENPI_AUTO_PUBLISH_COMMENT=1
TICENPI_AUTO_PUBLISH_MARKETPLACE=1
TICENPI_AUTO_PUBLISH_RELIST=1
TICENPI_AUTO_PUBLISH_DELETE=1

join_group must also be covered by the same fail-closed action-gate framework and enabled normally.

Inspect the current gate implementation first:
backend/app/core/publish_gate.py
and all call sites.

Use the existing architecture.
Do not invent a second gate system.

---

# 4. ADD JOIN_GROUP TO THE EXISTING ACTION GATE SYSTEM

Current problem:

join_group bypasses the existing master/action gate and directly calls the driver.

Fix this minimally.

Requirements:

1. join_group must obey the same master gate.
2. join_group must have an action-specific gate.
3. use the current ACTION_GATE_ENV / current convention where possible.
4. preferred env name if consistent with current architecture:
   TICENPI_AUTO_PUBLISH_JOIN_GROUP
   but inspect existing conventions before finalizing.
5. If a better already-existing naming convention is present, use it and document why.
6. Do not refactor unrelated worker/task code.
7. Gate OFF must block before any Facebook click.
8. Gate ON allows the existing join flow unchanged.

Tests required:

master OFF + join action ON → blocked
master ON + join action OFF → blocked
master ON + join action ON → allowed to reach mocked driver
no actual Facebook write in automated tests

Also ensure existing publish/comment/marketplace/relist/delete tests remain valid.

---

# 5. CONFIGURE NORMAL RUNTIME = ALL USER ACTIONS ON

Update the correct deployment sources so the intended effective runtime becomes ON for BOTH:

Staging
Production

Required:

MASTER=1

POST=1
COMMENT=1
MARKETPLACE=1
RELIST=1
DELETE=1
JOIN_GROUP=1

Do not enable by changing code defaults globally unless the existing deployment policy requires it.

Prefer explicit environment declaration in deploy manifests so runtime intent is auditable.

Do not rely on an untracked VPS .env value when the production manifest can explicitly declare the intended non-secret boolean.

Do not touch API keys/secrets.

---

# 6. IMPORTANT — ENABLING GATES DOES NOT MEAN RUNNING REAL FACEBOOK E2E HERE

This task turns the product features ON.

It does NOT perform:

- a real Marketplace public listing
- a real comment
- a real delete
- a real relist
- a real group join
- a real production Facebook post

Those are separate controlled E2E tasks.

After deployment, verify configuration and request routing only.

Do not create public Facebook side effects in this task.

---

# 7. STAGING-FIRST VALIDATION

Before Production promotion:

Deploy according to the existing Post Staging release workflow.

Use the correct current release branch / canonical release process.
Do not assume the dirty local HEAD is canonical.

Required Staging checks:

## Identity
- /api/health = 200
- environment=staging
- expected release/commit/digest
- immutable image identity known

## Auth
- Commercial gate remains ON
- Seat policy remains require
- ALLOW_TENANT_KEY=0
- normal Supabase/Google auth smoke passes using existing safe test method
- legacy X-Tenant-Key unit/integration contract denies when flag=0
- Bind Token contract passes
- Device Token contract passes

## Gates
running worker/container effective values:

MASTER=1
POST=1
COMMENT=1
MARKETPLACE=1
RELIST=1
DELETE=1
JOIN_GROUP=1

## No real Facebook write
Only mocked/contract/smoke validation.

Do not call production or staging endpoints that cause real Facebook mutation merely to prove gate ON.

---

# 8. DEVICE TOKEN DENY CHECK — VERIFY BEFORE FIXING

This is a focused Staging-only verification.

Goal:
determine whether ADR-003 is an actual remaining runtime authorization gap.

Use an existing NON-PRODUCTION test identity/device token if one safely exists.

Do not create a new auth design.

Preferred test shape:

A.
User has valid commercial/seat access
→ existing Device Token is valid
→ establish the affected background/device route currently accepts the token

B.
Remove/revoke the user's entitlement/seat using the established Staging test fixture/process ONLY if this can be done safely and reversibly.

C.
Without re-login and without issuing a new Device Token, retry the exact affected route with the OLD Device Token.

Expected secure result:

401 / 403 / explicit authorization denial

If DENIED:

DEVICE_TOKEN_REVOKE_RESULT = PASS
ADR003_RUNTIME_GAP_REPRODUCED = NO

Then:
- record evidence
- do NOT modify device-token authorization source
- restore the test fixture through the normal existing Staging process if needed
- leave ADR-003 with a note/evidence only if documentation update is appropriate

If STILL AUTHORIZED:

DEVICE_TOKEN_REVOKE_RESULT = FAIL
ADR003_RUNTIME_GAP_REPRODUCED = YES

Then:
HARD STOP.
Do not auto-fix ADR-003 in this task.
Report exact route, status, and sanitized evidence.

If a safe Staging fixture/token does not exist:

DEVICE_TOKEN_REVOKE_RESULT = UNVERIFIED

Do not manufacture a risky test.
Continue the gate / tenant-key task.

---

# 9. PROMOTE TO PRODUCTION

Only after Staging gates/auth checks pass.

Production must receive the same tested application source/image according to existing immutable promotion policy.

If code changed for join_group gating:
build once through official CI,
validate that image in Staging,
then promote the exact same digest to Production.

Do not rebuild a different Production image.

If only deployment config changed after an existing immutable image:
follow the existing config-only release policy if documented.
Do not invent one.

Production target:

TICENPI_ALLOW_TENANT_KEY=0

Commercial Gate unchanged ON
Seat policy unchanged require

MASTER=1
POST=1
COMMENT=1
MARKETPLACE=1
RELIST=1
DELETE=1
JOIN_GROUP=1

No real Facebook E2E on Production in this task.

---

# 10. PRODUCTION POST-DEPLOY VERIFICATION

Read-only/runtime validation:

- /api/health 200
- environment=production
- correct release
- correct commit
- expected image digest
- Commercial gate remains ON
- Seat policy remains require
- ALLOW_TENANT_KEY=0
- all action gates effective ON
- join_group gate present/effective ON
- no config drift introduced by accidental stale manifest
- Extension/Studio binding contracts still intact via tests/known smoke
- no secrets printed

Use docker inspect/read-only env validation as needed.

Do not attempt a guessed X-Tenant-Key.
No brute force.

---

# 11. REQUIRED TESTS

Run focused tests first, then existing relevant gate/auth suites.

Minimum new/updated coverage:

## Tenant key
- allow=0 rejects legacy tenant key
- normal JWT path unaffected

## Binding
- bind token path unaffected
- device token path unaffected according to current contract

## Action gates
For each:
publish
comment
marketplace_publish
relist
delete
join_group

prove:

master off → blocked
master on + action off → blocked
master on + action on → dispatch allowed to mock/stub

Do not add browser/Facebook live writes to unit suite.

Run any existing release gates required by Post CI before promotion.

---

# 12. SOURCE / BRANCH HYGIENE

Current local worktree was previously:

release/post-central-seat-shadow-staging-20260923
HEAD 7c90916
dirty with unrelated WIP.

Do not overwrite those unrelated changes.

Use a clean isolated worktree/branch based on the actual current canonical/release ancestor.

Before editing:

identify current canonical source and production source.

Do not build from a dirty working directory.

Do not merge unrelated Central Seat shadow WIP.

---

# 13. DO NOT CHANGE THESE THINGS

Out of scope:

- Central Seat schema/RPC redesign
- Seat fixture redesign
- LINE/LIFF
- PWA
- Marketplace unpublish
- Marketplace address resolver
- group search
- AI group classification
- hidden route cleanup
- orphan worker cleanup
- config_drift mount change
- canonical branch consolidation beyond what is strictly necessary to safely branch/release this task

These remain later tasks.

---

# 14. SUCCESS CRITERIA

PASS only when:

1. Current Extension/Studio formal paths are proven not to require X-Tenant-Key.
2. Production ALLOW_TENANT_KEY=0.
3. Staging ALLOW_TENANT_KEY remains 0.
4. Existing legacy key cannot authenticate when flag=0 by test/contract.
5. Commercial Gate still ON.
6. Seat policy still require.
7. Master Facebook action gate = ON in Staging/Production.
8. post/comment/marketplace/relist/delete = ON in Staging/Production.
9. join_group is now covered by the same gate architecture.
10. join_group gate = ON in Staging/Production.
11. No real Facebook side effect was generated during this task.
12. Staging validated before Production.
13. Production uses the exact Staging-validated immutable app image if code was rebuilt.
14. Device Token revoke test result is PASS, FAIL, or UNVERIFIED with evidence.
15. If Device Token test PASS: no unnecessary ADR-003 code fix was made.
16. No unrelated WIP was modified.

---

# 15. FINAL REPORT

PHASE = POST ACTION GATES + TENANT KEY CLOSURE
RESULT = PASS / PARTIAL / HARD_STOP

A. SOURCE
BASE_BRANCH =
BASE_COMMIT =
WORKTREE_ISOLATION =
CHANGED_FILES =
COMMIT =
PUSH =

B. X-TENANT-KEY DEPENDENCY
WEB_REQUIRES_X_TENANT_KEY =
EXTENSION_BIND_REQUIRES_X_TENANT_KEY =
STUDIO_BIND_REQUIRES_X_TENANT_KEY =
DEVICE_FLOW_REQUIRES_X_TENANT_KEY =

C. TENANT
STAGING_ALLOW_TENANT_KEY =
PRODUCTION_ALLOW_TENANT_KEY =
LEGACY_KEY_REJECT_TEST =
GOOGLE_JWT_PATH =
BIND_TOKEN_PATH =
DEVICE_TOKEN_PATH =

D. ACTION GATES
MASTER =
POST =
COMMENT =
MARKETPLACE =
RELIST =
DELETE =
JOIN_GROUP =
JOIN_GROUP_GATE_ENV_NAME =

E. STAGING
STAGING_RELEASE =
STAGING_COMMIT =
STAGING_DIGEST =
STAGING_HEALTH =
STAGING_GATE_RUNTIME =
STAGING_AUTH_SMOKE =

F. DEVICE TOKEN
DEVICE_TOKEN_REVOKE_RESULT = PASS / FAIL / UNVERIFIED
ADR003_RUNTIME_GAP_REPRODUCED = YES / NO / UNVERIFIED
DEVICE_TOKEN_SOURCE_CHANGED = YES/NO
NOTE =

G. PRODUCTION
PRODUCTION_RELEASE =
PRODUCTION_COMMIT =
PRODUCTION_DIGEST =
PRODUCTION_HEALTH =
PRODUCTION_GATE_RUNTIME =
PRODUCTION_ALLOW_TENANT_KEY =

H. FACEBOOK SIDE EFFECT
REAL_POST_CREATED = NO
REAL_COMMENT_CREATED = NO
REAL_MARKETPLACE_CREATED = NO
REAL_RELIST_CREATED = NO
REAL_DELETE_EXECUTED = NO
REAL_GROUP_JOIN_EXECUTED = NO

I. TESTS
FOCUSED_TESTS =
CI =
STAGING_GATE =
PRODUCTION_POST_DEPLOY_GATE =

J. FINAL
ALL_USER_ACTIONS_ENABLED = YES/NO
TENANT_KEY_FALLBACK_CLOSED = YES/NO
DEVICE_TOKEN_NEEDS_FOLLOWUP = YES/NO
READY_FOR_CONTROLLED_FACEBOOK_E2E = YES/NO

If PASS:
do not continue into Marketplace/automation E2E automatically.
Stop and report.
