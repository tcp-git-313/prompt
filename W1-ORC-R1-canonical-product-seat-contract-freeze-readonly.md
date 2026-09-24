# W1-ORC-R1 — ORC Canonical Product / Central Seat Contract Freeze (Read-Only)

ROLE: ORC COMMERCIAL CONTRACT OWNER

MODE: READ-ONLY / CONTRACT FREEZE / NO IMPLEMENTATION

WORKSPACE:
F:\00-Ticenpi-SaaS

ORC+LETTER REPO:
F:\00-Ticenpi-SaaS\TicenpiLetter

PLATFORM REPO:
F:\00-Ticenpi-SaaS\ticenpi-platform

GOAL:
Freeze the exact commercial product identity and Central Seat contract for ORC before implementation.

Do NOT rename products.
Do NOT apply migrations.
Do NOT modify source.
Do NOT mutate Staging/Production.
Do NOT create entitlement/Seat fixtures.
Do NOT deploy.

## VERIFIED CURRENT ARCHITECTURE

- User-facing product name: ORC
- Current commercial product_code used by source/config: "ocr"
- Staging products contains code="ocr"
- Staging products does not contain code="orc"
- ORC currently checks my_commercial_context() entitlement for the configured product code
- ORC does not currently call product_seat_status(...)
- Letter is an INTERNAL_COMPONENT in the same Vue/runtime deployment
- Letter does not need a separate purchased Seat unless contrary evidence is found
- ORC data isolation is currently user-owned via auth.uid()/user_id RLS, not customer-owned tenancy

## 1. INVENTORY ALL PRODUCT CODE REFERENCES

Search platform + ORC/Letter + deploy + docs for:
- "ocr"
- "orc"
- product registry
- entitlement code
- plan_products
- runtime ORC_PRODUCT_CODE
- Hub/Launcher product cards
- Seat calls
- audit logs
- migration constraints

Produce:
LOCATION | VALUE | PURPOSE | CANONICALITY

## 2. DETERMINE CANONICAL INTERNAL CODE

Answer exactly:

CANONICAL_COMMERCIAL_PRODUCT_CODE = ocr / orc / other
CANONICAL_CENTRAL_SEAT_PRODUCT_CODE = ocr / orc / other

Use current schema/data/contracts as authority.

Do not choose based on UI naming preference.

If "ocr" is already the consistent canonical DB code, freeze "ocr" for first commercial launch and treat "ORC" as display/product brand name.

If evidence truly requires rename/mapping, specify the required migration but do not execute.

## 3. LETTER BOUNDARY FREEZE

Confirm:

LETTER_MODE =
LETTER_REQUIRES_SEPARATE_ENTITLEMENT = YES/NO
LETTER_REQUIRES_SEPARATE_SEAT = YES/NO

Expected if current architecture holds:
Letter is internal functionality behind ORC access and does not duplicate commercial authorization.

## 4. ORC DATA BOUNDARY FREEZE

Confirm whether first commercial launch may safely retain:

auth.uid()
→ user-owned ORC records/projects/profile

while Central Seat controls only product eligibility.

Answer:
ORC_FIRST_LAUNCH_DATA_MODEL = user-owned / customer-owned / blocked

If user-owned is supported:
state explicitly that lack of customer_id/tenant_id is NOT itself a launch blocker.

If unsupported:
state exact reason.

## 5. FINAL TARGET ACCESS CHAIN

Write the exact implementation contract:

JWT
→ Commercial Core
→ Central Seat(<canonical product code>)
→ ORC user-owned RLS/data boundary
→ OCR functionality
→ Letter internal component

Define:
401 semantics
403 semantics
Central dependency failure semantics
platform_admin behavior
individual customer behavior
business assigned/unassigned behavior

## 6. EXACT IMPLEMENTATION INSERTION POINT

Identify exact files/symbols for next implementation:
- backend auth
- config
- frontend auth store if needed
- tests

Separate:
MUST_CHANGE
OPTIONAL
DO_NOT_TOUCH

## 7. FINAL REPORT

SOURCE_FILES_MODIFIED = NO
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO

RESULT = PASS/BLOCKED
ORC_CANONICAL_PRODUCT_CODE_FROZEN = YES/NO
ORC_CENTRAL_SEAT_CONTRACT_FROZEN = YES/NO
LETTER_SEAT_DUPLICATION_REQUIRED = YES/NO
ORC_DATA_MODEL_LAUNCH_BLOCKER = YES/NO
READY_FOR_ORC_F1 = YES/NO

If blocked:
BLOCKER = exact conflicting evidence

Finish and stop.
