# P4A-S1R — Resume Post Staging Shadow Deploy from Evidence

ROLE: POST STAGING SHADOW DEPLOY OWNER

MODE: RESUME / VERIFY ACTUAL CANDIDATE / STAGING DEPLOY IF SAFE

REPO:
F:\00-Ticenpi-SaaS\TicenpiPost

CONTEXT:
Previous P4A-S1 ended with:

LIVE_STAGING_AUDIT = PASS
CURRENT_RELEASE = 20260922-141422
LOCAL_SHADOW_TESTS = PASS (75 passed)
P4A_READY_FLAG = NOT_VERIFIED
POST_STAGING_SHADOW_DEPLOYED = NO
POST_STAGING_SHADOW_VERIFIED = NO
RESULT = HARD_STOP

The stop was caused by inability to re-verify a textual readiness flag from a prior report.

DO NOT require the prior textual flag.

Instead, reconstruct readiness only from the actual current repository state, exact intended diff, tests, and deploy preflight.

GOAL:
If the actual P4A implementation is present, isolated, and passes the required local verification, treat readiness as proven by evidence and continue the Post Staging shadow deployment.

DO NOT touch Production.
DO NOT make Central Seat authoritative.
DO NOT change SEAT_POLICY.
DO NOT change AUTO_TENANT_BOOTSTRAP.
DO NOT change ALLOW_TENANT_KEY.
DO NOT modify Central Seat SQL/RPC.
DO NOT deploy any other product.
DO NOT reset/clean/stash/rebase.
DO NOT commit/push unless the existing approved deploy mechanism strictly requires a commit; if it does, STOP and report that requirement rather than creating one automatically.

## 1. PREFLIGHT

Record:
- branch
- HEAD
- git status --short
- git diff --stat
- git diff --check

Expected baseline branch:
feat/post-tenant-provisioning

Expected baseline HEAD before P4A:
3681a1c23dbaaa22d0e630b93a887f15c540354d

Do not require HEAD to change, because P4A was intentionally left uncommitted.

## 2. RECONSTRUCT P4A CANDIDATE FROM ACTUAL DIFF

Verify the intended P4A implementation is present in exactly these functional areas:

backend/app/core/identity.py
backend/app/api/auth.py
backend/app/api/studio.py
backend/tests/test_tenant_bootstrap.py

Required semantics:

A.
Central product_seat_status("post") is queried from the shared Post authorization path using the same Bearer token.

B.
Legacy Post authorization remains authoritative.

C.
Shadow result is observational only.

D.
Central DENY/ERROR/UNKNOWN cannot change existing allow/deny.

E.
No route-level duplicated Seat checks.

F.
Post tenant_for()/tenant_members mapping remains required and unchanged.

G.
X-Tenant-Key path remains unchanged.

H.
Shadow is controlled by:
TICENPI_CENTRAL_SEAT_SHADOW_ENABLED=1

I.
No bearer/JWT/email/customer PII/secret logging.

If actual semantics differ materially:
STOP.

## 3. DIRTY SCOPE GATE

Determine whether the worktree contains only the intended P4A modifications plus known harmless generated/untracked artifacts.

If there are unrelated source changes that would enter the deploy build:
STOP.

Do NOT stop merely because the worktree is dirty.

The decision must be based on whether unrelated code would enter the artifact, not on cleanliness alone.

Report:

P4A_DIFF_ISOLATED = YES/NO

## 4. LOCAL READINESS GATE

Re-run:

- targeted P4A shadow/auth tests
- relevant CommercialGate / SeatChecker / identity tests
- git diff --check

The previous run had:
75 passed

Current minimum requirement:
all targeted P4A tests PASS.

Existing unrelated full-suite invariant failures outside the P4A changed files do not block this task if they are already known and can be proven unrelated.

Set:

P4A_READY_BY_EVIDENCE = YES

only if:
- intended implementation present
- diff isolated
- targeted tests PASS
- no user-visible auth behavior change
- no config cutover included

Do not look for a text flag from another report.

## 5. LIVE STAGING BASELINE

Reconfirm before deployment:

CURRENT_RELEASE = current actual Post Staging release
COMMERCIAL_GATE
AUTO_BOOTSTRAP
SEAT_POLICY
ALLOW_TENANT_KEY

Expected previously observed values:

COMMERCIAL_GATE = 1
AUTO_BOOTSTRAP = 1
SEAT_POLICY = skip
ALLOW_TENANT_KEY = 0

If values changed since the previous audit:
STOP and report exact drift.

## 6. DEPLOY CANDIDATE IDENTITY

Use the existing approved Post Staging deployment process.

Important:
The deployed artifact must include the verified P4A diff and no unrelated source work.

