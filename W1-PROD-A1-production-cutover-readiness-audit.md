# W1-PROD-A1 — Post / DM / ORC Production Cutover Readonly Readiness Audit

ROLE: FIRST COMMERCIAL WAVE PRODUCTION READINESS AUDIT OWNER

MODE: READ-ONLY / NO DEPLOY / NO MUTATION

WORKSPACE:
F:\00-Ticenpi-SaaS

PRODUCTS:
- Post
- DM
- ORC
- Letter as ORC dependency
- Launcher / Workbench
- Central Commercial Core

GOAL:
Prepare the first commercial wave for Production without deploying anything.

Determine exactly what is already present in Production and what must happen before Post, DM and ORC can be sold commercially.

Do NOT deploy.
Do NOT apply migrations.
Do NOT change Production config.
Do NOT create Production users/customers/Seats.
Do NOT modify files.

## 1. CENTRAL COMMERCIAL CORE PRODUCTION

Read-only verify Production state for:
- customers
- customer_members
- entitlements
- product_seat_assignments
- resolve_effective_entitlement_core
- effective_entitlements
- effective_entitlement_details
- product_seat_status
- assign_product_seat
- release_product_seat
- admin_list_product_seats

Report:
PRODUCTION_CENTRAL_CORE_STATE =
CURRENT / PARTIAL / ABSENT / UNKNOWN

Do not apply missing migrations.

## 2. POST PRODUCTION

Read-only capture:
- active Production release
- app source commit
- image digest
- deploy config identity
- Supabase project
- auth/commercial flags
- Seat policy
- tenant-key setting
- whether Seat module exists
- health
- rollback release

Compare against current successful Post Staging authoritative architecture.

List exact delta required for Production cutover.

## 3. DM PRODUCTION

Read-only capture:
- active release
- image digest
- auth config
- commercial gate
- current entitlement/Seat behavior
- RLS state
- local bypass safety
- health
- rollback release

Do not assume DM Staging authoritative is complete yet.

List Production deltas only.

## 4. ORC / LETTER PRODUCTION

Read-only capture:
- ORC release
- Letter release
- service topology
- auth mode
- tenant/data boundary
- health
- rollback identities
- whether ORC/Letter deploy independently

Do not decide final Seat placement until ORC-A1 architecture audit provides evidence.
If not available, mark dependency.

## 5. LAUNCHER PRODUCTION

Read-only verify:
- installed/stable Launcher environment target
- Production Supabase project
- current commercial contract
- current Seat Workbench capability
- current customer/member/entitlement admin capability
- Studio local handoff environment behavior

Do not switch Launcher environments.

## 6. PRODUCTION MIGRATION PLAN

Without executing, determine the exact ordered migrations/config changes needed for Central Commercial/Seat Production.

For each:
- source migration file
- checksum
- dependency
- rollback path
- backup prerequisite
- whether additive/destructive

Do not write or apply SQL.

## 7. BACKUP / ROLLBACK CONTRACT

For each product and Central Core identify:
- previous release
- DB backup/snapshot mechanism
- service rollback command/path
- config rollback
- health verification
- data migration rollback constraints

Return:
ROLLBACK_READY_POST
ROLLBACK_READY_DM
ROLLBACK_READY_ORC
ROLLBACK_READY_CENTRAL_CORE

## 8. PRODUCTION CUTOVER ORDER

Design the safest first-wave order.

Default candidate order:
1. Central Commercial Core Production migration/verification
2. Launcher Workbench Production readiness
3. Post Production cutover
4. DM Production cutover
5. ORC Production cutover
6. customer onboarding live smoke

But change the order if actual dependencies prove otherwise.

No deployments.

## 9. COMMERCIAL LAUNCH GATES

Define exact PASS gates for:
- Post
- DM
- ORC

Each must include:
- Production Central Seat authoritative
- assigned allow
- unassigned deny
- admin behavior
- invalid JWT
- product data boundary
- health
- rollback readiness
- no secret leakage
- customer onboarding path available

## 10. FINAL REPORT

A. CENTRAL CORE PROD STATE
B. POST PROD STATE / DELTA
C. DM PROD STATE / DELTA
D. ORC/LETTER PROD STATE / DELTA
E. LAUNCHER PROD STATE / DELTA
F. MIGRATION PLAN
G. BACKUP / ROLLBACK MATRIX
H. CUTOVER ORDER
I. COMMERCIAL LAUNCH GATES
J. HARD BLOCKERS
K. SAFETY

PRODUCTION_MUTATED = NO
STAGING_MUTATED = NO
FILES_MODIFIED = NO
DEPLOYED = NO

L. FINAL RESULT

RESULT = PASS/PARTIAL_PASS/BLOCKED
PRODUCTION_READINESS_MAPPED = YES/NO
READY_TO_DRAFT_PRODUCT_CUTOVER_TASKS = YES/NO
HARD_BLOCKERS = exact list

Finish audit and stop.
