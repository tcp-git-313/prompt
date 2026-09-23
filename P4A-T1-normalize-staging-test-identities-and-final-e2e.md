# P4A-T1 — Normalize Existing Staging Test Identities for Central Seat & Resume Final Post E2E

ROLE: POST CENTRAL SEAT STAGING TEST-DATA / FINAL E2E OWNER

MODE: STAGING ONLY / TEST DATA NORMALIZATION + FINAL E2E PREP

WORKSPACE:
F:\00-Ticenpi-SaaS

PRIMARY REPOS:
- F:\00-Ticenpi-SaaS\ticenpi-platform
- F:\00-Ticenpi-SaaS\TicenpiPost

GOAL:
Use EXISTING Staging test accounts, inspect their current commercial/access state, normalize them into the canonical Central Commercial + Central Seat + Post tenant model, and then resume the final Post Central Seat authoritative Staging E2E.

Do NOT create new permanent Auth users.
Do NOT touch Production.
Do NOT directly edit product_seat_assignments.
Do NOT bypass canonical RPCs.
Do NOT use service-role in browser/user flows.
Do NOT delete existing test users.

## CURRENT VERIFIED STATE

Central Seat Staging exists and is verified:
- resolve_effective_entitlement_core(uuid)
- effective_entitlements(uuid)
- effective_entitlement_details(uuid)
- product_seat_status(text)
- assign_product_seat(uuid,text,uuid)
- release_product_seat(uuid,text,uuid)
- admin_list_product_seats(uuid,text)

Post authoritative candidate:
- commit: 27e4efe
- CI: green
- image digest:
  sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0

Candidate is NOT yet deployed.

Current problem:
existing ordinary Staging login shows:
"尚未開通商用權限"

Therefore the existing test identity is authenticated, but its canonical commercial model is incomplete for the final Seat E2E.

## 1. DISCOVER EXISTING STAGING TEST IDENTITIES

Use existing secure/admin read paths only.

Do not ask the user to create accounts.

Inventory the EXISTING fixed Staging test identities already used by the project.

For each identity, inspect and report only safe metadata:

- label
- auth user exists YES/NO
- user_id
- platform role
- test_access if present
- profiles state if relevant
- customer_members rows
- customer id(s)
- customer membership status
- Post entitlement status
- Post entitlement seat_limit
- Central Post Seat assignment
- Post tenant_members mapping
- current expected access result

Do NOT print:
- passwords
- tokens
- JWTs
- refresh tokens
- Supabase keys

## 2. BUILD A CURRENT-STATE MATRIX

Produce:

Identity | Auth | platform_admin | Customer Member | Post Entitlement | Post Seat | Post Tenant Mapping | Current Expected Result

Classify each identity into:

ADMIN
ORDINARY_ASSIGNED_CANDIDATE
ORDINARY_UNASSIGNED_CANDIDATE
UNUSABLE_FOR_TEST
UNKNOWN

If fewer than two usable ordinary non-admin existing identities exist:
do NOT create a new account automatically.
Report the exact gap.

## 3. DEFINE THE FINAL TEST MODEL

Target final Staging test model:

### Identity A — Admin
- existing platform_admin
- Seat not required / exempt
- expected Post result = ALLOW

### Identity B — Ordinary Assigned
- existing non-admin Auth user
- active customer membership
- customer has effective Post entitlement
- Central Post Seat assigned
- Post tenant mapping valid
- expected Post result = ALLOW

### Identity C — Ordinary Unassigned
- existing non-admin Auth user
- active customer membership
- same test customer if safe, or another existing test customer
- customer has effective Post entitlement
- NO Central Post Seat assignment
- Post tenant mapping valid
- expected Post result = DENY after authoritative cutover

Do not use platform_admin as B or C.

## 4. PRESERVE ORIGINAL STATE

Before changing any Staging test data:

Record the exact original state for every identity/table/RPC-relevant object that may be changed.

At minimum:
- customer membership
- entitlement
- seat assignment
- Post tenant mapping

Use unique audit notes/markers only if existing admin tooling supports them safely.

