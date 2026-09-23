# P4B — DM Central Seat Shadow Integration

ROLE: DM CENTRAL SEAT SHADOW OWNER

MODE: IMPLEMENTATION, LOCAL ONLY, NON-AUTHORITATIVE

REPO:
F:\00-Ticenpi-SaaS\TicenpiDM

GOAL:
Implement the smallest safe DM shadow comparison against Central Seat without changing current user-visible authorization behavior.

Do NOT deploy.
Do NOT modify Staging/Production config.
Do NOT make Central Seat authoritative.
Do NOT modify ticenpi-platform, Post, or Launcher.
Do NOT touch Production.
Do NOT commit/push unless explicitly instructed.

KNOWN CURRENT STATE:
- DM auth path is get_current_user -> require_dm_access()/EntitledUserDep -> my_commercial_context() -> has_dm_access().
- DM currently does not enforce product_seat_status("dm").
- Live Staging Central Seat RPC exists.
- DM product RLS must remain.
- Current dirty WIP may include OAuth/YUCT extraction-shadow work unrelated to Seat.

TASKS:

1. PREFLIGHT
Record branch, HEAD, git status --short, git diff --stat.
Identify pre-existing dirty files.
Do not overwrite or edit unrelated dirty WIP.
No reset/clean/stash/rebase.

2. TRACE CURRENT PATH
Reconfirm:
- get_current_user
- require_dm_access()
- EntitledUserDep
- my_commercial_context()
- has_dm_access()
- all protected routers using EntitledUserDep
- related tests

3. ADD SHADOW ONLY
At require_dm_access() or the smallest shared entitlement guard:
call Central product_seat_status("dm") using the same caller bearer/session.

Requirements:
- existing has_dm_access() result remains authoritative
- central result MUST NOT change 401/403/allow behavior
- central RPC error MUST NOT change current behavior
- do not modify router-by-router
- do not modify DM RLS
- do not alter local/dev bypass semantics in this task

4. OBSERVABILITY
Add safe mismatch logging only.

May log:
- product=dm
- legacy decision
- central shadow decision
- normalized reason
- environment
- release/commit if already available
- non-reversible actor hash if existing helper exists

Must NOT log:
- bearer/JWT
- email
- service-role
- secrets
- raw customer identity

5. SHADOW DECISION MODEL
Normalize central result to:
ALLOW / DENY / ERROR / UNKNOWN

Legacy result remains authoritative.

Important expected mismatch:
business user with valid DM entitlement but no assigned DM Seat may be:
legacy=ALLOW
central=DENY

This is expected during shadow and must be observable, not "fixed" in this task.

6. TESTS
Add/adjust tests proving:
- legacy allow + central allow => allow unchanged
- legacy allow + central deny => allow unchanged + mismatch logged
- legacy deny + central allow => deny unchanged + mismatch logged
- central error => existing result unchanged
- platform_admin behavior unchanged
- Staging test_access behavior unchanged
- local/dev bypass unchanged
- DM RLS tests unaffected
- no secret logging
- protected routers still use shared dependency

Do not require live Staging.

7. NO CUTOVER
Do not:
- make Central authoritative
- remove has_dm_access()
- change HTTP semantics
- deploy
- change Staging/Production config

8. FINAL REPORT

A. PREFLIGHT
B. CURRENT AUTH PATH
C. DIRTY-WIP COLLISION CHECK
D. FILES CHANGED
E. SHADOW INSERTION POINT
F. DECISION/LOGGING CONTRACT
G. TESTS
H. SAFETY

Must state:
CENTRAL_AUTHORITATIVE = NO
USER_VISIBLE_AUTH_BEHAVIOR_CHANGED = NO
DM_RLS_CHANGED = NO
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO
COMMIT_CREATED = NO
PUSHED = NO

I. FINAL RESULT

If complete:

RESULT = PASS
DM_SHADOW_IMPLEMENTED = YES
LEGACY_AUTHORITY_PRESERVED = YES
READY_FOR_DM_STAGING_SHADOW = YES

If blocked by dirty file collision:

RESULT = BLOCKED
DM_SHADOW_IMPLEMENTED = NO
BLOCKER = exact file/symbol collision

Stop after local shadow implementation and tests.