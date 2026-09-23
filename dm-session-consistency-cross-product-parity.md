# DM SESSION CONSISTENCY + CROSS-PRODUCT LOGIN PARITY

## Goal

DM Staging has confirmed a protected request can reach the backend without the expected authenticated session context.

Observed evidence:

GET /api/designs
→ 403
→ MISSING_AUTH_TOKEN

The task is to:

1. Find the DM frontend lifecycle defect that allows protected requests before login/session initialization is complete.
2. Fix DM with one shared protected-request contract.
3. Add regression tests for startup, reload, refresh, logout and account switching.
4. Audit other Ticenpi products read-only for the same architectural risk.
5. Keep Central Seat, ExtractionHub, Production and other product source unchanged.

## Execution model

Use parallel READ-ONLY audits first:

- A: DM frontend session lifecycle and protected request callers
- B: DM backend protected route expectations
- C: DM local-vs-Staging parity
- D: cross-product read-only audit

Then:

READ-ONLY AUDITS
→ ROOT CAUSE
→ SINGLE WRITER DM FIX
→ PARALLEL TEST RUNS
→ STAGING RETEST

No parallel writers.

## Repositories

Writable:

F:\00-Ticenpi-SaaS\TicenpiDM

Read-only:

F:\00-Ticenpi-SaaS\ExtractionHub
F:\00-Ticenpi-SaaS\TicenpiPost
F:\00-Ticenpi-SaaS\Ticenpi591
F:\00-Ticenpi-SaaS\TicenpiORC
F:\00-Ticenpi-SaaS\TicenpiSign

Also discover other existing product repositories under F:\00-Ticenpi-SaaS that use login plus protected APIs.

Do not assume a repository exists before checking.

## Preflight

Record:

- branch
- HEAD
- git status
- git diff
- staged changes
- untracked files
- active writer evidence

Preserve existing WIP.

Do not reset, clean, stash or rebase.

Record current DM Staging release, health and runtime configuration identity.

If an active writer is changing the same DM session/request files, stop:

DM SESSION WRITE COLLISION

## Audit A — DM frontend session lifecycle

Map:

login
→ session restore
→ auth-ready state
→ protected API request

Find all protected API callers, including designs, scrape, settings, admin and shadow routes.

Build:

| Caller | Endpoint | Protected | Waits For Session Ready | Shared Request Layer | Risk |

Answer:

- Is there an explicit ready/hydrated state?
- Can page initialization call /api/designs before session restore completes?
- Are protected calls centralized or separately implemented?
- Can reload show a signed-in UI while an early protected call is still unauthenticated?
- Does session refresh update later protected requests?
- Does logout clear app state?
- Can account switching reuse prior state?

Do not modify source during this audit.

## Audit B — backend route contract

Map expected behavior for:

- /api/designs
- /api/scrape
- /api/admin/shared-engine/identity
- relevant shadow/admin routes

Record expected outcomes for:

- unauthenticated request
- authenticated allowed user
- authenticated user without required commercial access
- platform admin where applicable

Confirm MISSING_AUTH_TOKEN is an authentication-stage error and not a Seat decision.

Do not broaden backend permissions.

## Audit C — Local vs Staging parity

Compare local and Staging for:

- login
- session restore
- protected requests
- backend authentication
- commercial authorization
- admin authorization

Build:

| Capability | Local | Staging | Same Path | Risk |

If local interactive development skips the real Staging login/session lifecycle, record:

LOCAL_AUTH_PARITY_GAP

Keep local developer convenience if useful, but require CI coverage with auth enforcement enabled.

## Audit D — Cross-product read-only audit

For each real product with login plus protected APIs, inspect:

- local development bypass
- explicit auth-ready/hydrated state
- centralized protected request layer
- reload behavior
- refresh behavior
- logout cleanup tests
- account switch tests
- missing-auth regression coverage

Build:

| Product | Local Bypass | Auth Ready Gate | Central Request Layer | Reload Safe | Refresh Safe | Logout Safe | Risk |

Classify each product:

CONFIRMED_SAFE
RISK_ONLY
CONFIRMED_BUG
UNKNOWN

Do not change other products.

## Root-cause synthesis

Use evidence only.

Allowed classifications:

AUTH_HYDRATION_RACE
MISSING_AUTH_INJECTION
MULTIPLE_REQUEST_LAYERS
SESSION_REFRESH_SYNC
LOGOUT_STATE
ACCOUNT_SWITCH_STATE
INITIALIZATION_ORDER
MULTIPLE_CAUSES
OTHER

For the selected root cause provide:

- evidence
- trigger
- affected protected endpoints
- why local did not catch it
- minimal fix

If evidence is insufficient, stop:

DM SESSION ROOT CAUSE INCONCLUSIVE

## Fix principles

