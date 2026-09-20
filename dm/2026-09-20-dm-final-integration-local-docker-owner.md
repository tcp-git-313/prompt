# DM FINAL INTEGRATION + LOCAL DOCKER OWNER

## Mission

根據目前已經實測完成的狀態，建立 Ticenpi DM 的「真正最終 integration candidate」，然後用這個 integration HEAD 重建 localhost:9421 Local Docker，驗證通過後才 Freeze candidate。

不要直接從 559d6af 單獨重建後就當最終候選。
559d6af 只是其中一個 Owner package，還缺其他已知必需工作。

本任務只做到：

ALL_REQUIRED_DM_COMMITS_INTEGRATED
→ LOCAL_DOCKER_9421_REBUILT
→ LOCAL_DOCKER_VERIFIED
→ FROZEN_CANDIDATE_SHA_READY

完成後停止。
不要 push。
不要 CI。
不要 deploy Staging。
不要 deploy Production。

---

# 0. 已知真實狀態

## Local runtime

### Local Dev
- frontend: http://localhost:9422
- backend: http://127.0.0.1:8000
- 9422 / 8000 是既定設計的一組
- Capture 已真人實測 PASS
- AI / AGNES 已真人實測 PASS

### Local Docker
- localhost:9421 可用
- Capture 已真人實測 PASS
- AI / AGNES 已真人實測 PASS
- 但目前 9421 image 無法追溯：
  - image git-sha label 空
  - commit 顯示 working-tree
  - built from .staging-orchestration\dm
- verify_docker_is_latest 會判不合格
- 因此必須在最終 integration commit 後重新 build

---

# 1. 已知 Git packages

## Owner package — AI / Local tooling

COMMIT:
559d6af

BRANCH:
dm/ai-draw-local-tooling-20260920

BASE:
a199d90

PURPOSE:
- AI draw frontend changes
- reverse-side base image / related AI draw behavior
- removebg 504 retry
- AGNES download retry
- vite.config.js DM_BACKEND_URL override
- start_local_dev.ps1
- start_local_docker.ps1
- verify_docker_is_latest.ps1
- _charter/history.md note

KNOWN:
- staged secret scan clean
- docker-compose.yml not touched
- Dockerfile not touched
- auth.js not touched
- env loading logic not touched
- branch is clean except unrelated untracked files

## Staging digest pin

COMMIT:
0a32a59

KNOWN:
- exists on origin
- pins staging to ARM64 CI artifact produced from a199d90
- current 559d6af branch does NOT include it
- local release/staging-20260919-commercial remains at a199d90

IMPORTANT:
0a32a59 is a release/staging artifact pin related commit.
Inspect its diff and dependency before integration.
Do not blindly cherry-pick if it pins an old digest that must be regenerated after the final candidate.

If its semantics are "pin exact old a199d90 digest", it may need to be superseded rather than included as-is.
If it contains reusable workflow/runtime logic independent of the old digest, integrate only the correct reusable part.

## Entitlement security

COMMIT:
86f2c97

PURPOSE:
backend entitlement enforcement

KNOWN:
- protected APIs covered
- entitled allow
- no-entitlement deny
- expired deny
- wrong-product deny
- invalid JWT deny
- tests PASS

## Release Gates — DM

COMMIT:
2325cca

REPORTED BASE:
86f2c97

PURPOSE:
- app.release_gates
- Capture Gate A/B release blocking
- auth invalid JWT / no-entitlement / entitled release blocking
- Gate C hook
- AI E2E hook
- secret-safe gate loader
- fail-closed behavior

KNOWN:
isolated runner 22/22 PASS

## Older production/runtime commits to inspect for ancestry only

- 5e68034
- 5151699
- ef2e668

Do not automatically cherry-pick them.
First determine whether the chosen integration base already contains/supersedes their required changes.

---

# 2. Known untracked / unrelated files

There are untracked files such as:
- test_*.py
- DM_GOOGLE_OAUTH_UAT_*

These belong to other IDE/session work.

DO NOT:
- delete
- git clean
- commit them
- move them into this package

Keep them untouched and report them separately.

stash@{0} was reported equivalent to 43e59d3 and disposable, but DO NOT drop stash in this task unless direct evidence proves it is safe and necessary. Prefer leave unchanged.

---

# 3. Central deploy repo — DO NOT MIX YET

Central deploy currently has uncommitted modifications including:
- services.yaml
- deploy.sh
- audit.sh
- critical-smoke.sh
- 2 untracked files

Known problem:
services.yaml also changes Letter/orc port 9422 → 9423 so dmruntimestaging can use 9422.

This affects another service.

DO NOT commit or integrate central deploy in this task.
DO NOT include the Letter/orc port change accidentally.

Only inventory its state and report:
DEPLOY_REPO_PENDING = YES

A separate deploy-owner task will split DM-only deploy changes from unrelated Letter/orc port changes.

---

# 4. Integration strategy

Create a new clean integration branch/worktree:

BRANCH:
dm/integration-audit-20260920

WORKTREE:
choose a new non-conflicting path under F:\00-Ticenpi-SaaS\

Before mutation:
- git worktree list
- git branch --contains for all known commits
- git merge-base / ancestry
- git show --stat / diff for 559d6af, 86f2c97, 2325cca, 0a32a59
- determine the correct common base

