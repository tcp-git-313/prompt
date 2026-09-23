# P2 — Legacy Seat Read-Only Audit

## ROLE

You are the LEGACY SEAT AUDIT OWNER.

This is Phase 2 only.

Your job is to inspect the existing Seat / commercial-access implementations in Post, DM, and Launcher, identify every legacy source of truth and caller, and produce an evidence-backed migration map toward the already-established Central Commercial Core.

This phase is READ-ONLY.

Do not modify source code, SQL, migrations, tests, documentation, configuration, CI, Docker, Staging, Production, or Git history.

Do not implement the new Central Seat integration in this phase.

## 0. REPOSITORIES

Audit targets:

~~~
POST
F:\00-Ticenpi-SaaS\TicenpiPost

DM
F:\00-Ticenpi-SaaS\TicenpiDM

LAUNCHER
F:\00-Ticenpi-SaaS\Ticenpi-Launcher
~~~

Canonical reference only:

~~~
PLATFORM / COMMERCIAL CORE
F:\00-Ticenpi-SaaS\ticenpi-platform
~~~

ticenpi-platform is NOT an audit target for modification. It is the canonical reference for the new shared commercial contract.

## 1. VERIFIED BASELINE — DO NOT REDESIGN

Phase 1 and Phase 1.5 are complete.

Treat the following as established facts unless local evidence proves the working tree has drifted.

~~~ini
PHASE_1 = PASS
PHASE_1_5 = PASS

COMMERCIAL_CORE_CANONICAL_OWNER = ticenpi-platform
COMMERCIAL_CORE_SCHEMA_OWNER = ticenpi-platform
COMMERCIAL_CORE_MIGRATION_OWNER = ticenpi-platform
CENTRAL_SEAT_OWNER = ticenpi-platform
COMMERCIAL_CORE_REPRO_OWNER = ticenpi-platform

RUNTIME_SOURCE_OF_TRUTH = Central Supabase

LAUNCHER_CANONICAL_MIRROR_REMOVED = YES
~~~

Canonical Commercial Core domains:

~~~
customers
memberships
subscriptions
entitlements
seats
audit
~~~

Canonical reference location:

~~~
F:\00-Ticenpi-SaaS\ticenpi-platform\docs\commercial-core
F:\00-Ticenpi-SaaS\ticenpi-platform\supabase\migrations
F:\00-Ticenpi-SaaS\ticenpi-platform\supabase\repro
~~~

Central Seat contract:

~~~
F:\00-Ticenpi-SaaS\ticenpi-platform\docs\commercial-core\contracts\CENTRAL-SEAT-CONTRACT.md
~~~

Known Central Seat migration:

~~~
20260923_000600_central_product_seats.sql
~~~

Known SHA-256:

~~~
32EED072E422F576C9F5612BDD2E6E33B2F265A4D777B9BAC44AC2BE9A7B62CB
~~~

Current canonical Seat objects include:

~~~
public.product_seat_assignments

resolve_effective_entitlement_core(uuid)

effective_entitlements(uuid)
→ TABLE(product_code text, status text)

effective_entitlement_details(uuid)
→ product_code,status,seat_limit,entitlement_id

assign_product_seat(
  p_customer_id uuid,
  p_product_code text,
  p_user_id uuid
)

release_product_seat(
  p_customer_id uuid,
  p_product_code text,
  p_user_id uuid
)

product_seat_status(p_product_code text)
~~~

Canonical rules already verified:

~~~
- effective entitlement selection is shared
- seat_limit comes from the same selected entitlement row
- individual entitlement does not require manual Seat assignment
- business access uses central Seat assignment
- product isolation is required
- Post Seat must not grant DM Seat
- downgrade below assigned count is blocked
- assign/release writes are platform-admin protected
- product_seat_status derives identity from auth.uid()
- Central Supabase is runtime source of truth
~~~

Do NOT redesign these rules during this audit.

## 2. PRIMARY QUESTION

Determine exactly what happens today in each product before Central Seat cutover.

For each of Post, DM, and Launcher answer:

1. Where does the current user identity come from?
2. How is customer / tenant / organization context resolved?
3. How is membership checked?
4. How is entitlement checked?
5. Is seat_limit calculated locally?
6. Is Seat assignment stored locally?
7. Is there a legacy Seat table?
8. Is there a legacy Seat RPC / API / service?
9. Is there a local access Guard?
10. Which component makes the final ALLOW / DENY decision?
11. Does the frontend also perform access decisions?
12. Does an admin/workbench UI manage Seat assignments?
13. Are there triggers/jobs/hooks that change Seat state?
14. Are there tests encoding legacy behavior?
15. Are there bypasses for admin/dev/local/service-role/test identities?
16. Does any code directly read/write commercial tables?
17. Does any code depend on old Launcher commercial paths?
18. Which legacy behavior would conflict with Central Seat if both remain active?

