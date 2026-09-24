# P4C-A1 — Launcher / Studio Environment & Commercial Auth Readonly Audit

ROLE: LAUNCHER / STUDIO ENVIRONMENT & AUTH AUDIT OWNER

MODE: READ-ONLY AUDIT ONLY

WORKSPACE:
F:\00-Ticenpi-SaaS

PRIMARY:
- Launcher / Ticenpi Studio repo and runtime
- Post Staging runtime / handoff integration

GOAL:
Determine exactly why a user who is already valid in Post Staging Central Seat is shown by the installed Launcher / Ticenpi Studio as:

「此 Google 帳號尚未取得商用授權」

and why Post Staging cannot connect to Ticenpi Studio during Facebook account onboarding.

Do NOT fix anything in this task.
Do NOT change Launcher environment.
Do NOT change Supabase project.
Do NOT change Central Seat.
Do NOT change Post.
Do NOT change Production or Staging data.
Do NOT deploy.
Do NOT commit/push.

## KNOWN FACTS

Post Staging Central Seat authoritative E2E is already proven for the fixed regression users:

- assigned ordinary user → Post ALLOW
- unassigned ordinary user → Post DENY

Current Post Staging:
- Central Seat authoritative
- SEAT_POLICY=require
- tenant key disabled

The installed Launcher / Ticenpi Studio is the normal stable/production Launcher build.

Observed Launcher UI for the assigned Post Staging user:
- authenticated Google account is visible
- UI says: 「此 Google 帳號尚未取得商用授權」
- Studio does not become ready for Post Facebook account onboarding

Therefore the audit must determine whether this is:

A. expected Production-vs-Staging environment separation
B. Launcher still using an old commercial authorization contract
C. Launcher resolving a different user_id/session
D. Post Staging → Studio handoff has no environment contract
E. another exact root cause

Do not assume which one is true.

## 1. PREFLIGHT

Record:

Launcher repo:
- path
- branch
- HEAD
- git status --short
- git diff --stat

Post repo:
- branch
- HEAD
- relevant current Staging release identity

Installed Launcher:
- visible app version
- runtime config source
- installation/channel if discoverable
- stable/beta/dev channel if discoverable

Preserve all existing dirty WIP.

## 2. IDENTIFY LAUNCHER RUNTIME ENVIRONMENT

Determine from ACTUAL runtime/config/code:

- which Supabase URL/project ref Launcher uses
- whether it is Production or Staging
- whether environment is compile-time, runtime-config, env file, remote config, or hardcoded
- whether Launcher supports environment switching
- whether Studio/local service inherits the same environment
- whether stable Launcher is intentionally Production-only

Do NOT print secret values.

Report:

LAUNCHER_SUPABASE_PROJECT_REF =
LAUNCHER_ENVIRONMENT =
LAUNCHER_ENV_SOURCE =
LAUNCHER_ENV_SWITCH_SUPPORTED = YES/NO

## 3. IDENTIFY CURRENT LAUNCHER USER

Using safe local/session evidence only, determine:

- currently signed-in account label/email only if already visible in UI
- current Supabase auth user_id
- platform_role
- test_access if relevant
- session project/environment

Do NOT print:
- access token
- refresh token
- JWT
- cookies
- OAuth secrets

Compare the Launcher user_id with the known Post Staging assigned test user if possible.

Report:

LAUNCHER_AUTH_USER_ID =
LAUNCHER_USER_MATCHES_POST_STAGING_ASSIGNED_USER = YES/NO/UNKNOWN

If the same email resolves to a different user_id across Production vs Staging, state that explicitly.

## 4. TRACE LAUNCHER COMMERCIAL AUTH DECISION

Find the exact code path that renders:

「此 Google 帳號尚未取得商用授權」

Trace from:
Supabase session
→ commercial context query/RPC
→ entitlement/access evaluation
→ UI state/message
→ Studio ready/not-ready state

Identify exact RPCs/functions used.

Check whether Launcher currently uses:
- my_commercial_context()
- effective_entitlements()
- product_seat_status()
- legacy organization/membership model
- another contract

Do NOT infer.

Report exact files/symbols.

## 5. QUERY THE SAME COMMERCIAL CONTEXT LAUNCHER USES

Using the Launcher runtime's own project/environment and current authenticated user/session where safely possible:

Read the same commercial context the Launcher reads.

Do NOT switch environment.

Report normalized safe result:

CUSTOMER/MEMBERSHIP PRESENT = YES/NO
PRODUCT ENTITLEMENT PRESENT = YES/NO
POST ACCESS PRESENT = YES/NO
CENTRAL POST SEAT PRESENT = YES/NO/NOT_QUERIED
PLATFORM_ADMIN = YES/NO
LAUNCHER_FINAL_COMMERCIAL_DECISION = ALLOW/DENY

Do not print tokens or unnecessary PII.

## 6. COMPARE WITH POST STAGING

Compare:

Post Staging:
- Supabase project ref
- user_id
- customer membership
- Post entitlement
- Post Seat
- Post tenant mapping
- final access

Launcher:
- Supabase project ref
- user_id
- customer membership
- entitlement/access source
- Seat check if any
- final commercial decision

Produce a side-by-side table.

The key question:

IS THE LAUNCHER DENIAL EXPECTED BECAUSE IT IS PRODUCTION WHILE THE USER'S VALID COMMERCIAL/SEAT FIXTURE EXISTS ONLY IN STAGING?

