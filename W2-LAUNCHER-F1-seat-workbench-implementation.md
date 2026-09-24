# W2-LAUNCHER-F1 — Launcher Seat Workbench Implementation

ROLE: LAUNCHER COMMERCIAL WORKBENCH IMPLEMENTATION OWNER

MODE: IMPLEMENTATION + LOCAL TESTS + STAGING ADMIN E2E
NO PRODUCTION DEPLOY

WORKSPACE:
F:\00-Ticenpi-SaaS

LAUNCHER REPO:
F:\00-Ticenpi-SaaS\Ticenpi-Launcher

PLATFORM CONTRACT REFERENCE:
F:\00-Ticenpi-SaaS\ticenpi-platform

PLATFORM CONTRACT SOURCE COMMIT:
c7e589f21e019280ae7f83584bde3e6af255fd0f

GOAL:
Complete the first-launch Seat Workbench for Post, DM and ORC using the frozen Commercial Admin contracts, without touching Production.

DEPENDENCY:
Code implementation may begin immediately against the frozen contract.
Live Staging admin E2E must wait until W2-PLATFORM-S1 reports PASS.

FROZEN MUST-HAVE SCOPE:
Customer:
- search/list
- create
- detail
- status
- update allowed fields
- suspend/reactivate

Members:
- list using canonical stable user_id
- add existing Auth user
- suspend/reactivate/remove

Entitlement:
- product
- status
- seat_limit
- source/expiry where available
- manual admin set/update for first launch

Seats:
- assigned count / limit
- list assignments
- safe assign from active member list
- release

Products for first launch:
- post
- dm
- ocr

Do NOT create a separate orc commercial product code.

ABSOLUTE RULES:
- No Production deploy.
- Do NOT join Seat users by array position.
- Do NOT infer user_id from email.
- Do NOT direct-read protected tables from client.
- Use only canonical platform_admin RPCs.
- Do NOT weaken Hub/admin route guards.
- Preserve unrelated Launcher dirty WIP with isolated worktree/branch.
- Payment/checkout/webhook automation is out of scope.

1. PREFLIGHT
Inventory current Launcher Hub customer/admin/Seat Workbench files.
Identify current dirty WIP vs committed source.
Create isolated branch/worktree.

2. MEMBER LIST
Use:
admin_list_customer_members(customer_id)

Display:
- email
- role
- status
- safe stable user_id internally for actions

Never expose raw IDs unnecessarily in primary UI, but keep exact IDs in action payloads.

3. MEMBER LIFECYCLE UI
Implement:
- add existing Auth user
- suspend
- reactivate
- remove

Show clear confirmation for destructive/suspension actions.
After suspend/remove, refresh Seat counts/list.

4. CUSTOMER LIFECYCLE UI
Implement:
- update supported fields
- suspend
- reactivate

Clearly show suspended status.

5. ENTITLEMENT UI
For post/dm/ocr:
- show effective status
- seat_limit
- source/expiry if returned
- allow platform_admin manual set/update required for first launch

Do not build checkout/payment flow.

6. SEAT ASSIGN
Enable safe Assign only from canonical active customer members returned by server.

Before assign:
- member active
- product entitlement effective
- user not already assigned

Call canonical assign_product_seat.
Handle:
- already_assigned
- seat_limit reached
- inactive member
- no entitlement
- admin-required
- other canonical errors

7. SEAT RELEASE
Retain/complete existing release flow through release_product_seat.
Refresh state after release.

8. SECURITY / UX
- platform_admin only
- ordinary user cannot open/use Workbench
- no service-role in frontend
- no token/secret logging
- no cross-customer stale state after switching customers
- disable action buttons while mutation in flight
- show canonical error rather than generic silent failure

9. LOCAL TESTS
Required:
- customer/member rendering
- exact user_id action payload
- no email heuristic
- assign success/failure
- release
- suspend/member Seat refresh
- customer status
- product isolation post/dm/ocr
- non-admin denied
- stale customer state cleared

Run existing Hub tests plus new tests.

10. STAGING ADMIN E2E
ONLY after W2-PLATFORM-S1 PASS.

Use a platform_admin Staging session.

Verify:
A. create/select test customer
B. list exact members
C. set product entitlement/seat_limit
D. assign one Seat
E. release it
F. suspend member and observe automatic Seat release
G. reactivate does not auto-assign
H. customer suspend blocks commercial access
I. reactivate behaves according to canonical contract

Use test-only customer data.
Do not alter Production.

11. CANDIDATE
Create isolated commit containing only Workbench implementation/tests.
Do not deploy Production.

12. FINAL REPORT
Return:
SAFE_MEMBER_ASSIGN_UI =
MEMBER_LIFECYCLE_UI =
CUSTOMER_LIFECYCLE_UI =
ENTITLEMENT_UI =
SEAT_ASSIGN =
SEAT_RELEASE =
NON_ADMIN_DENIED =
LOCAL_TESTS =
STAGING_ADMIN_E2E =
PRODUCTION_MUTATED = NO
DEPLOYED_PRODUCTION = NO

If complete:
RESULT = PASS
LAUNCHER_SEAT_WORKBENCH_IMPLEMENTED = YES
READY_FOR_LAUNCHER_PRODUCTION_RELEASE_CANDIDATE = YES

If Platform-S1 not yet PASS:
RESULT = PARTIAL_PASS
LOCAL_IMPLEMENTATION = PASS
STAGING_ADMIN_E2E = NOT_RUN
READY_FOR_LAUNCHER_PRODUCTION_RELEASE_CANDIDATE = NO

Do not deploy Production.
