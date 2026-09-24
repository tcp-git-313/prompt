# W1-DM-R1 — DM Staging Deployment Evidence Recovery (Read-Only)

ROLE: DM STAGING DEPLOYMENT EVIDENCE RECOVERY OWNER

MODE: READ-ONLY / NO SOURCE CHANGES / NO DEPLOY

WORKSPACE:
F:\00-Ticenpi-SaaS

DM REPO:
F:\00-Ticenpi-SaaS\TicenpiDM

PLATFORM / DEPLOY REFERENCE:
F:\00-Ticenpi-SaaS\ticenpi-platform
F:\00-Ticenpi-SaaS\deploy

GOAL:
Recover trustworthy current DM Staging deployment evidence so W1-DM-F1 can resume safely.

Do NOT modify source.
Do NOT create a worktree.
Do NOT commit/push.
Do NOT mutate Staging or Production.
Do NOT restart containers/services unless explicitly required for a read-only probe and proven safe.
Do NOT deploy.

## KNOWN CURRENT EVIDENCE

- DM Staging /api/health returns 200
- reported release: 20260923-215914
- reported commit: 25db5f9b8b65a231dafb7cc8f1f05262fb7519aa
- /api/ready times out
- platform read-only audit cannot currently reach the Staging host
- older frozen candidate evidence does NOT match current health identity and must not be used as current truth

## REQUIRED OUTPUT

Recover and prove:

1. current DM Staging release
2. current app source commit
3. current backend/frontend image digest(s)
4. current deploy-config identity if separated
5. current effective auth/commercial/Seat-related runtime flags
6. current Supabase project/environment
7. current container/service health and restart count
8. exact previous release / rollback target
9. why /api/ready times out
10. why the platform read-only audit cannot reach the host

## 1. PREFLIGHT

Record:
- current DM repo branch/HEAD/status
- current deployment scripts/manifests used for DM Staging
- current health endpoint result
- current ready endpoint result
- whether the read-only audit failure is:
  - DNS
  - SSH
  - Tailscale/routing
  - firewall
  - host unavailable
  - wrong target
  - service port mismatch
  - TLS/proxy
  - script/config error
  - other exact cause

Do not alter network settings.

## 2. CURRENT RELEASE IDENTITY

Using the safest available read-only sources, cross-check at least two independent signals where possible:

- /api/health identity
- running container image reference
- Docker inspect labels
- active release directory
- release manifest
- deploy audit artifact
- compose config
- systemd/docker status

Return:

DM_STAGING_RELEASE =
DM_APP_SOURCE_COMMIT =
DM_BACKEND_IMAGE_DIGEST =
DM_FRONTEND_IMAGE_DIGEST =
DM_DEPLOY_CONFIG_COMMIT =
IDENTITY_CROSSCHECKED = YES/NO

If one source disagrees with another, do not choose one silently. Report the mismatch.

## 3. RUNTIME CONFIG

Read effective running configuration without printing secrets.

At minimum capture:
- environment
- Supabase project ref
- DM auth-required setting
- commercial gate / commercial context settings
- Central Seat shadow/require policy if present
- local-dev bypass flags
- test-access flags if runtime-configurable
- any tenant/customer isolation mode relevant to DM auth

Mask all secrets.

## 4. /api/ready TIMEOUT ROOT CAUSE

Trace what /api/ready actually checks.

Determine whether timeout is caused by:
- Supabase dependency
- database query
- external service
- ExtractionHub/YUCT dependency
- queue/Redis
- internal deadlock
- proxy/network path
- endpoint implementation bug
- timeout configuration
- other exact dependency

Use logs and safe read-only probes.

Do NOT modify readiness code.

Return:

READY_ENDPOINT_ROOT_CAUSE =
READY_ENDPOINT_AFFECTS_AUTH_CUTOVER = YES/NO/UNKNOWN

If ready timeout indicates a real runtime dependency failure that would make authoritative cutover unsafe:
set BLOCKER.

## 5. READ-ONLY AUDIT CONNECTIVITY ROOT CAUSE

Trace the exact audit path:
local machine / runner
→ network route
→ Staging host
→ service/SSH target

Identify why the existing platform audit cannot connect.

Do not change Tailscale/firewall/SSH config.

Return:

AUDIT_CONNECTIVITY_ROOT_CAUSE =
AUDIT_CONNECTIVITY_RECOVERABLE_WITHOUT_MUTATION = YES/NO

If an operator action is required, state one exact action only.

## 6. ROLLBACK TARGET

Prove the exact previous known-good DM Staging release.

Record:
- previous release ID
- app commit
- image digest(s)
- config identity
- health evidence if available
- rollback command/path used by the existing deployment framework

Do not execute rollback.

Required:

ROLLBACK_TARGET_PROVEN = YES/NO

## 7. FINAL SAFETY GATE

W1-DM-F1 may resume only if all are YES:

CURRENT_RELEASE_IDENTITY_PROVEN
CURRENT_RUNTIME_AUTH_CONFIG_PROVEN
ROLLBACK_TARGET_PROVEN
READY_ENDPOINT_RISK_CLASSIFIED
AUDIT_CONNECTIVITY_ROOT_CAUSE_PROVEN

## 8. FINAL REPORT

A. PREFLIGHT
B. CURRENT RELEASE IDENTITY
C. RUNTIME AUTH/COMMERCIAL CONFIG
D. /api/ready ROOT CAUSE
E. AUDIT CONNECTIVITY ROOT CAUSE
F. ROLLBACK TARGET
G. SAFETY

SOURCE_FILES_MODIFIED = NO
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO
COMMIT_CREATED = NO
PUSHED = NO

H. FINAL RESULT

If sufficient evidence is recovered:

RESULT = PASS
CURRENT_RELEASE_IDENTITY_PROVEN = YES
CURRENT_RUNTIME_AUTH_CONFIG_PROVEN = YES
ROLLBACK_TARGET_PROVEN = YES
READY_ENDPOINT_RISK_CLASSIFIED = YES
AUDIT_CONNECTIVITY_ROOT_CAUSE_PROVEN = YES
READY_FOR_DM_F1_RESUME = YES

If not:

RESULT = BLOCKED
READY_FOR_DM_F1_RESUME = NO
BLOCKER = exact missing evidence

Finish and stop.
