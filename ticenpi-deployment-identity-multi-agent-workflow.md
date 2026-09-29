# Ticenpi Deployment Identity — Multi-Agent Investigation, Review, and Execution Workflow

## PURPOSE

Resolve the current Ticenpi Deployment Identity problem without redoing the already-completed Product / Service Phase 1 investigation.

The current issue concerns these five deployment-identity fields:

```text
TICENPI_ENVIRONMENT
TICENPI_RELEASE_ID
TICENPI_COMMIT_SHA
TICENPI_SUPABASE_PROJECT_REF
TICENPI_DATABASE_TARGET
```

The goal is not to make all five keys exist in `shared/.env`.

The goal is to determine, from the actual implementation, which values are:

```text
RUNTIME_REQUIRED
DEPLOY_TIME_INJECTED
DERIVED
REDUNDANT
UNRESOLVED
```

and then make the smallest safe correction while preserving fail-closed Production identity validation.

---

# 0. AUTHORITATIVE WORKSPACE

Main workspace:

```text
F:\00-Ticenpi-SaaS
```

Deployment / platform repo:

```text
F:\00-Ticenpi-SaaS\deploy
```

System documentation directory:

```text
F:\00-Ticenpi-SaaS\deploy\docs\system
```

The target universal workflow document is:

```text
F:\00-Ticenpi-SaaS\deploy\docs\system\new-workflow.md
```

Related system documents:

```text
F:\00-Ticenpi-SaaS\deploy\docs\system\PRODUCT_TO_STAGING.md
F:\00-Ticenpi-SaaS\deploy\docs\system\STAGING_TO_PRODUCTION.md
F:\00-Ticenpi-SaaS\deploy\docs\system\SYSTEM_OWNERSHIP.md
F:\00-Ticenpi-SaaS\deploy\docs\system\PRODUCT_INTEGRATION_CONTRACT.md
F:\00-Ticenpi-SaaS\deploy\docs\system\TICENPI_SYSTEM_CURRENT_STATE.md
F:\00-Ticenpi-SaaS\deploy\docs\system\STAGING_TEST_IDENTITY_CONTRACT.md
```

Release evidence:

```text
F:\00-Ticenpi-SaaS\.release-evidence\
```

DM repo when product-level runtime inspection is needed:

```text
F:\00-Ticenpi-SaaS\TicenpiDM
```

Known Deployment Identity branch / commit from the prior investigation:

```text
branch: fix/dm-deployment-identity-20260929
commit: 1bae728
```

Do not assume that branch is already canonical `main`.
Verify current branch / HEAD before using it as an execution base.

---

# 1. ACCEPTED BASELINE — DO NOT REDO

The following findings are already accepted.

Do not repeat the entire Phase 1 investigation unless direct source evidence contradicts one of these statements.

## 1.1 Contract separation

```text
COMMERCIAL_AUTHORIZATION_CONTRACT
!=
DEPLOYMENT_IDENTITY_CONTRACT
```

Central Commercial / Central Seat answers:

```text
Can this user use this product?
```

Deployment Identity answers:

```text
What exact artifact/source/config/environment/database target is being deployed and running?
```

The five `TICENPI_*` fields belong to Deployment Identity.

They are not evidence that Central Seat is broken.

## 1.2 Incorrect model already identified

This model is considered incorrect:

```text
Every service must permanently store all five TICENPI_* keys
inside VPS shared/.env / requiredEnv.
```

The intended direction is:

```text
STATIC_SHARED_ENV
+
DEPLOY_TIME_IDENTITY_METADATA
=
EFFECTIVE_DEPLOYMENT_ENV
```

A runtime process may receive an identity value through environment variables.

That does not mean a human must maintain that value permanently in `shared/.env`.

## 1.3 Production promotion contract

For Docker → Docker products such as DM:

```text
STAGING ACCEPTED
→ accepted immutable artifact
→ Production preflight
→ promote same immutable artifact
→ no Production rebuild
```

Production must not ask a human to re-enter identity already determined by accepted Staging evidence.

## 1.4 Product / Service Phase 1 is separate

Do not redo:

