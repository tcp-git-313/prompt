# W1-DM-R4B — DM Readiness Timeout Audit

Read-only only. Inspect the current DM Gunicorn command, worker model, timeout, /api/ready implementation, dependency checks, per-check timeouts, execution order, readiness cache TTL, and health/readiness responsibilities. Confirm the exact cause of intermittent worker timeout and calculate worst-case cold-cache readiness latency.

Compare only evidence-backed fixes: increase Gunicorn timeout, increase cache TTL, parallelize independent checks, split shallow readiness from deep diagnostics, or the smallest safe combination. Do not implement.

Return READY_WORST_CASE_SECONDS, GUNICORN_TIMEOUT_SECONDS, READY_TIMEOUT_ROOT_CAUSE_CONFIRMED, RECOMMENDED_FIX, FILES_TO_TOUCH, TESTS_REQUIRED, ROLLBACK, READY_FOR_SEPARATE_IMPLEMENTATION_TASK. Do not modify files, restart services, change runtime/network config, mutate Staging/Production, deploy, commit or push.