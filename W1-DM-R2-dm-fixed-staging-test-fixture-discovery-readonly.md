# W1-DM-R2 — DM Fixed Staging Test Fixture Discovery (Read-Only)

ROLE: DM STAGING TEST FIXTURE DISCOVERY OWNER

MODE: READ-ONLY / NO MUTATION

WORKSPACE:
F:\00-Ticenpi-SaaS

DM REPO:
F:\00-Ticenpi-SaaS\TicenpiDM

CENTRAL PLATFORM REPO:
F:\00-Ticenpi-SaaS\ticenpi-platform

GOAL:
Identify two existing ordinary Staging Auth users that can be reused as the fixed DM Central Seat assigned/unassigned regression pair, without creating accounts or mutating Commercial Core / Seat state.

Do NOT create Auth users.
Do NOT assign/release Seats.
Do NOT create customers/memberships/entitlements.
Do NOT change test_access.
Do NOT modify source.
Do NOT commit/push/deploy.

## PREFERRED CANDIDATES

First inspect the same two ordinary Staging regression users already proven for Post, if discoverable from existing test fixture evidence.

Do not publish user UUIDs into source/docs unnecessarily.

The preferred DM pair must be:
- real Staging Auth users
- ordinary users, not platform_admin
- no test_access bypass
- active members of the same test-only business customer
- eligible to share the same DM entitlement
- suitable for one assigned / one unassigned DM Seat later
- suitable for DM data-boundary E2E later

## 1. PREFLIGHT

Record:
- Staging Supabase project ref
- current read-only access method
- DM product code as currently defined
- current DM entitlement state for the candidate test customer

Do not use Production.

## 2. DISCOVER EXISTING ORDINARY USERS

Search only approved existing Staging test identities.

For each candidate verify:
- Auth user exists
- profile exists
- platform_role = user
- test_access is null/none
- not platform_admin
- active customer membership
- no special bypass role

Reject identities that are:
- admin
- test_access allow/deny
- production-only
- missing Auth identity
- ambiguous duplicate identity

## 3. CUSTOMER / MEMBERSHIP COMPATIBILITY

Verify whether two ordinary candidates:
- belong to the same test-only business customer
- have active membership
- can be used under the same DM commercial entitlement

Do not mutate anything.

Return:

DM_TEST_CUSTOMER_ID = safe internal identifier if needed for execution
DM_ASSIGNED_CANDIDATE_FOUND = YES/NO
DM_UNASSIGNED_CANDIDATE_FOUND = YES/NO

## 4. DM ENTITLEMENT DISCOVERY

Read-only inspect:
- product_code="dm"
- effective entitlement status
- seat_limit
- resolver result
- entitlement_id from the detailed resolver if available

Verify the same selected effective entitlement row supplies the relevant seat_limit.

Return:

DM_EFFECTIVE_ENTITLEMENT_EXISTS = YES/NO
DM_EFFECTIVE_STATUS =
DM_SEAT_LIMIT =
SAME_EFFECTIVE_ROW_PROVEN = YES/NO

If no DM entitlement exists for the test customer:
do NOT create one.
Report that W1-DM-F1 must create/normalize it canonically later.

## 5. CURRENT DM SEAT STATE

Read-only inspect Central Seat status for both candidates.

Do not assign/release.

Return normalized:

CANDIDATE_A_CURRENT_DM_SEAT = assigned/not_assigned/not_required/unknown
CANDIDATE_B_CURRENT_DM_SEAT = assigned/not_assigned/not_required/unknown

This task is discovery only; existing state does not need to match final desired assigned/unassigned pattern yet.

## 6. DM DATA-BOUNDARY COMPATIBILITY

Inspect what DM needs for product data access.

Determine whether each candidate already has the required DM-side customer/data/RLS eligibility, if any.

Do not create mappings.

Return:

DM_DATA_BOUNDARY_TYPE =
CANDIDATE_A_DM_DATA_READY = YES/NO/NOT_APPLICABLE
CANDIDATE_B_DM_DATA_READY = YES/NO/NOT_APPLICABLE

If product-local provisioning is missing, identify the exact canonical operation/helper needed later.

## 7. FIXTURE REUSE DECISION

Set:

REUSE_EXISTING_POST_REGRESSION_USERS_FOR_DM = YES/NO

YES only if both candidates are:
- ordinary
- same Staging environment
- same test business customer
- suitable for DM entitlement/Seat E2E
- not bypassed

If NO, do not invent alternatives. Report the exact reason.

## 8. FINAL REPORT

A. STAGING IDENTITY
B. CANDIDATE USERS
C. CUSTOMER / MEMBERSHIP
D. DM ENTITLEMENT
E. CURRENT DM SEAT STATE
F. DM DATA BOUNDARY
G. REUSE DECISION
H. SAFETY

SOURCE_FILES_MODIFIED = NO
STAGING_DATA_MUTATED = NO
PRODUCTION_DATA_MUTATED = NO
DEPLOYED = NO
COMMIT_CREATED = NO
PUSHED = NO

I. FINAL RESULT

If two suitable existing users are proven:

RESULT = PASS
DM_FIXED_TEST_USERS_PROVEN = YES
REUSE_EXISTING_POST_REGRESSION_USERS_FOR_DM = YES
READY_FOR_DM_F1_RESUME = YES

If suitable identities cannot be proven:

RESULT = BLOCKED
DM_FIXED_TEST_USERS_PROVEN = NO
READY_FOR_DM_F1_RESUME = NO
BLOCKER = exact missing or disqualifying evidence

Finish and stop.