- Product Registry design
- Product vs Service separation
- Central Seat product integration
- fake Product prevention
- ExtractionHub / Launcher / Admin Workbench classification

Those are a separate Product Onboarding V2 workstream.

If you find a direct contradiction that materially affects Deployment Identity, report:

```text
BASELINE_CONTRADICTION =
EVIDENCE =
IMPACT =
```

Do not silently redesign the other workstream.

---

# 2. WORK ALLOCATION

Use three formal stages plus an optional cheap-worker pool.

```text
Stage 1 — Investigator
Kimi K3 Max

Stage 2 — Independent Reviewer
MiMo-V2.6-Pro High / Max

Stage 3 — Executor
GLM-5.3 High

Parallel support pool
MiMo-V2.6-Flash / DeepSeek V4 Flash
```

Important:

Do not have all models independently rescan the full repository.

The responsibility split is:

```text
Investigator = establish facts
Reviewer     = challenge conclusions
Executor     = execute approved exact plan
Flash pool   = mechanical evidence collection only
```

---

# TASK A — KIMI K3 MAX
# Deployment Identity Remaining-Gaps Audit

## ROLE

You are the primary investigator.

Your strongest required capabilities are:

- Agentic Coding / Software Engineering
- Repository understanding
- Git history investigation
- Tool use
- Long-context reasoning

This is a targeted remaining-gaps audit.

It is NOT a new Phase 1.

## MODE

READ ONLY.

Do not:

- modify source files
- modify documentation
- modify `services.yaml`
- add keys to `shared/.env`
- create secrets
- commit
- push
- deploy Staging
- deploy Production
- run a destructive DB operation

Read-only commands, tests that do not mutate environment state, Git inspection, and evidence inspection are allowed.

## A1. TRACE THE REAL CALL CHAIN

Trace the actual current implementation:

```text
promote.ps1
→ deploy.ps1
→ server/preflight.sh
→ server/deploy.sh
→ compose / runtime
→ server/audit.sh
→ server/rollback.sh
```

Also inspect where relevant:

```text
server/deployment_identity.py
server/manifest_tool.py
scripts/validate-services-manifest.py
schemas/services-manifest.schema.json
tools/release_evidence_tool.py
release.ps1
services.yaml
```

For DM inspect the actual staging / production compose and runtime identity consumers.

For every one of these variables:

```text
TICENPI_ENVIRONMENT
TICENPI_RELEASE_ID
TICENPI_COMMIT_SHA
TICENPI_SUPABASE_PROJECT_REF
TICENPI_DATABASE_TARGET
```

also search historical alias:

```text
TICENPI_GIT_SHA
```

Produce:

```text
VARIABLE =
DEFINED_AT =
SOURCE =
INJECTED_AT =
VALIDATED_AT =
RUNTIME_CONSUMER =
AUDIT_CONSUMER =
ROLLBACK_CONSUMER =
HEALTH_CONSUMER =
CURRENT_REQUIRED_ENV =
CURRENT_SHARED_ENV_REQUIRED =
```

Do not infer. Point to exact files / symbols / lines where possible.

## A2. FIND THE REQUIREDENV ROOT CAUSE

Inspect:

```text
F:\00-Ticenpi-SaaS\deploy\services.yaml
F:\00-Ticenpi-SaaS\deploy\schemas\services-manifest.schema.json
F:\00-Ticenpi-SaaS\deploy\scripts\validate-services-manifest.py
F:\00-Ticenpi-SaaS\deploy\server\manifest_tool.py
```

Determine exactly which layer currently requires the five identity keys to appear in `requiredEnv`.

Use:

```text
git log
git blame
git show
```

Find, if possible:

```text
INTRODUCING_COMMIT =
INTRODUCING_FILE =
INTRODUCING_RULE =
```

Do not report a commit unless evidence supports it.

## A3. EXPLAIN THE DM FAILURE PRECISELY

Determine why the recent DM Production preflight stopped before mutation because the five `TICENPI_*` keys were absent from:

```text
/opt/ticenpi/dm/shared/.env
```

Answer:

```text
DM_FAILURE_BRANCH =
DM_FAILURE_TOOLING_COMMIT =
DM_FAILURE_PREFLIGHT_VERSION =
DM_FAILURE_EXACT_CHECK =
DM_FAILURE_EXPECTED_SOURCE =
DM_FAILURE_ACTUAL_SOURCE =
```

Specifically determine whether that failure happened:

- before the `1bae728` effective-env fix,
- on `1bae728`,
- or on another partially rolled-out state.

## A4. INSPECT WHAT RELEASE EVIDENCE ALREADY KNOWS

Inspect:

```text
F:\00-Ticenpi-SaaS\.release-evidence\<product>\accepted-staging.json
F:\00-Ticenpi-SaaS\.release-evidence\<product>\history\*.json
release.ps1
promote.ps1
tools\release_evidence_tool.py
```

Use actual field names.

Confirm whether accepted Staging evidence already contains:

- source commit
- immutable artifact digest / artifact id
- deploy config commit
- Staging release id
- runtime identity verification state

Then confirm whether `promote.ps1` already:

- reads accepted Staging evidence
- pins the accepted immutable artifact
- blocks rebuild
- knows accepted source identity
- knows Staging release identity

Output:

```text
ACCEPTED_STAGING_ALREADY_KNOWS =
PROMOTE_ALREADY_KNOWS =
```

## A5. CLASSIFY EACH VARIABLE

For each of the five fields, choose exactly one primary classification:

```text
RUNTIME_REQUIRED
DEPLOY_TIME_INJECTED
DERIVED
REDUNDANT
UNRESOLVED
```

For every field output:

```text
VARIABLE =
CLASS =
CANONICAL_SOURCE =
DERIVED_FROM =
MUST_EXIST_IN_SHARED_ENV = YES/NO
RUNTIME_NEEDS_VALUE = YES/NO
WHY =
```

Do not assume all five survive.

Do not assume all five should be removed.

## A6. SHA ALIAS AUDIT

Investigate:

```text
TICENPI_GIT_SHA
TICENPI_COMMIT_SHA
```

Answer:

- which files consume each
- whether both remain active
- which one is canonical
- whether health / runtime / deployment identity can diverge because of the alias
- whether rollback preserves the correct source identity

Output:

```text
SHA_ALIAS_RESULT =
CANONICAL_SHA_FIELD =
MIGRATION_REQUIRED = YES/NO
```

## A7. DATABASE TARGET AUDIT

Do not redesign database architecture.

Determine whether:

```text
TICENPI_DATABASE_TARGET
```

has independent semantic meaning beyond the canonical Supabase project / DB target.

If DM Production DB is simply the Postgres database inside the one canonical Production Supabase project, determine whether this field is:

```text
DERIVED
REDUNDANT
```

If it has independent meaning, identify the canonical source.

Output:

```text
DATABASE_TARGET_CLASS =
CANONICAL_SOURCE =
DERIVED_FROM =
KEEP_OR_REMOVE_REASON =
```

## A8. DOCUMENT DELTA — ONLY RELEVANT SECTIONS

Do not re-review every document from scratch.

Compare actual implementation with:

```text
F:\00-Ticenpi-SaaS\deploy\docs\system\new-workflow.md
F:\00-Ticenpi-SaaS\deploy\docs\system\PRODUCT_TO_STAGING.md
F:\00-Ticenpi-SaaS\deploy\docs\system\STAGING_TO_PRODUCTION.md
F:\00-Ticenpi-SaaS\deploy\docs\system\SYSTEM_OWNERSHIP.md
F:\00-Ticenpi-SaaS\deploy\docs\system\PRODUCT_INTEGRATION_CONTRACT.md
F:\00-Ticenpi-SaaS\deploy\docs\system\TICENPI_SYSTEM_CURRENT_STATE.md
```

`STAGING_TEST_IDENTITY_CONTRACT.md` is Central Seat fixture documentation.

Do not modify or re-audit it unless direct Deployment Identity coupling is discovered.

For each relevant document output only:

```text
DOCUMENT =
SECTION =
CURRENT_RULE =
ACTUAL_IMPLEMENTATION =
CONFLICT =
CHANGE_REQUIRED = YES/NO
CORRECT_RULE =
```

