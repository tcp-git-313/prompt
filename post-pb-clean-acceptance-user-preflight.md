# POST P-B — CLEAN STAGING ACCEPTANCE USER PREFLIGHT

## ROLE
POST ACCEPTANCE IDENTITY PREFLIGHT OWNER

## MODEL
LUNA Medium

## MODE
READ-ONLY ONLY — ZERO-MUTATION PARALLEL TASK

Goal:
Find the safest clean Staging Acceptance identity for Post so release testing no longer depends on legacy `tcp.a2026i@gmail.com` data.

This task may run in parallel with:
- platform_admin identity audit
- Central Seat work
- extension/launcher readiness audit

## HARD RULES

DO NOT:
- create Auth users
- delete Auth users
- modify customer memberships
- assign/release Seat
- modify entitlement
- modify Post tenant mappings
- modify DB rows
- modify source
- commit
- push
- change Supabase settings

This task only discovers candidates and prepares the exact next action.

## 1. READ FROZEN POST IDENTITY RULES

Read:
- `F:\00-Ticenpi-SaaS\TicenpiPost\_charter\adr\ADR-001-tenant-identity-entry-point.md`
- `F:\00-Ticenpi-SaaS\TicenpiPost\_charter\adr\ADR-002-central-commercial-post-tenant-seat.md`
- relevant Post memory files if available

Frozen rules:
- 1 User = 1 Post Tenant
- user identity key = stable Auth user_id, not email
- new clean Acceptance user should not inherit legacy Post tenant ambiguity

## 2. DISCOVER EXISTING SAFE CANDIDATES

Read-only inspect Staging identities suitable for Acceptance.

A candidate is CLEAN only if all can be proven:

- exactly one Auth user_id for the chosen email
- no duplicate same-email Auth identity
- no existing conflicting Post tenant mapping
- no legacy TEST tenant ownership
- commercial access can be established through normal Staging flow
- not the legacy `tcp.a2026i@gmail.com` identity
- not a production-only identity

Prefer an already existing dedicated staging/test user if one is clearly intended for this purpose.

Do not choose a user merely because it has admin privileges.

## 3. CENTRAL SEAT AWARENESS

Central Seat may still be under construction.

For the current pre-Seat Post Acceptance phase, determine whether the candidate can use the existing Staging commercial path with Seat skip as already approved.

Do not assign Seat.

Record whether the same candidate can later be used for:
- individual entitlement E2E
- business assigned-seat E2E

These may be different users.

## 4. IF A CLEAN EXISTING USER EXISTS

Report:

CANDIDATE_EMAIL =
CANDIDATE_USER_ID =
DUPLICATE_EMAIL_AUTH_USERS = 0/1/...
EXISTING_POST_TENANT =
LEGACY_DATA_PRESENT =
COMMERCIAL_CONTEXT =
SAFE_FOR_PRE_SEAT_ACCEPTANCE = YES/NO

Do not log in or mutate anything unless login itself is already part of an explicitly existing safe test harness. Prefer no session mutation in this audit.

## 5. IF NO CLEAN USER EXISTS

Do NOT create one.

Prepare exact user action / later writer action:

REQUIRED_NEW_TEST_IDENTITY =
RECOMMENDED_PURPOSE =
MINIMUM_CENTRAL_MEMBERSHIP =
MINIMUM_ENTITLEMENT =
EXPECTED_FIRST_POST_FLOW =

Expected flow:

Google Login
→ Commercial Auth
→ POST /api/session/bootstrap
→ exactly one Phase2 tenant
→ Main UI

Do not include Seat assignment yet unless Central Seat is already READY.

## 6. FINAL REPORT

PHASE = CLEAN ACCEPTANCE USER PREFLIGHT
RESULT = PASS / NEED_NEW_USER / HARD_STOP

RECOMMENDED_ACCEPTANCE_EMAIL =
RECOMMENDED_USER_ID =
DUPLICATE_EMAIL_COUNT =
EXISTING_TENANT_COUNT =
LEGACY_DATA_PRESENT =

SAFE_FOR_PRE_SEAT_ACCEPTANCE =
SAFE_FOR_LATER_SEAT_E2E =

IF_NEW_USER_REQUIRED:
USER_ACTION =
WRITER_ACTION_AFTER_APPROVAL =

NEXT =

No mutation.
No commit.
No push.
