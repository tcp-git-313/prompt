# TICENPIPOST — PRODUCTION + STAGING RUNTIME TRUTH AUDIT

## ROLE
TicenpiPost Runtime Configuration Truth Auditor

## RECOMMENDED MODEL
LUNA Medium

## MODE
READ-ONLY RUNTIME AUDIT
NO SOURCE MODIFICATION
NO DEPLOY
NO RESTART
NO FACEBOOK WRITE ACTION

Project:
F:\00-Ticenpi-SaaS\TicenpiPost

Relevant prior reconciliation:
ticenpipost-production-baseline-reconciliation-recovery-prioritization.md

Known baseline from the latest reconciliation report:

- Production public URL: https://post.ticenpi.com
- Production runtime previously reported:
  - environment=production
  - release=20260925-035353
  - commit=27e4efedbd91
  - port=9418
- Production expected image digest:
  sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0
- Staging runtime is expected on VPS localhost 127.0.0.1:19419
- Staging compose project was reported as ticenpi-post-runtime-staging
- Staging expected app source: 27e4efe
- Staging expected image digest: same d6a0c592...
- Previous report could not verify Staging runtime directly.
- Previous report also could not verify the effective runtime values of several flags.

## PRIMARY MISSION

Determine the **actual effective runtime truth** for Production and Staging.

Do not infer runtime state from code defaults alone.
Do not infer runtime state from compose templates alone.
Do not infer runtime state from .env examples alone.

For every important setting, distinguish these layers:

1. CODE_DEFAULT
2. COMPOSE_DECLARATION
3. ENV_FILE / DEPLOY_INPUT
4. EFFECTIVE_RUNNING_CONTAINER_VALUE
5. EFFECTIVE_APPLICATION_BEHAVIOR

The final answer must tell us what the currently running container actually sees.

---

# 0. ABSOLUTE SAFETY RULES

READ-ONLY ONLY.

Forbidden:

- edit files
- create files in product repo
- commit
- push
- checkout
- reset
- restore
- stash
- clean
- merge
- rebase
- cherry-pick
- docker compose up/down
- docker restart
- docker stop/start
- recreate container
- pull image
- build image
- deploy
- alter VPS files
- alter .env
- alter secrets
- alter Supabase
- alter Cloudflare
- alter database
- call any endpoint that writes state
- execute any Facebook publish/comment/join/delete/relist action
- change a feature flag
- change permissions

Allowed:

- git status/log/show/diff/grep
- read source/docs/deploy manifests
- SSH read-only commands
- docker ps / docker inspect
- docker compose config if it does not mutate
- read-only container exec commands such as environment inspection
- curl GET /api/health
- curl GET other proven read-only status endpoints
- process listing
- file metadata/stat
- grep/sed/cat against deployment config
- hash/digest comparison

Do not expose secret values in the report.

---

# 1. SECRET HANDLING

Never print raw values for:

- TICENPI_TENANT_KEYS
- Supabase service-role keys
- JWT secrets
- API keys
- cookies
- bind/device tokens
- passwords
- private URLs containing credentials

For secret-bearing variables report only:

SET / UNSET

and when useful:

- entry count
- masked prefix only if already public and non-sensitive
- length
- structural validity
- whether parser accepts the shape
- whether a value appears weak/short **without printing it**

For TICENPI_TENANT_KEYS specifically:

Report:

TICENPI_TENANT_KEYS_PRESENT =
TICENPI_TENANT_KEYS_ENTRY_COUNT =
TICENPI_TENANT_KEYS_PARSE_VALID =
TICENPI_TENANT_KEYS_DIRECTION_MATCHES_CODE =
TICENPI_TENANT_KEYS_WEAK_OR_GUESSABLE = YES/NO/UNVERIFIED

Do NOT print the keys themselves.

---

# 2. FREEZE CURRENT LOCAL STATE

Record only:

REPO =
BRANCH =
HEAD =
WORKTREE_STATUS =

If dirty:
continue.
Do not touch or restore anything.

Read the prior reconciliation report and the relevant deployment docs before touching the VPS.

At minimum inspect:

- deploy/staging/compose.yml
- deploy/runtime-staging/compose.yml
- release identity files
- production-promotion.json if present
- backend/app/core/commercial.py
- backend/app/core/seat.py
- backend/app/core/identity.py
- backend/app/core/publish_gate.py
- backend/app/api/auth.py
- backend/app/api/studio.py
- _charter/adr/ADR-002-central-commercial-post-tenant-seat.md
- _charter/adr/ADR-003-device-token-entitlement-revalidation-gap.md
- docs/DEPLOY-google-auth-2026-09-04.md

Do not assume old line numbers remain correct.

---

# 3. IDENTIFY ACTUAL RUNNING CONTAINERS

On the VPS, read-only identify the real running Post Production and Staging containers.

Do not hardcode container names if Docker can prove them.

Use:

- docker ps
- labels
- compose project labels
- exposed ports
- image digest / image ID

Record:

PRODUCTION_CONTAINER =
PRODUCTION_COMPOSE_PROJECT =
PRODUCTION_IMAGE =
PRODUCTION_IMAGE_DIGEST =
PRODUCTION_PORTS =

STAGING_CONTAINER =
STAGING_COMPOSE_PROJECT =
STAGING_IMAGE =
STAGING_IMAGE_DIGEST =
STAGING_PORTS =

If multiple candidates exist:
resolve using compose labels + ports + /api/health identity.

Do not choose by name alone.

---

# 4. VERIFY RUNTIME IDENTITY

Production:

GET https://post.ticenpi.com/api/health

Staging:

from the VPS itself, GET the actual local Staging health endpoint,
expected around:

http://127.0.0.1:19419/api/health

but first prove the port/container mapping.

Record exact returned non-secret fields:

environment
release
commit
service
port
config_drift if present

Compare them to image/container identity.

Output:

PRODUCTION_HEALTH =
STAGING_HEALTH =

PRODUCTION_COMMIT_MATCHES_IMAGE =
STAGING_COMMIT_MATCHES_IMAGE =

PRODUCTION_EXPECTED_DIGEST_MATCH =
STAGING_EXPECTED_DIGEST_MATCH =

If Staging is unhealthy or unreachable:
do not restart it.
record the evidence and continue with available container/config inspection.

---

# 5. BUILD THE FLAG INVENTORY FROM SOURCE

Before inspecting container env, determine from source exactly how each flag is interpreted.

Required flags:

## Commercial / Identity / Seat

- TICENPI_COMMERCIAL_GATE_ENABLED
- TICENPI_SEAT_POLICY
- TICENPI_CENTRAL_SEAT_SHADOW_ENABLED
- TICENPI_AUTO_TENANT_BOOTSTRAP
- TICENPI_ALLOW_TENANT_KEY
- TICENPI_TENANT_KEYS
- TICENPI_REQUIRE_STUDIO
- TICENPI_LOCAL_DEVELOPMENT

## Publish / Facebook actions

- TICENPI_AUTO_PUBLISH
- TICENPI_AUTO_PUBLISH_POST
- TICENPI_AUTO_PUBLISH_COMMENT
- TICENPI_AUTO_PUBLISH_MARKETPLACE
- TICENPI_AUTO_PUBLISH_RELIST
- TICENPI_AUTO_PUBLISH_DELETE

Also search source for any additional current Post action-specific gates that affect:

- publish
- comment
- marketplace_publish
- relist
- delete
- join_group
- group sync

Do not assume this list is exhaustive.

For each, identify:

SOURCE_FILE =
DEFAULT =
PARSER =
EFFECTIVE_LOGIC =

Example:

MASTER_FLAG AND ACTION_FLAG?
ACTION_FLAG overrides master?
empty string means false?
missing in production means fail-closed?

Prove it from code.

---

# 6. INSPECT COMPOSE / DEPLOY DECLARATIONS

For Production and Staging separately, record the declared values or substitutions in the compose source.

Do not confuse:

\${VAR:-default}

with actual runtime value.

Output:

FLAG =
PROD_COMPOSE_DECLARATION =
STAGING_COMPOSE_DECLARATION =

Also identify which env file / deployment input each compose invocation uses.

Do not print the full env file.

---

# 7. INSPECT EFFECTIVE RUNNING CONTAINER ENV

This is the core step.

Use read-only Docker inspection of the **running containers** to determine the actual values they received.

For non-secret booleans/policies, report exact effective value.

For secret values, report SET/UNSET only.