## A9. TARGET DESIGN TO TEST AGAINST

Use this as the hypothesis to verify, not as a fact to force:

```text
Accepted Staging Evidence
+ Deploy Target
+ Canonical Environment Config
+ Release Tooling Identity
=
EXPECTED DEPLOYMENT IDENTITY

EXPECTED DEPLOYMENT IDENTITY
vs
EFFECTIVE DEPLOYMENT IDENTITY
=
Preflight / Audit decision
```

Expected direction if implementation supports it:

```text
TICENPI_ENVIRONMENT
← deploy/promote target

TICENPI_RELEASE_ID
← canonical release/promote tooling identity

TICENPI_COMMIT_SHA
← accepted source commit

TICENPI_SUPABASE_PROJECT_REF
← canonical Supabase URL / target

TICENPI_DATABASE_TARGET
← canonical DB target, possibly derived
```

Preflight must fail closed on mismatch.

Preflight should not fail merely because a derived deploy-time identity value is absent from persistent `shared/.env`.

## A10. FINAL OUTPUT

Return exactly these major sections:

```text
CURRENT_CALL_CHAIN =

DM_FAILURE_EXACT_CAUSE =

INTRODUCING_COMMIT =

VARIABLE_MATRIX =

WRONG_REQUIREDENV_RULE =

PREFLIGHT_CURRENT_MODEL =

ACCEPTED_STAGING_ALREADY_KNOWS =

PROMOTE_ALREADY_KNOWS =

SHA_ALIAS_RESULT =

DATABASE_TARGET_RESULT =

CODE_FILES_TO_FIX =

DOC_SECTIONS_TO_FIX =

MINIMAL_FIX_PLAN =

TEST_PLAN =

BASELINE_CONTRADICTION = NONE / <details>

PLAN_READY_FOR_REVIEW = YES/NO
```

If any material conclusion still requires the Executor to guess:

```text
PLAN_READY_FOR_REVIEW = NO
```

---

# TASK B — MIMO-V2.6-PRO HIGH / MAX
# Independent Adversarial Review

## ROLE

You are the independent reviewer.

You are not the primary investigator.

Your job is to challenge the Stage 1 conclusions before any implementation begins.

## INPUT REQUIRED

You must receive the full Task A output, including its evidence references.

Also read:

```text
F:\00-Ticenpi-SaaS\deploy\docs\system\new-workflow.md
```

Read only the source files needed to verify disputed claims.

Do not redo a full repository audit.

## MODE

READ ONLY.

Do not modify, commit, push, deploy, or repair anything.

## B1. VERIFY THE CLASSIFICATION

For every variable:

```text
TICENPI_ENVIRONMENT
TICENPI_RELEASE_ID
TICENPI_COMMIT_SHA
TICENPI_SUPABASE_PROJECT_REF
TICENPI_DATABASE_TARGET
```

independently assess whether Task A correctly classified it as:

```text
RUNTIME_REQUIRED
DEPLOY_TIME_INJECTED
DERIVED
REDUNDANT
UNRESOLVED
```

Challenge any classification that relies only on:

- variable name
- documentation wording
- validator policy
- assumptions from another product

Prefer actual consumers and actual deployment flow.

## B2. CHECK SOURCE-OF-TRUTH DUPLICATION

For every proposed retained value ask:

```text
Does another authoritative source already uniquely determine this value?
```

Look specifically for duplicate truths between:

- accepted-staging evidence
- service manifest
- command target
- Supabase URL
- Production DB config
- compose env
- shared/.env
- runtime labels
- deployment identity JSON

Flag any proposal that creates two independently maintained values for the same identity fact.

## B3. CHECK IMMUTABLE PROMOTION SAFETY

Verify that the proposed fix preserves:

```text
STAGING ACCEPTED ARTIFACT
=
PRODUCTION PROMOTED ARTIFACT
```

For Docker→Docker products.

Reject any design that:

- rebuilds Production
- derives Production commit independently from a mutable branch
- lets runtime identity disagree with accepted artifact provenance
- overwrites accepted-staging evidence
- permits a mutable image tag to substitute for digest identity

