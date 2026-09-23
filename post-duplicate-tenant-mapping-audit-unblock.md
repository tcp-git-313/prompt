# POST — DUPLICATE TENANT MAPPING AUDIT + ACCEPTANCE UNBLOCK

## ROLE
POST TENANT CONSISTENCY OWNER

## MODEL
LUNA High

## MODE
STRICT EXECUTOR MODE — PLAN IS FROZEN

Project:
`F:\00-Ticenpi-SaaS\TicenpiPost`

Branch:
`feat/post-tenant-provisioning`

Current state:

- Bootstrap route idempotency contract test = PASS
- Live browser bootstrap not directly proven
- Acceptance DB has TWO active tenant_members rows matching the displayed account email
- Tenant consistency therefore cannot be proven
- 9458 detects Extension 0.4.8
- 19418 does not detect Extension injection
- Studio currently offline
- Git worktree clean
- Four commits ahead of origin
- No push / CI / Staging yet

Mission:

ONLY resolve the duplicate active User→Post Tenant mapping safely enough to unblock 19418 acceptance.

Do not work on:
- Central Seat
- Production
- Staging deploy
- Launcher implementation
- Extension source changes
- Facebook execution egress

You are Executor, not Planner.

==================================================
1. ABSOLUTE DATA SAFETY
==================================================

Do NOT:

- DELETE tenant_members
- UPDATE tenant_id
- merge tenants
- move account rows
- move task rows
- change account ownership
- truncate data
- reset DB
- rebuild volume
- use destructive SQL
- assume which tenant is canonical from email alone

Until canonical ownership is proven.

If evidence is insufficient:
STOP.

==================================================
2. READ FROZEN TENANT RULES FIRST
==================================================

Read:

`C:\Users\Tu\.claude\projects\F--\memory\project_post_phase2_tenant_seat.md`

`C:\Users\Tu\.claude\projects\F--\memory\project_post_two_accounts_two_profiles.md`

`F:\00-Ticenpi-SaaS\TicenpiPost\_charter\adr\ADR-002-central-commercial-post-tenant-seat.md`

`F:\00-Ticenpi-SaaS\TicenpiPost\_charter\adr\ADR-001-tenant-identity-entry-point.md`

Frozen target:

1 User = 1 Post Tenant
1 Tenant = 1 User

Tenant bootstrap must be idempotent.

Do not redesign this.

==================================================
3. IDENTIFY THE CURRENT AUTH USER
==================================================

Do not use email as the final identity key.

Find the exact current Supabase Auth user_id for the 19418 logged-in account using existing safe sources only, for example:

- profiles
- commercial context
- auth/session logs
- central membership records
- existing DB mappings

Record:

CURRENT_EMAIL =
CURRENT_USER_ID =

If exact user_id cannot be proven:
STOP.

==================================================
4. LIST ALL TENANT MEMBERSHIP ROWS FOR THAT USER
==================================================

Read-only query the Acceptance DB.

For the exact CURRENT_USER_ID, list all matching tenant_members rows.

Record for each:

- tenant_id
- user_id
- active/status if present
- created_at
- updated_at
- source/provenance columns if any

Do not collapse by email.

Expected target:
exactly one active mapping for CURRENT_USER_ID.

==================================================
5. DETERMINE PROVENANCE OF BOTH TENANTS
==================================================

For each tenant_id, determine whether it came from:

- old manual/test tenant
- pre-Phase2 mapping
- deterministic Phase2 bootstrap
- another historical migration
- accidental duplicate insert

Check existing deterministic tenant rule.

If current implementation uses:

`u- + sha256(user_id)[:24]`

compute the expected deterministic tenant_id for CURRENT_USER_ID.

Record:

EXPECTED_PHASE2_TENANT_ID =
TENANT_A =
TENANT_A_ORIGIN =
TENANT_B =
TENANT_B_ORIGIN =

Do NOT assume deterministic tenant automatically wins if historical production data ownership would be lost.

==================================================
6. INVENTORY DATA OWNERSHIP BY TENANT
==================================================

Read-only inventory all Post domain data keyed by each duplicate tenant.

At minimum inspect whichever tables actually exist for:

- accounts
- tasks
- scheduled jobs
- drafts
- groups
- cookie/device binding metadata
- bind tokens
- device tokens
- budget_usage_v2
- any other tenant_id scoped tables

For each tenant, record counts and important identifiers.

Do not expose secrets/cookies/token bodies.
Only counts / IDs / metadata needed for ownership.

Goal:
understand whether one tenant is empty and the other owns real user data.

==================================================
7. CHECK LIVE SESSION / RESOLUTION PATH
==================================================

Read the current tenant resolver / principal resolution code.

