# P4A-D3 — Final Post Staging Central Seat Deploy & Authoritative E2E

ROLE: POST FINAL STAGING DEPLOY / CENTRAL SEAT CUTOVER OWNER

MODE: STAGING ONLY / FINAL DEPLOY + AUTHORITATIVE E2E

REPO:
F:\00-Ticenpi-SaaS\TicenpiPost

GOAL:
Deploy the already-verified split-identity Post Staging candidate, activate Central Seat authoritative enforcement, and complete the final Assigned/Unassigned/Admin E2E.

DO NOT modify CI architecture.
DO NOT rebuild application image.
DO NOT touch Production.
DO NOT recreate test users.
DO NOT change Central Seat SQL/RPC.
DO NOT alter the prepared fixed Staging regression fixture.

## VERIFIED RELEASE IDENTITIES

APP_SOURCE_COMMIT:
27e4efedbd9157293b95b56cd9cecaeeadbf6cac

APP_IMAGE_DIGEST:
sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0

APP_BUILD_RUN_ID:
35885450561

DEPLOY_CONFIG_COMMIT:
f4ecbf0d3d292ac4af668484e6327aadfdcb18f9

CONFIG VALIDATION CI:
GREEN
Run:
35911744520

RECURSIVE_DIGEST_BLOCKER_CLEARED = YES
CONFIG_ONLY_COMMIT_REBUILDS_IMAGE = NO
PROVENANCE_CONTRACT_ACCEPTS_SPLIT_IDENTITY = YES
READY_FOR_P4A_FINAL_STAGING_DEPLOY = YES

## VERIFIED FIXTURE STATE

FIXED_STAGING_TEST_FIXTURE_READY = YES

The existing Staging regression identities are already configured as:

- ADMIN test identity
  - platform_admin
  - Seat exempt

- POST_TEST_ASSIGNED
  - ordinary non-admin
  - active customer membership
  - effective Post entitlement
  - Post Seat assigned
  - valid Post tenant mapping

- POST_TEST_UNASSIGNED
  - ordinary non-admin
  - active customer membership
  - effective Post entitlement
  - no Post Seat assignment
  - valid Post tenant mapping

Do NOT modify these fixture semantics before final E2E.

## CURRENT STAGING BEFORE DEPLOY

Current release:
20260923-183949

Current flags:
TICENPI_COMMERCIAL_GATE=1
TICENPI_AUTO_TENANT_BOOTSTRAP=1
TICENPI_SEAT_POLICY=skip
TICENPI_ALLOW_TENANT_KEY=0
TICENPI_CENTRAL_SEAT_SHADOW_ENABLED=1

## ABSOLUTE RULES

- Staging only.
- Post service only.
- No Production changes.
- No new image build.
- No config commit changes unless a factual drift is discovered.
- No Central Seat fixture changes unless necessary to restore exact prior test state after a failed temporary verification.
- Do not use X-Tenant-Key.
- Do not use service-role for user login E2E.
- Do not run another product deployment in parallel.
- Roll back immediately on mandatory failure.

## 1. PREFLIGHT

Record:

- active Staging release
- active app commit
- active image digest
- current effective runtime flags
- current deploy config commit
- current health

Verify deployment candidate identity:

APP_SOURCE_COMMIT =
27e4efedbd9157293b95b56cd9cecaeeadbf6cac

APP_IMAGE_DIGEST =
sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0

DEPLOY_CONFIG_COMMIT =
f4ecbf0d3d292ac4af668484e6327aadfdcb18f9

If any mismatch:
STOP.

## 2. FINAL DEPLOY GATE

Prove:

APP_ARTIFACT_PROVEN = YES
CONFIG_VALIDATION_CI = GREEN
COMPOSE_DIGEST_MATCHES_VERIFIED_APP_ARTIFACT = YES
STAGING_SCOPE_ONLY = YES
PRODUCTION_UNCHANGED = YES
FIXED_STAGING_TEST_FIXTURE_READY = YES

Only then continue.

## 3. DEPLOY POST STAGING ONLY

Use the approved single-service Post Staging deploy chain.

Deploy:

APP_SOURCE_COMMIT =
27e4efedbd9157293b95b56cd9cecaeeadbf6cac

APP_IMAGE_DIGEST =
sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0

DEPLOY_CONFIG_COMMIT =
f4ecbf0d3d292ac4af668484e6327aadfdcb18f9

