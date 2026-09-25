# PRODUCTION RESUME — DM OPTION A + OCR PROD SCHEMA + CONTINUE TO GO-LIVE

## OWNER
PRODUCTION RELEASE OWNER

## USER APPROVAL / AUTHORIZATION

The user has explicitly approved:

1. **DM Production = Option A**
   - Keep the real DM Production runtime on `DM_SEAT_POLICY=require`.
   - For the **synthetic DM auth smoke process only**, allow `DM_SEAT_POLICY=shadow`.
   - This is a narrowly scoped test-process exception, not a Production runtime policy change.
   - Do not weaken real Production seat enforcement.
   - Do not broaden the exception to Post/OCR/Launcher or general smoke behavior.

2. **OCR Production schema migration = APPROVED**
   - The user authorizes applying the OCR Production DB changes required by the current accepted OCR release:
     - `db/v2_schema.sql`
     - `migrate-from-v1.sql`
     - `20260923_orc_staging_contract.sql`
   - Existing Production OCR v1 data must be preserved and migrated.
   - The previously observed Production legacy OCR record count is 4; verify the actual count again immediately before migration.

3. **Continue automatically through the remaining Production release flow**
   - You are authorized to edit the exact required deployment/config/script files.
   - You are authorized to enter/run the required commands automatically.
   - You are authorized to commit and push the narrowly scoped release/deploy fixes required for this release.
   - You are authorized to run CI, promotion, Production deploy, DB migration, smoke, audit, and evidence recording.
   - Do **not** pause for routine confirmations inside this approved scope.
   - Stop only on a defined HARD STOP condition below.

4. **Do NOT implement DM Option B in this run**
   - Option B is a follow-up hardening task after the immediate Production rollout.
   - Final report must explicitly include:
     `DM_SEAT_B_FOLLOWUP=REQUIRED`
   - Option B means: make the synthetic DM seat test run under `require` with proper test/mock seat context, then rebuild and re-run Staging before future Production promotions.

---

# KNOWN RESUME STATE

Do not redo completed work unless verification shows it is invalid.

## Commercial Core

Known state:

```text
PRODUCTION_COMMERCIAL_CORE = PASS
BACKUP_VERIFIED            = YES
MIGRATIONS_APPLIED         = 000600 -> 000700 -> 000800
REGRESSION                 = PASS
ROLLBACK_VERIFIED          = YES
```

The Production Commercial Core backup/restore/regression/rollback rehearsal is already complete.

Do not reapply 000600/000700/000800 unless evidence proves they are missing.

## Post Production

Known state:

```text
POST_PRODUCTION = DEPLOYED
release = 20260925-035353
accepted staging digest = d6a0c592...
runtime validation = PASS
public domain = post.ticenpi.com
seat policy = require
commercial gate = enabled
auto tenant creation = disabled
auth fail-closed = PASS
Post DB migration = applied
rollback target = 20260917-150808
```

Post still needs real Production canary assigned/unassigned verification before evidence can be promoted from DEPLOYED to ACCEPTED.

Do not redeploy Post unless required by a newly discovered blocker.

## DM Production

Known state:

```text
DM_PRODUCTION_COMPOSE_READY = YES
DM release commit = b51bbab
DM CI = 4/4 GREEN
DM promote gate = PASS
DM Production = NOT DEPLOYED
```

The only known blocker is the synthetic auth smoke seat mode.

## OCR Production

Known state:

```text
OCR_AUTHORITATIVE_PORT = 9423
OCR_PRODUCTION = NOT DEPLOYED
Production v2 schema = not yet applied
Production legacy OCR rows observed = 4
```

The local deploy manifest was already corrected from the stale 9468 planning value to authoritative Production port 9423.

## Launcher

Known state:

```text
LAUNCHER_PRODUCTION = NOT STARTED
```

## Overall

```text
PRODUCTION_CANARY = NOT COMPLETE
GO_LIVE = NO
```

---

# PRIMARY GOAL