Prefer one shared frontend contract for protected requests.

Required behavior:

- protected requests wait until session initialization finishes
- current authenticated session is used
- missing session state is handled before the protected request is sent
- refresh updates later requests
- logout invalidates app auth state
- account switching cannot reuse prior state
- no sensitive session material is logged

Do not patch only /api/designs if the same pattern affects other protected callers.

## Required frontend tests

1. App startup while session restore is pending
   → protected request not sent

2. Restore completes successfully
   → protected request follows authenticated path

3. Restore completes without a valid session
   → deterministic unauthenticated behavior

4. Authenticated reload
   → /api/designs follows authenticated path

5. Session refresh
   → later protected request uses current session

6. Logout
   → prior authenticated state is not reused

7. Account switch
   → only the new account is used

8. All protected callers follow the same contract

## Backend regression

Verify existing contract still holds:

- missing authentication denied
- invalid authentication denied
- allowed user succeeds
- commercial authorization remains unchanged
- admin endpoint remains admin-only

Do not weaken backend checks to hide a frontend defect.

## Parallel tests after the fix

May run in parallel:

- frontend session tests
- backend auth tests
- shadow/shared-engine regression
- cookie/tenant/commercial-access regression

Test tracks are read-only.

Only the single writer may modify source after failures.

## Local parity gate

Interactive local development may keep convenience behavior.

CI must include an auth-enforced path covering:

- unauthenticated request
- authenticated request
- normal user
- authorized commercial user
- platform admin
- reload
- refresh
- logout
- account switch

Document:

LOCAL DEV CONVENIENCE != STAGING AUTH PROOF

## Staging retest

If source changes are required, use the normal release process:

commit
→ CI
→ immutable image
→ manifest
→ Staging

No hot patch.
No Production deploy.

Use a fresh normal login flow.

Verify:

- /api/designs no longer fails with MISSING_AUTH_TOKEN
- protected requests wait for session readiness
- /api/admin/shared-engine/identity follows existing admin authorization

If /api/designs changes from MISSING_AUTH_TOKEN to a commercial-access denial, record:

SESSION PROPAGATION FIXED
NEXT GATE = AUTHORIZATION

Do not mix the two issues.

## Cross-product follow-up

LOW:
no action

MEDIUM:
recommend regression task

HIGH:
recommend product-specific parity task

CONFIRMED_BUG:
recommend blocking that product's release path until fixed

Do not fix other products in this task.

## Regression

Run relevant DM suites:

- frontend session
- backend auth
- cookie isolation
- tenant/commercial access
- shared engine identity
- shadow
- legacy YUCT regression
- frontend build
- git diff --check

If packaging/runtime config changes, also run the existing ARM64/Staging release gates.

## Final report

# DM SESSION CONSISTENCY + CROSS-PRODUCT PARITY REPORT

## 1. DM Baseline

Branch:
HEAD:
Staging release:
Health:
Collision:

## 2. DM Root Cause

Classification:
Evidence:
Affected endpoints:
Why local missed it:

## 3. Protected Request Audit

Protected callers:
Centralized callers:
Direct callers:
Auth-ready gap:
Initialization race:

## 4. Fix

Files:
Ready-state behavior:
Protected request contract:
Refresh:
Logout:
Account switch:

Sensitive logging added:
NO

Auth bypass added:
NO

## 5. Tests

Startup:
Reload:
Refresh:
Logout:
Account switch:
Protected callers:
Backend auth:
Shadow:
Cookie/Tenant/Commercial:
Build:
git diff --check:

## 6. Staging Retest

Release:
Fresh browser:
Login:

/api/designs:
Result:
MISSING_AUTH_TOKEN resolved:
YES / NO

/api/admin/shared-engine/identity:
Result:

## 7. Cross-Product Audit

| Product | Local Bypass | Auth Ready Gate | Central Request Layer | Risk | Classification |

## 8. Cross-Product Findings

Confirmed bugs:
Risk-only:
Confirmed safe:
Unknown:

## 9. Isolation

Central Seat:
UNCHANGED

ExtractionHub:
UNCHANGED

Other product writes:
NONE

Production:
UNCHANGED

Production DB:
UNCHANGED

## 10. Next Gate

If DM session propagation is fixed:

READY FOR AUTHORIZATION / PLATFORM_ADMIN VERIFICATION

If protected APIs and admin identity both pass:

READY FOR YUCT UI SMOKE + ROLLBACK CLOSEOUT

## 11. Final Result

Only:

DM SESSION PROPAGATION FIXED

or

DM SESSION PROPAGATION BLOCKED

## Hard stop

Do not modify Central Seat, other product source, Production, Production DB, ExtractionHub, Core Primary or Legacy scraper.

Stop after DM Staging evidence and cross-product audit are complete.