## B4. CHECK FAIL-CLOSED BEHAVIOR

Removing a static `requiredEnv` requirement must not weaken identity validation.

Confirm the replacement still fails closed for at least:

- Production command but effective environment != production
- accepted commit != runtime commit
- accepted digest != promoted/running digest
- Production Supabase target != effective Supabase target
- canonical DB target != effective DB target when DB target has independent meaning
- release identity cannot be correlated to evidence

## B5. CHECK ROLLBACK SAFETY

Inspect whether the proposed identity derivation still allows rollback to restore:

- previous artifact
- previous config
- previous release identity
- correct environment identity
- correct Supabase / DB target

Reject a change that causes rollback to inherit identity from the new failed release.

## B6. CHECK SHA ALIAS RESOLUTION

Verify Task A's conclusion on:

```text
TICENPI_GIT_SHA
TICENPI_COMMIT_SHA
```

Reject any migration plan that breaks existing health / audit / rollback consumers without compatibility handling.

## B7. CHECK DOCUMENT CHANGES

Ensure the plan correctly separates:

```text
Static Runtime Config / Secrets
vs
Deployment Identity Metadata
```

At minimum verify planned deltas for:

```text
PRODUCT_TO_STAGING.md
STAGING_TO_PRODUCTION.md
SYSTEM_OWNERSHIP.md
PRODUCT_INTEGRATION_CONTRACT.md
TICENPI_SYSTEM_CURRENT_STATE.md
new-workflow.md
```

Do not require a change to `STAGING_TEST_IDENTITY_CONTRACT.md` unless there is direct coupling.

## B8. SCOPE CONTROL

Reject unnecessary redesign.

This task is not permission to redesign:

- Central Seat
- Commercial Core
- Product Registry
- Product / Service classification
- tenant/RLS
- OAuth architecture

## B9. FINAL OUTPUT

Return:

```text
REVIEW_RESULT = APPROVED / CHANGES_REQUIRED

VARIABLE_CLASSIFICATION_REVIEW =

SOURCE_OF_TRUTH_REVIEW =

IMMUTABLE_PROMOTION_REVIEW =

FAIL_CLOSED_REVIEW =

ROLLBACK_REVIEW =

SHA_ALIAS_REVIEW =

DATABASE_TARGET_REVIEW =

DOC_DELTA_REVIEW =

OVERREACH_FOUND =

REQUIRED_PLAN_CHANGES =

EXECUTION_PLAN_APPROVED = YES/NO
```

Only output:

```text
EXECUTION_PLAN_APPROVED = YES
```

when the plan is specific enough that an Executor does not need to redesign anything.

---

# TASK C — GLM-5.3 HIGH
# Approved-Plan Executor

## ROLE

You are the Executor.

You do not redesign Deployment Identity.

You execute only a plan that passed Task B with:

```text
EXECUTION_PLAN_APPROVED = YES
```

## REQUIRED INPUT

You must receive:

1. Full Task A investigation output.
2. Full Task B review output.
3. The final approved exact file plan.
4. Current authoritative branch / commit.

If any of those are missing:

```text
EXECUTION_BLOCKED = MISSING_APPROVED_PLAN
```

and stop.

## C1. WORKTREE SAFETY

Before editing:

- verify repo / branch / HEAD
- inspect dirty tracked + untracked state
- do not overwrite unrelated WIP
- use an isolated worktree / branch when the primary checkout is dirty or contains unrelated work
- do not use `git add -A`

Report:

```text
SOURCE_BASE =
WORKTREE =
WORKTREE_CLEAN =
UNRELATED_WIP_PRESERVED =
```

## C2. EXECUTION RULE

For every approved change, follow:

```text
FILE
→ SYMBOL / SECTION
→ APPROVED CHANGE
→ TEST
```

Do not expand scope.

If source reality requires a materially different design than the approved plan:

```text
PLAN_DEVIATION_REQUIRED = YES
REASON =
EVIDENCE =
```

Stop.

Do not improvise a replacement architecture.

## C3. REQUIRED BEHAVIOR

The final implementation must preserve the approved contract:

```text
STATIC_SHARED_ENV
+
DEPLOY_TIME_IDENTITY_METADATA
=
EFFECTIVE_DEPLOYMENT_ENV
```

and:

```text
EXPECTED DEPLOYMENT IDENTITY
=
accepted evidence
+ deploy target
+ canonical config
+ canonical release tooling identity
```

Then preflight/audit compare expected identity against actual effective/running identity.

Do not make all five `TICENPI_*` permanent human-managed `shared/.env` values merely to pass validation.

## C4. TESTING

Run all approved targeted tests.

At minimum, where applicable, include regression coverage for:

- identity metadata injected without persistent shared-env duplicate
- true static requiredEnv still enforced
- missing required secret still fails
- Production environment mismatch fails
- Supabase target mismatch fails
- source commit mismatch fails
- artifact digest mismatch fails
- release identity remains traceable
- rollback identity remains correct
- SHA alias compatibility / migration
- derived DATABASE_TARGET behavior, if approved
- existing manifest collision tests
- existing deployment identity tests
- existing preflight tests

Then run the repository's approved broader regression suite.

Do not weaken or delete a failing test merely to obtain green status.

## C5. DIFF REVIEW

Before commit, inspect:

```text
git status
git diff --check
git diff
```

Classify every changed file:

```text
APPROVED_CHANGE
UNEXPECTED_CHANGE
GENERATED_CHANGE
UNRELATED_CHANGE
```

There must be no unexplained changed file.

## C6. DOCUMENT SYNCHRONIZATION

Update only approved documents.

Expected likely documents, subject to Task A/B exact plan:

```text
F:\00-Ticenpi-SaaS\deploy\docs\system\PRODUCT_TO_STAGING.md
F:\00-Ticenpi-SaaS\deploy\docs\system\STAGING_TO_PRODUCTION.md
F:\00-Ticenpi-SaaS\deploy\docs\system\SYSTEM_OWNERSHIP.md
F:\00-Ticenpi-SaaS\deploy\docs\system\PRODUCT_INTEGRATION_CONTRACT.md
F:\00-Ticenpi-SaaS\deploy\docs\system\TICENPI_SYSTEM_CURRENT_STATE.md
F:\00-Ticenpi-SaaS\deploy\docs\system\new-workflow.md
```

Do not modify `TICENPI_SYSTEM_CURRENT_STATE.md` to claim something is live before tests / rollout evidence actually proves it.

Do not modify `STAGING_TEST_IDENTITY_CONTRACT.md` unless the approved plan found direct coupling.

## C7. PREFLIGHT

After code and tests pass, run only the approved non-destructive preflight / dry-run path.

Do not perform Production mutation without explicit current user authorization.

If the task reaches the Production mutation boundary, output:

```text
READY_FOR_PRODUCTION_AUTHORIZATION = YES
EXACT_COMMAND =
EXPECTED_RESULT =
ROLLBACK_TARGET =
```

and stop.

## C8. COMMIT / PUSH

Only if the approved plan explicitly permits it:

- commit logical changes in controlled commits
- do not include unrelated files
- push only the approved branch

If push was not approved, stop after local verified commit or verified diff, as specified by the approved plan.

## C9. FINAL OUTPUT

Return:

```text
EXECUTION_RESULT = PASS / FAIL / BLOCKED

SOURCE_BASE =
WORKTREE =
CHANGED_FILES =
COMMITS =
TESTS =
VALIDATOR_RESULT =
PREFLIGHT_RESULT =
IDENTITY_DERIVATION_RESULT =
IDENTITY_CROSSCHECK_RESULT =
ROLLBACK_VERIFICATION =
DOCS_UPDATED =
UNEXPECTED_CHANGES =
PLAN_DEVIATION_REQUIRED =
READY_FOR_PRODUCTION_AUTHORIZATION =
EXACT_NEXT_ACTION =
```

---

# TASK D — MIMO-V2.6-FLASH / DEEPSEEK V4 FLASH
# Parallel Cheap Worker Pool

## ROLE

You are a mechanical evidence worker.

You do not make architecture decisions.