The output must be based on evidence from actual files and call chains.

Do not infer from filenames alone.

## 3. ABSOLUTE READ-ONLY RULE

This phase must not change anything.

Forbidden:

~~~
git commit
git push
git tag
git checkout
git switch
git reset
git restore
git clean
git stash
git rebase

file edits
file creation
file deletion
formatting
auto-fix

SQL write
DDL
DML
migration apply
Supabase push
RPC mutation
Staging mutation
Production mutation

Docker rebuild
deploy
service restart
CI trigger
release
~~~

Do not "fix while inspecting".

Do not create audit documents inside the repositories.

Return the audit only in your final response.

If a tool would modify repository state merely by running, do not run it.

## 4. PREFLIGHT

For all four repositories, record:

~~~
path
branch
HEAD
git status --short
git diff --stat
~~~

Repositories:

~~~
TicenpiPost
TicenpiDM
Ticenpi-Launcher
ticenpi-platform
~~~

Dirty working trees are evidence.

Do not clean or alter them.

Identify which changes were already present before this audit.

If the repo path is missing, report it as a blocker for that repository only; do not invent an alternate path unless repository evidence identifies it.

## 5. CANONICAL CONTRACT FIRST

Before auditing legacy Seat implementations, read the canonical Platform contract and domain documentation.

At minimum inspect:

~~~
docs/commercial-core/README.md
docs/commercial-core/customers.md
docs/commercial-core/memberships.md
docs/commercial-core/subscriptions.md
docs/commercial-core/entitlements.md
docs/commercial-core/seats.md
docs/commercial-core/audit.md
docs/commercial-core/contracts/CENTRAL-SEAT-CONTRACT.md
~~~

Then inspect the relevant canonical migrations.

Build a concise TARGET CONTRACT containing:

~~~
identity source
customer source
membership source
entitlement source
seat assignment source
seat_limit source
allow/deny RPC
admin assign/release mechanism
audit source
~~~

This target contract is the comparison baseline.

Do not modify it.

## 6. SEARCH SCOPE

Search code, SQL, tests, scripts, config, and docs for both obvious and indirect Seat/access behavior.

At minimum search variants of:

~~~
seat
seats
seat_limit
seat_count
assignment
assigned
licensed
license
member
membership
customer_member
organization
tenant
customer
subscription
entitlement
commercial
access
allow
deny
guard
permission
product_code
is_platform_admin
service_role
platform_admin
my_commercial_context
effective_entitlements
effective_entitlement_details
product_seat_status
assign_product_seat
release_product_seat
~~~

Also inspect:

~~~
backend dependencies
frontend stores/composables/services
API clients
Supabase clients
FastAPI dependencies/middleware
route guards
Electron/Launcher bridge code
admin/workbench screens
SQL migrations
schema snapshots
tests
fixtures
cron/background jobs
triggers
Docker/env wiring
CI smoke tests
local development bypasses
~~~

Do not rely on grep hits alone.

Trace callers until the final user-facing behavior is understood.

## 7. POST AUDIT

For F:\00-Ticenpi-SaaS\TicenpiPost construct the actual current access chain.

Example format only:

~~~
login
↓
JWT/session
↓
customer/tenant resolution
↓
membership
↓
entitlement
↓
legacy Seat check
↓
backend Guard
↓
protected route
~~~

Do not assume this exact chain.

Prove the real chain.

Identify all Post-specific sources of:

~~~
Seat capacity
Seat assignment
user-to-product access
tenant/customer context
commercial entitlement
ALLOW/DENY
~~~

Explicitly determine whether Post currently has:

~~~ini
POST_LEGACY_SEAT_TABLE =
POST_LEGACY_SEAT_RPC =
POST_LEGACY_SEAT_API =
POST_LEGACY_SEAT_GUARD =
POST_LEGACY_SEAT_FRONTEND_LOGIC =
POST_LEGACY_SEAT_ADMIN_UI =
POST_LEGACY_SEAT_TESTS =
~~~

For each: YES / NO / AMBIGUOUS, with file evidence.

## 8. DM AUDIT

For F:\00-Ticenpi-SaaS\TicenpiDM specifically verify the known commercial path around:

~~~
get_current_user
my_commercial_context()
EntitledUserDep
protected routes
~~~

Do not assume the remembered architecture is still correct.

Trace the current code.

