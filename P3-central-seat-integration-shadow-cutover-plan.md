# P3 — Central Seat Integration & Shadow Cutover Plan

ROLE: CENTRAL SEAT INTEGRATION PLANNER

MODE: PLAN ONLY / READ-ONLY

Do not modify source, SQL, migrations, config, CI, Docker, Staging, Production, or Git history. Do not commit, push, deploy, restart services, apply migrations, or delete legacy code.

## Repositories

POST
F:\00-Ticenpi-SaaS\TicenpiPost

DM
F:\00-Ticenpi-SaaS\TicenpiDM

LAUNCHER
F:\00-Ticenpi-SaaS\Ticenpi-Launcher

PLATFORM / CANONICAL REFERENCE
F:\00-Ticenpi-SaaS\ticenpi-platform

## Verified baseline

Phase 2 is complete:

RESULT = PASS
PHASE_2_LEGACY_SEAT_AUDIT = PASS
LEGACY_SEAT_MAP_COMPLETE = YES
READY_FOR_PHASE_3_INTEGRATION_PLAN = YES

Canonical owner:
ticenpi-platform

Runtime source of truth:
Central Supabase

Canonical Seat contract:
F:\00-Ticenpi-SaaS\ticenpi-platform\docs\commercial-core\contracts\CENTRAL-SEAT-CONTRACT.md

Central Seat authority:
- product_seat_assignments
- product_seat_status(product_code)
- assign_product_seat(...)
- release_product_seat(...)
- effective_entitlement_details(...)

Do not redesign these contracts.

## Phase 2 facts

### Post
- Already has CentralSeatStatusClient / SeatChecker.
- Current call is product_seat_status("post").
- Seat checking is conditional.
- SeatChecker is injected only when TICENPI_AUTO_TENANT_BOOTSTRAP=1.
- TICENPI_SEAT_POLICY=skip can bypass Central Seat.
- X-Tenant-Key can bypass JWT, Commercial Gate, Seat, and Post tenant membership.
- Post tenant_members is product tenant mapping, NOT Commercial Core customer_members.
- Best future shadow insertion point: authorize_commercial_and_seat().

### DM
- require_dm_access()/EntitledUserDep currently checks my_commercial_context() + has_dm_access().
- DM does not currently call product_seat_status("dm").
- A business user with entitlement may currently pass without an assigned DM Seat.
- DM product RLS remains required.
- Best future shadow insertion point: require_dm_access().

### Launcher
- Hub/Electron gates product launch by entitlement.
- Launcher is not the API security boundary for Post/DM.
- Workbench can manage entitlement / seat_limit.
- Workbench currently has no Seat assignment/release UI.
- Legacy organizations/memberships/devices are not Product Seat tables and must remain separate.

## Goal

Produce an implementation-ready Phase 4 plan with exactly one Product Seat authority:

Central Supabase / Commercial Core
→ product_seat_status("post") → Post backend decision
→ product_seat_status("dm") → DM backend decision
→ assign/release/status → Launcher Workbench

Never merge:
- customer_members
- Post tenant_members
- Launcher organizations/memberships/devices
- product_seat_assignments

These are different responsibilities.

## Task A — Read-only runtime/config evidence

Verify, where safely possible, current effective Post values:

TICENPI_COMMERCIAL_GATE_ENABLED
TICENPI_AUTO_TENANT_BOOTSTRAP
TICENPI_SEAT_POLICY
TICENPI_ALLOW_TENANT_KEY

Report separately for:
- local/root compose
- Staging
- Production

Read-only evidence only.
Do not expose secret values.
If an effective value cannot be proven, use UNKNOWN.
Do not guess.

## Task B — Final access contract

Define exact desired backend behavior for Post and DM:

1. JWT validation
2. platform_admin behavior
3. Staging test_access behavior
4. individual entitlement behavior
5. business membership behavior
6. Seat assignment behavior
7. expired entitlement
8. suspended/removed membership
9. Central RPC failure behavior
10. HTTP status/error semantics
11. product tenant/RLS behavior after commercial authorization

## Task C — Post plan

Produce an ordered plan covering:

1. Shadow mode at authorize_commercial_and_seat().
2. Keep legacy result authoritative during shadow.
3. Compare legacy result vs product_seat_status("post").
4. Log mismatch safely.
5. Decouple Seat enforcement from TICENPI_AUTO_TENANT_BOOTSTRAP.
6. Make bootstrap control tenant creation only.
7. Ensure formal Staging/Production cannot silently use SEAT_POLICY=skip.
8. Inventory and retire X-Tenant-Key separately, only after caller proof.
9. Preserve Post tenant_members as product tenant mapping.
10. Final sequence must be:
   Central commercial/Seat authorization
   → Post tenant mapping
   → route/data access.
11. Define exact gate to switch Central result authoritative.
12. Define rollback to previous product guard without rolling back Central DB.

## Task D — DM plan

Produce an ordered plan around require_dm_access()/EntitledUserDep:

1. Shadow legacy has_dm_access() against product_seat_status("dm").
2. Same bearer/session context for both comparisons.
3. Legacy authoritative during shadow only.
4. Central errors must not silently allow access.
5. Define whether my_commercial_context() remains for UI/role/test context after cutover.
6. Remove duplicate entitlement final-decision responsibility only after cutover.
7. Preserve JWT validation and DM product RLS.
8. Define exact cutover gate and rollback.

## Task E — Launcher / Workbench plan

Plan Workbench support for:

- customer members display
- effective entitlement
- effective seat_limit
- assigned Seat count
- assigned users
- assign_product_seat()
- release_product_seat()
- seats_full
- seat_downgrade_blocked
- existing audit visibility where suitable

Keep membership management, entitlement management, and Seat assignment as separate concepts.

Determine whether existing canonical RPCs provide enough READ capability for admin Seat listing.

If a missing read contract exists, identify it exactly, but do not create it.

## Task F — Shadow verification matrix

For Post and DM include:

- individual entitlement
- business assigned Seat
- business no Seat
- Seat full
- expired entitlement
- suspended membership
- removed membership
- Post-only Seat
- DM-only Seat
- platform_admin
- Staging test allow
- Staging test deny
- repeated assign
- release/reassign
- downgrade blocked
- Central RPC unavailable/error

For each:
legacy expected
central expected
shadow authority
mismatch severity
cutover blocker YES/NO

## Task G — Observability

Plan safe mismatch logging.

Do not log:
- bearer/JWT
- secrets
- email
- raw identifiers unless strictly necessary

Prefer:
- product
- legacy decision
- central decision
- normalized reason
- environment
- release/commit
- non-reversible actor hash if needed

## Task H — Cutover gates

Define objective PASS criteria for:

CANONICAL_SQL_VERIFIED
SHADOW_TEST_MATRIX_PASS
NO_UNEXPLAINED_MISMATCH
POST_BOOTSTRAP_DECOUPLED
POST_PROTECTED_ENV_SEAT_SKIP_DISABLED
POST_TENANT_KEY_BYPASS_RESOLVED
DM_CENTRAL_SHADOW_PASS
LAUNCHER_ADMIN_ASSIGNMENT_READY
STAGING_E2E_PASS
ROLLBACK_DEFINED

State which gates are required before:
- Post cutover
- DM cutover
- Launcher Seat UI release
- Production rollout

## Task I — Workstreams

Split future execution into collision-safe workstreams:

P3A — Post auth shadow/cutover
P3B — DM auth shadow/cutover
P3C — Launcher Seat admin/read model
P3D — Integration E2E/cutover gates

For each report:
repo
allowed files/areas
forbidden overlap
dependencies
parallel-safe YES/NO
recommended model
engineering size

## Task J — Commit boundaries

Plan separate future commits for:
- Post shadow
- Post cutover
- DM shadow
- DM cutover
- Launcher Seat admin UI
- E2E/cutover evidence

Do not commit now.

## Task K — Delete-later / keep

List components that can only be deleted after successful cutover and caller proof.

Likely candidates:
- Post duplicate commercial final-decision code
- DM duplicate entitlement final-decision code
- X-Tenant-Key fallback if caller-free
- formal-environment Seat skip paths
- obsolete legacy-only tests

Also list permanent KEEP items.

## Required final report

A. PREFLIGHT
Repo | Branch | HEAD | Dirty | Notes

B. LIVE POST CONFIG EVIDENCE
LOCAL_COMMERCIAL_GATE
LOCAL_AUTO_TENANT_BOOTSTRAP
LOCAL_SEAT_POLICY
LOCAL_ALLOW_TENANT_KEY
STAGING equivalents
PRODUCTION equivalents
Use UNKNOWN when not proven.

C. FINAL ACCESS CONTRACT
Post and DM.

D. POST PLAN

E. DM PLAN

F. LAUNCHER / WORKBENCH PLAN

G. SHADOW MATRIX

H. CUTOVER GATES

I. ROLLBACK PLAN

J. WORKSTREAMS
Task | Repo | Scope | Dependencies | Parallel-safe | Recommended model | Engineering size

K. COMMIT PLAN

L. DELETE-LATER / KEEP MATRIX

M. SAFETY

SOURCE_FILES_MODIFIED = NO
SQL_MODIFIED = NO
CONFIG_MODIFIED = NO
MIGRATIONS_APPLIED = NO
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
COMMIT_CREATED = NO
PUSHED = NO
DEPLOYED = NO

N. FINAL RESULT

If complete:

RESULT = PASS
PHASE_3_INTEGRATION_PLAN = PASS
POST_PLAN_READY = YES
DM_PLAN_READY = YES
LAUNCHER_PLAN_READY = YES
READY_FOR_PHASE_4_EXECUTION = YES

If live evidence is insufficient:

RESULT = PARTIAL_PASS
PHASE_3_INTEGRATION_PLAN = PASS
READY_FOR_PHASE_4_EXECUTION = NO
BLOCKERS = exact factual blockers

## Core rule

Plan only.

One Product Seat authority.
No silent formal-environment bypass.
Do not confuse commercial customer membership with product tenant mapping.
No production behavior change until shadow evidence and Staging gates pass.

Finish the plan and stop.