You may run in parallel with Task A.

## ALLOWED WORK

Examples:

### Worker D1 — Reference inventory

Search exact references for:

```text
TICENPI_ENVIRONMENT
TICENPI_RELEASE_ID
TICENPI_COMMIT_SHA
TICENPI_GIT_SHA
TICENPI_SUPABASE_PROJECT_REF
TICENPI_DATABASE_TARGET
```

Return:

```text
VARIABLE | FILE | LINE/SYMBOL | READ/WRITE/VALIDATE
```

No conclusions about which field should be removed.

### Worker D2 — Git history inventory

Use Git history to find commits touching:

- requiredEnv validation
- Deployment Identity
- preflight identity checks
- deploy-time env injection

Return candidate commits and exact diffs.

Do not declare root cause.

### Worker D3 — Release evidence inventory

Inspect accepted Staging / history JSON files.

Return actual field names and values with secrets omitted.

Do not decide architecture.

### Worker D4 — Documentation delta inventory

Search only relevant sections in:

```text
F:\00-Ticenpi-SaaS\deploy\docs\system\
```

Return locations mentioning:

- requiredEnv
- shared/.env
- release identity
- runtime identity
- accepted Staging
- Production promotion
- Supabase target
- Deployment Identity

Do not rewrite documents.

### Worker D5 — Test / log classification

If Task C produces failures, classify logs into:

```text
NEW_REGRESSION
PRE_EXISTING_FAILURE
ENVIRONMENT_FAILURE
TEST_ASSUMPTION_MISMATCH
UNKNOWN
```

Do not modify tests.

## FORBIDDEN FOR FLASH WORKERS

Do not decide:

- canonical source of truth
- whether DATABASE_TARGET should be deleted
- Production safety
- rollback architecture
- Central Seat design
- Commercial model
- Product Registry model

Do not edit source.

Do not commit.

Do not deploy.

## FINAL OUTPUT

Every cheap-worker task must return facts only:

```text
WORKER_TASK =
FILES_INSPECTED =
FACTS =
CANDIDATE_EVIDENCE =
UNRESOLVED =
NO_ARCHITECTURE_DECISION_MADE = YES
```

---

# 3. HANDOFF PROTOCOL

The flow is:

```text
Task D workers ──facts──┐
                       ↓
Task A Kimi K3 Max
→ investigation report
                       ↓
Task B MiMo-V2.6-Pro
→ adversarial review
                       ↓
EXECUTION_PLAN_APPROVED = YES
                       ↓
Task C GLM-5.3 High
→ implementation + tests + preflight
                       ↓
Production authorization boundary
```

Task B must not approve based only on prose confidence.

Task A must provide source evidence.

Task C must not act on an unapproved plan.

---

# 4. GLOBAL STOP CONDITIONS

Stop instead of guessing if any of these occur:

- authoritative source cannot be determined
- accepted Staging evidence disagrees with running artifact and the discrepancy is unexplained
- Production target cannot be uniquely determined
- Supabase / DB target cannot be uniquely determined
- rollback identity cannot be reconstructed
- Task B requires a materially different architecture
- current repo state contains unrelated WIP that cannot be safely isolated
- Production mutation would be required without explicit user approval

Output the smallest next action required to unblock.

---

# 5. SUCCESS CONDITION

This workstream is successful only when the implementation can prove:

```text
1. Staging ACCEPTED identity is authoritative for accepted source/artifact provenance.

2. Production Promotion reuses the accepted immutable artifact where the product delivery model supports same-artifact promotion.

3. Deployment target determines environment identity.

4. Canonical Supabase / DB configuration determines data target identity without creating a second manually maintained truth.

5. Release tooling determines release identity.

6. Runtime receives the identity values it actually needs.

7. Preflight / audit fail closed on identity mismatch.

8. Static secrets/config remain protected by requiredEnv where they are genuinely runtime-required.

9. Deploy-time or derived identity metadata is not incorrectly treated as a permanent human-managed shared/.env requirement.

10. Rollback restores a self-consistent previous artifact/config/identity state.
```

Final architecture principle:

```text
Verify identity.
Do not duplicate identity.
```
