# W2-DM-C1 — DM Central Seat Response Contract Compatibility Fix

ROLE: DM CENTRAL SEAT CONTRACT COMPATIBILITY OWNER

MODE: ROOT-CAUSE + MINIMAL FIX + TESTS + CI ARTIFACTS
NO STAGING DEPLOY / NO PRODUCTION

WORKSPACE:
F:\00-Ticenpi-SaaS

DM REPO:
F:\00-Ticenpi-SaaS\TicenpiDM

PLATFORM CONTRACT REFERENCE:
F:\00-Ticenpi-SaaS\ticenpi-platform

POST REFERENCE:
F:\00-Ticenpi-SaaS\TicenpiPost

GOAL:
Fix the exact contract mismatch that caused the DM F3 Staging cutover to fail when consuming public.product_seat_status("dm").

Do not redo unrelated DM work.
Do not change Central Seat semantics unless live contract evidence proves the Platform contract itself is wrong.

CURRENT VERIFIED STATE:
- F3 deployment attempt rolled back successfully.
- Current DM Staging is healthy on previous release:
  20260923-215914
- Current Staging app commit:
  25db5f9b8b65a231dafb7cc8f1f05262fb7519aa
- /api/health = 200
- /api/ready = 200
- Production unchanged
- F3 deploy-config candidate commit:
  047f84db653945347356ad353a977bdca4663636
- Previous final app candidate source:
  2dd02e2378600f02b9d14590391b9edf84bc14a2
- Previous CI run:
  35968635281
- Previous backend digest:
  sha256:7f5012ee4acad7267b164e1c3320df88e99bee6691e143303de942175a835468
- Previous frontend digest:
  sha256:e731f573843874e9436e6654eabaf54f79f4d92c85636410667215fb6f41d33a
- Failure point:
  product_seat_status("dm") returned data, but DM rejected the response as invalid/unexpected format.
- Rollback was automatic and successful.
- Post already has a successful authoritative Central Seat integration against the same Central Seat system.
- Platform Central Seat regressions previously passed.

ABSOLUTE RULES:
- Do NOT deploy Staging.
- Do NOT touch Production.
- Do NOT change DM readiness hardening.
- Do NOT change extraction-shadow behavior.
- Do NOT change OAuth.
- Do NOT change RLS semantics.
- Do NOT change entitlement/Seat fixture data in this task.
- Do NOT change DM_SEAT_POLICY rollout logic.
- Do NOT add fallback that silently converts malformed Central Seat responses into ALLOW.
- Do NOT bypass product_seat_status.
- Preserve fail-closed behavior.
- Use an isolated worktree/branch.
- Preserve all unrelated dirty WIP.

1. REPRODUCE THE EXACT CONTRACT FAILURE

Using the final DM candidate source and focused tests, reproduce the same failure class seen during F3.

Capture:
- exact DM function/symbol calling product_seat_status("dm")
- exact HTTP/RPC response payload shape received
- exact parser/model/validation code that rejects it
- exact exception / error classification
- whether Supabase returns:
  - a JSON object
  - a one-row array
  - a scalar
  - a wrapped result
  - null
  - another shape

Return:
DM_SEAT_CALL_SITE =
LIVE_RESPONSE_SHAPE =
DM_EXPECTED_RESPONSE_SHAPE =
REJECTION_REASON =

Do not infer the live response shape from type hints alone.

2. PROVE THE CANONICAL PLATFORM CONTRACT

Inspect the canonical implementation of:
public.product_seat_status(product_code)

Determine:
- SQL return type
- returned column names
- allowed state/reason values
- null behavior
- individual-customer behavior
- platform_admin/exempt behavior
- business assigned behavior
- business unassigned behavior
- inactive/no-entitlement behavior

Cross-check against live Staging read-only RPC output using approved test identities if available.

Return:
PLATFORM_CANONICAL_RESPONSE_CONTRACT =
PLATFORM_LIVE_MATCHES_CANONICAL = YES/NO

Do not mutate Platform DB.

3. COMPARE THE SUCCESSFUL POST CONSUMER

Inspect the Post authoritative consumer that already works.

Compare:
- RPC invocation
- bearer token forwarding
- Supabase REST/RPC response decoding
- object/array normalization
- field extraction
- state mapping
- fail-closed behavior
- error handling

