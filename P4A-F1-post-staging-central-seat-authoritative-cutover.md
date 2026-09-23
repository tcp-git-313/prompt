# P4A-F1 — Post Staging Central Seat Authoritative Cutover & Final E2E

ROLE: POST CENTRAL SEAT FINAL CUTOVER OWNER

MODE: STAGING ONLY / AUTHORITATIVE CUTOVER / FINAL E2E

REPO:
F:\00-Ticenpi-SaaS\TicenpiPost

GOAL:
Complete the Post Central Seat integration in Staging.

This is the final Post Staging phase:

1. prove a real non-admin Staging user actually reaches Central Seat
2. decouple Seat enforcement from tenant auto-bootstrap
3. make Central Seat authoritative for normal Post commercial access
4. use existing Staging test identities to prove assigned/unassigned/admin behavior
5. preserve Post tenant mapping as a separate required boundary
6. rollback immediately if final access behavior is wrong

DO NOT touch Production.

## KNOWN CURRENT STAGING RELEASE

Release:
20260923-183949

Source commit:
b07e3d71b32a4aad2d3339418ffb707418f581c2

Image digest:
sha256:42974599edc1c40bcd298206ec112cfb5f469a56155c0a3e04410f8e952d5cd6

Current known flags before final cutover:

TICENPI_COMMERCIAL_GATE=1
TICENPI_AUTO_TENANT_BOOTSTRAP=1
TICENPI_SEAT_POLICY=skip
TICENPI_ALLOW_TENANT_KEY=0
TICENPI_CENTRAL_SEAT_SHADOW_ENABLED=1

Central Seat Staging is already verified:

- product_seat_status(text)
- assign_product_seat(uuid,text,uuid)
- release_product_seat(uuid,text,uuid)
- admin_list_product_seats(uuid,text)
- same-effective-row resolver proof PASS

## ABSOLUTE RULES

- Staging only.
- Production must remain untouched.
- Use existing Staging test identities.
- Do NOT ask the user to manually create new accounts.
- Do NOT create permanent test accounts.
- Do NOT use X-Tenant-Key for final user-path validation.
- Do NOT remove Post tenant_for()/tenant_members mapping.
- Do NOT add service-role credentials to user/browser flow.
- Do NOT log bearer/JWT/email/password/secrets.
- Do NOT overwrite unrelated work.
- Do NOT run another product deploy in parallel.
- Do NOT change Central Seat SQL semantics.

If existing Staging test identities are insufficient for assigned/unassigned comparison, temporary Seat assignment/release on an existing approved test identity is allowed, but the prior state must be recorded and restored in finally.

## 1. PREFLIGHT

Record:

- branch
- HEAD
- git status --short
- git diff --stat
- active Staging release/commit/image
- current effective runtime flags

Confirm active runtime is still:

release=20260923-183949
commit=b07e3d71b32a4aad2d3339418ffb707418f581c2

If active runtime drifted:
STOP.

## 2. PROVE LIVE NON-ADMIN SHADOW CALL FIRST

Before changing authority:

Use an existing ordinary Staging test account.

Authenticate through the normal Staging user login path.

Send a request that traverses:

resolve_principal()
→ authorize_commercial_and_seat()
→ Post tenant_for()
→ protected/session/bootstrap path

Do not use platform_admin.

Prove the same request causes Central:

product_seat_status("post")

to execute with the same caller bearer/session.

Acceptable proof:
- existing safe observability/log event tied to that request
- existing instrumentation that proves the Central RPC call

Do not print the token.

Required:

NON_ADMIN_LIVE_CENTRAL_CALL_PROVEN = YES

If this cannot be proven:
STOP before cutover.

## 3. INSPECT CURRENT SEAT ENFORCEMENT WIRING

Trace current code for:

- SeatChecker
- CentralSeatStatusClient
- authorize_commercial_and_seat()
- TICENPI_SEAT_POLICY
- TICENPI_AUTO_TENANT_BOOTSTRAP
- dependency construction/injection
- session/bootstrap paths
- normal protected API path

Prove whether Seat enforcement is still conditionally constructed only when:

TICENPI_AUTO_TENANT_BOOTSTRAP=1

If yes, this must be fixed before final cutover.

## 4. DECOUPLE SEAT ENFORCEMENT FROM AUTO BOOTSTRAP

Implement the smallest safe change so:

Seat enforcement depends on:
- authenticated commercial access
- Central Seat policy/config

and NOT on whether tenant auto-bootstrap is enabled.

Required conceptual rule:

AUTO_TENANT_BOOTSTRAP
controls tenant bootstrap/provisioning only

SEAT_POLICY
controls Central Seat enforcement only

Do not change current tenant bootstrap semantics.

Do not change Post tenant_members semantics.

Do not duplicate Seat checks route-by-route.

Use the shared authorization path.

## 5. FINAL AUTHORITATIVE CONTRACT

