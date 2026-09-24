# W2-PLATFORM-C1 — Commercial Core Admin Contracts Implementation

ROLE: CENTRAL COMMERCIAL CORE CONTRACT IMPLEMENTATION OWNER

MODE: CODE + MIGRATION SOURCE + TESTS ONLY / NO STAGING APPLY / NO PRODUCTION

WORKSPACE:
F:\00-Ticenpi-SaaS

PLATFORM REPO:
F:\00-Ticenpi-SaaS\ticenpi-platform

GOAL:
Convert the already-validated Central Seat / Commercial Core source into a clean canonical committed implementation for first commercial launch, and add the minimum missing admin contracts required by Launcher Seat Workbench.

This task MUST NOT apply migrations to Staging or Production.

VERIFIED REQUIRED CONTRACTS:
1. Safe member identity read
2. Member lifecycle
3. Customer lifecycle
4. Existing Central Seat source stabilization

ABSOLUTE RULES:
- Do NOT mutate Staging or Production.
- Do NOT run migration apply against live databases.
- Do NOT modify Launcher.
- Do NOT deploy.
- Preserve all unrelated dirty WIP.
- Use isolated worktree/branch.
- Do NOT rewrite existing Central Seat semantics.
- Do NOT weaken RLS/security-definer protections.
- Do NOT expose direct table read grants.

1. PREFLIGHT
Inventory:
- existing 20260923_000600 Central Seat migration
- existing 20260923_000700 admin Seat list migration
- current committed vs untracked/dirty state
- current commercial admin RPCs
- tests/repro scripts

2. CANONICALIZE EXISTING CENTRAL SEAT SOURCE
Bring the already-Staging-validated Central Seat migrations/contracts into canonical git source without semantic changes.

Preserve:
- same effective entitlement resolver
- seat_limit from selected row
- assign/release locking
- product isolation
- downgrade protection
- audit behavior
- platform_admin authorization

Do not refactor for style.

3. SAFE MEMBER READ CONTRACT
Implement one canonical admin-only read contract:

Preferred:
admin_list_customer_members(p_customer_id uuid)

Return at minimum:
- member_id
- user_id
- email
- role
- status
- created_at

Requirements:
- platform_admin only
- SECURITY DEFINER/search_path consistent with existing admin RPCs
- stable user_id from customer_members
- no heuristic email join
- no direct client table grant
- clear errors

If extending admin_customer_detail is materially safer/smaller, document why and use that instead.

4. MEMBER LIFECYCLE
Implement minimal admin-only contracts for:
- suspend
- reactivate
- remove

Preserve existing trigger behavior:
- suspend/remove releases active Seats for that customer
- reactivation does NOT auto-reassign Seat

Add audit events.

5. CUSTOMER LIFECYCLE
Implement minimal admin-only contracts for:
- update allowed customer fields
- suspend/disable
- reactivate

Use existing customer status model.
Do not delete historical customer records.

Customer suspension must make access deny while preserving history.

6. TESTS
Add tests for:
- platform_admin allowed
- ordinary user denied
- cross-customer isolation
- member user_id exactness
- suspend releases Seats
- reactivate does not restore Seats
- remove releases Seats
- customer suspend denies effective access
- customer reactivate behavior
- Central Seat existing regression unchanged
- no direct table grants
- search_path/security-definer safety
- concurrency invariants unchanged

Run canonical repro/security tests.

7. MIGRATION / ROLLBACK SOURCE
Create forward migrations and rollback source files consistent with repository conventions.

Do NOT apply them to live Staging/Production.

Record checksums.

8. CLEAN CANDIDATE
Create isolated branch and commit only:
- canonical Central Seat source
- new admin contracts
- tests
- docs needed to freeze the contract

Push normally if required for CI.
No force push.

9. CI
Run required Platform CI/security tests.
No database mutation outside disposable/local test environments.

10. FINAL REPORT
Return:
CENTRAL_SEAT_SOURCE_CANONICALIZED =
SAFE_MEMBER_ID_CONTRACT_IMPLEMENTED =
MEMBER_LIFECYCLE_IMPLEMENTED =
CUSTOMER_LIFECYCLE_IMPLEMENTED =
FORWARD_MIGRATIONS_READY =
ROLLBACK_SOURCE_READY =
TESTS =
CI =
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO

If complete:
RESULT = PASS
READY_FOR_PLATFORM_STAGING_APPLY = YES
READY_FOR_LAUNCHER_WORKBENCH_IMPLEMENTATION = YES

If source conflict/semantic drift appears:
RESULT = BLOCKED
BLOCKER = exact issue

Stop before any Staging apply.