Do not rebuild.

Record:

PREVIOUS_RELEASE =
NEW_RELEASE =
DEPLOYED_APP_SOURCE_COMMIT =
DEPLOYED_IMAGE_DIGEST =
DEPLOYED_CONFIG_COMMIT =
DEPLOY_AUDIT =

Required:
DEPLOYED_IMAGE_DIGEST == APP_IMAGE_DIGEST

## 4. VERIFY EFFECTIVE RUNTIME POLICY

Read the actual running container environment.

Required:

TICENPI_COMMERCIAL_GATE = 1
TICENPI_SEAT_POLICY = require
TICENPI_ALLOW_TENANT_KEY = 0

Record:

TICENPI_AUTO_TENANT_BOOTSTRAP =
TICENPI_CENTRAL_SEAT_SHADOW_ENABLED =

Prove:

SEAT_ENFORCEMENT_DECOUPLED_FROM_AUTO_BOOTSTRAP = YES

Central Seat must now be authoritative.

Set only if proven:

CENTRAL_AUTHORITATIVE = YES

## 5. VERIFY FIXTURE BEFORE E2E

Read-only recheck:

ADMIN:
platform_admin = YES

POST_TEST_ASSIGNED:
- ordinary user
- active membership
- effective Post entitlement
- Seat assigned
- Post tenant mapping valid

POST_TEST_UNASSIGNED:
- ordinary user
- active membership
- effective Post entitlement
- Seat NOT assigned
- Post tenant mapping valid

If fixture drifted:
STOP before user E2E.

Do not normalize again unless exact prior fixture contract can be restored canonically.

## 6. FINAL LIVE E2E

Use normal Staging browser/login/session.

Do not use X-Tenant-Key.

### Case A — ADMIN

Expected:
ALLOW

Central Seat:
exempt / not required according to canonical admin contract

Required:
ADMIN_ALLOW = PASS

### Case B — ASSIGNED ORDINARY USER

Expected:
- authenticated
- membership valid
- Post entitlement valid
- Central Post Seat = ALLOW
- Post tenant mapping valid
- final access = ALLOW

Required:
BUSINESS_ASSIGNED_ALLOW = PASS

### Case C — UNASSIGNED ORDINARY USER

Expected:
- authenticated
- membership valid
- Post entitlement valid
- Central Post Seat = DENY/not_assigned
- Post tenant mapping valid
- final response = 403 / no Post access

Required:
BUSINESS_UNASSIGNED_DENY = PASS

### Case D — INVALID JWT

Expected:
401

Required:
INVALID_JWT_401 = PASS

If the execution agent cannot interactively log in as B/C because it has no browser session or credentials:

Do NOT change fixture.
Do NOT create users.
Do NOT revert the deployment solely for lack of interactive credentials.

Return:

RESULT = PARTIAL_PASS
DEPLOY = PASS
AUTHORITATIVE_POLICY = PASS
MANUAL_LOGIN_E2E_REQUIRED = YES

and provide exactly these two operator steps:

1. login as POST_TEST_ASSIGNED and report whether Post opens
2. login as POST_TEST_UNASSIGNED and report whether Post is denied

No password should be requested.

## 7. PROVE CENTRAL SEAT CAUSALITY

For B and C prove they share:

- valid Auth identity class
- active commercial membership
- same effective Post entitlement eligibility
- valid Post tenant mapping

and differ in:

Central Post Seat assignment

Required:

CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES

Expected:

Assigned → ALLOW
Unassigned → DENY

## 8. PROVE POST TENANT MAPPING REMAINS REQUIRED

Central Seat ALLOW must not replace Post product data isolation.

Verify:

POST_TENANT_MAPPING_PRESERVED = YES

If possible, prove the authorization chain remains:

JWT
→ Commercial Core
→ Central Seat
→ Post tenant mapping
→ protected API/data

Do not mutate tenant mappings solely to force a negative case.

## 9. ERROR SEMANTICS

Verify live or targeted runtime evidence for:

missing/invalid JWT → 401

valid JWT without Seat → 403

Central dependency unavailable → 503 / fail closed according to implemented authoritative contract

If Central failure cannot be safely simulated live:
mark NOT_REACHED.
Do not change infrastructure just to force it.

## 10. LOGGING SAFETY

Inspect auth/Seat logs.

