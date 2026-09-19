# DM STAGING COMPLETE ORCHESTRATOR OWNER

## Goal
從目前已完成的 DM 狀態一路執行到 STAGING_ACCEPTED=YES；禁止 Production deploy。不要再重做今天已證明的 audit。能平行就平行，只有硬依賴才等待。

## Known baseline
- Production runtime currently healthy; current production release is older than the pending entitlement fix.
- Staging public URL is the existing dm-staging endpoint and uses the staging Supabase project.
- Local Dev frontend is localhost:9422; backend is localhost:8000.
- Local Docker localhost:9421 is stale and must be rebuilt from the final integrated source.
- Entitlement source fix commit: 86f2c97.
- DM release-gate commit: 2325cca.
- Central deploy release-gate commit: f8d4f0b.
- Capture Gate A/B already exist and are release-blocking.
- Auth invalid-JWT / no-entitlement / entitled gates already exist and are release-blocking.
- Capture C and AI browser E2E hooks exist but must not be marked PASS without real browser evidence.
- AGNES_API_KEY already exists in the intended env source; previous failure indicates env→runtime wiring/loading must be checked before assuming the key is missing.
- Admin / positive / negative Google test identities are already configured locally. Do not publish account identifiers or credentials.

## HARD RULES
1. Freeze the existing Capture network path unless new direct evidence proves it is broken. Do not first modify CORS, JWT contract, YUCT/SOCKS proxy, nginx /api routing, or /api/scrape.
2. AGNES troubleshooting order is fixed:
   env source → env loader → process/container injection → backend runtime → config object → /api/ai-draw → AGNES.
   Do not first change CORS/JWT/proxy/timeouts.
3. Release identity invariant:
   - Local Docker MUST be rebuilt from the exact integrated candidate source and pass before any push/release build.
   - Freeze the candidate Git SHA only after localhost:9421 proves it is no longer serving the stale image.
   - GitHub Actions ARM64 MUST build the release image from that exact frozen candidate Git SHA.
   - Staging MUST deploy the exact CI-produced ARM64 image digest.
   - Do NOT require the local Windows/x86 Docker digest to equal the ARM64 release digest; architecture-specific image digests may differ.
   - The immutable invariant for release promotion is:
     FROZEN_CANDIDATE_GIT_SHA = CI_ARM64_SOURCE_SHA
     and
     STAGING_IMAGE_DIGEST = CI_ARM64_IMAGE_DIGEST.
   - No VPS source-tree build and no rebuild after CI ARM64 publication.
4. No secrets in Git/image/logs. Presence checks only.
5. No Production deploy or Production data mutation.
6. Do not use git reset --hard or git clean.
7. Do not stop for routine confirmation. Stop only on a true hard blocker.

## Phase 1 — Workspace / integration
Use isolated integration worktrees. Confirm commits 86f2c97, 2325cca, f8d4f0b exist. Integrate the DM commits into one candidate branch and the deploy commit into the central deploy candidate. Do not touch other agents' dirty worktrees.

## Phase 2 — Parallel engineering

### A. AGNES runtime wiring
- Confirm AGNES_API_KEY is PRESENT in the intended env source without printing it.
- Confirm the running Local Dev backend sees it.
- Trace actual env loading/injection and fix only the broken wiring if runtime is missing it.
- Restart/recreate only the required local backend process/container.
- Perform one minimal authenticated AI generation.
- Require: AGNES_RUNTIME_KEY=PRESENT and LOCAL_AI_GENERATION=PASS.
- Add low-cost configured/reachable health coverage; do not generate images on every health check.
- Add/adjust tests for env wiring and AI API behavior.

### B. Local Docker canonical flow
- Determine why the root compose became production-profile and restore the intended responsibility split without blind revert.
- Local Docker must use local/disposable runtime only, no Production Supabase/origin/targets.
- Staging and Production are runtime injection targets, not separate application rebuilds.
- STOP the stale localhost:9421 stack and rebuild localhost:9421 from the integrated candidate.
- Verify the new 9421 is actually serving the integrated candidate, not the old bundle/image.
- Required proof before leaving this phase:
  - LOCAL_DOCKER_REBUILT = YES
  - LOCAL_DOCKER_STALE = NO
  - /api/health = 200
  - runtime-config = local/disposable
  - candidate Git SHA / release identity matches the integrated candidate
  - frontend bundle/build identity differs from the known stale 9421 bundle
  - auth gates PASS
  - Capture A/B PASS
  - AGNES runtime presence PASS
  - one minimal AI generation PASS
- HARD STOP: if 9421 is still stale or source identity is ambiguous, do NOT push, do NOT trigger CI, and do NOT deploy Staging.

### C. Capture regression protection
- Re-run Capture Gate A and Gate B against the integrated candidate.
- Confirm /api/scrape still follows the existing httpx + proxy contract.
- Do not rewrite Capture unless a direct failure boundary is proven.

Run A/B/C in parallel where possible.

## Phase 3 — Candidate validation
Run relevant backend tests, auth/entitlement tests, scrape tests, AI tests, release gates, frontend tests/typecheck/build, compose validation, and secret scan. Any new regression blocks the candidate.

## Phase 4 — Freeze source candidate
Only after the rebuilt Local Docker on 9421 passes:
- record FROZEN_CANDIDATE_GIT_SHA
- local image ID/digest as local evidence only
- release candidate ID
- source state

After freeze, no source changes. Any source change invalidates the candidate and requires Local Docker rebuild/revalidation.