If the current deploy pipeline cannot safely build/deploy an uncommitted isolated working-tree candidate:
STOP and report:

DEPLOY_REQUIRES_COMMITTED_CANDIDATE = YES

Do not create a commit automatically.

If the pipeline supports an isolated local candidate/artifact with provable source identity, proceed.

Record:
- source identity
- whether working-tree diff is included
- image digest
- release identity
- runtime config identity

## 7. ENABLE SHADOW ONLY

Enable:

TICENPI_CENTRAL_SEAT_SHADOW_ENABLED=1

Do not change:

TICENPI_SEAT_POLICY
TICENPI_AUTO_TENANT_BOOTSTRAP
TICENPI_ALLOW_TENANT_KEY
TICENPI_COMMERCIAL_GATE

After deployment, read the actual running container environment and prove all five effective values.

Required:

COMMERCIAL_GATE = 1
AUTO_BOOTSTRAP = 1
SEAT_POLICY = skip
ALLOW_TENANT_KEY = 0
CENTRAL_SEAT_SHADOW_ENABLED = 1

If any protected flag changes unexpectedly:
rollback and report failure.

## 8. LIVE SHADOW VERIFICATION

Using existing Staging test identities and current Staging Commercial Core data, verify the safest reachable cases:

- legacy allow + central allow
- legacy allow + central deny, if an existing test identity can produce it
- platform_admin legacy behavior unchanged
- Post tenant mapping remains required
- Central error does not change legacy result, only if safely simulatable without changing shared Central Seat or Staging config

Do not mutate Production.
Do not create unsafe permanent Staging state solely to force every mismatch case.

For cases not safely reachable, mark:
NOT_REACHED
not FAIL.

## 9. LOGGING SAFETY

Verify live logs:

Allowed:
- product=post
- legacy decision
- central decision
- normalized reason
- environment/release if already present

Forbidden:
- bearer
- JWT
- email
- service-role
- secrets
- raw customer identity

## 10. HEALTH / SMOKE

Run existing Post Staging health and critical smoke.

Confirm:
- /api/health
- authenticated request path
- session/bootstrap path
- tenant mapping
- no user-visible auth regression

## 11. ROLLBACK

Before deployment record previous release:

PREVIOUS_RELEASE =

If deploy or verification fails:
use the existing approved single-service rollback mechanism.

After rollback prove actual active release returned to the prior release.

Do not modify other services.

## 12. FINAL REPORT

### A. PREFLIGHT

Branch:
HEAD:
Dirty:
Diff isolated:
git diff --check:

### B. P4A READINESS RECONSTRUCTION

Implementation present:
Legacy authority preserved:
Shadow observational only:
Targeted tests:
P4A_READY_BY_EVIDENCE = YES/NO

### C. STAGING BASELINE

Previous active release:
COMMERCIAL_GATE:
AUTO_BOOTSTRAP:
SEAT_POLICY:
ALLOW_TENANT_KEY:

### D. DEPLOY IDENTITY

Source identity:
Image digest:
Release:
Runtime config:

### E. EFFECTIVE FLAGS AFTER DEPLOY

COMMERCIAL_GATE:
AUTO_BOOTSTRAP:
SEAT_POLICY:
ALLOW_TENANT_KEY:
CENTRAL_SEAT_SHADOW_ENABLED:

### F. LIVE SHADOW MATRIX

Case | Legacy | Central | Final result | Event | PASS/NOT_REACHED

### G. HEALTH / SMOKE

### H. LOGGING SAFETY

### I. ROLLBACK READINESS

### J. SAFETY

CENTRAL_AUTHORITATIVE = NO
USER_VISIBLE_AUTH_BEHAVIOR_CHANGED = NO
SEAT_POLICY_CHANGED = NO
AUTO_BOOTSTRAP_CHANGED = NO
TENANT_KEY_SETTING_CHANGED = NO
PRODUCTION_MUTATED = NO

### K. FINAL RESULT

If successful:

RESULT = PASS
P4A_READY_BY_EVIDENCE = YES
POST_STAGING_SHADOW_DEPLOYED = YES
POST_STAGING_SHADOW_VERIFIED = YES
LEGACY_AUTHORITY_PRESERVED = YES
READY_FOR_POST_CUTOVER_PLANNING = YES

If the only blocker is deploy mechanism requiring a commit:

RESULT = BLOCKED
P4A_READY_BY_EVIDENCE = YES
POST_STAGING_SHADOW_DEPLOYED = NO
DEPLOY_REQUIRES_COMMITTED_CANDIDATE = YES
BLOCKER = exact deploy requirement

If actual implementation/diff/test gate fails:

RESULT = HARD_STOP
P4A_READY_BY_EVIDENCE = NO
BLOCKER = exact factual blocker

Stop after Staging shadow verification.
