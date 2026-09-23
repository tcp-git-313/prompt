# POST P-A — PLATFORM_ADMIN IDENTITY AUDIT

## ROLE
POST ADMIN IDENTITY AUDITOR

## MODEL
LUNA High

## MODE
READ-ONLY ONLY — ZERO-MUTATION PARALLEL TASK

Project context:
`F:\00-Ticenpi-SaaS\TicenpiPost`

Goal:
Confirm exactly which Supabase Auth `user_id` for `tcp.a2026i@gmail.com` owns `platform_admin`, and classify the duplicate-email identities without modifying any data.

This task is intentionally safe to run in parallel with:
- Central Seat work
- Post clean acceptance-user preflight
- other read-only audits

## HARD RULES

DO NOT:
- modify source
- commit
- push
- update DB
- delete/merge users
- change platform_admin
- change memberships
- change entitlements
- change tenant_members
- change Supabase settings
- create test users
- rotate tokens

If any write would be required:
STOP.

## 1. READ FIRST

Read only the minimum relevant central auth/commercial role code and schema.

Find the actual source of truth for:
- Supabase Auth user id
- platform_admin
- customer membership
- current user / profile mapping

Do not infer admin from email.

## 2. IDENTIFY BOTH AUTH USERS

For email:

`tcp.a2026i@gmail.com`

List every known Auth user identity.

For each record:
- user_id
- email
- created_at if available
- profile mapping
- active customer membership(s)
- platform_admin status
- any explicit test/admin marker

Do not expose secrets.

## 3. CONFIRM PLATFORM_ADMIN OWNER

Determine exactly which user_id has platform_admin authority.

Record:

ADMIN_USER_ID =
SECONDARY_SAME_EMAIL_USER_ID =
ADMIN_SOURCE_OF_TRUTH =
ADMIN_STATUS_PROVEN = YES/NO

If admin status cannot be proven from a trusted source:
HARD_STOP.

## 4. LEGACY CLASSIFICATION

Classify each same-email user only from evidence:

- CURRENT_ADMIN_IDENTITY
- LEGACY_TEST_IDENTITY
- UNKNOWN

Do not change labels in DB.
This is a reporting classification only.

## 5. SAFETY CHECK

Confirm whether leaving both Auth users and both Post tenants untouched creates any immediate admin-rights regression.

Do not evaluate future cleanup here.

Report:

LEAVING_LEGACY_DATA_UNTOUCHED_BREAKS_ADMIN = YES/NO/UNKNOWN

## 6. CENTRAL SEAT INTERACTION

Read-only answer:

Would Central Seat key access by:
`user_id + customer + product`

rather than email?

Confirm from the actual in-progress/committed central contract if available.

Do not modify Central Seat.

If Central Seat files are changing during this audit, record the exact commit/timestamp used.

## 7. FINAL REPORT

PHASE = PLATFORM_ADMIN IDENTITY AUDIT
RESULT = PASS / HARD_STOP

EMAIL =
AUTH_USER_IDS =

ADMIN_USER_ID =
ADMIN_SOURCE_OF_TRUTH =
ADMIN_STATUS_PROVEN =

SECONDARY_USER_ID =
SECONDARY_CLASSIFICATION =

LEAVING_LEGACY_DATA_UNTOUCHED_BREAKS_ADMIN =

CENTRAL_SEAT_IDENTITY_KEY =
CENTRAL_SEAT_REFERENCE_COMMIT_OR_STATE =

SAFE_TO_EXCLUDE_TCP_A2026I_FROM_ACCEPTANCE = YES/NO

NEXT =

No mutations.
No commit.
No push.