Required exact non-secret runtime truth:

TICENPI_ENVIRONMENT
TICENPI_COMMERCIAL_GATE_ENABLED
TICENPI_SEAT_POLICY
TICENPI_CENTRAL_SEAT_SHADOW_ENABLED
TICENPI_AUTO_TENANT_BOOTSTRAP
TICENPI_ALLOW_TENANT_KEY
TICENPI_REQUIRE_STUDIO
TICENPI_LOCAL_DEVELOPMENT
TICENPI_AUTO_PUBLISH
TICENPI_AUTO_PUBLISH_POST
TICENPI_AUTO_PUBLISH_COMMENT
TICENPI_AUTO_PUBLISH_MARKETPLACE
TICENPI_AUTO_PUBLISH_RELIST
TICENPI_AUTO_PUBLISH_DELETE

For TICENPI_TENANT_KEYS:
presence/shape only, never raw content.

Do this separately for:

PRODUCTION
STAGING

If a variable is absent:
write ABSENT, not empty.

If present as empty string:
write EMPTY_STRING.

These are different.

---

# 8. DETERMINE EFFECTIVE BEHAVIOR, NOT JUST RAW FLAGS

Using source logic + runtime env, compute the actual effective behavior.

For Production and Staging, report:

## Commercial

COMMERCIAL_GATE_EFFECTIVE =
ENTITLEMENT_REQUIRED =
FAIL_OPEN_OR_FAIL_CLOSED =

## Seat

SEAT_POLICY_EFFECTIVE =
CENTRAL_SEAT_SHADOW_EFFECTIVE =
CENTRAL_SEAT_AUTHORITATIVE_EFFECTIVE =
PLATFORM_ADMIN_EXEMPTION =
TEST_ACCESS_ALLOW_EXEMPTION =

Do not equate:

SEAT_POLICY=require

with:

CENTRAL_SEAT_SHADOW_ENABLED=1

They are separate concepts.

## Tenant

AUTO_TENANT_BOOTSTRAP_EFFECTIVE =
TENANT_KEY_FALLBACK_EFFECTIVE =
TENANT_KEYS_PRESENT =
TENANT_KEY_BYPASS_POSSIBLE = YES/NO/UNVERIFIED

## Studio

REQUIRE_STUDIO_EFFECTIVE =

## Publish actions

POST_PUBLISH_EFFECTIVE =
COMMENT_EFFECTIVE =
MARKETPLACE_PUBLISH_EFFECTIVE =
RELIST_EFFECTIVE =
DELETE_EFFECTIVE =

Use:

ALLOWED
BLOCKED_BY_GATE
CONDITIONALLY_ALLOWED
UNVERIFIED

Do not execute the action to determine this.

---

# 9. TENANT KEY SECURITY CHECK — READ ONLY

The previous reconciliation found a possible P0 issue:

Production compose may still default:

TICENPI_ALLOW_TENANT_KEY=1

and an older deployment doc states the tenant key mapping may have been reversed / weak.

Verify this carefully without revealing secrets.

Required checks:

1. What does current code expect the mapping direction to be?
2. What structural shape does the Production runtime value use?
3. Does it parse successfully?
4. Is ALLOW_TENANT_KEY actually enabled in the running Production container?
5. Is Google/Supabase identity also enabled?
6. Is the legacy tenant-key route still a viable authentication path?
7. Does the runtime key material appear short/guessable based on length/structure only?

Output:

PRODUCTION_LEGACY_TENANT_KEY_PATH_ACTIVE =
PRODUCTION_TENANT_KEY_MAPPING_CORRECT =
PRODUCTION_TENANT_KEY_SECURITY_RISK =
P0_CONFIRMED = YES/NO/UNVERIFIED

Do not test guessed keys against Production.

No brute force.
No authentication attempt with fabricated keys.

---

# 10. MARKETPLACE RUNTIME TRUTH

This task must settle whether Marketplace is currently merely present in source or actually allowed by runtime gates.

Report:

MARKETPLACE_SOURCE_PRESENT =
MARKETPLACE_UI_PRESENT =
MARKETPLACE_ACTION_REGISTERED =
MARKETPLACE_MASTER_GATE =
MARKETPLACE_ACTION_GATE =
MARKETPLACE_EFFECTIVE_RUNTIME =
MARKETPLACE_REAL_PUBLISH_E2E_VERIFIED = NO unless evidence already exists