Return:
POST_CONSUMER_PATTERN =
DM_CONSUMER_DIFFERENCE =

The goal is compatibility, not code-copying for its own sake.

4. CLASSIFY ROOT CAUSE

Choose exactly one primary class:
A. DM parser/decoder mismatch
B. DM model/schema mismatch
C. shared Supabase RPC client wrapper mismatch
D. Platform canonical contract mismatch
E. live Staging Platform drift
F. other proven cause

Return:
ROOT_CAUSE_CLASS =
ROOT_CAUSE =

If Platform canonical/live contract is correct and Post consumes it successfully, fix DM only.

If Platform contract itself is wrong, STOP before changing it and report the exact contract defect. Do not broaden scope automatically.

5. IMPLEMENT THE SMALLEST COMPATIBILITY FIX

Allowed primary scope:
- DM Central Seat RPC client/decoder
- DM Seat response model/parser
- focused auth/Seat tests
- minimal shared helper directly required

Requirements:
- accept only the canonical Platform response shape
- preserve strict validation
- preserve fail-closed behavior
- preserve canonical 401/403/503 semantics
- do not convert unknown state/reason values to ALLOW
- do not add service-role credentials
- do not direct-read Seat tables
- do not add product-specific hacks outside the DM consumer boundary

6. REQUIRED TEST MATRIX

Add or update focused tests for the exact canonical RPC shapes:

- business assigned -> ALLOW
- business unassigned -> 403
- individual valid entitlement -> canonical allowed/not_required behavior
- platform_admin/exempt -> ALLOW according to Platform contract
- inactive/no entitlement -> deny
- malformed response -> fail closed
- empty response -> fail closed
- unknown state -> fail closed
- RPC unavailable/error -> 503/fail closed
- wrong product code -> deny/fail closed
- array-vs-object decoding behavior exactly matching the real Supabase client response
- existing DM auth/RLS behavior unchanged

Also run:
- focused Central Seat tests
- existing DM auth tests
- readiness tests
- extraction shadow tests
- auth/RLS verifier
- relevant API tests
- full backend suite if practical
- git diff --check

7. ISOLATED COMMIT

Create one minimal isolated fix commit.

Return:
DM_CONTRACT_FIX_COMMIT =

Do not include deploy-config changes in this commit.

8. CI

Run normal DM CI for the exact fix commit.

Required:
CI = PASS

Capture:
NEW_DM_APP_SOURCE_COMMIT =
NEW_BACKEND_IMAGE_DIGEST =
NEW_FRONTEND_IMAGE_DIGEST =
BUILD_RUN_ID =

Cross-check CI source SHA and both immutable digests.

Do not deploy.

9. F3 RESUME HANDOFF

Produce exact next-step handoff for rerunning the standard DM Staging cutover:

- source commit
- backend digest
- frontend digest
- expected DM_SEAT_POLICY=require
- existing F3 deploy-config baseline relationship
- note that live fixture must still be verified before deployment

Return:
READY_TO_RESUME_DM_F3 = YES/NO

10. FINAL REPORT

Return exactly:

RESULT =
DM_SEAT_CALL_SITE =
LIVE_RESPONSE_SHAPE =
DM_EXPECTED_RESPONSE_SHAPE =
PLATFORM_CANONICAL_RESPONSE_CONTRACT =
PLATFORM_LIVE_MATCHES_CANONICAL =
POST_CONSUMER_PATTERN =
DM_CONSUMER_DIFFERENCE =
ROOT_CAUSE_CLASS =
ROOT_CAUSE =
DM_CONTRACT_FIX_COMMIT =
FOCUSED_SEAT_TESTS =
AUTH_REGRESSION =
READINESS_REGRESSION =
EXTRACTION_REGRESSION =
AUTH_RLS_REGRESSION =
FULL_BACKEND =
CI =
NEW_DM_APP_SOURCE_COMMIT =
NEW_BACKEND_IMAGE_DIGEST =
NEW_FRONTEND_IMAGE_DIGEST =
BUILD_RUN_ID =
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO
READY_TO_RESUME_DM_F3 =

If complete:
RESULT = PASS
READY_TO_RESUME_DM_F3 = YES

If Platform contract itself is proven wrong:
RESULT = BLOCKED
READY_TO_RESUME_DM_F3 = NO
BLOCKER = exact Platform contract defect

Finish and stop.