All changes must be reversible.

## 5. NORMALIZE COMMERCIAL CORE FIRST

For Identity B and C:

Required:
- Auth user already exists
- active customer membership exists
- customer has effective Post entitlement

If missing, use EXISTING canonical platform-admin RPC/admin flows to create or update the missing commercial relationship.

Do not directly edit canonical tables if an existing admin RPC exists.

Before invoking any mutation:
- inspect the available admin RPCs/contracts
- choose the canonical supported operation

Do NOT redesign Commercial Core.

If no canonical supported write path exists for a required missing object:
STOP and report:
MISSING_CANONICAL_ADMIN_WRITE_PATH = <object>

Do not invent raw SQL mutations merely to make the test pass.

## 6. NORMALIZE CENTRAL SEAT

For Identity B:

Use canonical:
assign_product_seat(
  customer_id,
  "post",
  user_id
)

Then verify with:
admin_list_product_seats(customer_id, "post")

Required:
Identity B appears assigned.

For Identity C:

Use canonical:
release_product_seat(
  customer_id,
  "post",
  user_id
)

only if currently assigned.

Then verify:
Identity C does NOT appear assigned.

Do not directly write product_seat_assignments.

## 7. NORMALIZE POST TENANT MAPPING

For both B and C:

Verify valid Post tenant_members mapping exists.

Use the existing supported Post provisioning/admin mechanism.

Do not confuse:
customer_members
with
Post tenant_members

They are separate models.

Required:

Identity B:
Post tenant mapping = YES

Identity C:
Post tenant mapping = YES

This ensures the final ALLOW/DENY difference is caused by Central Seat, not by missing Post tenant mapping.

## 8. VERIFY PRE-CUTOVER STATE

Before deploying authoritative candidate:

For B:
- authenticated
- commercial membership valid
- Post entitlement valid
- Post Seat assigned
- Post tenant mapping valid

For C:
- authenticated
- commercial membership valid
- Post entitlement valid
- Post Seat NOT assigned
- Post tenant mapping valid

Required:

ASSIGNED_TEST_IDENTITY_READY = YES
UNASSIGNED_TEST_IDENTITY_READY = YES

If either is NO:
STOP before deploy.

## 9. DEPLOY THE ALREADY-GREEN AUTHORITATIVE CANDIDATE

Deploy the exact candidate:

commit = 27e4efe
image digest =
sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0

Use the approved Post Staging deploy chain.

Do NOT rebuild from arbitrary working tree.
Do NOT deploy another commit.
Do NOT touch Production.

Record:
- previous release
- new release
- deployed commit
- deployed digest
- audit result

## 10. FINAL STAGING RUNTIME FLAGS

After deploy, verify actual running values:

TICENPI_COMMERCIAL_GATE = 1
TICENPI_SEAT_POLICY = require
TICENPI_ALLOW_TENANT_KEY = 0

AUTO_TENANT_BOOTSTRAP may be 1 or 0 according to current provisioning needs, but Seat enforcement must be proven independent from it.

Verify:
SEAT_ENFORCEMENT_DECOUPLED_FROM_AUTO_BOOTSTRAP = YES

## 11. FINAL LIVE E2E

Use the existing Staging login flow.

### Case A — Admin
Login existing platform_admin.

Expected:
ALLOW

### Case B — Ordinary Assigned
Login Identity B.

Required:
- normal Supabase auth session
- commercial context valid
- Central product_seat_status("post") = ALLOW
- Post tenant mapping valid
- final HTTP/user access = ALLOW

### Case C — Ordinary Unassigned
Login Identity C.

Required:
- normal Supabase auth session
- commercial context valid
- Central product_seat_status("post") = DENY / not_assigned
- Post tenant mapping valid
- final result = 403 / no Post access

### Case D — Invalid JWT
Expected:
401

Do not use X-Tenant-Key.

## 12. PROVE THE DIFFERENCE IS CENTRAL SEAT

For B and C, prove:

same commercial eligibility class
+
valid Post tenant mapping
+
different Central Seat assignment

causes:

B = ALLOW
C = DENY

