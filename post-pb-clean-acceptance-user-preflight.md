# POST P-B — CLEAN STAGING ACCEPTANCE USER PREFLIGHT

## TASK
P-B — Clean Staging Acceptance User Preflight

## ROLE
POST ACCEPTANCE IDENTITY PREFLIGHT OWNER

## RECOMMENDED MODEL
LUNA Medium

## ESTIMATED ENGINEERING SIZE
Small to Medium

## MODE
READ-ONLY PREFLIGHT — ZERO MUTATION

## OBJECTIVE
Identify whether Post Staging already has one clean, existing Auth identity suitable for the next Post Acceptance run, independent of ambiguous legacy data associated with `tcp.a2026i@gmail.com`.

Produce evidence-backed identity and readiness findings, or specify the minimum later action needed. This task does not create or repair an identity and does not run acceptance E2E.

## SCOPE AND ENVIRONMENT
- Work only on the Post repository and Staging resources.
- Repository: `F:\00-Ticenpi-SaaS\TicenpiPost`
- The known Staging Supabase project is `jlsqjvehwblkeuycjoyj`. Verify the active project from current, non-secret configuration before any data inspection. If configuration is missing, ambiguous, or points elsewhere, stop with `HARD_STOP`.
- Never inspect or contact the Production Supabase project for this task.
- Do not infer current runtime state from old notes, a project name alone, or a configured URL that cannot be tied to the active project. Record what was actually observed and when.
- Do not print, copy into the report, or persist access tokens, JWTs, passwords, OAuth credentials, service-role keys, or other secrets. Redact secret values in command output and artifacts.

This is a read-only task and may run in parallel with:
- platform_admin identity audit
- Central Seat work
- extension/Launcher readiness audit

## REQUIRED READING
Before inspecting identities, read:
- `_charter/adr/ADR-001-tenant-identity-entry-point.md`
- `_charter/adr/ADR-002-central-commercial-post-tenant-seat.md`
- the current Post Acceptance / Staging handoff or checklist, if one exists
- directly relevant project memory or evidence records, if available

Treat ADR-002 v3 (accepted 2026-09-22) as the current design contract unless the repository contains a newer accepted decision. Verify its current status. Relevant rules include:
- 1 Auth user = 1 Post tenant; identity is the stable Auth `user_id`, never email alone.
- With commercial gate enabled, access requires active commercial eligibility; for company customers, Seat policy may also require an assigned Seat.
- `TICENPI_SEAT_POLICY=skip` is permitted only in local/Staging and is the temporary pre-Central-Seat Acceptance path when explicitly configured and approved. Never assign a Seat in this task.
- `GET /api/session/state` is read-only. `POST /api/session/bootstrap` is the explicit tenant creation path. This task must not call bootstrap.
- Staging acceptance is not Production readiness.

If a newer accepted ADR, task handoff, or runtime configuration conflicts with these notes, stop and report the conflict; do not resolve it by changing settings.

## ABSOLUTE NO-MUTATION RULES
Do not:
- create, invite, delete, disable, update, or sign in as any Auth user
- issue OAuth login or start a browser session
- create, change, assign, or release customer memberships, entitlements, products, test-access flags, Post tenant mappings, or Seats
- call any write RPC, bootstrap endpoint, migration, seed, repair, or application write API
- change Supabase settings, secrets, environment variables, or project configuration
- modify source, local data, test data, or repository files
- commit, push, deploy, or open a PR
- use credentials against Production

Use read-only queries only. If the available access path cannot guarantee read-only behavior, do not use it; report the missing evidence. Do not expose secrets while proving which project or account is active.

## PREFLIGHT STEPS

### 1. Establish the current source and environment
Record:
- current date/time and timezone
- repository path and current Git branch/commit
- clean/dirty working-tree state (do not alter it)
- accepted ADR version/status
- Staging project identity proven from current configuration or dashboard context, with secret values redacted
- whether current Post Staging runtime/config evidence is available and its timestamp

Stop with `HARD_STOP` if the active target cannot be proven to be Staging. Do not query Auth or database data before resolving that boundary.

### 2. Read identity and tenant rules
From the accepted ADR and implementation, determine the actual current contracts for:
- Auth identity key and duplicate-email handling
- `tenant_members` uniqueness and whether a user can already have a tenant
- Post tenant ownership and legacy/test-tenant definitions
- commercial context and the current Staging pre-Seat path
- the effect of `TICENPI_COMMERCIAL_GATE_ENABLED`, `TICENPI_AUTO_TENANT_BOOTSTRAP`, and `TICENPI_SEAT_POLICY`

Distinguish design requirements from observed current implementation/configuration. Do not claim a setting is active based on ADR text alone.

### 3. Discover existing candidate identities using read-only evidence
Prefer an already designated Staging test/acceptance identity. Consider alternatives only if their Staging purpose and ownership are clear.

For each plausible candidate, establish as much of the following as the available read-only access allows:
- candidate email and stable Auth `user_id`
- exact number of Auth identities with that email, including deleted/banned state if the API exposes it
- whether the identity belongs to Staging and is not Production-only
- active customer membership count and customer type
- active Post entitlement / commercial-context result and source
- existing Post tenant count, tenant ID(s), and owner mapping
- whether any mapping or tenant is known to be legacy, test-only, conflicting, or ambiguous
- relevant `test_access` state, if applicable
- whether the account is the excluded legacy identity `tcp.a2026i@gmail.com`