Determine whether EntitledUserDep or adjacent dependencies perform:

~~~
entitlement validation
membership validation
Seat validation
seat_limit calculation
legacy assignment lookup
platform_admin bypass
service_role bypass
local/dev bypass
test bypass
~~~

Explicitly determine:

~~~ini
DM_LEGACY_SEAT_TABLE =
DM_LEGACY_SEAT_RPC =
DM_LEGACY_SEAT_API =
DM_LEGACY_SEAT_GUARD =
DM_LEGACY_SEAT_FRONTEND_LOGIC =
DM_LEGACY_SEAT_ADMIN_UI =
DM_LEGACY_SEAT_TESTS =
~~~

For each: YES / NO / AMBIGUOUS, with evidence.

## 9. LAUNCHER AUDIT

For F:\00-Ticenpi-SaaS\Ticenpi-Launcher remember:

~~~
Launcher is no longer the canonical Commercial Core owner.
~~~

Audit Launcher only as:

~~~
consumer
admin/workbench surface
legacy product/access implementation
~~~

Trace:

1. user/customer creation
2. membership management
3. subscription/entitlement management
4. product access display
5. Seat assignment display
6. Seat assignment writes
7. Seat release writes
8. capacity display
9. admin authorization
10. legacy commercial APIs/RPCs still referenced by UI

Separate company membership from product Seat assignment.

Explicitly determine:

~~~ini
LAUNCHER_LEGACY_SEAT_TABLE =
LAUNCHER_LEGACY_SEAT_RPC =
LAUNCHER_LEGACY_SEAT_API =
LAUNCHER_LEGACY_SEAT_GUARD =
LAUNCHER_LEGACY_SEAT_FRONTEND_LOGIC =
LAUNCHER_LEGACY_SEAT_ADMIN_UI =
LAUNCHER_LEGACY_SEAT_TESTS =
~~~

For each: YES / NO / AMBIGUOUS, with evidence.

## 10. DATA-OBJECT CLASSIFICATION

Every discovered DB object must be classified into exactly one primary category:

~~~
IDENTITY
CUSTOMER
MEMBERSHIP
SUBSCRIPTION
ENTITLEMENT
SEAT_ASSIGNMENT
AUDIT
PRODUCT_RUNTIME
TEST_ONLY
UNKNOWN
~~~

Do not delete conceptually valid membership/subscription/entitlement objects merely because they participate in access control.

Seat cleanup is not commercial-schema cleanup.

## 11. LEGACY COMPONENT CLASSIFICATION

Every discovered relevant component must receive one migration classification:

~~~
KEEP
REPLACE_WITH_COMMERCIAL_CORE
DEPRECATE
DELETE_LATER
AMBIGUOUS
~~~

KEEP = still valid under the target architecture.

REPLACE_WITH_COMMERCIAL_CORE = current responsibility should be served by the canonical Commercial Core.

DEPRECATE = should stop being authoritative at cutover but may remain temporarily for shadow/rollback evidence.

DELETE_LATER = deletion candidate only after cutover, caller scan, and rollback window.

AMBIGUOUS = insufficient evidence.

Do not force a classification.

## 12. FINAL ALLOW / DENY OWNERS

For each product identify the exact current final authorization decision point.

Required:

~~~ini
POST_FINAL_ALLOW_DENY_OWNER =
DM_FINAL_ALLOW_DENY_OWNER =
LAUNCHER_FINAL_ALLOW_DENY_OWNER =
~~~

Give:

~~~
file
symbol/function
caller
input
output/error behavior
~~~

If there are multiple gates, show their order.

This is critical for Phase 3.

## 13. DUPLICATE AUTHORITY / CONFLICT ANALYSIS

Identify all situations where Central Seat integration could create double authority.

Examples:

~~~
legacy Seat AND Central Seat
legacy Seat OR Central Seat
frontend allow while backend denies
Post-local seat_limit vs canonical seat_limit
DM-local assignment vs canonical product_seat_assignments
Launcher UI writing legacy table while products read Central Seat
admin bypass behavior differs between old and new systems
membership source differs
tenant/customer identity differs
~~~

For each conflict provide:

~~~
LEGACY_SOURCE
CENTRAL_SOURCE
CURRENT_CALLER
RISK
CUTOVER_REQUIREMENT
~~~

Do not implement the fix.

## 14. SPECIAL ACCESS / BYPASS AUDIT

Search for and document all exceptional access paths:

~~~
platform_admin
admin
superuser
service_role
local development
offline mode
test identities
mock auth
X-Tenant-Key
feature flags
environment bypass
staging-only bypass
debug bypass
~~~

