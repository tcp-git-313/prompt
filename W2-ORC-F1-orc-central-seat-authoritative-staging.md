# W2-ORC-F1 — ORC Central Seat Authoritative Staging

ROLE: ORC CENTRAL SEAT STAGING IMPLEMENTATION OWNER

MODE: IMPLEMENTATION + STAGING FIXTURE + STAGING DEPLOY + E2E

WORKSPACE:
F:\00-Ticenpi-SaaS

ORC+LETTER REPO:
F:\00-Ticenpi-SaaS\TicenpiLetter

PLATFORM REPO (CONTRACT REFERENCE ONLY):
F:\00-Ticenpi-SaaS\ticenpi-platform

GOAL:
Make ORC authoritative on Central Seat in Staging using the already-frozen commercial contract, then prove assigned/unassigned behavior end-to-end.

VERIFIED CONTRACT:
- User-facing brand: ORC
- Canonical commercial product code: ocr
- Canonical Central Seat product code: ocr
- Letter mode: INTERNAL_COMPONENT
- Letter requires separate Seat: NO
- ORC data model may remain user-owned via auth.uid()/user_id RLS
- Existing ordinary Staging regression users from Post may be reused
- Shared test customer already exists
- Current ORC entitlement for that customer is absent
- Current ORC Seat assignments are absent
- Post-proven ordinary users are non-admin and test_access=null

ABSOLUTE RULES:
- Staging only.
- Do NOT touch Production.
- Do NOT rename ocr to orc.
- Do NOT create a second ORC product code.
- Do NOT add a separate Letter entitlement/Seat.
- Do NOT redesign ORC data model into customer tenancy.
- Preserve existing dirty WIP using an isolated worktree/branch.
- Do NOT use service-role in end-user runtime auth.
- Do NOT weaken JWT, RLS, or fail-closed behavior.
- Do NOT rely on frontend Seat checks as authoritative.
- Backend must be authoritative.

RUNTIME INPUTS:
ORC_TEST_ASSIGNED_USER_ID=<existing ordinary Staging Auth UUID>
ORC_TEST_UNASSIGNED_USER_ID=<existing ordinary Staging Auth UUID>

Use the same two ordinary users already proven by Post.

1. PREFLIGHT
Record:
- branch/HEAD/status
- Staging release identity
- current ORC runtime auth config
- current product code
- current protected route behavior
- rollback target

2. ISOLATE CANDIDATE
Create a dedicated worktree/branch from the correct current ORC integration baseline.
Do not include unrelated dirty WIP.

Required:
ORC_CANDIDATE_ISOLATED = YES

3. BACKEND AUTHORITATIVE SEAT
Modify the shared backend auth dependency centered on:
backend/app/auth.py::require_ocr_access

Target chain:
Bearer JWT
→ JWKS verification
→ my_commercial_context()
→ product_seat_status('ocr')
→ authoritative decision
→ protected ORC API

Do not duplicate Seat logic per route.

Required behavior:
- platform_admin → ALLOW/exempt
- individual valid entitlement → ALLOW according to canonical Seat RPC result
- business + valid entitlement + assigned ocr Seat → ALLOW
- business + valid entitlement + no ocr Seat → 403
- missing/invalid JWT → 401
- Central dependency failure → fail closed / 503
- wrong product code → deny

4. FRONTEND UX MIRROR
Frontend may mirror Seat state only for UX.
Authoritative decision remains backend.

Inspect/update only if needed:
- frontend/src/stores/auth.js
- frontend/src/lib/supabase.js
- frontend/src/lib/orcDataContract.js

Do not let frontend bypass backend.

5. LETTER
Do not add separate Letter auth.
Letter remains behind ORC access:
OCR records
→ letterBatch
→ LetterView

6. STAGING FIXTURE
Using canonical platform_admin RPCs only:
- ensure the shared test customer has effective ocr entitlement
- seat_limit = 2
- assign ocr Seat to ORC_TEST_ASSIGNED_USER_ID
- ensure ORC_TEST_UNASSIGNED_USER_ID has no ocr Seat

Do not directly write product_seat_assignments.

Record entitlement_id and Seat state.

7. TESTS
Add/adjust:
- assigned ordinary → allow
- unassigned ordinary → 403
- platform_admin → allow
- individual valid entitlement behavior
- invalid JWT → 401
- Central error → fail closed
- wrong product code regression
- test_access allow/deny existing behavior
- user-owned RLS unchanged
- Letter remains internal and available only after ORC access
- no secret logging

Run relevant backend/frontend tests and git diff --check.

8. RELEASE CANDIDATE / CI
Create exact immutable candidate.
Record:
ORC_APP_SOURCE_COMMIT
ORC_IMAGE_DIGEST
ORC_DEPLOY_CONFIG_COMMIT if separated
CI status

Do not deploy unless required CI is green.

9. STAGING DEPLOY
Deploy ORC Staging only.
Record previous/new release, commit, digest, config identity.

10. FINAL E2E
Mandatory:
A. platform_admin → ALLOW
B. assigned ordinary user → ALLOW
C. unassigned ordinary user → 403
D. invalid JWT → 401
E. user-owned RLS boundary preserved
F. Letter accessible only through authorized ORC session

11. HEALTH / SECURITY
Verify:
- health 200
- no crash loop
- no token/JWT/service-role/secret leakage
- Production untouched

12. ROLLBACK
If mandatory runtime/E2E/security fails:
rollback ORC Staging only to previous known-good release and prove health 200.

13. FINAL REPORT
Return:
ORC_CANDIDATE_ISOLATED =
ORC_PRODUCT_CODE = ocr
ORC_ENTITLEMENT_READY =
ORC_ASSIGNED_SEAT =
ORC_UNASSIGNED_SEAT =
CENTRAL_AUTHORITATIVE =
ORC_ASSIGNED_ALLOW =
ORC_UNASSIGNED_DENY =
ORC_RLS_PRESERVED =
LETTER_INTERNAL_COMPONENT_PRESERVED =
HEALTH_200 =
PRODUCTION_MUTATED = NO

If complete:
RESULT = PASS
ORC_CENTRAL_SEAT_STAGING_CUTOVER = PASS
READY_FOR_ORC_PRODUCTION_CUTOVER = YES

If browser login only remains:
RESULT = PARTIAL_PASS
MANUAL_LOGIN_E2E_REQUIRED = YES
READY_FOR_ORC_PRODUCTION_CUTOVER = NO

Do not deploy Production.
