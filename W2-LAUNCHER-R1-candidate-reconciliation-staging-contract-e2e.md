# W2-LAUNCHER-R1 — Launcher Candidate Reconciliation + Staging Contract E2E

ROLE: LAUNCHER COMMERCIAL WORKBENCH RELEASE-CANDIDATE OWNER

MODE: CANDIDATE RECONCILIATION + LOCAL/DEV VALIDATION AGAINST STAGING CONTRACTS

IMPORTANT:
There is NO separate Launcher Staging deployment for this task.

The stable Launcher remains Production-only.
Do NOT create a Staging Launcher app, Staging Studio build, alternate stable channel, or Production environment switch.

The purpose of "Staging Admin E2E" here is ONLY:
- run the isolated Launcher/Hub candidate locally or in its existing supported dev mode
- point it at the already-approved Central Supabase Staging contracts
- prove the Workbench calls the live Staging admin RPCs correctly
- do not deploy Launcher anywhere

WORKSPACE:
F:\00-Ticenpi-SaaS

LAUNCHER REPO:
F:\00-Ticenpi-SaaS\Ticenpi-Launcher

PLATFORM CONTRACT SOURCE:
F:\00-Ticenpi-SaaS\ticenpi-platform

PLATFORM STAGING CONTRACT:
W2-PLATFORM-S1 = PASS
Source commit:
c7e589f21e019280ae7f83584bde3e6af255fd0f

Known Launcher candidate reports:
Candidate A:
366cd05484fbce635b8b058298f917d57658d3f2

Candidate B:
b7f85bf9dc0c81ca3c555c44ebaa952ce2826df7

Both were reported as isolated Workbench implementations.
One report says 24/24 tests; another says 23/23 tests.

GOAL:
Resolve the two Launcher candidate commits into one canonical candidate, then prove that candidate against the live Staging Commercial Admin contracts WITHOUT deploying a Launcher Staging environment.

ABSOLUTE RULES:
- Do NOT deploy Launcher to Staging.
- Do NOT deploy Launcher to Production.
- Do NOT modify Production Supabase.
- Do NOT switch the stable Production Launcher to Staging.
- Do NOT create a second Launcher environment.
- Do NOT guess which candidate is newer/better.
- Do NOT merge both blindly.
- Preserve unrelated dirty WIP.
- Use existing local/dev Launcher mode only.
- If the current Launcher architecture cannot safely target Staging from local/dev without changing stable Production behavior, STOP and report the exact blocker.

1. CANDIDATE RECONCILIATION

Inspect both commits:
366cd05484fbce635b8b058298f917d57658d3f2
b7f85bf9dc0c81ca3c555c44ebaa952ce2826df7

Determine:
- parent/ancestor relationship
- changed files
- semantic differences
- test differences
- whether one supersedes the other
- whether either contains unrelated changes
- whether both implement the frozen MUST_HAVE scope

Return:
CANDIDATE_A_STATUS =
CANDIDATE_B_STATUS =
CANONICAL_LAUNCHER_CANDIDATE =
RECONCILIATION_REASON =

Do not create a new merge commit unless neither candidate alone is complete and a minimal reconciliation is necessary.

2. FROZEN MUST_HAVE CHECK

Canonical candidate must support:

Customer:
- search/list
- create
- detail
- status
- update supported fields
- suspend/reactivate

Member:
- canonical list with stable user_id
- add existing Auth user
- suspend/reactivate/remove

Entitlement:
- post
- dm
- ocr
- status
- seat_limit
- manual admin update for first launch

Seat:
- count
- list
- safe assign from active canonical member list
- release

Security:
- platform_admin only
- no email/user heuristic
- no direct protected-table reads
- no service-role in frontend
- stale customer state cleared

3. LOCAL TESTS

Run the full Launcher/Hub test suite from the canonical candidate.
Explain the 23/23 vs 24/24 discrepancy.

Required:
LAUNCHER_LOCAL_TESTS = PASS

4. STAGING CONTRACT E2E — NO LAUNCHER DEPLOY

Use the canonical candidate in local/dev mode only.

Target:
Central Supabase Staging project
jlsqjvehwblkeuycjoyj

Precondition:
W2-PLATFORM-S1 PASS.

Use a platform_admin Staging login/session.

Verify through the Launcher Workbench UI:
A. customer list/search/detail
B. canonical member list returns stable server user_id
C. entitlement status/seat_limit render
D. assign Seat to an eligible active member
E. release Seat
F. suspend member causes Seat release
G. reactivate member does not auto-reassign
H. customer suspend blocks commercial access
I. customer reactivate restores customer status but does not invent Seat assignment
J. product isolation across post/dm/ocr

Use only test customer data.

If OAuth/browser login requires operator interaction:
pause only for login, then continue from the preserved local session.

5. PRODUCTION SAFETY

Prove:
- stable Launcher config unchanged
- Production Supabase untouched
- no Production deployment
- no updater/channel mutation
- no Staging Launcher build created

6. FINAL REPORT

Return:
CANONICAL_LAUNCHER_CANDIDATE =
LOCAL_TESTS =
STAGING_CONTRACT_E2E =
LAUNCHER_STAGING_DEPLOYED = NO
STABLE_LAUNCHER_CHANGED = NO
PRODUCTION_MUTATED = NO

If all E2E passes:
RESULT = PASS
LAUNCHER_WORKBENCH_STAGING_CONTRACT_VALIDATED = YES
READY_FOR_LAUNCHER_PRODUCTION_RELEASE_CANDIDATE = YES

If operator login remains:
RESULT = PARTIAL_PASS
MANUAL_LOGIN_REQUIRED = YES
READY_FOR_LAUNCHER_PRODUCTION_RELEASE_CANDIDATE = NO

If safe local/dev Staging targeting is unsupported:
RESULT = BLOCKED
BLOCKER = exact architecture/config limitation
DO NOT invent a Staging Launcher environment.
