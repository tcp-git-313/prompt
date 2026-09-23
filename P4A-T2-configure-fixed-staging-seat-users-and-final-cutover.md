# P4A-T2 — Configure Fixed Post Staging Seat Test Users & Resume Final Cutover

ROLE: POST CENTRAL SEAT STAGING TEST FIXTURE + FINAL CUTOVER OWNER

MODE: STAGING ONLY

WORKSPACE:
F:\00-Ticenpi-SaaS

PRIMARY REPOS:
- F:\00-Ticenpi-SaaS\ticenpi-platform
- F:\00-Ticenpi-SaaS\TicenpiPost

REQUIRED RUNTIME INPUTS FROM OPERATOR:
- POST_TEST_ASSIGNED_USER_ID
- POST_TEST_UNASSIGNED_USER_ID

These are existing Staging Supabase Auth user UUIDs.
Do not create replacement users.
Do not print credentials or tokens.

KNOWN ADMIN TEST IDENTITY:
Use the already-discovered existing Staging platform_admin identity from prior P4A inventory.

GOAL:
Configure the two existing ordinary Staging Auth users into the canonical Commercial Core + Central Seat + Post tenant model, then resume the final Post authoritative Staging cutover.

Final intended model:

ADMIN
→ platform_admin
→ Seat exempt
→ ALLOW

POST_TEST_ASSIGNED
→ ordinary non-admin
→ active customer membership
→ effective Post entitlement
→ Central Post Seat assigned
→ valid Post tenant mapping
→ ALLOW

POST_TEST_UNASSIGNED
→ ordinary non-admin
→ same test customer/commercial eligibility where safe
→ effective Post entitlement
→ NO Central Post Seat
→ valid Post tenant mapping
→ DENY / 403

The intended material difference between the two ordinary users is Central Post Seat assignment.

CURRENT VERIFIED CANDIDATE:
- authoritative Post candidate commit: 27e4efe
- CI: GREEN
- image digest: sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0

CURRENT STAGING:
- release: 20260923-183949
- TICENPI_COMMERCIAL_GATE=1
- TICENPI_AUTO_TENANT_BOOTSTRAP=1
- TICENPI_SEAT_POLICY=skip
- TICENPI_ALLOW_TENANT_KEY=0
- TICENPI_CENTRAL_SEAT_SHADOW_ENABLED=1

ABSOLUTE RULES:
- Staging only.
- Production untouched.
- No new Auth users.
- No deletion of existing users.
- Do not make either ordinary test user platform_admin.
- Do not use test_access allow/deny to fake Seat behavior.
- Use canonical Commercial Core admin operations.
- Use assign_product_seat/release_product_seat for Seat mutation.
- Do not directly write product_seat_assignments.
- Do not use X-Tenant-Key for final E2E.
- Do not use service-role in browser/user flow.
- Do not run another product deployment in parallel.

## 1. VALIDATE INPUT IDENTITIES

Require both runtime inputs to be present and valid UUIDs.

Read-only verify each Staging Auth user exists.

For both ordinary users verify:
- Auth exists
- platform_admin = NO
- no test-access bypass invalidating Seat E2E

If either user is admin or missing:
STOP.

## 2. SELECT A TEST-ONLY CUSTOMER

Find an existing clearly test-only Staging customer suitable for permanent Central Seat regression.

If none exists, create one through the canonical platform-admin customer creation path.

Do not reuse real customer data.

Record:
TEST_CUSTOMER_ID
TEST_CUSTOMER_IS_TEST_ONLY = YES

## 3. ENSURE ACTIVE MEMBERSHIP

Using canonical admin membership operations, ensure both ordinary users are active members of TEST_CUSTOMER_ID.

Do not use raw table writes when a supported admin operation exists.

Verify both after mutation.

If the canonical membership write path is missing:
STOP and report it.

## 4. ENSURE EFFECTIVE POST ENTITLEMENT

Ensure TEST_CUSTOMER_ID has one effective Post entitlement:

- product_code=post
- active/trial according to canonical resolver
- seat_limit >= 2
- starts_at <= now
- not expired

Prefer a valid existing entitlement.
Otherwise use canonical platform-admin entitlement write/update operation.

Verify with the authenticated canonical resolver:
effective_entitlement_details(TEST_CUSTOMER_ID)

Record:
EFFECTIVE_POST_ENTITLEMENT_ID
EFFECTIVE_POST_STATUS
EFFECTIVE_POST_SEAT_LIMIT

## 5. CONFIGURE CENTRAL SEAT

For POST_TEST_ASSIGNED_USER_ID:
call canonical:
assign_product_seat(TEST_CUSTOMER_ID, "post", POST_TEST_ASSIGNED_USER_ID)

For POST_TEST_UNASSIGNED_USER_ID:
ensure no assignment using canonical:
release_product_seat(TEST_CUSTOMER_ID, "post", POST_TEST_UNASSIGNED_USER_ID)

Then call:
admin_list_product_seats(TEST_CUSTOMER_ID, "post")

Required:
- assigned test user appears
- unassigned test user does not appear
- seat_limit comes from effective resolver

## 6. CONFIGURE POST TENANT MAPPING

Both ordinary test users must have valid Post tenant_members mapping using the existing supported Post provisioning/admin mechanism.

Required:
POST_TEST_ASSIGNED_TENANT_MAPPING = YES
POST_TEST_UNASSIGNED_TENANT_MAPPING = YES

