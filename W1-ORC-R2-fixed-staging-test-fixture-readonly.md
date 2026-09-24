# W1-ORC-R2 — ORC Fixed Staging Test Fixture Validation (Read-Only)

ROLE: ORC STAGING FIXTURE VALIDATION OWNER

MODE: READ-ONLY / NO FIXTURE MUTATION

WORKSPACE:
F:\00-Ticenpi-SaaS

ORC+LETTER REPO:
F:\00-Ticenpi-SaaS\TicenpiLetter

PLATFORM REPO:
F:\00-Ticenpi-SaaS\ticenpi-platform

GOAL:
Validate whether the two already-proven ordinary Post Staging regression users can be reused for ORC Central Seat assigned/unassigned E2E.

Do NOT create users.
Do NOT create customer/member/entitlement/Seat rows.
Do NOT assign/release Seats.
Do NOT modify test_access.
Do NOT deploy.

## REQUIRED RUNTIME INPUTS

ORC_TEST_ASSIGNED_USER_ID=<existing ordinary Staging Auth UUID>
ORC_TEST_UNASSIGNED_USER_ID=<existing ordinary Staging Auth UUID>

Use the same two ordinary Staging users already proven in Post Central Seat E2E.

They are expected to be:
- platform_role=user
- test_access=null
- non-admin
- existing Auth users in Staging

Do not scan only migration fixtures and conclude they do not exist.
Validate the runtime-supplied user IDs against live Staging data.

## 1. VERIFY IDENTITIES

For both runtime user IDs confirm:
- auth user exists
- profile exists
- platform_role=user
- test_access null/none
- non-admin
- same Staging project

## 2. VERIFY CUSTOMER MEMBERSHIP

Determine:
- active customer membership
- whether both can use the same existing test-only business customer
- customer status/type

Return:
ORC_TEST_CUSTOMER_ID =
SAME_ACTIVE_TEST_CUSTOMER = YES/NO

Do not mutate.

## 3. VERIFY PRODUCT CONTRACT

Use the product code proven by W1-ORC-R1 if available.

If W1-ORC-R1 has not completed:
inspect both "ocr" and "orc" read-only and report uncertainty.
Do NOT create either product.

Return:
ORC_CANONICAL_PRODUCT_CODE =
PRODUCT_EXISTS = YES/NO

## 4. VERIFY ENTITLEMENT READINESS

Read-only inspect whether the chosen test customer currently has an effective ORC entitlement.

Return:
ORC_EFFECTIVE_ENTITLEMENT_EXISTS =
STATUS =
SEAT_LIMIT =
ENTITLEMENT_ID =

If absent:
that is not a failure of user identity.
Report that ORC-F1 must canonically create/normalize the test entitlement before E2E.

## 5. VERIFY CURRENT SEAT STATE

Read-only inspect Central Seat for both users/product.

Return:
ASSIGNED_CANDIDATE_CURRENT_SEAT =
UNASSIGNED_CANDIDATE_CURRENT_SEAT =

Do not assign/release.

## 6. VERIFY ORC DATA-BOUNDARY READINESS

Because current ORC data is user-owned, determine whether each user can safely have independent ORC profile/data rows after authorization.

Do not create rows.

Return:
ORC_DATA_BOUNDARY_TYPE =
ASSIGNED_DATA_READY =
UNASSIGNED_DATA_READY =

## 7. FINAL FIXTURE PLAN

Produce the exact canonical mutations ORC-F1 will later need, if any:

- add/confirm membership
- create/update ORC entitlement
- set seat_limit
- assign Seat to assigned user
- ensure unassigned user has no Seat
- any ORC profile provisioning after auth

Do NOT execute.

## 8. FINAL REPORT

SOURCE_FILES_MODIFIED = NO
STAGING_DATA_MUTATED = NO
PRODUCTION_DATA_MUTATED = NO
DEPLOYED = NO

RESULT = PASS/PARTIAL_PASS/BLOCKED

ORC_FIXED_ORDINARY_USERS_PROVEN =
SAME_ACTIVE_TEST_CUSTOMER =
ORC_PRODUCT_CONTRACT_IDENTIFIED =
ORC_ENTITLEMENT_READY =
READY_FOR_ORC_F1 =

A PARTIAL_PASS is acceptable if users/customer are valid but entitlement/Seat rows still need canonical creation during ORC-F1.

Finish and stop.