Candidate status:
- **CLEAN** only when evidence proves one unambiguous Staging Auth user, no conflicting/legacy Post tenant association, and a viable current Staging commercial path.
- **NOT CLEAN** when evidence demonstrates duplicate identity, conflicting or legacy tenant ownership, Production-only use, or unavailable commercial eligibility.
- **UNKNOWN** when any required property cannot be proven. Do not convert missing evidence into a pass.

Do not choose a candidate merely because it is an administrator or has broad privileges. `platform_admin` does not by itself prove ordinary-user Acceptance readiness.

Do not log in, create a session, call `/api/session/bootstrap`, or mutate any state while assessing candidates.

### 4. Assess the pre-Seat Staging path
Determine from current Staging configuration and available read-only evidence whether a CLEAN candidate can use the explicitly approved pre-Seat path. Confirm separately:
- commercial gate state
- auto-tenant-bootstrap state
- Seat policy state
- whether the candidate's customer/product context qualifies under the current implementation
- whether the first successful product flow would be Google Login → Commercial Auth → `POST /api/session/bootstrap` → exactly one Phase 2 tenant → Main UI

This is a readiness assessment only. Do not execute that flow. If any required flag or business state is unknown, report `UNKNOWN` and do not mark the path safe.

Record whether this same identity is suitable for later:
- individual entitlement E2E
- company assigned-Seat E2E

These use cases may require different identities. Seat E2E is `NO` unless an existing assignment and all required Central Seat conditions are proven; do not create one.

### 5. If no CLEAN candidate is proven
Do not create a user or alter access. Prepare the minimum later action without assuming an email address or inventing user IDs:
- whether the user must first complete a user-controlled Google sign-up in Staging
- the minimum Central customer membership/product entitlement required
- whether customer type and Seat policy imply a later Seat assignment
- what exact read-only checks must pass before a separately authorized writer action
- expected first Post flow

Clearly separate user action from a future authorized writer action. No writer action is part of this task.

## EVIDENCE QUALITY
For each conclusion, cite the exact source (file and line, sanitized query/result, or dashboard/API observation), timestamp, and environment. Keep raw evidence local and avoid saving secrets or unnecessary personal data.

Do not treat these as sufficient by themselves:
- historical memory or handoff claims
- static source code when claiming current runtime configuration
- a successful identity lookup without checking tenant ownership and commercial eligibility
- an Auth email match without stable `user_id` and duplicate count
- `platform_admin` privilege
- local/fixture evidence as Staging evidence
- readiness assessment as completed login or human E2E

If a database/API query is unavailable or its read-only nature is uncertain, mark the affected fields `UNKNOWN`; do not improvise a privileged query.

## REQUIRED FINAL REPORT
Return a concise report using this exact structure:

```text
PHASE = CLEAN ACCEPTANCE USER PREFLIGHT
RESULT = PASS / NEED_NEW_USER / HARD_STOP

ENVIRONMENT = STAGING / UNKNOWN
STAGING_PROJECT_ID = <verified ID or UNKNOWN>
EVIDENCE_AS_OF = <timestamp and timezone>
REPOSITORY_COMMIT = <sha>
WORKTREE = CLEAN / DIRTY

RECOMMENDED_ACCEPTANCE_EMAIL = <value or NONE/UNKNOWN>
RECOMMENDED_USER_ID = <stable Auth user_id or NONE/UNKNOWN>
DUPLICATE_EMAIL_COUNT = <integer or UNKNOWN>
EXISTING_TENANT_COUNT = <integer or UNKNOWN>
EXISTING_TENANT_IDS = <IDs or NONE/UNKNOWN>
LEGACY_DATA_PRESENT = YES / NO / UNKNOWN
COMMERCIAL_CONTEXT = PASS / FAIL / UNKNOWN
COMMERCIAL_GATE_ENABLED = YES / NO / UNKNOWN
AUTO_TENANT_BOOTSTRAP_ENABLED = YES / NO / UNKNOWN
SEAT_POLICY = require / skip / UNKNOWN
SAFE_FOR_PRE_SEAT_ACCEPTANCE = YES / NO / UNKNOWN
SAFE_FOR_INDIVIDUAL_E2E = YES / NO / UNKNOWN
SAFE_FOR_LATER_SEAT_E2E = YES / NO / UNKNOWN

EVIDENCE =
- <claim>: <source, environment, timestamp>

IF_NEW_USER_REQUIRED:
USER_ACTION = <specific action or NONE>
MINIMUM_FUTURE_WRITER_ACTION = <specific action or NONE>
PRECONDITIONS_FOR_WRITER = <checks or NONE>

BLOCKERS =
- <specific blocker or NONE>

NEXT = <single concrete next step>
MUTATION = NONE
COMMIT = NONE
PUSH = NONE
```

Use:
- `PASS` only when a CLEAN candidate and current pre-Seat Staging readiness are proven.
- `NEED_NEW_USER` when no CLEAN candidate exists or can be proven, but Staging boundary and required next action are clear.
- `HARD_STOP` when the Staging boundary is unproven, requirements conflict, or continuing could risk Production or mutation.

Never report Acceptance E2E as completed by this preflight.
