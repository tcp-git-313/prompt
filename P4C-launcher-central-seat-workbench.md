# P4C — Launcher Central Seat Workbench Integration

ROLE: LAUNCHER CENTRAL SEAT WORKBENCH OWNER

MODE: LOCAL IMPLEMENTATION ONLY

REPO:
F:\00-Ticenpi-SaaS\Ticenpi-Launcher

GOAL:
Integrate the canonical Central Seat admin read/assign/release contract into Launcher Workbench as the platform-admin management surface.

This task is UI/client integration only.

DO NOT modify Central Seat SQL.
DO NOT modify ticenpi-platform.
DO NOT modify Post or DM.
DO NOT deploy.
DO NOT touch Production.
DO NOT commit/push unless explicitly instructed.

## VERIFIED SERVER CONTRACT

Live Staging already has:

admin_list_product_seats(
  p_customer_id uuid,
  p_product_code text
)

and existing:

assign_product_seat(
  p_customer_id uuid,
  p_product_code text,
  p_user_id uuid
)

release_product_seat(
  p_customer_id uuid,
  p_product_code text,
  p_user_id uuid
)

Canonical source remains Central Supabase.

admin_list_product_seats returns enough data for:

- customer_id
- product_code
- effective_entitlement_id
- effective_status
- seat_limit
- assigned_count
- is_full
- assignments
  - assignment_id
  - user_id
  - assigned_at
  - member_status when available

IMPORTANT CURRENT LIMITATION:

admin_customer_detail() members do NOT currently expose user_id.

Therefore:

- DO NOT match assignment user_id to member email/name by array index
- DO NOT match by email heuristics
- DO NOT invent identity joins
- display opaque user_id + member_status when identity cannot be safely resolved
- leave a clear future enhancement boundary for adding user_id to admin_customer_detail()

## 1. PREFLIGHT

Record:

- branch
- HEAD
- git status --short
- git diff --stat

Known branch baseline may be:

release/launcher-0.2.23-baseline

Known HEAD baseline may be:

7e140f34f9a990e901860c4e4f30267e0fddc5c8

Do not assume worktree is unchanged.

Preserve all existing dirty WIP.

Do not reset/clean/stash/rebase.

Before editing, identify:

- current Workbench admin files
- existing customer detail UI
- current entitlement/seat_limit display
- current admin RPC wrapper/helper
- platform_admin gating
- any existing assign/release UI or placeholder
- dirty-file collision risk

If the required UI files have overlapping unknown dirty WIP:
STOP and report exact collision.

## 2. CURRENT WORKBENCH FLOW

Trace current flow:

platform_admin login
→ customer list/detail
→ entitlement display
→ Workbench actions

Identify the smallest existing admin area where Product Seat management belongs.

Do not create a second admin application.

## 3. ADD CENTRAL SEAT READ CLIENT

Add a typed/client wrapper for:

admin_list_product_seats(customer_id, product_code)

Use the signed-in platform_admin Supabase session.

Do NOT use:
- service_role
- raw DB connection
- direct SELECT from product_seat_assignments
- custom backend proxy unless already required by Launcher architecture

Normalize server errors without hiding them.

## 4. SEAT PANEL

For selected customer + product, show:

- product code/name
- effective entitlement status
- seat limit
- assigned count
- remaining capacity if derivable
- full/not-full state
- assignment rows

Assignment row display:

- user_id
- member_status
- assigned_at

If email/name cannot be safely resolved:
show user_id only.

Do not fake friendly identity labels.

## 5. ASSIGN FLOW

Use canonical:

assign_product_seat(customer_id, product_code, user_id)

The selectable candidates must come only from an existing authoritative customer-member source already available to the platform-admin UI.

If the current member data does not expose user_id:
DO NOT implement an unsafe selector.

In that case:

- keep assignment action disabled or clearly unavailable
- explain in UI/state that user identity cannot yet be safely resolved
- still implement read/list and release for existing assignments if safe

Never infer user_id from list position or email.

If an existing safe user_id source already exists elsewhere in Launcher, document and use it.

## 6. RELEASE FLOW

Use canonical:

release_product_seat(customer_id, product_code, user_id)

Release is allowed only for an assignment returned by admin_list_product_seats().

After success:
refresh list from server.

No optimistic local truth.