Do not assume a199d90 is the correct final base until ancestry is proven.

---

# 5. Required integration logic

The final DM integration candidate MUST contain the functional intent of:

1. entitlement enforcement — 86f2c97
2. release gates — 2325cca
3. AI/local tooling — 559d6af
4. all still-required runtime/build fixes from 5e68034 / 5151699 / ef2e668, but only if not already in ancestry
5. staging digest-pin logic only if semantically correct for a future final candidate

IMPORTANT:
Do NOT freeze a candidate containing an old digest pin that points to a199d90 artifact.

If 0a32a59 hardcodes/pins the old a199d90 CI digest:
- do not treat that digest as final
- preserve the mechanism if needed
- mark FINAL_STAGING_DIGEST_PIN = WAITING_FOR_NEW_CI_ARTIFACT

---

# 6. Conflict / overlap audit

Before cherry-pick/merge, build a file overlap map.

Pay special attention to:
- app/auth/*
- app/routers/*
- app/release_gates*
- app/agnes.py
- frontend AiDrawPanel.vue
- A4Preview.vue
- assets.js
- vite.config.js
- docker-compose.yml
- runtime-config generation
- _charter/history.md

If two packages modify the same file:
resolve by behavior/intent, not by "ours" or "theirs".

No blanket ours/theirs.

---

# 7. Tests after integration

Run at least:

## Backend
- auth tests
- entitlement tests
- release gate tests
- scrape tests
- AI/AGNES tests
- relevant full regression

Known pre-existing failures must be reproduced on the verified base before classifying them as pre-existing.

## Frontend
- targeted AI draw tests
- npm test
- typecheck
- build

## Static / scripts
- PowerShell syntax / safe execution for:
  - start_local_dev.ps1
  - start_local_docker.ps1
  - verify_docker_is_latest.ps1
- python compile where relevant
- compose config validation

## Secret scan
Tracked diff must contain:
- no AGNES key
- no JWT secret
- no passwords/tokens

---

# 8. Rebuild localhost:9421 from the integration HEAD

This is mandatory.

Use the integrated candidate HEAD, not 559d6af alone.

Use the intended local/disposable runtime contract.

Requirements:
- stop/replace stale 9421 Docker as required
- rebuild from integration HEAD
- no Production Supabase
- no Production origin
- no Production DB/queue/storage targets

Then verify:

LOCAL_DOCKER_9421 = PASS

Evidence required:
- http://localhost:9421 = 200
- /api/health = 200
- runtime-config = local/disposable
- frontend bundle is newly built from integration candidate
- image git/source identity is traceable
- verify_docker_is_latest.ps1 -RequireClean = PASS
- Capture real test = PASS
- AI / AGNES real minimal generation = PASS
- auth/release gates = PASS
- tracked worktree clean

Do not accept only "container healthy".

If UI is old or image identity is still working-tree/blank:
LOCAL_DOCKER_9421 = FAIL

---

# 9. Freeze candidate

Only after all integration tests and Local Docker PASS:

record:

INTEGRATION_BRANCH =
dm/integration-audit-20260920

INTEGRATION_HEAD =
<sha>

LOCAL_DOCKER_IMAGE_ID =
...

LOCAL_DOCKER_SOURCE_SHA =
...

LOCAL_DOCKER_VERIFIED =
YES

FROZEN_CANDIDATE_SHA =
<same integration HEAD>

At this point:
READY_TO_PUSH_FOR_CI_ARM64 = YES

But DO NOT push in this task.

---

# 10. Final report

Return:

INTEGRATION_BRANCH =
...

INTEGRATION_HEAD =
...

BASE =
...

COMMITS_INCLUDED =
- ...
- ...

COMMITS_SUPERSEDED =
- ...

0A32A59_DECISION =
INCLUDED / PARTIALLY_REIMPLEMENTED / SUPERSEDED / WAITING_FOR_NEW_DIGEST

ENTITLEMENT_86F2C97 =
INCLUDED / MISSING

RELEASE_GATES_2325CCA =
INCLUDED / MISSING

AI_LOCAL_TOOLING_559D6AF =
INCLUDED / MISSING

OLDER_RUNTIME_FIXES =
...

CONFLICTS =
...

BACKEND_TESTS =
...

FRONTEND_TESTS =
...

SCRIPT_VALIDATION =
...

SECRET_SCAN =
PASS / FAIL

LOCAL_DOCKER_9421 =
PASS / FAIL

VERIFY_DOCKER_IS_LATEST =
PASS / FAIL

CAPTURE_LOCAL_DOCKER =
PASS / FAIL

AI_LOCAL_DOCKER =
PASS / FAIL

WORKTREE_CLEAN_TRACKED =
YES / NO

UNTRACKED_OTHER_AGENT_FILES =
PRESERVED / NOT_PRESERVED

DEPLOY_REPO_PENDING =
YES

FROZEN_CANDIDATE_SHA =
...

READY_TO_PUSH_FOR_CI_ARM64 =
YES / NO

READY_FOR_STAGING =
NO
(reason: CI ARM64 + deploy repo split still pending)

BLOCKERS =
...

## Hard stop
Do not push.
Do not trigger CI.
Do not modify central deploy.
Do not deploy Staging.
Do not deploy Production.
