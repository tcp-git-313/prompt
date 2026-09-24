# W2-ORC-B1 — Final Browser Central Seat E2E

ROLE: ORC STAGING FINAL BROWSER E2E OWNER

MODE: BROWSER E2E ONLY
NO CODE / NO DB MUTATION / NO DEPLOY

TARGET:
ORC Staging release 20260924-143226

VERIFIED DEPLOYED STATE:
- Central Seat authoritative = YES
- ORC product code = ocr
- ORC_CENTRAL_SEAT_ENABLED=true
- OCR entitlement active
- seat_limit=2
- assigned ordinary user exists
- unassigned ordinary user exists
- health 200
- invalid/no token = 401
- RLS preserved
- Letter remains internal ORC component
- Production untouched

GOAL:
Complete only the two real browser-login E2E checks required to close ORC Staging.

ABSOLUTE RULES:
- Do NOT modify code.
- Do NOT modify entitlement/Seat fixtures.
- Do NOT deploy.
- Do NOT touch Production.
- Do NOT swap the assigned/unassigned fixture roles.
- Do NOT use platform_admin as a substitute for ordinary-user proof.

1. ASSIGNED USER

Pause for operator Google login if required.

After login, prove:
- authenticated ordinary user
- ORC application loads
- protected ORC API succeeds
- OCR flow is available
- ORC user-owned data boundary remains scoped to auth.uid()
- Letter internal route/component is reachable through authorized ORC access

Return:
ORC_ASSIGNED_ALLOW = PASS/FAIL

2. LOGOUT / SESSION ISOLATION

Fully logout/clear the ORC session through supported UI/session flow.
Prove the second account is not inheriting the first account session.

3. UNASSIGNED USER

Pause for operator Google login if required.

After login, prove:
- authentication itself succeeds
- ORC commercial access is denied
- backend denial is 403 / Seat not assigned according to canonical behavior
- protected ORC data/API is not loaded
- Letter does not become an authorization bypass

Return:
ORC_UNASSIGNED_DENY = PASS/FAIL

4. FINAL CAUSAL PROOF

Confirm both users:
- are ordinary, non-admin
- use the same active business customer
- share the same active ocr entitlement eligibility
- differ in ocr Seat assignment

Return:
ORC_CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES/NO

5. FINAL REPORT

If both pass:
RESULT = PASS
ORC_CENTRAL_SEAT_STAGING_CUTOVER = PASS
ORC_ASSIGNED_ALLOW = PASS
ORC_UNASSIGNED_DENY = PASS
ORC_CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES
RLS_PRESERVED = YES
LETTER_INTERNAL_COMPONENT_PRESERVED = YES
READY_FOR_ORC_PRODUCTION_CUTOVER = YES
PRODUCTION_MUTATED = NO

If login interaction remains:
RESULT = PARTIAL_PASS
MANUAL_LOGIN_REQUIRED = YES
READY_FOR_ORC_PRODUCTION_CUTOVER = NO

No code changes. No deploy.
