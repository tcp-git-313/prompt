# P4A-S1 — Post Staging Shadow Deploy & Verification

ROLE: POST STAGING SHADOW DEPLOY OWNER

MODE: STAGING DEPLOY / NON-AUTHORITATIVE

REPO:
F:\00-Ticenpi-SaaS\TicenpiPost

GOAL:
Deploy the already-implemented Post Central Seat shadow to Staging and verify live shadow behavior without changing user-visible authorization.

PREREQUISITE:
P4A local result:
POST_SHADOW_IMPLEMENTED = YES
LEGACY_AUTHORITY_PRESERVED = YES
READY_FOR_POST_STAGING_SHADOW = YES

ABSOLUTE RULES:
- Central Seat remains NON-authoritative.
- Do NOT change SEAT_POLICY from skip to require.
- Do NOT change AUTO_TENANT_BOOTSTRAP.
- Do NOT change ALLOW_TENANT_KEY.
- Do NOT touch Production.
- Do NOT modify Central Seat SQL/RPC.
- Do NOT change Post tenant_members semantics.
- Do NOT run another product deployment in parallel.
- Preserve unrelated worktree changes.
- Do not commit/push unless explicitly instructed.

1. PREFLIGHT
Record branch, HEAD, git status --short, git diff --stat.
Verify only the intended P4A files are part of this candidate.
Re-run targeted P4A tests and git diff --check.
If unrelated source changes are present in the deploy candidate, STOP.

2. RELEASE IDENTITY
Build/deploy using the existing approved Post Staging deployment path.
Record:
- source commit / working tree identity
- image digest
- release identity
- runtime config identity
- health result

Do not modify Production.

3. ENABLE SHADOW ONLY
Enable only:
TICENPI_CENTRAL_SEAT_SHADOW_ENABLED=1

Do not alter:
TICENPI_SEAT_POLICY
TICENPI_AUTO_TENANT_BOOTSTRAP
TICENPI_ALLOW_TENANT_KEY
commercial gate semantics

If enabling the shadow requires changing a shared config file in a way that can affect another service, STOP.

4. LIVE STAGING VERIFICATION
Using existing Staging test identities, verify at minimum:
- legacy allow + central allow
- legacy allow + central deny
- central error handling if safely simulatable without mutating Central Seat
- platform_admin existing behavior
- Post tenant mapping still required
- no user-visible authorization behavior changed

For mismatch cases:
legacy decision must remain final.

5. OBSERVABILITY
Confirm logs contain normalized shadow events only.
No bearer/JWT/email/secret/customer PII.

Expected events include:
MATCH_ALLOW
MATCH_DENY if reachable
MISMATCH_LEGACY_ALLOW_CENTRAL_DENY
MISMATCH_LEGACY_DENY_CENTRAL_ALLOW if reachable
CENTRAL_ERROR
CENTRAL_UNKNOWN

Do not force synthetic destructive data if existing Staging test accounts can prove the cases.

6. HEALTH / SMOKE
Run existing Post Staging health and critical smoke checks.
Verify:
- /api/health
- authenticated access
- bootstrap/session path
- tenant mapping
- no auth regression

7. ROLLBACK READINESS
Record the exact previous Staging release identity and rollback command/path.
Do not perform rollback unless verification fails.

If verification fails:
rollback using existing safe release mechanism and report.

8. FINAL REPORT

A. PREFLIGHT
B. RELEASE IDENTITY
C. RUNTIME FLAGS
D. LIVE SHADOW MATRIX
E. LOGGING SAFETY
F. HEALTH/SMOKE
G. ROLLBACK READINESS
H. SAFETY

Must state:
CENTRAL_AUTHORITATIVE = NO
USER_VISIBLE_AUTH_BEHAVIOR_CHANGED = NO
SEAT_POLICY_CHANGED = NO
AUTO_BOOTSTRAP_CHANGED = NO
TENANT_KEY_SETTING_CHANGED = NO
PRODUCTION_MUTATED = NO

I. FINAL RESULT

If successful:
RESULT = PASS
POST_STAGING_SHADOW_DEPLOYED = YES
POST_STAGING_SHADOW_VERIFIED = YES
LEGACY_AUTHORITY_PRESERVED = YES
READY_FOR_POST_CUTOVER_PLANNING = YES

If blocked:
RESULT = BLOCKED
POST_STAGING_SHADOW_DEPLOYED = NO/YES
POST_STAGING_SHADOW_VERIFIED = NO
BLOCKER = exact blocker

Stop after Staging shadow verification.