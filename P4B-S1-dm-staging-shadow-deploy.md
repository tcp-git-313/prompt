# P4B-S1 — DM Staging Shadow Deploy & Verification

ROLE: DM STAGING SHADOW DEPLOY OWNER

MODE: STAGING DEPLOY / NON-AUTHORITATIVE

REPO:
F:\00-Ticenpi-SaaS\TicenpiDM

GOAL:
Deploy the already-implemented DM Central Seat shadow to Staging and verify live shadow behavior without changing user-visible authorization.

PREREQUISITE:
P4B-R2 local result:
DM_SHADOW_IMPLEMENTED = YES
LEGACY_AUTHORITY_PRESERVED = YES
DIRTY_YUCT_WIP_PRESERVED = YES
READY_FOR_DM_STAGING_SHADOW = YES

IMPORTANT:
Run this only when no other product Staging deployment is active.

ABSOLUTE RULES:
- Central Seat remains NON-authoritative.
- Preserve all existing YUCT/OAuth dirty WIP semantics.
- Do NOT modify local-dev bypass semantics.
- Do NOT change platform_admin/test_access behavior.
- Do NOT change DM RLS.
- Do NOT touch Production.
- Do NOT modify Central Seat SQL/RPC.
- Do NOT run another product deployment in parallel.
- Do not commit/push unless explicitly instructed.

1. PREFLIGHT
Record branch, HEAD, git status --short, git diff --stat.
Confirm P4B-R2 intended files are present and YUCT/OAuth unrelated WIP is not accidentally included in deploy artifact.
Re-run:
- Seat shadow tests
- entitlement tests
- verify_auth_rls.py
- git diff --check

If the deploy artifact would include unrelated dirty WIP not intended for Staging, STOP.

2. RELEASE IDENTITY
Use the existing approved DM Staging deployment path.
Record:
- source identity
- image digest
- release identity
- runtime config identity
- health result

Do not touch Production.

3. LIVE SHADOW
Enable only the DM shadow capability required by the implementation.
Do not alter legacy authorization precedence.

Local/dev bypass is irrelevant in Staging and must remain unchanged in source.

4. LIVE STAGING MATRIX
Using existing Staging test identities, verify:
- platform_admin legacy allow unchanged
- test_access=allow unchanged
- test_access=deny unchanged
- ordinary legacy entitlement allow + central allow
- ordinary legacy entitlement allow + central deny if available
- legacy deny + central allow if safely available
- Central RPC error does not change legacy result

Central result must never become the final decision in this task.

5. OBSERVABILITY
Verify safe events:
MATCH_ALLOW
MATCH_DENY
MISMATCH_LEGACY_ALLOW_CENTRAL_DENY
MISMATCH_LEGACY_DENY_CENTRAL_ALLOW
CENTRAL_ERROR
CENTRAL_UNKNOWN

No bearer/JWT/email/secret/raw customer identity.

6. HEALTH / SMOKE
Run existing DM Staging health/auth/RLS smoke checks.
Confirm protected routes still use shared EntitledUserDep and DM RLS remains effective.

7. ROLLBACK READINESS
Record previous Staging release identity and exact rollback path.
Rollback only if verification fails.

8. FINAL REPORT

A. PREFLIGHT
B. RELEASE IDENTITY
C. LIVE SHADOW MATRIX
D. LOGGING SAFETY
E. HEALTH/AUTH/RLS
F. ROLLBACK READINESS
G. SAFETY

Must state:
CENTRAL_AUTHORITATIVE = NO
USER_VISIBLE_AUTH_BEHAVIOR_CHANGED = NO
PLATFORM_ADMIN_BEHAVIOR_CHANGED = NO
TEST_ACCESS_BEHAVIOR_CHANGED = NO
DM_RLS_CHANGED = NO
PRODUCTION_MUTATED = NO

H. FINAL RESULT

If successful:
RESULT = PASS
DM_STAGING_SHADOW_DEPLOYED = YES
DM_STAGING_SHADOW_VERIFIED = YES
LEGACY_AUTHORITY_PRESERVED = YES
READY_FOR_DM_CUTOVER_PLANNING = YES

If blocked:
RESULT = BLOCKED
DM_STAGING_SHADOW_DEPLOYED = NO/YES
DM_STAGING_SHADOW_VERIFIED = NO
BLOCKER = exact blocker

Stop after Staging shadow verification.