Determine which tenant the current authenticated user would resolve to if multiple active rows exist.

If query semantics are nondeterministic:
record that as a bug.

Do not patch yet unless this Prompt explicitly reaches an approved remediation case.

Record:

CURRENT_RESOLVER_BEHAVIOR =
MULTIPLE_ROW_BEHAVIOR =
CURRENT_SESSION_TENANT_CAN_BE_PROVEN = YES/NO

==================================================
8. CLASSIFY INTO ONE OF THREE CASES
==================================================

CASE A — SAFE STALE EMPTY DUPLICATE

Conditions:

- exact user_id proven
- one tenant is canonical by frozen Phase2 rule
- other tenant is historical/stale
- stale tenant has NO meaningful domain data
- no active account/task/device ownership depends on stale tenant
- removing stale mapping does not orphan data

Then:
prepare MINIMAL REMEDIATION only.

Do not execute until Section 9 checks pass.


CASE B — BOTH TENANTS CONTAIN USER DATA

Do NOT merge automatically.

STOP and report:

DATA_MIGRATION_REQUIRED = YES

Include:
- which tables contain data
- counts
- canonical candidate
- minimum migration plan

Do not mutate DB.


CASE C — CANONICAL TENANT CANNOT BE PROVEN

STOP.

Do not mutate DB.

==================================================
9. SAFE REMEDIATION RULE FOR CASE A ONLY
==================================================

Only if CASE A is fully proven:

Before mutation:

1. export/read-only snapshot of the two tenant_members rows
2. record DB counts for affected tenant-scoped tables
3. verify stale tenant has no meaningful owned data
4. verify exact rollback SQL

Allowed mutation:

- deactivate/remove ONLY the stale tenant_members mapping
- preserve canonical tenant row
- do not delete tenant-scoped data
- do not touch other users

Use existing migration/admin-safe mechanism if present.

Do not introduce a general-purpose destructive script.

After mutation verify:

- CURRENT_USER_ID has exactly one active tenant mapping
- canonical tenant_id unchanged
- domain data counts unchanged
- account list still loads
- reload resolves same tenant
- bootstrap idempotency test remains PASS

If any post-check fails:
rollback immediately using recorded row snapshot.

==================================================
10. DO NOT FIX EXTENSION OR STUDIO YET
==================================================

Extension injection and Studio online are separate browser/user-action gates.

Do NOT change extension source in this task.

Do NOT change Launcher code in this task.

After tenant consistency becomes PASS, report the remaining manual actions only.

==================================================
11. FINAL ACCEPTANCE EVIDENCE AFTER TENANT FIX
==================================================

Once exactly one active mapping exists for CURRENT_USER_ID:

prove:

AUTH_USER_ID =
TENANT_BEFORE_RELOAD =

reload 19418

TENANT_AFTER_RELOAD =

run/verify bootstrap idempotency path

TENANT_AFTER_BOOTSTRAP =

PASS only if all are identical.

==================================================
12. GIT RULES
==================================================

Do not alter the existing four commits.

Do not:
- amend
- squash
- rebase
- force push

If no source change is required:
make no commit.

If a source defect is discovered that allows duplicate active mappings despite the existing DB invariant:
STOP and report it separately.
Do not silently patch architecture in this task.

==================================================
13. REPORT FORMAT
==================================================

PHASE = DUPLICATE TENANT MAPPING AUDIT
RESULT = PASS / HARD_STOP

CURRENT_EMAIL =
CURRENT_USER_ID =

EXPECTED_PHASE2_TENANT_ID =

TENANT_1 =
TENANT_1_ORIGIN =
TENANT_1_DATA_SUMMARY =

TENANT_2 =
TENANT_2_ORIGIN =
TENANT_2_DATA_SUMMARY =

CURRENT_RESOLVER_BEHAVIOR =
MULTIPLE_ROW_BEHAVIOR =

CLASSIFICATION = CASE_A / CASE_B / CASE_C

REMEDIATION_EXECUTED = YES/NO
ROLLBACK_EVIDENCE =
POST_FIX_ACTIVE_MAPPING_COUNT =

TENANT_BEFORE_RELOAD =
TENANT_AFTER_RELOAD =
TENANT_AFTER_BOOTSTRAP =
TENANT_ID_STABLE = PASS/FAIL

REMAINING_USER_ACTIONS =
- Launch existing Ticenpi Launcher/Studio
- Confirm Chrome Extension 0.4.8 site access/injection on localhost:19418

GIT_STATUS =
NEXT =

If RESULT = PASS:
return to the existing frozen POST RELEASE plan at the remaining Launcher/Extension browser gates.

Do not Push until those gates also PASS.
