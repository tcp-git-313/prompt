# P4A — Post Central Seat Shadow Integration

ROLE: POST CENTRAL SEAT SHADOW OWNER

MODE: IMPLEMENTATION, LOCAL ONLY, NON-AUTHORITATIVE

REPO:
F:\00-Ticenpi-SaaS\TicenpiPost

GOAL:
Implement the smallest safe Post shadow comparison against Central Seat without changing current user-visible authorization behavior.

Do NOT deploy.
Do NOT modify Staging/Production config.
Do NOT make Central Seat authoritative.
Do NOT modify ticenpi-platform, DM, or Launcher.
Do NOT touch Production.
Do NOT commit/push unless explicitly instructed.

KNOWN CURRENT STATE:
- Post already has CentralSeatStatusClient / SeatChecker.
- product_seat_status("post") exists in Live Staging.
- current final path is authorize_commercial_and_seat() then Post tenant mapping.
- current Staging uses SEAT_POLICY=skip.
- AUTO_TENANT_BOOTSTRAP currently influences SeatChecker injection.
- X-Tenant-Key remains a separate bypass risk.
- Post tenant_members must remain; it is product-tenant mapping, not Commercial Core membership.

TASKS:

1. PREFLIGHT
Record branch, HEAD, git status --short, git diff --stat.
Preserve existing work.
No reset/clean/stash/rebase.

2. TRACE CURRENT PATH
Reconfirm exact symbols/files for:
- authorize_commercial_and_seat()
- CommercialGate
- SeatChecker / CentralSeatStatusClient
- resolve_principal()
- tenant_for()/tenant_members
- X-Tenant-Key path
- related tests

3. ADD SHADOW ONLY
At the smallest shared backend authorization point, add a shadow call to:
product_seat_status("post")

Requirements:
- use same authenticated bearer/session context
- shadow result MUST NOT change allow/deny
- shadow error MUST NOT change allow/deny
- do not add new per-route duplicated checks
- do not alter Post tenant mapping
- do not alter X-Tenant-Key behavior in this task
- do not alter AUTO_TENANT_BOOTSTRAP or SEAT_POLICY behavior in this task

4. OBSERVABILITY
Add safe mismatch logging only.

May log:
- product=post
- legacy decision
- central shadow decision
- normalized reason
- environment
- release/commit if already available
- non-reversible actor hash if existing safe helper exists

Must NOT log:
- bearer token
- JWT
- email
- raw secret
- service-role
- full customer identity

5. SHADOW SEMANTICS
Normalize central outcomes into a small decision model:
ALLOW / DENY / ERROR / UNKNOWN

Legacy result remains authoritative.

Central RPC unavailable/invalid response:
- record shadow ERROR
- do not alter response

6. TESTS
Add/adjust tests proving:
- legacy allow + central allow => legacy allow unchanged
- legacy allow + central deny => legacy allow unchanged + mismatch logged
- legacy deny + central allow => legacy deny unchanged + mismatch logged
- central error => legacy result unchanged
- platform_admin path remains unchanged
- Post tenant mapping unchanged
- no route-level duplication
- no secret logging
- existing SeatChecker tests still pass

Do not require live Staging.

7. NO CUTOVER
Do not:
- change SEAT_POLICY
- decouple bootstrap
- remove X-Tenant-Key
- make Central authoritative
- deploy

8. FINAL REPORT

A. PREFLIGHT
B. CURRENT AUTH PATH
C. FILES CHANGED
D. SHADOW INSERTION POINT
E. DECISION/LOGGING CONTRACT
F. TESTS
G. SAFETY

Must state:
CENTRAL_AUTHORITATIVE = NO
USER_VISIBLE_AUTH_BEHAVIOR_CHANGED = NO
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO
COMMIT_CREATED = NO
PUSHED = NO

H. FINAL RESULT

If complete:

RESULT = PASS
POST_SHADOW_IMPLEMENTED = YES
LEGACY_AUTHORITY_PRESERVED = YES
READY_FOR_POST_STAGING_SHADOW = YES

If blocked:

RESULT = BLOCKED
POST_SHADOW_IMPLEMENTED = NO
BLOCKER = exact blocker

Stop after local shadow implementation and tests.