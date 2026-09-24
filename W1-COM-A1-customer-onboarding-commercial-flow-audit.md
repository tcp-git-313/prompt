# W1-COM-A1 — Customer Onboarding Commercial Flow Readonly Audit

ROLE: COMMERCIAL CUSTOMER ONBOARDING CONTRACT AUDIT OWNER

MODE: READ-ONLY AUDIT / NO IMPLEMENTATION

WORKSPACE:
F:\00-Ticenpi-SaaS

PRIMARY SYSTEMS:
- ticenpi-platform Commercial Core
- Launcher / Workbench
- Post
- DM
- ORC
- Supabase Auth integration
- existing admin RPCs / provisioning helpers

GOAL:
Define the exact canonical customer onboarding flow required to commercially sell Post, DM and ORC without manual Supabase table editing.

Target business flow:

Customer signs up / exists
→ Customer entity
→ Customer membership
→ Product entitlement
→ seat_limit
→ assign Seat
→ product-specific data/tenant provisioning
→ product access

This task must identify what already exists, what is missing, and which system owns each step.

Do NOT implement.
Do NOT mutate Staging/Production.
Do NOT create users/customers/entitlements/Seats.
Do NOT modify Launcher.

## 1. INVENTORY CANONICAL WRITE PATHS

For each object/action, identify the existing supported admin RPC/API/helper:

- create customer
- update customer
- add member
- remove/suspend member
- create entitlement
- update entitlement
- set seat_limit
- assign product Seat
- release product Seat
- list Seat assignments
- provision Post product tenant/data mapping
- provision DM data scope if applicable
- provision ORC data scope if applicable

For each record:
OWNER SYSTEM
RPC/API/SYMBOL
AUTHORIZATION
INPUTS
OUTPUTS
STAGING/PRODUCTION availability

If missing:
MISSING

## 2. AUTH USER CREATION / INVITATION

Determine the intended account flow:
- Google self-signup
- email/password
- admin invite
- invite link
- other

Identify who creates the Supabase Auth user and when.

Do not assume Launcher should create Auth users unless code supports it.

Map:
Auth User ↔ customer_members

## 3. CUSTOMER CREATION OWNERSHIP

Determine whether Launcher Workbench already creates/manages customers or only reads them.

Identify:
- current UI
- current admin RPCs
- missing write paths

## 4. ENTITLEMENT / PLAN OWNERSHIP

Determine how Post/DM/ORC entitlements are currently created:
- manual admin RPC
- payment webhook
- test fixture
- not implemented

Identify seat_limit source.

Do not invent billing behavior.

## 5. SEAT MANAGEMENT OWNERSHIP

Confirm canonical Seat owner:
Central Commercial Core

Map:
admin_list_product_seats
assign_product_seat
release_product_seat

Determine exactly what Launcher still needs for safe UI.

Check whether member user_id is safely available for assignment candidate selection.

If not, identify the smallest missing server contract.

## 6. PRODUCT-SPECIFIC PROVISIONING

For Post, DM, ORC separately:

After Central Seat assignment, what product-local provisioning is still required?

Examples:
- Post tenant_members
- DM RLS/customer context
- ORC tenant/data mapping

Do not assume all products have the same model.

Return one exact chain per product.

## 7. CUSTOMER OFFBOARDING / DOWNGRADE

Map canonical supported behavior for:
- remove member
- suspend member
- release Seat
- entitlement expiry
- reduce seat_limit
- customer disable

Confirm which actions are already enforced atomically and which require orchestration.

## 8. MINIMUM COMMERCIAL LAUNCH FLOW

Define the smallest operational flow required to onboard a paying customer for each product.

For each step specify:
SYSTEM
ACTOR
ACTION
CAN_AUTOMATE_NOW = YES/NO

The goal is to eliminate direct Supabase table editing for normal operations.

## 9. LAUNCHER WORKBENCH SCOPE

Define exactly what Launcher Workbench must support before commercial launch:

- customer lookup/create?
- member management?
- entitlement/plan display?
- seat_limit display?
- Seat assign/release?
- product provisioning?
- audit/history?

Separate:
MUST_HAVE_FOR_LAUNCH
NICE_TO_HAVE_LATER

## 10. FINAL REPORT

A. AUTH USER FLOW
B. CUSTOMER WRITE PATHS
C. MEMBERSHIP WRITE PATHS
D. ENTITLEMENT WRITE PATHS
E. CENTRAL SEAT WRITE PATHS
F. POST PROVISIONING
G. DM PROVISIONING
H. ORC PROVISIONING
I. OFFBOARDING / DOWNGRADE
J. LAUNCHER MUST-HAVE SCOPE
K. MISSING CONTRACTS
L. MINIMUM COMMERCIAL ONBOARDING SEQUENCE
M. SAFETY

SOURCE_FILES_MODIFIED = NO
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO

N. FINAL RESULT

RESULT = PASS/PARTIAL_PASS/BLOCKED
CUSTOMER_ONBOARDING_FLOW_MAPPED = YES/NO
MISSING_LAUNCH_BLOCKERS = exact list
READY_FOR_LAUNCHER_WORKBENCH_IMPLEMENTATION = YES/NO

Finish audit and stop.