Resume from the current Production state and continue as far as technically safe, without asking routine questions:

```text
DM Option A approval
↓
repair/sync exact smoke gate
↓
deploy DM Production
↓
validate DM runtime + live seat behavior
↓
backup OCR Production
↓
apply OCR v2 schema/migration/contract
↓
verify migrated data
↓
prepare/deploy OCR Production
↓
validate OCR Production
↓
prepare/deploy Launcher Production
↓
run Product canaries
↓
promote evidence to ACCEPTED
↓
GO_LIVE only if every gate passes
```

---

# REPOS / PATHS

Expected Windows workspace root:

`F:\00-Ticenpi-SaaS`

Known important repos/paths include:

- Deploy:
  `F:\00-Ticenpi-SaaS\deploy`

- DM:
  `F:\00-Ticenpi-SaaS\TicenpiDM`

- OCR / Letter:
  inspect the actual current repo/path before writing; do not guess

- Platform:
  `F:\00-Ticenpi-SaaS\ticenpi-platform`

- DM Production release worktree:
  `F:\00-Ticenpi-SaaS\.release-worktrees\dm-production-20260925`

Known evidence:
- Commercial Core:
  `F:\00-Ticenpi-SaaS\deploy\db-backup\artifacts\production\w5a-20260925\commercial-core-evidence.json`