Return:
ENVIRONMENT_MISMATCH_CONFIRMED = YES/NO

## 7. TRACE POST STAGING → STUDIO HANDOFF

Inspect the Post Facebook onboarding flow that produces:

「無法連接 Ticenpi Studio。請確認 Ticenpi Studio 已開啟並完成登入，再按下方按鈕重新偵測」

Determine:

- how Post detects local Studio
- local port/protocol
- handshake endpoint(s)
- what identity/auth data is exchanged
- whether Post sends environment information
- whether Studio reports its own environment
- whether Post Staging can distinguish Production Studio from Staging Studio
- whether the handoff requires the SAME user identity
- whether commercial auth is re-evaluated by Launcher before Studio is declared ready

Do not modify either side.

## 8. FACEBOOK FLOW BOUNDARY

Determine where the flow currently stops:

Post Staging
→ Studio detection
→ Studio auth/commercial gate
→ Facebook OAuth launch
→ Facebook callback
→ account binding

Return exactly the furthest reached stage.

Set:

FACEBOOK_OAUTH_ACTUALLY_REACHED = YES/NO

If NO, do not diagnose Facebook OAuth itself as broken.

## 9. ARCHITECTURE OPTIONS — ANALYSIS ONLY

Based on actual implementation, evaluate the smallest correct future model.

At minimum compare:

OPTION A:
Stable Launcher remains Production-only.
Post Staging uses a separate Staging/Dev Studio mode/runtime.

OPTION B:
One Launcher supports explicit Production/Staging environment switching with strong isolation.

OPTION C:
Post Staging is allowed to use Production Launcher without sharing commercial state.

For each:
- code/config scope
- auth/session implications
- risk of cross-environment contamination
- compatibility with auto-update/stable channel
- whether separate local ports/profile storage are required

Do NOT implement.

Prefer the model most consistent with current architecture and safest environment separation.

## 10. CENTRAL SEAT RELATION

Explicitly answer:

- Is Launcher denial caused by Central Seat not being implemented?
- Is Launcher already consuming Central Commercial/Seat?
- Is the observed denial simply because Production and Staging commercial data are separate?
- Does Launcher need Seat enforcement itself, or only enough commercial context to decide whether Studio may serve a product?
- Should Production Launcher ever read Staging Central Seat directly?

Required rule:
Production Launcher must NOT be changed to read Staging Supabase merely to make this test pass.

## 11. FINAL REPORT

### A. PREFLIGHT

### B. LAUNCHER RUNTIME ENVIRONMENT

LAUNCHER_VERSION =
LAUNCHER_ENVIRONMENT =
LAUNCHER_SUPABASE_PROJECT_REF =
LAUNCHER_ENV_SOURCE =
LAUNCHER_ENV_SWITCH_SUPPORTED =

### C. CURRENT LAUNCHER IDENTITY

LAUNCHER_AUTH_USER_ID =
PLATFORM_ROLE =
SESSION_ENVIRONMENT =
MATCHES_POST_STAGING_USER =

### D. COMMERCIAL AUTH PATH

Exact code/RPC chain:

### E. COMMERCIAL RESULT

CUSTOMER_MEMBERSHIP =
POST_ENTITLEMENT =
POST_SEAT =
LAUNCHER_FINAL_DECISION =

### F. POST STAGING COMPARISON

Table:
Dimension | Post Staging | Launcher

### G. ENVIRONMENT DIAGNOSIS

ENVIRONMENT_MISMATCH_CONFIRMED =
LAUNCHER_OLD_AUTH_CONTRACT_CONFIRMED =
USER_ID_MISMATCH_CONFIRMED =

ROOT_CAUSE = exact factual root cause

### H. STUDIO HANDOFF

Detection protocol:
Port:
Handshake:
Environment exchanged:
User identity matched:
Commercial auth rechecked:

### I. FACEBOOK FLOW

FACEBOOK_OAUTH_ACTUALLY_REACHED =
FURTHEST_STAGE_REACHED =

### J. RECOMMENDED FUTURE MODEL

OPTION =
WHY =
REQUIRED_FOLLOWUP_TASKS =

### K. CENTRAL SEAT CONCLUSION

POST_CENTRAL_SEAT_STAGING_STATUS = PASS
LAUNCHER_DENIAL_CAUSED_BY_POST_SEAT_FAILURE = YES/NO
PRODUCTION_LAUNCHER_SHOULD_READ_STAGING_SEAT = NO

### L. SAFETY

SOURCE_FILES_MODIFIED = NO
CONFIG_MODIFIED = NO
STAGING_DATA_MUTATED = NO
PRODUCTION_DATA_MUTATED = NO
DEPLOYED = NO
COMMIT_CREATED = NO
PUSHED = NO

### M. FINAL RESULT

If root cause is proven:

RESULT = PASS
LAUNCHER_ENV_AUTH_ROOT_CAUSE_PROVEN = YES
READY_FOR_LAUNCHER_STUDIO_ENVIRONMENT_FIX_PLAN = YES

If identity/environment cannot be proven:

RESULT = BLOCKED
LAUNCHER_ENV_AUTH_ROOT_CAUSE_PROVEN = NO
BLOCKER = exact missing evidence

Finish audit and stop.