Required:

CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES

This is the key final acceptance result.

## 13. OPTIONAL PRODUCT ISOLATION

If an existing safe identity has DM Seat but no Post Seat:

verify:
Post = DENY

Otherwise:
NOT_REACHED

Do not create extra state just for this optional case.

## 14. LOG / SECRET SAFETY

Confirm no live logs expose:
- bearer
- JWT
- email
- password
- service-role
- Supabase key
- cookie
- refresh token

AUTH_LOG_SECRET_LEAK = NO

## 15. HEALTH / SMOKE

Verify:
- /api/health = 200
- external Post Staging = 200
- container ready
- authenticated assigned path works
- unassigned path denied
- session/bootstrap works
- tenant mapping still enforced
- no crash loop

## 16. TEST DATA FINAL STATE

Because these are dedicated Staging test identities, the normalized A/B/C state MAY remain in Staging for future regression if:
- all identities are confirmed test-only
- no real customer data is involved
- the resulting state is clearly documented

Do NOT restore B/C back to the pre-Seat legacy state if that would destroy the canonical regression fixtures.

Instead document them as the fixed Central Seat Staging regression identities.

If any temporary intermediate mutation was used, clean that temporary state.

Required:
REAL_CUSTOMER_DATA_MUTATED = NO
TEMP_RESIDUE = 0

## 17. FINAL REPORT

### A. EXISTING IDENTITY INVENTORY

Table:
Identity label | Auth | Role | Customer | Membership | Post entitlement | Seat | Post tenant | Classification

### B. NORMALIZATION ACTIONS

For each identity:
what was missing
what canonical RPC/admin operation was used
final state

### C. FINAL TEST MODEL

ADMIN =
ASSIGNED_USER =
UNASSIGNED_USER =

Do not print credentials.

### D. PRE-CUTOVER READINESS

ASSIGNED_TEST_IDENTITY_READY =
UNASSIGNED_TEST_IDENTITY_READY =

### E. DEPLOY

Previous release:
New release:
Commit:
Digest:
Audit:

### F. RUNTIME FLAGS

COMMERCIAL_GATE =
SEAT_POLICY =
ALLOW_TENANT_KEY =
AUTO_TENANT_BOOTSTRAP =
SEAT_ENFORCEMENT_DECOUPLED_FROM_AUTO_BOOTSTRAP =

### G. FINAL E2E

Case | Identity Type | Membership | Entitlement | Seat | Post tenant | Central result | Final result | PASS

Required PASS:
- Admin
- Assigned ordinary user
- Unassigned ordinary user
- Invalid JWT

### H. CENTRAL SEAT CAUSAL PROOF

CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES/NO

### I. SECURITY / HEALTH

AUTH_LOG_SECRET_LEAK =
HEALTH_200 =
CRASH_LOOP =

### J. DATA SAFETY

REAL_CUSTOMER_DATA_MUTATED = NO
TEMP_RESIDUE = 0
FIXED_STAGING_REGRESSION_IDENTITIES_DOCUMENTED = YES/NO

### K. PRODUCTION

PRODUCTION_MUTATED = NO

### L. FINAL RESULT

If complete:

RESULT = PASS
EXISTING_STAGING_USERS_NORMALIZED = YES
POST_CENTRAL_SEAT_STAGING_CUTOVER = PASS
CENTRAL_AUTHORITATIVE = YES
BUSINESS_ASSIGNED_ALLOW = PASS
BUSINESS_UNASSIGNED_DENY = PASS
CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES
READY_FOR_POST_PRODUCTION_CUTOVER = YES

If there are insufficient existing non-admin identities:

RESULT = BLOCKED
BLOCKER = insufficient existing ordinary Staging identities
NEW_ACCOUNT_REQUIRED = YES
Do not create it automatically.

If a canonical admin write path is missing:

RESULT = BLOCKED
BLOCKER = missing canonical admin write path for <object>

If deployment/E2E fails:
rollback Post Staging only and report exact failure.

Finish after final Post Staging authoritative E2E.
Do not deploy Production.