- Post evidence:
  `F:\00-Ticenpi-SaaS\.release-evidence\post\`

First inspect the actual current workspace, branches, HEADs, worktrees, dirty state, and release evidence before modifying anything.

---

# GLOBAL SAFETY RULES

## Preserve current release state

Do not:
- reset unrelated dirty WIP
- delete untracked files
- switch branches over active work
- rebuild a service that already has an accepted immutable Staging digest unless source changes truly require it
- substitute a new image for an accepted Staging image without repeating Staging acceptance
- modify Production secrets unless explicitly required and already authorized
- print secret values
- weaken runtime authentication/authorization

## Immutable promotion principle

For Post/DM/OCR/Launcher whenever an accepted Staging image exists:

```text
accepted Staging digest
→ same digest in Production
→ NO rebuild
```

If a code change is required after Staging acceptance:
- build a new image
- run Staging acceptance again
- only then promote that exact digest

Do not silently rebuild during Production promotion.

---

# PHASE 0 — PREFLIGHT / AUTHORITATIVE STATE

Before making changes:

1. Inspect:
   - repo paths
   - branches
   - HEADs
   - worktrees
   - git status
   - current services.yaml / manifest
   - current VPS active manifest
   - current Production runtimes
   - current evidence
   - current CI states

2. Verify:
   - Post is still on expected release/digest
   - DM commit `b51bbab` CI is green
   - OCR authoritative port is still 9423
   - Commercial Core migrations are present
   - no new operator/manual changes occurred after the recorded state

If observed state materially differs, adapt using current evidence and report it.

Do not redo completed phases merely because the prompt lists them.

---

# PHASE 1 — DM OPTION A: EXACT SYNTHETIC SMOKE EXCEPTION

## Intent

Production DM runtime must remain:

```text
DM_SEAT_POLICY=require
```

Only the **in-process synthetic auth smoke execution** may use:

```text
DM_SEAT_POLICY=shadow
```

This is user-approved.

## Critical smoke source of truth

Previously observed:
- VPS `/opt/ticenpi/deploy/critical-smoke.sh` differs from local deploy repo.
- VPS version is the actual runtime baseline.

Therefore:

1. Read the VPS current `critical-smoke.sh`.
2. Read the deploy repo version.
3. Diff them.
4. Use the **VPS current version as the functional baseline**.
5. Make the smallest possible change:
   - for service `dm` synthetic auth gate only
   - inject `DM_SEAT_POLICY=shadow` into that smoke process
6. Do not modify the running DM service env.
7. Do not change Post/OCR/Launcher smoke semantics.
8. Do not remove any security/auth checks.
9. Do not skip the synthetic auth gate.
10. Do not convert failure to pass.

The test must still execute; only its isolated seat policy context differs.

## Sync back to repo

After validating the exact VPS baseline + approved change:
- sync the resulting authoritative `critical-smoke.sh` back to the deploy repo
- preserve unrelated VPS-only legitimate logic
- do not overwrite newer VPS behavior with a stale repo copy

Run relevant tests/lint/diff checks.

Commit/push the exact scoped deploy-script change if required by the normal promotion gate.

Record in evidence:

```text
DM_RUNTIME_SEAT_POLICY=require
DM_SYNTHETIC_SMOKE_SEAT_POLICY=shadow
USER_APPROVED_OPTION_A=YES
```

---

# PHASE 2 — DM PRODUCTION PROMOTION

Known command:

```powershell
powershell -NoProfile -Command "cd F:\00-Ticenpi-SaaS\deploy; .\promote.ps1 dm production -SourceOverride F:\00-Ticenpi-SaaS\.release-worktrees\dm-production-20260925 -Yes"
```

Before executing:
- confirm DM release worktree is clean and points to intended commit
- confirm exact accepted digest
- confirm promotion gate uses `SourceOverride` correctly
- confirm no rebuild occurs
- confirm rollback target exists

Then run promotion automatically.

## DM post-deploy validation

Verify:

- health
- runtime release identity
- runtime commit
- exact image digest
- Production environment
- Production Supabase project
- `DM_SEAT_POLICY=require` in the real service
- commercial auth gate enabled
- auth fail-closed
- no test bypass
- no Staging test_access overlay
- public DM domain routing
- post-deploy audit
- rollback evidence

## DM live seat validation

The synthetic smoke is not sufficient.

Perform live E2E if credentials/session capabilities are available:

- assigned seat user:
  - authenticated
  - entitled
  - assigned
  - DM access succeeds

- unassigned user:
  - authenticated
  - otherwise eligible but without assigned seat
  - DM access denied as intended

- platform_admin behavior:
  verify documented intended behavior; do not invent exemption semantics

If real login needs human Google/Facebook interaction, MFA, CAPTCHA, or credentials unavailable to the agent:
- do not bypass
- record `MANUAL_CANARY_REQUIRED`
- continue other independent deployment work

---

# PHASE 3 — OCR PRODUCTION BACKUP BEFORE SCHEMA CHANGE

The user has authorized the OCR Production schema migration.

Before any OCR SQL:

1. Identify the exact Production Supabase project/database.
2. Verify it is Production, not Staging.
3. Take a fresh backup/export sufficient to restore:
   - OCR legacy tables
   - relevant v2 tables if partially present
   - functions/RPC definitions needed by OCR
4. Hash/record backup evidence.
5. Count current legacy OCR records again.

Expected historical observation:
`4 rows`

Do not assume 4 if current live count differs; record actual count.

If backup cannot be verified/restored or clearly validated, HARD STOP before mutation.

---

# PHASE 4 — OCR PRODUCTION SCHEMA MIGRATION

User-approved SQL set:

1. `db/v2_schema.sql`
2. `migrate-from-v1.sql`
3. `20260923_orc_staging_contract.sql`

## Before apply

For each file:
- locate the authoritative source file
- compute sha256
- compare with the exact file used/accepted in Staging where evidence exists
- inspect for Production-specific unsafe assumptions
- check idempotency / transaction semantics
- check dependencies/order

Do not silently alter SQL unless a clearly necessary Production compatibility repair is found.

If SQL must be changed:
- make the smallest change
- explain reason in evidence
- test on a restored Production backup copy first
- do not mutate live Production until rehearsal passes

## Rehearsal

Restore fresh Production backup into an isolated database/environment and apply:

```text
v2_schema
→ migrate-from-v1
→ staging_contract
```

Verify:
- schema succeeds
- 4/current legacy rows migrate correctly
- no data loss
- keys/relationships remain correct
- expected RPC/functions exist
- RLS / grants match intended Production behavior
- OCR app contract tests pass

Only then apply to live Production.

## Live apply

Apply in the same tested order.

Capture:
- start/end time
- SQL file hashes
- pre/post row counts
- affected tables/functions
- migration output/errors
- post-migration fingerprint where possible

Do not apply Commercial Core 000600/000700/000800 again.

---

# PHASE 5 — OCR PRODUCTION APP PREP / PROMOTION

After DB contract is verified:

1. Locate OCR's Staging-accepted release evidence.
2. Identify:
   - accepted commit
   - accepted frontend/backend digest(s)
   - accepted runtime config
   - accepted Production-capable compose pattern
3. Verify Production authoritative port:
   `9423`
4. Prepare Production compose/config from accepted artifact(s).
5. Do not use stale `9468`.
6. Ensure:
   - Production Supabase
   - Production environment
   - no Staging-only test overlay
   - correct auth/entitlement/seat behavior as applicable
   - no source build during promotion if immutable accepted digest exists

If OCR Production compose/config does not yet exist:
- create the minimal Production config based on the accepted Staging release and current deploy architecture
- run config validation
- commit/push the scoped deployment config as required
- wait for required CI
- promote automatically when green

## OCR deploy validation

Verify:
- runtime release
- commit
- digest
- health
- Production DB
- Production Supabase
- port 9423
- Cloudflare/public domain
- OCR API functionality
- upload/extraction basic smoke
- quota RPC/contract
- migrated legacy data visible/usable where relevant
- rollback target/evidence

---

# PHASE 6 — LAUNCHER PRODUCTION

Once Post, DM, and OCR are Production-deployed or independently blocked only by manual canary:

1. Inspect Launcher current accepted Staging evidence.
2. Inspect current Production Launcher state.
3. Confirm Central Commercial Core integration.
4. Confirm product cards/routes for:
   - Post
   - DM
   - OCR/Letter
   - other existing products
5. Prepare Production config using accepted immutable artifacts.
6. Run CI/config gates.
7. Promote Launcher Production automatically if all gates pass.

Do not alter product business logic simply to make promotion easier.

Verify:
- Production auth
- Workbench/Launcher loads
- product cards visible according to entitlement
- routes point to Production domains
- logout/session behavior
- no Staging URLs
- no localhost URLs
- public domain/Cloudflare health

---

# PHASE 7 — CANARY / LIVE E2E

Run every canary that can be automated with existing authorized credentials/session mechanisms.

Do not print credentials or tokens.

## Post

Need:
- assigned seat access success
- unassigned seat access denied
- Production login
- basic product operation

If human login is required:
`POST_MANUAL_CANARY_REQUIRED=YES`

## DM

Need:
- Production login
- assigned seat success
- unassigned seat denied
- basic design read/write
- A4/editor basic operation
- scraper/extraction basic operation if safe

If human login is required:
`DM_MANUAL_CANARY_REQUIRED=YES`

## OCR

Need:
- Production login
- basic OCR/Letter operation
- Production DB contract
- migrated data access where applicable

If human login is required:
`OCR_MANUAL_CANARY_REQUIRED=YES`

## Launcher

Need:
- Production login
- Workbench loads
- entitlement-based product navigation
- no Production route regressions

---

# PHASE 8 — EVIDENCE PROMOTION

For each product maintain explicit state:

```text
NOT_STARTED
READY
DEPLOYED
ACCEPTED
BLOCKED
```

Do not label ACCEPTED until required automated + manual canary criteria are truly satisfied.

Update evidence files with:
- release
- commit
- digest
- environment
- runtime identity
- auth/seat policy
- migration evidence
- smoke result
- rollback target
- canary status
- remaining manual action

---

# PHASE 9 — GO-LIVE DECISION

Set:

`GO_LIVE=YES`

only if:

- Commercial Core = PASS
- Post = ACCEPTED
- DM = ACCEPTED
- OCR = ACCEPTED
- Launcher = ACCEPTED
- required Production canaries = PASS
- rollback paths verified
- no critical blocker remains

If all technical deployment is complete but human login canaries remain:

```text
GO_LIVE=NO
TECHNICAL_DEPLOYMENT_COMPLETE=YES
MANUAL_CANARY_REQUIRED=YES
```

Do not fake GO_LIVE.

---

# AUTOMATIC EXECUTION POLICY

Within this prompt's approved scope, do not stop to ask things like:

- "Should I edit the compose?"
- "Should I run the SQL?"
- "Should I push this config fix?"
- "Should I execute the promotion?"
- "Should I type this command?"
- "Should I continue to OCR?"
- "Should I continue to Launcher?"

The answer is already YES if the step is required by this plan and passes its safety gate.

You may automatically:
- edit scoped scripts/config
- run PowerShell/bash commands
- run SQL migrations
- create backups
- commit scoped release/deploy fixes
- push scoped release/deploy fixes
- wait for CI
- rerun failed deterministic checks after fixing the exact issue
- promote Production
- run verification
- record evidence

Do not ask for permission again for the exact DM Option A or the exact three OCR SQL files.

---

# HARD STOP CONDITIONS

Stop and report only if one of these occurs:

1. Fresh OCR Production backup cannot be verified.
2. OCR rehearsal shows data loss/corruption.
3. Actual OCR migration requires SQL beyond the three explicitly approved files and it is materially schema-changing/destructive.
4. A required fix would weaken real Production auth/seat policy.
5. DM real runtime would need `shadow` instead of `require`.
6. Security gate requires bypass beyond the exact user-approved DM synthetic-smoke exception.
7. Accepted Staging digest cannot be identified and promotion would require an unvalidated rebuild.
8. Production secret value is missing and cannot be sourced from the existing authorized secret system.
9. Irreversible/destructive operation is required without verified rollback.
10. Human Google/Facebook/MFA/CAPTCHA interaction is the only remaining blocker.
11. A different operator changed Production state such that current evidence is no longer trustworthy.
12. CI reveals a real product/security regression rather than a release-pipeline/config issue.

Routine CI waiting, config edits, migration execution, promotion, smoke, and evidence updates are NOT hard stops.

---

# DM OPTION B — FOLLOW-UP, NOT NOW

Do not implement Option B in this run.

At the end, always output:

```text
DM_SEAT_B_FOLLOWUP=REQUIRED
```

Follow-up B goal:

```text
synthetic DM auth/seat gate
→ DM_SEAT_POLICY=require
→ deterministic test/mock seat context
→ assigned PASS
→ unassigned DENY
→ new image
→ full Staging acceptance
→ future Production promotion uses require even in synthetic seat test
```

This is the long-term cleanup after the immediate release is stabilized.

---

# FINAL REPORT FORMAT

Return one consolidated report:

```text
PRODUCTION RELEASE RESUME REPORT

COMMERCIAL CORE
Status:
Evidence:

POST
Status:
Release:
Digest:
Runtime:
Canary:

DM
Status:
Release:
Commit:
Digest:
Real runtime seat policy:
Synthetic smoke seat policy:
Smoke:
Live seat E2E:
Canary:
Rollback:

OCR
Status:
Backup:
Pre-migration legacy rows:
SQL 1 hash/result:
SQL 2 hash/result:
SQL 3 hash/result:
Post-migration rows:
Contract verification:
Release:
Digest:
Port:
Runtime:
Canary:
Rollback:

LAUNCHER
Status:
Release:
Runtime:
Canary:

GO_LIVE:
TECHNICAL_DEPLOYMENT_COMPLETE:
MANUAL_CANARY_REQUIRED:

Unexpected blockers:
- ...

DM_SEAT_B_FOLLOWUP=REQUIRED

Docker rebuild after accepted Staging digest:
- Post:
- DM:
- OCR:
- Launcher:

Production security weakened:
NO

Secrets printed:
NO
```
