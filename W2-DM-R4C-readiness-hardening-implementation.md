# W2-DM-R4C — DM Readiness Hardening Implementation

ROLE: DM RUNTIME RELIABILITY IMPLEMENTATION OWNER

MODE: CODE + TESTS ONLY / NO DEPLOY

WORKSPACE:
F:\00-Ticenpi-SaaS

DM REPO:
F:\00-Ticenpi-SaaS\TicenpiDM

GOAL:
Implement the already-audited readiness hardening with the smallest safe change, without touching Commercial/Seat logic and without deploying.

VERIFIED ROOT CAUSE:
- one UvicornWorker
- Gunicorn default timeout 30s
- /api/ready currently calls deep dependency diagnostics
- deep checks are synchronous/blocking and sequential
- DNS can be unbounded
- cold-cache deep checks may starve the event loop and trigger worker timeout

VERIFIED DESIGN:
- /api/ready should be shallow and evaluate runtime contract only
- /api/health/detail remains deep diagnostics
- deep diagnostics should run blocking checks off the event loop
- independent deep checks should run concurrently
- increasing Gunicorn timeout is NOT the primary fix

ABSOLUTE RULES:
- Do NOT change Central Commercial/Seat/auth semantics.
- Do NOT deploy.
- Do NOT modify Staging/Production.
- Do NOT change network config.
- Do NOT touch unrelated dirty WIP.
- Use isolated worktree/branch.
- Do NOT change Dockerfile timeout unless tests prove the audited design still requires it.

ALLOWED PRIMARY FILES:
- src/backend/app/routers/health.py
- src/backend/app/services/health.py
- tests/api/test_health.py
- minimal adjacent test helper only if required

1. PREFLIGHT
Record branch/HEAD/status and current health implementation.
Create isolated worktree/branch.

2. SHALLOW /api/ready
Change /api/ready so it:
- evaluates readiness(contract)
- does NOT call HealthService.detail()
- returns 200/503 based on contract only

3. DEEP DIAGNOSTICS OFF EVENT LOOP
Keep /api/health/detail as deep diagnostics.
Run synchronous blocking checks off-thread.
Run independent checks concurrently.
Preserve existing check semantics and cache semantics.

4. TESTS
Required:
- /api/ready never invokes HealthService.detail()
- invalid runtime contract → 503
- valid runtime contract → fast 200 even if deep check would sleep/fail
- deep health still returns failure correctly
- independent deep checks complete approximately in max duration, not sum
- sleeping sync check does not block unrelated request/event loop
- TTL cache semantics unchanged
- existing health tests remain green

5. PERFORMANCE PROOF
Provide measured test evidence showing:
- shallow ready path bounded and fast
- deep checks no longer sequentially block single event loop

6. CANDIDATE
Commit only readiness hardening files/tests.
Do not include Seat/auth/release identity work.

7. FINAL REPORT
Return:
READY_SHALLOW =
DEEP_CHECKS_OFF_EVENT_LOOP =
DEEP_CHECKS_CONCURRENT =
CACHE_SEMANTICS_PRESERVED =
AUTH_SEAT_LOGIC_CHANGED = NO
FILES_CHANGED =
TESTS =
COMMIT =
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO

If complete:
RESULT = PASS
READY_HARDENING_IMPLEMENTED = YES
READY_FOR_DM_FINAL_CANDIDATE_INTEGRATION = YES

Do not deploy.
