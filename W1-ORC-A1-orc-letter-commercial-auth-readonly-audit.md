# W1-ORC-A1 — ORC + Letter Commercial Auth / Seat / Service Boundary Readonly Audit

ROLE: ORC + LETTER COMMERCIAL ARCHITECTURE AUDIT OWNER

MODE: READ-ONLY AUDIT ONLY

WORKSPACE:
F:\00-Ticenpi-SaaS

EXPECTED PRODUCT MODEL FROM OPERATOR:
- ORC is a directly purchased user-facing product.
- User logs directly into ORC.
- ORC calls Letter.
- ORC and Letter are one product suite from the user's perspective.
- Letter is not assumed to be a separately purchased Seat product unless the code proves otherwise.

GOAL:
Map the exact current ORC + Letter auth, tenancy, commercial entitlement, deployment and service-call architecture so the next task can implement Central Seat for ORC without accidentally duplicating Seat inside Letter or breaking internal service calls.

Do NOT modify files.
Do NOT deploy.
Do NOT mutate Staging/Production.
Do NOT create users.
Do NOT create Seat assignments.
Do NOT change ORC or Letter auth.

## 1. DISCOVER REPOS / RUNTIMES

Identify exact repo paths for:
- ORC
- Letter
- shared auth/commercial modules they import
- deployment manifests/services
- Staging endpoints/ports
- Production endpoints/ports if discoverable read-only

Record branch, HEAD, status for ORC and Letter.

## 2. TRACE ORC USER AUTH

Trace exact ORC user entry path:

login
→ JWT/session validation
→ commercial context
→ entitlement/product check
→ tenant/customer/data scope
→ protected ORC API

Identify:
- Supabase project/runtime config
- product code currently used
- whether ORC already calls my_commercial_context()
- whether ORC already calls product_seat_status("orc")
- any local bypass
- platform_admin handling
- test_access handling
- HTTP 401/403/503 behavior

## 3. TRACE ORC TENANCY / DATA ISOLATION

Identify ORC's actual data boundary:
- customer_id
- tenant_id
- organization_id
- RLS
- local tenant_members
- other exact model

Do not assume it matches Post or DM.

Map:
User → Commercial Customer → ORC data boundary

Identify cross-tenant negative tests if any.

## 4. TRACE ORC → LETTER CALL CHAIN

Determine exactly how ORC calls Letter:

- in-process library
- local HTTP
- remote HTTP
- queue/worker
- internal RPC
- another mechanism

For each call identify:
- endpoint/function
- caller identity
- auth header/credential type
- tenant/customer context passed
- product context passed
- correlation/job identifiers
- whether Letter can be called directly from the public network
- whether user bearer is forwarded
- whether service credentials exist
- how Letter scopes data

Do not print secrets.

## 5. CLASSIFY LETTER

Return exactly one primary classification:

LETTER_MODE = INTERNAL_COMPONENT
or
LETTER_MODE = INTERNAL_SERVICE
or
LETTER_MODE = DIRECT_USER_PRODUCT
or
LETTER_MODE = MIXED
or
LETTER_MODE = UNKNOWN

For each classification, explain the evidence.

Preferred commercial design assumption unless contradicted by code:
ORC owns the user-facing Central Seat gate.
Letter does NOT have a separate purchased Seat.
Letter trusts only authenticated ORC/service calls plus tenant/customer scope.

But do not force this conclusion if the code says otherwise.

## 6. COMMERCIAL CORE / CENTRAL SEAT GAP

For ORC determine:

CURRENT_CENTRAL_SEAT_STATE =
NONE
SHADOW
AUTHORITATIVE
PARTIAL
UNKNOWN

Identify exact insertion point for canonical:
product_seat_status("orc")

Determine whether existing commercial entitlement already uses product_code="orc".

If product_code differs, report exact current code/value.
Do not rename anything.

## 7. REQUIRED FINAL ORC COMMERCIAL CONTRACT

Based on current architecture, describe the minimal correct target:

User
→ ORC Auth
→ Commercial Core
→ Central Seat(product=orc)
→ ORC data boundary
→ ORC functionality
→ Letter internal call if needed

For Letter, specify the minimum boundary required:
- service authenticity
- tenant/customer propagation
- data ownership check
- no duplicate Central Seat if ORC already gated the user

Do not implement.

## 8. STAGING TEST FIXTURE DISCOVERY

Read-only inspect whether existing fixed ordinary Staging regression users can be reused for ORC.

Determine:
- Auth users available
- active customer membership
- existing ORC entitlement
- existing ORC Seat assignments
- ORC tenant/data mappings if applicable

Do not mutate.

Return:
ORC_ASSIGNED_TEST_USER_AVAILABLE = YES/NO
ORC_UNASSIGNED_TEST_USER_AVAILABLE = YES/NO
ORC_TEST_CUSTOMER_AVAILABLE = YES/NO

## 9. DEPLOY / RELEASE RISK

Inspect ORC and Letter deployment architecture.

Determine:
- separate images/services or same artifact
- shared manifest risk
- whether deploying ORC changes Letter
- whether deploying Letter changes ORC
- rollback boundaries
- health endpoints
- current Staging release identity
- Production release identity read-only if available

Return:
ORC_AND_LETTER_CAN_DEPLOY_INDEPENDENTLY = YES/NO
SHARED_DEPLOY_RISK = exact explanation

## 10. FINAL IMPLEMENTATION PLAN

Produce the smallest implementation plan for the next task, broken into exact files/symbols if discoverable:

A. ORC Central Seat integration
B. ORC authoritative policy toggle
C. ORC test fixture
D. ORC Staging deploy
E. ORC assigned/unassigned/admin E2E
F. Letter service boundary, only if required
G. rollback/health smoke

Explicitly list:
FILES_TO_MODIFY
SYMBOLS_TO_MODIFY
DO_NOT_TOUCH

## 11. FINAL REPORT

A. PREFLIGHT
B. ORC AUTH CHAIN
C. ORC DATA BOUNDARY
D. ORC → LETTER CALL CHAIN
E. LETTER CLASSIFICATION
F. CENTRAL SEAT GAP
G. TARGET CONTRACT
H. TEST FIXTURE AVAILABILITY
I. DEPLOY / ROLLBACK MODEL
J. EXACT NEXT IMPLEMENTATION SCOPE
K. SAFETY

SOURCE_FILES_MODIFIED = NO
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO
COMMIT_CREATED = NO
PUSHED = NO

L. FINAL RESULT

If architecture is fully mapped:

RESULT = PASS
ORC_COMMERCIAL_ARCHITECTURE_MAPPED = YES
LETTER_BOUNDARY_CLASSIFIED = YES
READY_FOR_ORC_CENTRAL_SEAT_IMPLEMENTATION = YES

If key facts cannot be proven:

RESULT = BLOCKED
READY_FOR_ORC_CENTRAL_SEAT_IMPLEMENTATION = NO
BLOCKER = exact missing evidence

Finish audit and stop.
