# W2-PLATFORM-S1 — Apply Commercial Admin Contracts to Staging

ROLE: CENTRAL COMMERCIAL CORE STAGING MIGRATION OWNER

MODE: STAGING APPLY + VERIFICATION ONLY

WORKSPACE:
F:\00-Ticenpi-SaaS

PLATFORM REPO:
F:\00-Ticenpi-SaaS\ticenpi-platform

VERIFIED SOURCE CANDIDATE:
Branch: w2-platform-c1-commercial-admin
Commit: c7e589f21e019280ae7f83584bde3e6af255fd0f

GOAL:
Apply the already-tested Commercial Admin contracts to Central Supabase Staging, verify them live, and stop before any Launcher deployment or Production change.

VERIFIED IMPLEMENTATION:
- Central Seat source canonicalized
- admin_list_customer_members implemented
- member suspend/reactivate/remove implemented
- customer update/suspend/reactivate implemented
- audit/RLS/SECURITY DEFINER/search_path protections implemented
- disposable PostgreSQL tests PASS
- Central Seat regression PASS
- concurrency PASS
- rollback PASS

ABSOLUTE RULES:
- Staging only: jlsqjvehwblkeuycjoyj
- Do NOT touch Production.
- Do NOT modify migration semantics.
- Do NOT include unrelated Platform dirty WIP.
- Use exact commit c7e589f...
- Do NOT run ORC/DM fixture mutations concurrently with this migration apply.
- Take a fresh Staging logical backup before apply.
- Stop on checksum/schema drift.

1. PREFLIGHT
Prove:
- exact Platform candidate commit
- working tree isolation
- target Supabase project ref = jlsqjvehwblkeuycjoyj
- Production ref is NOT target
- current live presence/checksum of Central Seat migrations 000600 and 000700
- 000800 is not already partially applied

Record checksums for all migration files involved.

2. FRESH STAGING BACKUP
Create the approved logical backup / snapshot required by the deployment framework.

Verify backup readability and record artifact/location without exposing secrets.

Required:
STAGING_BACKUP_READY = YES

3. LIVE PRE-MIGRATION READONLY CHECK
Verify current:
- customers/memberships
- effective entitlement resolver
- product_seat_assignments
- assign/release/status/list Seat RPCs
- existing test customer used by Post regressions

No mutation yet.

4. APPLY ONLY REQUIRED NEW MIGRATION
Apply only the new Commercial Admin contract migration(s) from exact commit c7e589f....

Do not replay unrelated migrations.
Do not apply any Production migration.

5. LIVE CONTRACT VERIFICATION
As platform_admin verify:
- admin_list_customer_members returns stable member_id/user_id/email/role/status/created_at
- user_id matches customer_members exactly
- ordinary user cannot invoke admin RPC
- cross-customer read is rejected/isolated according to contract
- member suspend releases Seats
- member reactivate does not auto-restore Seats
- member remove releases Seats
- customer suspend blocks effective access
- customer reactivate preserves explicit Seat semantics
- audit events written correctly

Use a disposable/test-only customer fixture for lifecycle mutation checks.
Restore/clean up test-only state through canonical RPCs where the contract requires it.

Do not mutate the existing Post/DM/ORC regression customer unless explicitly necessary for a non-destructive read.

6. CENTRAL SEAT REGRESSION
Re-run:
- same effective entitlement row
- seat_limit
- assign/release
- product isolation
- concurrency/over-allocation protection
- admin Seat listing

Required:
CENTRAL_SEAT_REGRESSION = PASS

7. ROLLBACK PROOF
Do not roll back the successful Staging migration.
Instead verify the rollback source against a disposable/local restore or equivalent safe rehearsal.

Required:
ROLLBACK_REHEARSAL = PASS

8. FINAL REPORT
Return:
PLATFORM_SOURCE_COMMIT =
STAGING_PROJECT_REF =
STAGING_BACKUP_READY =
MIGRATION_APPLIED =
ADMIN_LIST_CUSTOMER_MEMBERS_LIVE =
MEMBER_LIFECYCLE_LIVE =
CUSTOMER_LIFECYCLE_LIVE =
CENTRAL_SEAT_REGRESSION =
ROLLBACK_REHEARSAL =
PRODUCTION_MUTATED = NO

If complete:
RESULT = PASS
PLATFORM_STAGING_ADMIN_CONTRACTS = PASS
READY_FOR_LAUNCHER_WORKBENCH_STAGING_E2E = YES

If any live mismatch occurs:
RESULT = ROLLED_BACK or BLOCKED
BLOCKER = exact issue

Do not touch Production.
