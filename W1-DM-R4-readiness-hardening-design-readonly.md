# W1-DM-R4 — DM Readiness / Gunicorn Timeout Hardening Design (Read-Only)

ROLE: DM RUNTIME RELIABILITY DESIGN OWNER

MODE: READ-ONLY / DESIGN ONLY

WORKSPACE:
F:\00-Ticenpi-SaaS
DM REPO:
F:\00-Ticenpi-SaaS\TicenpiDM

GOAL:
Design the smallest safe fix for intermittent /api/ready worker timeouts before commercial launch, without changing runtime now.

Do NOT modify files.
Do NOT deploy.
Do NOT restart services.
Do NOT mutate Staging/Production.

## VERIFIED SYMPTOM

/api/ready performs sequential dependency checks and may exceed Gunicorn's default 30s worker timeout on cold/slow network conditions.

Previously observed checks include:
- YUCT DNS/TLS
- proxy
- Supabase auth/JWKS
- AGNES
- other readiness dependencies

The endpoint can return 200 when checks complete, but worker timeout events have occurred.

## 1. TRACE EXACT CURRENT IMPLEMENTATION

Identify:
- Gunicorn CMD/args
- worker count/class
- timeout
- keepalive/graceful timeout
- /api/ready implementation
- dependency checks and per-check timeout
- sequential vs parallel behavior
- cache TTL
- health vs readiness responsibility

## 2. QUANTIFY WORST-CASE LATENCY

Calculate upper-bound and realistic cold-cache ready latency from code-configured timeouts.

Return:
READY_WORST_CASE_SECONDS =
GUNICORN_TIMEOUT_SECONDS =
TIMEOUT_MARGIN_SECONDS =

## 3. EVALUATE FIX OPTIONS

Compare only evidence-backed options:

A. Increase Gunicorn timeout
B. Increase readiness cache TTL
C. Parallelize independent checks
D. Separate shallow readiness from deep diagnostics
E. Combination

For each:
- files/symbols
- operational risk
- false-positive/false-negative health risk
- rollback simplicity
- whether app image rebuild is required

## 4. RECOMMENDED MINIMAL FIX

Choose the smallest commercial-safe change.

Do not optimize unrelated health code.

Required:
RECOMMENDED_FIX =
WHY =
FILES_TO_TOUCH =
TESTS_REQUIRED =
ROLLBACK =

## 5. FINAL REPORT

SOURCE_FILES_MODIFIED = NO
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO

RESULT = PASS/BLOCKED
READY_TIMEOUT_ROOT_CAUSE_CONFIRMED = YES/NO
READY_HARDENING_PLAN_COMPLETE = YES/NO
READY_FOR_SEPARATE_IMPLEMENTATION_TASK = YES/NO

Finish and stop.