For ordinary authenticated Post users:

commercial entitlement/membership validity
+
Central Post Seat requirement
+
Post tenant mapping
=
final access

Central Seat decision must now be authoritative according to canonical contract.

Expected rules:

platform_admin:
ALLOW / exempt according to Central contract

individual valid entitlement:
ALLOW, manual Seat not required

business user with active membership + effective Post entitlement + assigned Post Seat:
ALLOW

business user with valid entitlement but no assigned Post Seat:
DENY

Post Seat assignment must not imply DM Seat access.

inactive/suspended/removed invalid commercial state:
DENY

Central service/RPC failure:
FAIL CLOSED using the existing planned service-error semantics.
Do not silently fall back to legacy allow.

Missing/invalid JWT:
401

Valid JWT but no commercial/Seat access:
403

Central dependency/service unavailable:
503 if that is consistent with existing backend error conventions.

Do not convert all Central errors to 401.

## 6. SHADOW TRANSITION

The existing shadow observer may remain temporarily for comparison if it does not duplicate or override the authoritative result.

But after cutover:

CENTRAL_AUTHORITATIVE = YES

There must be exactly one final Central Seat authority.

Do not keep a path where legacy can override a Central DENY.

If shadow code becomes redundant but is harmless, leave cleanup for a later small task rather than broad refactor.

## 7. TEST IDENTITIES

Discover and use existing approved Staging identities.

At minimum classify available accounts into:

A. platform_admin
B. ordinary non-admin business user
C. another ordinary non-admin user if available
D. individual entitlement user if available

Do not print credentials.

If B/C need assigned/unassigned states, use canonical admin Seat RPCs against existing test identities.

Use:

admin_list_product_seats()
assign_product_seat()
release_product_seat()

Do not directly edit Seat tables.

## 8. TEMPORARY TEST STATE RULE

Before any Seat mutation:

- capture current Post Seat assignment state
- capture effective entitlement/status
- record test customer/product/user IDs internally
- do not print secrets

If changing an existing test user's Seat for E2E:

try/finally cleanup is mandatory.

After test:
restore the exact original assignment state.

Set:

TEST_STATE_RESTORED = YES

If state cannot be restored:
HARD STOP and report.

## 9. LOCAL TESTS BEFORE DEPLOY

Add/update tests for:

1. Seat enforcement constructed even when AUTO_TENANT_BOOTSTRAP=0
2. AUTO_TENANT_BOOTSTRAP no longer controls Seat authority
3. SEAT_POLICY=skip preserves non-enforcing behavior for non-formal local/test use where intentionally supported
4. SEAT_POLICY=require enforces Central result
5. platform_admin exemption
6. individual entitlement not requiring manual Seat
7. business assigned => allow
8. business unassigned => deny
9. product isolation
10. Central failure => fail closed / correct 503 semantics
11. missing/invalid JWT unchanged
12. Post tenant mapping still required after Central allow
13. X-Tenant-Key remains disabled in formal Staging path
14. no secret logging
15. session/bootstrap and protected API share the same authorization rule

Run targeted Post auth/CommercialGate/SeatChecker/identity/bootstrap suites.

git diff --check must PASS.

## 10. RELEASE CANDIDATE / CI

Use the existing approved GitHub candidate workflow.

The exact cutover code/config candidate must:

- be committed on a dedicated release branch
- be pushed
- have required green CI
- have immutable commit SHA
- have image digest

Do not deploy an uncommitted worktree.

Do not include unrelated local commits/WIP.

If safe candidate isolation cannot be achieved:
STOP.

## 11. STAGING CONFIG CUTOVER

After CI green, change Staging effective configuration to the final formal policy:

TICENPI_COMMERCIAL_GATE=1
TICENPI_SEAT_POLICY=require
TICENPI_ALLOW_TENANT_KEY=0

TICENPI_AUTO_TENANT_BOOTSTRAP may remain 1 if currently needed for tenant provisioning, but Seat enforcement MUST no longer depend on it.

TICENPI_CENTRAL_SEAT_SHADOW_ENABLED may remain 1 during final verification if harmless.

Do not change Production.

## 12. DEPLOY STAGING

Deploy only Post Staging.

Record:

previous release
new release
candidate commit
image digest
runtime config identity

Run deployment audit.

Required:
0 blocking failures.

## 13. VERIFY EFFECTIVE RUNNING FLAGS

Read actual container environment after deployment.

Required:

COMMERCIAL_GATE = 1
SEAT_POLICY = require
ALLOW_TENANT_KEY = 0

Also prove Seat enforcement is active regardless of AUTO_TENANT_BOOTSTRAP dependency wiring.

## 14. FINAL LIVE E2E MATRIX

Run live authenticated Staging tests.

### Case 1 — platform_admin

Expected:
ALLOW

Seat assignment:
not required / exempt

### Case 2 — business assigned Post Seat