## Phase 5 — Push + CI ARM64 release build
Push the exact frozen candidate Git SHA and required central deploy commit.
Trigger the existing GitHub Actions ARM64 release build.

Required proof:
- CI_ARM64_SOURCE_SHA = FROZEN_CANDIDATE_GIT_SHA
- CI_ARM64_IMAGE_DIGEST recorded
- release manifest/checksum recorded
- CI build passes

Do NOT build release images on the VPS.
Do NOT compare the Windows/x86 local image digest to the ARM64 release digest as an equality requirement.

## Phase 6 — Staging preflight
Use central deploy candidate including f8d4f0b. Validate:
- target is staging
- exact release/image digest/manifest
- staging env contract
- staging Supabase/origin
- rollback target
- AGNES key presence in staging runtime source, value-blind
- no Production target selected

## Phase 7 — Deploy exact CI ARM64 digest to Staging
Deploy the exact CI-produced ARM64 image digest. Do not rebuild on the VPS and do not deploy from an extracted source tree. Inject only staging runtime config/secrets.

After deploy verify:
- environment=staging
- release ID = new candidate release
- Git SHA = FROZEN_CANDIDATE_GIT_SHA
- STAGING_IMAGE_DIGEST = CI_ARM64_IMAGE_DIGEST
- staging Supabase is correct
- /api/health = 200
- AGNES runtime key = PRESENT (value-blind)
- frontend is NOT the old staging UI/bundle/release

HARD STOP + rollback if any of these are false.
In particular, if opening https://dm-staging.ticenpi.com still shows the old UI or old release identity, STAGING_DEPLOY = FAIL even if containers are healthy.

## Phase 8 — Automated Staging gates
Run all release-blocking gates:
- health
- Capture A/B
- auth invalid JWT
- auth no-entitlement
- auth entitled
- critical smoke
- post-deploy audit
- AGNES configured/reachable health

Any blocking failure => STAGING_ACCEPTED=NO and use existing rollback behavior.

## Phase 9 — Browser UAT
Browser control is needed only here. Do all previous phases first even if Browser MCP is unavailable.

Use the already configured local test identities:
- admin positive path
- ordinary positive path
- ordinary negative path

Run:
1. Google OAuth/session baseline.
2. Non-entitled authenticated user must get 403 on protected DM API.
3. Entitled user must be allowed.
4. Capture E2E: real browser → /api/scrape → visible result → image naturalWidth/naturalHeight > 0 → no Failed to fetch/CORS error.
5. AI E2E: real browser → /api/ai-draw → rendered image → no Failed to fetch/blocking console error.

If Browser MCP is unavailable at this final phase, report BROWSER_UAT=BLOCKED_BY_BROWSER_MCP; do not fake PASS.

If Browser UAT proves a source defect, fix only the proven boundary, invalidate the candidate, and repeat Local Docker → freeze → publish → Staging with a new candidate.

## Phase 10 — Staging acceptance
STAGING_ACCEPTED=YES only if all are true:
- entitlement backend PASS
- invalid JWT denied
- no-entitlement denied
- entitled allowed
- Capture A/B PASS
- Capture Browser E2E PASS
- AGNES runtime key PRESENT
- AGNES health PASS
- AI API PASS
- AI Browser E2E PASS
- Local Docker PASS
- same artifact Local→Staging YES
- Staging runtime identity PASS
- release gates PASS
- no new regressions
- no tracked secrets

## Phase 11 — SSOT
Only after STAGING_ACCEPTED=YES, update DM current-state/history with candidate SHA, image digest, staging release, AGNES wiring fix, Capture/AI/auth results, release gates, and staging acceptance. Do not modify the frozen ecosystem snapshot.

## Final output
DM_STAGING_ORCHESTRATION =
PASS / BLOCKED

CANDIDATE_GIT_SHA =
...

LOCAL_DOCKER =
PASS / FAIL

LOCAL_DOCKER_REBUILT =
YES / NO

LOCAL_DOCKER_STALE =
YES / NO

LOCAL_DOCKER_IMAGE_DIGEST =
...

FROZEN_CANDIDATE_GIT_SHA =
...

CI_ARM64_SOURCE_SHA =
...

CI_ARM64_IMAGE_DIGEST =
...

AGNES_ENV_SOURCE =
PRESENT / MISSING

AGNES_RUNTIME_KEY =
PRESENT / MISSING

LOCAL_AI_GENERATION =
PASS / FAIL

CAPTURE_GATE_A =
PASS / FAIL

CAPTURE_GATE_B =
PASS / FAIL

AUTH_INVALID_JWT =
DENIED / FAIL

AUTH_NO_ENTITLEMENT =
DENIED / FAIL

AUTH_ENTITLED =
ALLOW / FAIL

PUBLISHED_IMAGE_DIGEST =
...

CI_SOURCE_MATCHES_FROZEN_CANDIDATE =
YES / NO

STAGING_DIGEST_MATCHES_CI_ARM64 =
YES / NO

STAGING_RELEASE =
...

STAGING_IMAGE_DIGEST =
...

STAGING_AUTOMATED_GATES =
PASS / FAIL

GOOGLE_OAUTH =
PASS / FAIL / BLOCKED

CAPTURE_BROWSER_E2E =
PASS / FAIL / BLOCKED

AI_BROWSER_E2E =
PASS / FAIL / BLOCKED

STAGING_ACCEPTED =
YES / NO

PRODUCTION_CHANGED =
NO

BLOCKERS =
...

If STAGING_ACCEPTED=YES: stop. Do not deploy Production.