For each:

~~~
product
location
condition
what it bypasses
whether it would bypass Central Seat
classification
~~~

Do not remove bypasses in this phase.

## 15. FRONTEND VS BACKEND

For every product determine:

~~~ini
FRONTEND_ACCESS_DISPLAY =
BACKEND_ACCESS_ENFORCEMENT =
~~~

Explicitly flag any case where the frontend appears to be the only enforcement.

Central Seat cutover must ultimately be server-side enforced, but this phase must only report the current state.

## 16. WORKBENCH / ADMIN TABLE IMPACT

For Launcher Workbench/Admin, produce a current-to-target map.

Required target concepts:

~~~
Company users
→ customer_members

Product purchased / effective plan
→ entitlement / effective_entitlement_details

Seat capacity
→ seat_limit from selected effective entitlement row

Who owns a product Seat
→ product_seat_assignments

Seat assign
→ assign_product_seat()

Seat release
→ release_product_seat()
~~~

Compare this target with the actual current Workbench data sources.

Classify each current UI data source:

~~~
KEEP
REWIRE
REMOVE_LATER
AMBIGUOUS
~~~

Do not modify the Workbench.

## 17. TEST CONTRACT AUDIT

Locate tests that encode existing commercial/Seat behavior.

For every relevant test group report:

~~~
repo
test file
behavior tested
legacy dependency
future value
classification
~~~

Classify tests:

~~~
KEEP_AS_REGRESSION
ADAPT_DURING_CUTOVER
LEGACY_ONLY_DELETE_LATER
AMBIGUOUS
~~~

Do not modify tests.

## 18. SHADOW-CUTOVER DESIGN — PLAN ONLY

Based on the evidence, propose the safest future verification sequence.

Do not implement it.

Target model:

~~~
LEGACY RESULT = allow/deny
CENTRAL RESULT = allow/deny

legacy remains authoritative temporarily
central result is shadow-only

compare
log mismatch
do not change customer behavior
~~~

Required cases:

~~~
individual entitlement
business + assigned Seat
business + no Seat
Seat full
expired entitlement
suspended membership
removed membership
Post-only Seat
DM-only Seat
platform_admin
repeated assignment
release/reassign
downgrade blocked state
~~~

For Post and DM identify the smallest location where shadow comparison can later be inserted without duplicating logic across every route.

PLAN ONLY.

## 19. DELETION GATES — PLAN ONLY

No legacy component may later be deleted until all of these are proven:

~~~
Central result is authoritative
Staging E2E passes
legacy vs central mismatch is understood
no runtime caller remains
no frontend caller remains
no trigger/job caller remains
no CI/test required caller remains
rollback point exists
production cutover is separately approved
~~~

List which discovered legacy components would need these gates.

Do not delete anything now.

## 20. SOURCE-OF-TRUTH PRIORITY

Documentation may be stale.

Priority order:

~~~
1. Current executable source
2. Current SQL/migrations/schema definitions
3. Current tests
4. Current runtime/config wiring
5. Current docs
6. Historical docs
~~~

If docs conflict with source, report the conflict.

## 21. EVIDENCE STANDARD

Every material claim must include evidence.

Preferred format:

~~~
repo
relative file path
symbol/function/table
short factual finding
~~~

Do not paste large source blocks.

Summarize.

## 22. HARD STOP CONDITIONS

Do not resolve these by editing.

Report them:

A. Unrelated dirty work prevents reliable attribution.
B. A final ALLOW/DENY owner cannot be traced.
C. Legacy Seat logic depends on unavailable remote-only code/config.
D. Canonical Commercial Core materially differs from the verified baseline.
E. Necessary inspection would require remote mutation.
F. Actual repo path cannot be proven.
G. Two active implementations disagree and authority cannot be determined.

A blocker in one repo does not excuse skipping safe read-only work in the others.

## 23. REQUIRED FINAL REPORT

Return exactly these sections.

### A. PREFLIGHT

Table:

~~~
Repo | Branch | HEAD | Dirty | Notes
~~~

Include all four repos.

### B. TARGET CENTRAL CONTRACT

~~~
Identity:
Customer:
Membership:
Subscription:
Entitlement:
Seat assignment:
Seat limit:
Allow/Deny:
Assign:
Release:
Audit:
Canonical owner:
Runtime source:
~~~

### C. POST CURRENT ACCESS CHAIN

Show the real chain.

Then:

~~~ini
POST_LEGACY_SEAT_TABLE =
POST_LEGACY_SEAT_RPC =
POST_LEGACY_SEAT_API =
POST_LEGACY_SEAT_GUARD =
POST_LEGACY_SEAT_FRONTEND_LOGIC =
POST_LEGACY_SEAT_ADMIN_UI =
POST_LEGACY_SEAT_TESTS =
POST_FINAL_ALLOW_DENY_OWNER =
~~~

### D. DM CURRENT ACCESS CHAIN

Show the real chain.

Then:

~~~ini
DM_LEGACY_SEAT_TABLE =
DM_LEGACY_SEAT_RPC =
DM_LEGACY_SEAT_API =
DM_LEGACY_SEAT_GUARD =
DM_LEGACY_SEAT_FRONTEND_LOGIC =
DM_LEGACY_SEAT_ADMIN_UI =
DM_LEGACY_SEAT_TESTS =
DM_FINAL_ALLOW_DENY_OWNER =
~~~

### E. LAUNCHER CURRENT ACCESS / ADMIN CHAIN

Show customer, membership, subscription, entitlement, Seat/admin/workbench chain.

Then:

~~~ini
LAUNCHER_LEGACY_SEAT_TABLE =
LAUNCHER_LEGACY_SEAT_RPC =
LAUNCHER_LEGACY_SEAT_API =
LAUNCHER_LEGACY_SEAT_GUARD =
LAUNCHER_LEGACY_SEAT_FRONTEND_LOGIC =
LAUNCHER_LEGACY_SEAT_ADMIN_UI =
LAUNCHER_LEGACY_SEAT_TESTS =
LAUNCHER_FINAL_ALLOW_DENY_OWNER =
~~~

### F. LEGACY COMPONENT MATRIX

Table:

~~~
Repo
Component
Type
Current responsibility
Current caller
Current source of truth
Final classification
Evidence
~~~

Classification:

~~~
KEEP
REPLACE_WITH_COMMERCIAL_CORE
DEPRECATE
DELETE_LATER
AMBIGUOUS
~~~

### G. DB OBJECT MATRIX

Table:

~~~
Object
Repo/reference
Category
Current writer
Current reader
Canonical target
Conflict risk
~~~

### H. BYPASS MATRIX

Table:

~~~
Repo
Bypass
Condition
What it bypasses
Central Seat impact
~~~

### I. WORKBENCH IMPACT

Table:

~~~
UI area
Current data source
Current write target
Target source
Action later
~~~

Actions:

~~~
KEEP
REWIRE
REMOVE_LATER
AMBIGUOUS
~~~

### J. CONFLICTS

For every conflict:

~~~
LEGACY_SOURCE
CENTRAL_SOURCE
RISK
REQUIRED_CUTOVER_ACTION
~~~

### K. TEST CONTRACTS

Table:

~~~
Repo
Test
Behavior
Legacy dependency
Classification
~~~

### L. SHADOW PLAN

For Post and DM separately:

~~~
best insertion point
legacy authoritative result source
central shadow result source
comparison/logging location
cases to test
~~~

No implementation.

### M. DELETION CANDIDATES

Separate:

~~~
DELETE AFTER CUTOVER
KEEP PERMANENTLY
AMBIGUOUS / NEEDS MORE EVIDENCE
~~~

### N. MODIFICATION SAFETY

Must state:

~~~ini
SOURCE_FILES_MODIFIED = NO
SQL_MODIFIED = NO
MIGRATIONS_APPLIED = NO
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
COMMIT_CREATED = NO
PUSHED = NO
DEPLOYED = NO
~~~

### O. FINAL RESULT

If enough evidence exists:

~~~ini
RESULT = PASS
PHASE_2_LEGACY_SEAT_AUDIT = PASS
LEGACY_SEAT_MAP_COMPLETE = YES
READY_FOR_PHASE_3_INTEGRATION_PLAN = YES
~~~

If useful but incomplete:

~~~ini
RESULT = PARTIAL_PASS
PHASE_2_LEGACY_SEAT_AUDIT = PARTIAL
LEGACY_SEAT_MAP_COMPLETE = NO
READY_FOR_PHASE_3_INTEGRATION_PLAN = NO
BLOCKERS = <exact factual blockers>
~~~

If materially blocked:

~~~ini
RESULT = BLOCKED
PHASE_2_LEGACY_SEAT_AUDIT = BLOCKED
BLOCKERS = <exact factual blockers>
~~~

## 24. CORE RULE

This phase answers:

What old Seat/access logic exists, who calls it, what must stay, what must later be replaced, and where can Central Seat be introduced without double authority?

It does NOT answer by changing the system.

Do not edit.
Do not clean.
Do not integrate.
Do not delete.
Do not deploy.

Finish the evidence-backed audit and stop.