Both should map to the same intended test Post tenant when safe.

This is separate from customer_members.

If no supported Post tenant provisioning path exists:
STOP.

## 7. PRE-CUTOVER FIXTURE GATE

Required:

POST_TEST_ASSIGNED:
Auth YES
Admin NO
Membership active
Effective Post entitlement YES
Post Seat YES
Post tenant mapping YES

POST_TEST_UNASSIGNED:
Auth YES
Admin NO
Membership active
Effective Post entitlement YES
Post Seat NO
Post tenant mapping YES

Set:
FIXED_STAGING_TEST_FIXTURE_READY = YES

Only then continue.

## 8. DEPLOY EXACT AUTHORITATIVE CANDIDATE

Deploy exact candidate:
commit 27e4efe
digest sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0

Use approved Post Staging deploy chain.

Do not rebuild another commit.

Record:
PREVIOUS_RELEASE
NEW_RELEASE
DEPLOYED_COMMIT
DEPLOYED_DIGEST
DEPLOY_AUDIT

## 9. SWITCH STAGING TO AUTHORITATIVE CENTRAL SEAT

Required effective policy:

TICENPI_COMMERCIAL_GATE=1
TICENPI_SEAT_POLICY=require
TICENPI_ALLOW_TENANT_KEY=0

AUTO_TENANT_BOOTSTRAP may remain 1, but prove:
SEAT_ENFORCEMENT_DECOUPLED_FROM_AUTO_BOOTSTRAP = YES

Shadow flag may remain 1 during validation if harmless.

Read actual running container values.

## 10. FINAL LIVE E2E

Use normal Staging login/session only.

A. platform_admin
Expected ALLOW

B. POST_TEST_ASSIGNED
Expected:
Central Post Seat ALLOW
Post tenant mapping valid
Final ALLOW

C. POST_TEST_UNASSIGNED
Expected:
Central Post Seat DENY/not_assigned
Post tenant mapping valid
Final 403

D. Invalid JWT
Expected 401

Do not use X-Tenant-Key.

If agent cannot interactively login as B/C because it has no credentials/session:
do not undo fixture or deployment.
Return PARTIAL_PASS and request only manual browser login result for these two existing accounts.
Never request the passwords.

## 11. CAUSAL PROOF

Prove B and C have:
- same customer
- active commercial membership
- same effective Post entitlement class
- valid Post tenant mapping
- different Central Post Seat assignment

Required:
CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES

Assigned → ALLOW
Unassigned → DENY

## 12. HEALTH / SECURITY

Verify:
- /api/health 200
- external Staging 200
- correct release/commit/digest
- no crash loop
- no bearer/JWT/password/email/service-role/Supabase secret leakage

AUTH_LOG_SECRET_LEAK = NO

## 13. KEEP THESE AS FIXED REGRESSION IDENTITIES

After successful setup, preserve the test fixture:

- platform_admin
- assigned ordinary user
- unassigned ordinary user

Do not release the assigned Seat after success.
Do not assign a Seat to the unassigned user.

Document their labels and UUID references in the appropriate private/local test documentation; do not publish credentials.

## 14. ROLLBACK

If authoritative deployment causes mandatory health/auth failure:
- rollback Post Staging only
- restore previous runtime config
- leave the canonical test fixture available for retry
- prove health 200

## 15. FINAL REPORT

A. INPUT IDENTITIES
B. TEST CUSTOMER
C. MEMBERSHIPS
D. EFFECTIVE POST ENTITLEMENT
E. CENTRAL SEAT
F. POST TENANT MAPPING
G. FIXTURE READINESS
H. DEPLOY
I. EFFECTIVE RUNTIME FLAGS
J. FINAL E2E
K. CENTRAL SEAT CAUSAL PROOF
L. HEALTH / SECURITY
M. PRODUCTION SAFETY

Required final fields:

FIXED_STAGING_TEST_FIXTURE_READY =
POST_TEST_ASSIGNED_SEAT =
POST_TEST_UNASSIGNED_SEAT =
POST_TEST_ASSIGNED_TENANT_MAPPING =
POST_TEST_UNASSIGNED_TENANT_MAPPING =
SEAT_POLICY =
CENTRAL_AUTHORITATIVE =
BUSINESS_ASSIGNED_ALLOW =
BUSINESS_UNASSIGNED_DENY =
CENTRAL_SEAT_CAUSAL_E2E_PROVEN =
PRODUCTION_MUTATED = NO

If full E2E passes:

RESULT = PASS
POST_CENTRAL_SEAT_STAGING_CUTOVER = PASS
CENTRAL_AUTHORITATIVE = YES
READY_FOR_POST_PRODUCTION_CUTOVER = YES

If fixture + deploy succeed but B/C interactive login is unavailable:

RESULT = PARTIAL_PASS
FIXED_STAGING_TEST_FIXTURE_READY = YES
POST_CENTRAL_SEAT_STAGING_CUTOVER = PENDING_MANUAL_LOGIN_E2E
MANUAL_LOGIN_E2E_REQUIRED = YES
READY_FOR_POST_PRODUCTION_CUTOVER = NO

If canonical configuration path is missing:

RESULT = BLOCKED
FIXED_STAGING_TEST_FIXTURE_READY = NO
BLOCKER = exact missing canonical write/provisioning path

Do not deploy Production.