No actual Marketplace publish in this task.

If runtime gate is ALLOWED:
say "ALLOWED_BY_CONFIGURATION, NOT E2E_VERIFIED".

Do not say COMPLETE_ACTIVE.

---

# 11. AUTOMATION RUNTIME TRUTH

For:

- relist
- cleanup/delete
- daily_comment
- scheduled publish

determine:

SOURCE_PRESENT
WORKER_PRESENT
SCHEDULER_PRESENT
ACTION_GATE_EFFECTIVE
RUNTIME_ENABLED_BY_CONFIG
E2E_VERIFIED

Do not trigger jobs.

---

# 12. CENTRAL SEAT TRUTH

The previous report found confusing branch/config state.

This audit must explicitly distinguish:

A. Seat code present in image?
B. TICENPI_SEAT_POLICY effective value?
C. TICENPI_CENTRAL_SEAT_SHADOW_ENABLED effective value?
D. Is authoritative seat enforcement actually in the request path?
E. Is shadow observation active?
F. Is Production using central seat?
G. Is Staging using central seat?

Output separately:

PRODUCTION_SEAT_CODE_PRESENT =
PRODUCTION_SEAT_POLICY =
PRODUCTION_SEAT_SHADOW =
PRODUCTION_SEAT_AUTHORITATIVE =
PRODUCTION_CENTRAL_SEAT_EFFECTIVE =

STAGING_SEAT_CODE_PRESENT =
STAGING_SEAT_POLICY =
STAGING_SEAT_SHADOW =
STAGING_SEAT_AUTHORITATIVE =
STAGING_CENTRAL_SEAT_EFFECTIVE =

Use YES/NO/PARTIAL/UNVERIFIED with evidence.

Do not infer from branch names.

---

# 13. COMMERCIAL GATE TRUTH

The previous reconciliation had a classification inconsistency:

code default = 0
compose = 1
runtime = unverified

Resolve it.

Output:

PRODUCTION_COMMERCIAL_GATE_CODE_DEFAULT =
PRODUCTION_COMMERCIAL_GATE_COMPOSE =
PRODUCTION_COMMERCIAL_GATE_CONTAINER =
PRODUCTION_COMMERCIAL_GATE_EFFECTIVE =

STAGING_COMMERCIAL_GATE_CODE_DEFAULT =
STAGING_COMMERCIAL_GATE_COMPOSE =
STAGING_COMMERCIAL_GATE_CONTAINER =
STAGING_COMMERCIAL_GATE_EFFECTIVE =

Effective state must be based on the running container.

---

# 14. DRIFT DETECTION

Compare:

SOURCE EXPECTATION
vs
COMPOSE
vs
RUNNING CONTAINER
vs
HEALTH IDENTITY

List every mismatch.

Classes:

EXPECTED_OVERRIDE
BENIGN_DRIFT
STALE_CONTAINER
STALE_MANIFEST
SECURITY_RELEVANT_DRIFT
UNKNOWN

Examples:

- compose says 1, container says absent
- expected digest differs
- source commit differs from health commit
- Staging image same but config different intentionally

Do not repair drift.

---

# 15. REQUIRED TRUTH TABLE

Produce one table per environment.

Columns:

| Variable / Behavior | Code Default | Compose Declaration | Running Container | Effective Behavior | Evidence | Risk |

Must include all required flags above.

For TICENPI_TENANT_KEYS:
mask the Running Container column as SET/UNSET only.

---

# 16. DECISION OUTPUTS

At the end, explicitly answer:

1. Is Marketplace currently enabled in Production?
2. Is Marketplace currently enabled in Staging?
3. Is Commercial Gate actually enabled in Production?
4. Is Commercial Gate actually enabled in Staging?
5. Is Production still accepting X-Tenant-Key?
6. Is the P0 tenant-key security issue real?
7. Is Central Seat authoritative in Production?
8. Is Central Seat authoritative in Staging?
9. Is Central Seat shadow active anywhere?
10. Are relist/comment/delete automation actions currently allowed by config?
11. Is Production running the expected immutable image?
12. Is Staging running the expected immutable image?