Confirm no leakage of:

- bearer token
- JWT
- refresh token
- password
- email
- service-role
- Supabase secret
- cookie

Required:

AUTH_LOG_SECRET_LEAK = NO

If secret leakage occurs:
rollback and HARD STOP.

## 11. HEALTH / SMOKE

Run:

- internal /api/health
- external Post Staging endpoint
- container readiness
- authenticated assigned flow
- unassigned denial
- session/bootstrap flow
- protected API smoke
- tenant mapping smoke
- critical smoke

Required:

HEALTH_200 = YES
CRASH_LOOP = NO

## 12. ROLLBACK

Before deployment record the exact previous release/config identity.

If any mandatory deployment/runtime/security gate fails:

- rollback Post Staging only
- restore previous config
- keep fixed Central Seat regression fixture intact
- verify previous release active
- verify health 200

Do not touch other services.

## 13. FINAL REPORT

### A. PREDEPLOY IDENTITY

PREVIOUS_RELEASE =
APP_SOURCE_COMMIT =
APP_IMAGE_DIGEST =
DEPLOY_CONFIG_COMMIT =

### B. DEPLOY GATE

APP_ARTIFACT_PROVEN =
CONFIG_VALIDATION_CI =
COMPOSE_DIGEST_MATCHES_VERIFIED_APP_ARTIFACT =
FIXED_STAGING_TEST_FIXTURE_READY =

### C. DEPLOY RESULT

NEW_RELEASE =
DEPLOYED_APP_SOURCE_COMMIT =
DEPLOYED_IMAGE_DIGEST =
DEPLOYED_CONFIG_COMMIT =
DEPLOY_AUDIT =

### D. RUNTIME FLAGS

COMMERCIAL_GATE =
SEAT_POLICY =
ALLOW_TENANT_KEY =
AUTO_TENANT_BOOTSTRAP =
CENTRAL_SEAT_SHADOW_ENABLED =
SEAT_ENFORCEMENT_DECOUPLED_FROM_AUTO_BOOTSTRAP =
CENTRAL_AUTHORITATIVE =

### E. FIXTURE RECHECK

ADMIN =
ASSIGNED =
UNASSIGNED =

### F. FINAL E2E

Case | Central Seat | Post tenant mapping | Final result | PASS/NOT_REACHED

Required:
ADMIN_ALLOW =
BUSINESS_ASSIGNED_ALLOW =
BUSINESS_UNASSIGNED_DENY =
INVALID_JWT_401 =

### G. CAUSAL PROOF

CENTRAL_SEAT_CAUSAL_E2E_PROVEN =

### H. TENANT ISOLATION

POST_TENANT_MAPPING_PRESERVED =

### I. ERROR SEMANTICS

CENTRAL_FAILURE_FAIL_CLOSED =
CENTRAL_FAILURE_HTTP =

### J. HEALTH / SECURITY

HEALTH_200 =
CRASH_LOOP =
AUTH_LOG_SECRET_LEAK =

### K. PRODUCTION

PRODUCTION_MUTATED = NO

### L. FINAL RESULT

If all mandatory E2E passes:

RESULT = PASS
POST_CENTRAL_SEAT_STAGING_CUTOVER = PASS
CENTRAL_AUTHORITATIVE = YES
BUSINESS_ASSIGNED_ALLOW = PASS
BUSINESS_UNASSIGNED_DENY = PASS
CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES
POST_TENANT_MAPPING_PRESERVED = YES
READY_FOR_POST_PRODUCTION_CUTOVER = YES

If deploy/policy passes but B/C interactive login is unavailable:

RESULT = PARTIAL_PASS
POST_CENTRAL_SEAT_STAGING_CUTOVER = PENDING_MANUAL_LOGIN_E2E
CENTRAL_AUTHORITATIVE = YES
MANUAL_LOGIN_E2E_REQUIRED = YES
READY_FOR_POST_PRODUCTION_CUTOVER = NO

If mandatory runtime/E2E/security fails and rollback succeeds:

RESULT = ROLLED_BACK
POST_CENTRAL_SEAT_STAGING_CUTOVER = FAIL
CENTRAL_AUTHORITATIVE = NO
ROLLBACK = PASS
READY_FOR_POST_PRODUCTION_CUTOVER = NO
BLOCKER = exact failure

Do not deploy Production.