Expected:
Central = ALLOW
Post tenant mapping valid
Final = ALLOW

### Case 3 — same/other business user without Post Seat

Expected:
Central = DENY
Final = 403

This is mandatory.

### Case 4 — product isolation

User with DM Seat but no Post Seat, if available:
Post = DENY

If no safe existing identity can represent this:
NOT_REACHED is acceptable.

### Case 5 — individual valid entitlement

If existing safe fixture available:
manual Seat absent
Final = ALLOW

Otherwise:
NOT_REACHED

### Case 6 — invalid/missing JWT

Expected:
401

### Case 7 — Central service error

If safely testable without changing shared infrastructure:
Expected:
fail closed / 503

Otherwise:
NOT_REACHED

## 15. POST TENANT MAPPING PROOF

Central ALLOW must NOT bypass Post product tenant mapping.

Prove:

Central ALLOW
+
missing/invalid Post tenant mapping
≠ automatic access

Post tenant_for()/tenant_members remains a separate product data boundary.

Required:

POST_TENANT_MAPPING_PRESERVED = YES

## 16. LOGGING SAFETY

Inspect final live logs.

Must not contain:

- bearer
- JWT
- refresh token
- email
- password
- service-role
- Supabase secret/key
- cookie
- raw credential material

Set:

AUTH_LOG_SECRET_LEAK = NO

If YES:
rollback and HARD STOP.

## 17. HEALTH / SMOKE

Run:

- /api/health
- external Staging endpoint
- authenticated access
- session/bootstrap
- protected API
- tenant mapping
- critical smoke
- container readiness

No crash loop.

## 18. ROLLBACK

Before deploy record:

PREVIOUS_RELEASE =

If any mandatory final E2E case fails:

- rollback Post Staging only
- restore previous config
- restore temporary Seat test state
- prove previous release active
- prove health 200

Do not touch other services.

## 19. FINAL STATE CLEANUP

Restore all temporary Seat assignments to their exact pre-test state.

Do not remove real customer assignments.

No synthetic/test residue.

Required:

TEST_STATE_RESTORED = YES
TEMP_RESIDUE = 0

## 20. FINAL REPORT

### A. PREFLIGHT

### B. LIVE PRE-CUTOVER CENTRAL CALL

NON_ADMIN_LIVE_CENTRAL_CALL_PROVEN = YES/NO

### C. BOOTSTRAP DECOUPLING

SEAT_ENFORCEMENT_DECOUPLED_FROM_AUTO_BOOTSTRAP = YES/NO

### D. LOCAL TESTS

### E. RELEASE CANDIDATE

Branch:
Commit:
CI:
Image digest:

### F. STAGING DEPLOY

Previous release:
New release:
Audit:

### G. EFFECTIVE FLAGS

COMMERCIAL_GATE =
AUTO_BOOTSTRAP =
SEAT_POLICY =
ALLOW_TENANT_KEY =
CENTRAL_SEAT_SHADOW_ENABLED =

### H. FINAL E2E MATRIX

Table:
Case | Identity type | Central result | Tenant mapping | HTTP/final result | PASS/NOT_REACHED

Mandatory PASS:
- platform_admin
- business assigned
- business unassigned
- invalid JWT

### I. PRODUCT ISOLATION

### J. TENANT MAPPING

POST_TENANT_MAPPING_PRESERVED =

### K. ERROR SEMANTICS

### L. LOGGING SAFETY

AUTH_LOG_SECRET_LEAK =

### M. HEALTH / SMOKE

### N. TEST STATE CLEANUP

TEST_STATE_RESTORED =
TEMP_RESIDUE =

### O. PRODUCTION SAFETY

PRODUCTION_MUTATED = NO

### P. FINAL RESULT

If complete:

RESULT = PASS
POST_CENTRAL_SEAT_STAGING_CUTOVER = PASS
CENTRAL_AUTHORITATIVE = YES
SEAT_ENFORCEMENT_DECOUPLED_FROM_AUTO_BOOTSTRAP = YES
BUSINESS_ASSIGNED_ALLOW = PASS
BUSINESS_UNASSIGNED_DENY = PASS
POST_TENANT_MAPPING_PRESERVED = YES
STAGING_FINAL_E2E = PASS
READY_FOR_POST_PRODUCTION_CUTOVER = YES

If mandatory E2E fails and rollback succeeds:

RESULT = ROLLED_BACK
POST_CENTRAL_SEAT_STAGING_CUTOVER = FAIL
CENTRAL_AUTHORITATIVE = NO
READY_FOR_POST_PRODUCTION_CUTOVER = NO
ROLLBACK = PASS
BLOCKER = exact failure

If pre-cutover proof/candidate isolation fails:

RESULT = BLOCKED
READY_FOR_POST_PRODUCTION_CUTOVER = NO
BLOCKER = exact factual blocker

Finish after final Staging authoritative E2E.
Do not deploy Production.