Do not answer any item with inference if runtime evidence is missing.
Use UNVERIFIED.

---

# 17. NEXT ACTION CLASSIFICATION

This task does not implement anything.

Based only on proven runtime truth, classify next actions:

SAFE_TO_PROCEED
NEEDS_SECURITY_FIX
NEEDS_E2E_VERIFY
NEEDS_CONFIG_DECISION
NEEDS_DEPLOY_RECONCILIATION
NO_ACTION

For each issue output:

ISSUE =
RUNTIME_TRUTH =
CLASS =
WHY =
NEXT_TASK_NAME =

Do not write the next implementation prompt.

---

# 18. HARD STOP CONDITIONS

HARD_STOP only if:

- cannot identify Production container
- cannot identify Staging container and no deploy evidence exists
- SSH/VPS read-only access unavailable for both environments
- Docker runtime cannot be inspected
- evidence contradicts itself so severely that effective runtime truth cannot be established

Do NOT HARD STOP because:

- worktree dirty
- a flag is absent
- a feature is disabled
- a container is unhealthy
- Staging is down

Report those states and continue wherever possible.

---

# 19. FINAL REPORT FORMAT

PHASE = POST PRODUCTION + STAGING RUNTIME TRUTH AUDIT
RESULT = PASS / PARTIAL / HARD_STOP

A. LOCAL
BRANCH =
HEAD =
WORKTREE =

B. PRODUCTION IDENTITY
CONTAINER =
COMPOSE_PROJECT =
IMAGE =
DIGEST =
HEALTH =
COMMIT =
RELEASE =

C. STAGING IDENTITY
CONTAINER =
COMPOSE_PROJECT =
IMAGE =
DIGEST =
HEALTH =
COMMIT =
RELEASE =

D. COMMERCIAL
PROD_COMMERCIAL_GATE =
STAGING_COMMERCIAL_GATE =

E. TENANT
PROD_AUTO_TENANT_BOOTSTRAP =
STAGING_AUTO_TENANT_BOOTSTRAP =
PROD_ALLOW_TENANT_KEY =
STAGING_ALLOW_TENANT_KEY =
PROD_TENANT_KEYS_PRESENT =
STAGING_TENANT_KEYS_PRESENT =
PROD_TENANT_KEY_MAPPING_CORRECT =
P0_TENANT_KEY_RISK_CONFIRMED =

F. SEAT
PROD_SEAT_POLICY =
STAGING_SEAT_POLICY =
PROD_CENTRAL_SEAT_SHADOW =
STAGING_CENTRAL_SEAT_SHADOW =
PROD_CENTRAL_SEAT_AUTHORITATIVE =
STAGING_CENTRAL_SEAT_AUTHORITATIVE =

G. FACEBOOK ACTION GATES
PROD_POST =
PROD_COMMENT =
PROD_MARKETPLACE =
PROD_RELIST =
PROD_DELETE =
STAGING_POST =
STAGING_COMMENT =
STAGING_MARKETPLACE =
STAGING_RELIST =
STAGING_DELETE =

H. MARKETPLACE
MARKETPLACE_PRODUCTION_CONFIG =
MARKETPLACE_STAGING_CONFIG =
MARKETPLACE_REAL_E2E_VERIFIED =

I. DRIFT
SECURITY_RELEVANT_DRIFT =
STALE_CONTAINER =
STALE_MANIFEST =
OTHER_DRIFT =

J. DECISIONS
MARKETPLACE_PROD_ENABLED =
MARKETPLACE_STAGING_ENABLED =
COMMERCIAL_PROD_ENABLED =
COMMERCIAL_STAGING_ENABLED =
TENANT_KEY_PROD_ACTIVE =
CENTRAL_SEAT_PROD_AUTHORITATIVE =
CENTRAL_SEAT_STAGING_AUTHORITATIVE =
CENTRAL_SEAT_SHADOW_ACTIVE_ANYWHERE =
EXPECTED_PROD_IMAGE =
EXPECTED_STAGING_IMAGE =

K. NEXT
ISSUES =
NEXT_TASKS =
READY_FOR_WRITER_PROMPTS = YES/NO

NO SOURCE CHANGES.
NO COMMIT.
NO PUSH.
NO DEPLOY.
NO FACEBOOK WRITE ACTION.