## 7. SERVER-TRUTH RULE

The UI must never compute authoritative seat occupancy locally.

After assign/release:

call admin_list_product_seats() again

and render returned truth.

Do not mutate local counters as authoritative state.

## 8. STATES

Handle explicitly:

- loading
- no effective entitlement
- zero assignments
- partial capacity
- full capacity
- RPC unauthorized
- customer_not_found
- product_not_found
- admin_required
- network/server error
- assignment conflict/error
- release error

Do not expose raw SQL/PostgREST internals to normal UI.

## 9. PLATFORM ADMIN SECURITY

Seat Workbench must only be visible/usable under the existing platform_admin admin surface.

Do not add client-side role assumptions as the only protection.

Server RPC remains authoritative.

No service-role key in browser/Electron renderer.

No direct table access.

## 10. PRODUCT ISOLATION

Seat panel must be product-scoped.

Post assignment rows must not appear under DM.

DM assignments must not appear under Post.

Switching selected product must fetch fresh server data.

## 11. TESTS

Add/extend Launcher tests for:

1. platform_admin can open Seat panel
2. non-admin cannot access Seat management UI
3. list RPC uses selected customer + product
4. zero assignment state
5. partial state
6. full state
7. product isolation
8. release uses exact returned user_id
9. release refreshes server truth
10. no direct product_seat_assignments table access
11. no service-role in client
12. no email/name guessed from array order
13. missing safe member user_id does not enable unsafe assign selector
14. RPC errors render safely
15. existing Workbench entitlement display remains functional

If a safe candidate user_id source already exists:
add assign success/failure tests.

If it does not:
test assignment UI remains safely disabled.

## 12. UI SCOPE

Keep the UI compact.

Preferred shape:

Customer Workbench
→ Product row/card
→ Seats

Example:

Post
Entitlement: active
Seats: 2 / 5

Assignments:
[user UUID] active [Release]
[user UUID] active [Release]

[Assign Seat]

If safe user_id candidate source is unavailable:

[Assign Seat unavailable — member user identity not yet exposed]

Do not redesign unrelated Launcher pages.

## 13. NO DEPLOY

This task ends after:

- implementation
- local tests
- local UI verification if available

Do not deploy Staging.

A separate task will perform Launcher Staging deployment/E2E after this is reviewed.

## 14. FINAL REPORT

### A. PREFLIGHT

Branch:
HEAD:
Dirty collision:

### B. CURRENT WORKBENCH ARCHITECTURE

### C. FILES CHANGED

### D. CENTRAL RPC CLIENT

Read:
Assign:
Release:

### E. UI

Displayed fields:
Safe identity source:
Assign enabled:
Release enabled:

### F. IDENTITY LIMITATION

ADMIN_CUSTOMER_DETAIL_HAS_USER_ID = YES/NO

If NO:

UNSAFE_EMAIL_OR_INDEX_JOIN_USED = NO
ASSIGN_SELECTOR_SAFELY_DISABLED = YES/NO

### G. TESTS

### H. SECURITY

DIRECT_SEAT_TABLE_ACCESS = NO
SERVICE_ROLE_IN_CLIENT = NO
SERVER_TRUTH_AFTER_MUTATION = YES
PLATFORM_ADMIN_SERVER_AUTHORITY_PRESERVED = YES

### I. SAFETY

PLATFORM_SQL_MODIFIED = NO
POST_MODIFIED = NO
DM_MODIFIED = NO
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO
COMMIT_CREATED = NO
PUSHED = NO

### J. FINAL RESULT

If read/list/release UI is complete but safe assign candidate identity is unavailable:

RESULT = PARTIAL_PASS
LAUNCHER_SEAT_READ_UI = PASS
LAUNCHER_SEAT_RELEASE_UI = PASS
LAUNCHER_SEAT_ASSIGN_UI = BLOCKED_SAFE
BLOCKER = admin_customer_detail does not expose safe member user_id
READY_FOR_LAUNCHER_STAGING_READ_E2E = YES

If safe assign source exists and complete:

RESULT = PASS
LAUNCHER_SEAT_WORKBENCH = PASS
READY_FOR_LAUNCHER_STAGING_E2E = YES

If collision prevents safe implementation:

RESULT = BLOCKED
BLOCKER = exact file/symbol collision

Stop after local implementation and tests.
