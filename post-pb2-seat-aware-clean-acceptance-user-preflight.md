# POST P-B2 — SEAT-AWARE CLEAN ACCEPTANCE IDENTITY PREFLIGHT

## ROLE
POST ACCEPTANCE IDENTITY PREFLIGHT OWNER

## MODEL
LUNA Medium

## MODE
READ-ONLY ONLY — PLAN FROZEN

Project:
F:\00-Ticenpi-SaaS\TicenpiPost

Current facts:
- P-A admin identity audit PASS.
- tcp.a2026i@gmail.com is platform_admin on Staging and MUST NOT be used for assigned/unassigned Seat acceptance.
- Legacy TEST mapping belongs to a different old user_id and remains untouched.
- P-C source audit found no code change required for 19418 Extension/Studio readiness.
- Staging project: jlsqjvehwblkeuycjoyj.
- Staging now contains public.product_seat_assignments.
- Central Seat schema fingerprint says Seat migration was applied on 2026-09-23.
- This does NOT by itself prove the full CENTRAL_SEAT_STAGING_READY gate.
- Post ADR-002 may be stale relative to current Staging state.

Mission:
Identify the correct existing NON-ADMIN Staging Seat test identities for Post Acceptance and report their exact current central commercial + Seat state.

Do not create users.
Do not assign/release Seat.
Do not modify Post tenant data.
Do not modify source.
Do not commit or push.
Do not update ADRs in this task.

## 1. DO NOT STOP ON STALE ADR ALONE

ADR-002 saying Central Seat is not live is no longer sufficient reason to stop.

Treat current live Staging schema + current central contract as higher-priority operational evidence for this preflight.

However, do NOT assume CENTRAL_SEAT_STAGING_READY = PASS.

Verify the actual current Staging state read-only.

## 2. FIND EXISTING TEST IDENTITIES FIRST

Search current Staging Auth / profiles / customer_members / commercial data for existing dedicated Seat test identities.

Prioritize identities explicitly intended for:
- platform_test_allow
- platform_test_deny
- seat allow / deny
- staging acceptance

Do not create a new identity if suitable existing identities exist.

For each candidate record:
- email
- auth user_id
- profile/platform role
- customer_id
- membership role/status
- entitlement for Post
- product seat assignment state
- whether platform_admin exemption applies
- current Post tenant mapping count
- legacy TEST data presence

## 3. REQUIRED CANDIDATE PROPERTIES

Assigned-seat candidate must be:
- non-platform_admin
- exactly one current Staging Auth user_id
- active customer membership
- active Post entitlement path
- explicitly assigned Post Seat, if Seat contract is already active
- no ambiguous legacy TEST identity collision

Denied-seat candidate must be:
- non-platform_admin
- exactly one current Staging Auth user_id
- valid active customer membership if the intended test is "member but unassigned"
- active Post entitlement path
- NOT assigned a Post Seat
- no platform_admin/test exemption that would bypass Seat deny

If existing test identities have exemption semantics that invalidate normal Seat testing, report that and do not use them.

## 4. VERIFY CENTRAL SEAT CONTRACT READ-ONLY

Find current exact Staging contract for:
- product_seat_assignments
- status lookup RPC/function
- assign/release RPC names if present
- platform_admin exemption
- seat_limit source
- individual-customer behavior

Record exact current migration/contract commit or schema fingerprint timestamp used.

Do not modify central files.

## 5. CHECK WHETHER CENTRAL SEAT IS FULLY READY

Read-only assess these minimum readiness items:

- assignment storage exists
- status lookup exists
- seat_limit enforcement exists
- entitlement inactive -> deny
- membership inactive -> deny
- platform_admin exemption explicit
- audit path exists
- Staging migration applied

Report:
CENTRAL_SEAT_SCHEMA_PRESENT = YES/NO
CENTRAL_SEAT_CONTRACT_PRESENT = YES/NO
CENTRAL_SEAT_RUNTIME_READY = YES/NO/UNPROVEN

Do not promote UNPROVEN to YES.

## 6. POST TENANT STATE

For each selected candidate:
- count active tenant_members rows by exact auth user_id
- list tenant_id if exactly one
- mark NO_TENANT if none
- mark AMBIGUOUS if more than one for same exact user_id

Do not use email alone to decide tenant ownership.

No cleanup in this task.

## 7. FINAL SELECTION

Select:
ALLOW_USER = best existing assigned-seat candidate
DENY_USER = best existing unassigned-seat candidate

If only one side exists, report the missing side.

If neither exists:
RESULT = NEED_TEST_IDENTITY_SETUP

Do not create them.

## 8. REPORT FORMAT

PHASE = SEAT-AWARE CLEAN ACCEPTANCE USER PREFLIGHT
RESULT = PASS / NEED_TEST_IDENTITY_SETUP / HARD_STOP

STAGING_PROJECT =
EVIDENCE_AS_OF =

CENTRAL_SEAT_SCHEMA_PRESENT =
CENTRAL_SEAT_CONTRACT_PRESENT =
CENTRAL_SEAT_RUNTIME_READY =
CENTRAL_SEAT_REFERENCE =

ALLOW_EMAIL =
ALLOW_USER_ID =
ALLOW_PLATFORM_ROLE =
ALLOW_CUSTOMER_ID =
ALLOW_MEMBERSHIP =
ALLOW_POST_ENTITLEMENT =
ALLOW_SEAT_ASSIGNED =
ALLOW_TENANT_STATE =
ALLOW_SAFE_FOR_ACCEPTANCE = YES/NO

DENY_EMAIL =
DENY_USER_ID =
DENY_PLATFORM_ROLE =
DENY_CUSTOMER_ID =
DENY_MEMBERSHIP =
DENY_POST_ENTITLEMENT =
DENY_SEAT_ASSIGNED =
DENY_TENANT_STATE =
DENY_SAFE_FOR_ACCEPTANCE = YES/NO

TCP_A2026I_EXCLUDED = YES

REQUIRED_USER_ACTION =
NEXT =

No mutation.
No commit.
No push.
