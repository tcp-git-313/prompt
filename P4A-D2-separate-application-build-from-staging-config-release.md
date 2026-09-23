# P4A-D2 — Separate Immutable Application Artifact Build from Staging Config-Only Release

ROLE: RELEASE PIPELINE / ARTIFACT IDENTITY REPAIR OWNER

MODE: STAGING-SAFE CI REPAIR / NO DEPLOYMENT / NO PRODUCTION MUTATION

REPOSITORY:
F:\00-Ticenpi-SaaS\TicenpiPost

## GOAL

Repair the Post release pipeline so that:

1. An application-source commit builds and publishes one immutable container image whose digest is the canonical release artifact identity.
2. A later Staging deployment/config-only commit can pin that already-verified digest and run validation without rebuilding or republishing application code.
3. The config-only commit cannot create a recursive “new commit -> new image digest -> compose pin mismatch” loop.
4. Production remains completely untouched.

This task repairs the release mechanism only. It must not perform the final Central Seat cutover or any Staging deployment.

## HARD SAFETY BOUNDARIES

Do not:

- deploy Post Staging;
- change any running container or runtime policy;
- touch Production compose, environment, release directories, containers, DNS, or deployment state;
- change Central Seat SQL/RPC semantics or recreate/reassign the existing fixture users;
- modify Post authentication, tenant isolation, or business behavior;
- reset, clean, stash, rebase, or overwrite unrelated dirty work;
- weaken branch protection, required checks, registry permissions, or secret handling;
- use a mutable tag as the authoritative deployment identity;
- claim that a green CI run proves a runtime or E2E result.

If the repository's current release process cannot safely support the split without a broader product or Production change, stop with `HARD_STOP` and document the exact blocker.

## VERIFIED CONTEXT FROM P4A-D1

The fixed Staging Central Seat fixture is already ready and must be preserved:

```text
FIXED_STAGING_TEST_FIXTURE_READY = YES
POST_TEST_ASSIGNED_SEAT = YES
POST_TEST_UNASSIGNED_SEAT = NO
POST_TEST_ASSIGNED_TENANT_MAPPING = YES
POST_TEST_UNASSIGNED_TENANT_MAPPING = YES
```

Test customer:

```text
5b3b204b-25a5-411b-aacb-ca8b4c617482
```

The previous authoritative application candidate was commit `27e4efe`.

The previously verified application image digest was:

```text
sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0
```

The D1 failure was caused by CI rebuilding on the config-only commit. The config candidate pinned one digest while its own CI produced another digest (`sha256:c03f6f9c…`). Therefore, D2 must make build identity independent from deployment/config commit identity.

Prior evidence:

- D1 prompt: https://raw.githubusercontent.com/tcp-git-313/prompt/main/P4A-D1-reconcile-staging-image-digest-and-final-cutover.md
- D1 CI evidence: https://github.com/tcp-git-313/ticenpi-post/actions/runs/35903771176

Treat these values as starting evidence, not as a substitute for inspecting the current repository and current CI configuration.

## 1. PREFLIGHT AND DIRTY-WORK PROTECTION

Before editing, record:

- repository root and current branch;
- HEAD and `git status --short`;
- dirty files and any existing worktrees;
- current workflow files and deployment/config paths;
- current image build/push jobs, triggers, tags, digests, labels, and required check names;
- current Staging and Production deployment entrypoints.

Use an isolated worktree or a dedicated branch from the correct repository state. Preserve the user's primary checkout and all unrelated WIP. Do not infer that a clean-looking worktree means Production is safe; prove the changed-file scope.

## 2. MODEL THE TWO RELEASE PATHS

Document the current flow and implement the smallest safe separation:

### A. Application artifact build path

Triggered by an application-source change or an explicit manual build for an exact source commit. It may:

- run application tests and the existing security checks;
- build the application image;
- push the image;
- publish the immutable digest;
- publish immutable provenance metadata linking at least:
  - source commit SHA;
  - image digest;
  - CI workflow/run identity;
  - image name and target platform(s);
  - Dockerfile/build context identity where available.

The digest, not a tag and not the later deployment commit, is the artifact identity.

### B. Staging config-only release path

Triggered by a Staging deployment/config-only change or an explicit validation of such a change. It may:

- validate that changed files are within the approved Staging/config allowlist;
- validate compose syntax and effective configuration;
- validate that every pinned digest is immutable, pullable, and backed by the approved artifact provenance;
- run deployment preflight and relevant config/contract tests;
- emit a stable green config/release check.

It must not:

- run `docker build`, BuildKit, `docker/build-push-action`, or an equivalent image-build step;
- push an image or overwrite a tag;
- generate a new application digest;
- silently substitute `HEAD` of the config-only commit for the application source commit;
- make a Staging deployment as part of D2.

If the current workflow is monolithic, split it into separate jobs/workflows or use a safe reusable workflow with explicit inputs. Do not rely only on a job name or a human convention: make the no-build property enforceable and testable.

## 3. IMMUTABLE ARTIFACT CONTRACT

Choose the repository's smallest durable implementation for an artifact manifest or equivalent release evidence. The contract must make a config-only commit able to reference an artifact built from an earlier application commit.

At minimum, the Staging pin or its adjacent release metadata must identify:

```text
APPLICATION_SOURCE_COMMIT = <exact source SHA>
IMAGE = <exact image name>
IMAGE_DIGEST = sha256:<full digest>
ARTIFACT_BUILD_RUN = <immutable CI run/workflow identity>
ARTIFACT_PROVENANCE = <existing verifiable evidence or repository-native metadata>
```

The validator must reject:

- missing or abbreviated digests;
- tags without a digest;
- a digest that cannot be resolved in the intended registry;
- a digest whose provenance does not match the declared application source commit;
- a digest produced by an untrusted branch/event or unapproved workflow;
- a config-only change that changes application source identity without a new artifact build;
- a Production image/config reference in a Staging-only candidate.

Prefer existing registry labels, OCI metadata, attestations, release manifests, or CI artifacts already used by the repository. Do not introduce a new external service or secret merely to store a manifest. Never print credentials or secret values in logs.

## 4. WORKFLOW TRIGGER AND JOB DESIGN

Inspect the actual repository and implement the appropriate mechanism. The result must satisfy all of the following:

- application build jobs run only for application-source/build requests;
- Staging-only config changes do not schedule an image build or image push;
- the config-only workflow still performs meaningful validation and cannot be bypassed by a false path classification;
- source changes that affect the image cannot be classified as config-only;
- workflow dispatch inputs, if supported, require an exact source/ref and make the resulting artifact identity explicit;
- pull requests from untrusted forks cannot push images or use privileged deployment credentials;
- required check names remain stable enough for branch protection and release gates;
- a skipped build is distinguishable from a failed build and from a successful config validation;
- the workflow does not use a mutable `latest`, branch tag, or run-number tag as the release authority.

If path filters are used, define and test the allowlist/denylist precisely. Include Dockerfile, dependency lockfiles, build scripts, application source, migrations that are packaged into the image, and workflow/build configuration in the application-build set whenever they affect the image.

## 5. CONFIG-ONLY NEGATIVE GUARANTEE

Add a repository-native check, test, or CI assertion that demonstrates:

```text
config-only candidate -> build job NOT RUN
config-only candidate -> image push NOT RUN
config-only candidate -> pinned artifact validation RUN
config-only candidate -> compose/config tests RUN
```

The evidence must come from the actual workflow graph/run summary or an equivalent machine-verifiable result, not from a comment claiming that no build occurred.

Also add a negative test showing that an application-source change cannot pass through the config-only path without invoking the artifact-build path.

## 6. STAGING-ONLY RELEASE MANIFEST VALIDATION

Make the Staging deployment manifest/config validator prove:

- the service image is pinned by full digest;
- the digest is tied to the declared application source commit;
- only allowed Staging services/configuration changed;
- the intended Staging service port and dependency wiring are unchanged;
- Production files and references are unchanged;
- no unrelated environment variable drift exists;
- the Central Seat runtime policy is not changed by D2;
- the existing Seat fixture remains data-only evidence and is not mutated.

The final D2 candidate may contain a sample or test config-only pin, but it must not be deployed. If changing the real Staging pin is needed to exercise the validator, keep it in the isolated candidate and report it as not deployed.

## 7. VALIDATION

Run the smallest complete validation set supported by the repository, including:

- workflow syntax/lint validation;
- YAML/JSON/schema validation for release metadata;
- compose syntax and effective-config validation;
- deployment preflight in no-op/read-only mode;
- `git diff --check`;
- targeted tests for image identity, provenance, path classification, and config-only no-build behavior;
- existing application tests required by the build path;
- a dry-run or test invocation for both an application-source candidate and a config-only candidate.

Do not claim a Docker build was avoided merely because a local test did not run Docker. Prove it in the CI workflow design or run evidence.

## 8. CI ACCEPTANCE GATES

The task is complete only when the evidence supports all of these:

```text
APPLICATION_BUILD_PATH_DEFINED = YES
APPLICATION_ARTIFACT_IMMUTABLE = YES
ARTIFACT_SOURCE_PROVENANCE = YES
CONFIG_ONLY_PATH_DEFINED = YES
CONFIG_ONLY_BUILD_SKIPPED = YES
CONFIG_ONLY_IMAGE_PUSH_SKIPPED = YES
CONFIG_ONLY_DIGEST_VALIDATION = YES
APPLICATION_CHANGE_CANNOT_BYPASS_BUILD = YES
STAGING_ONLY_SCOPE = YES
PRODUCTION_UNCHANGED = YES
CENTRAL_SEAT_FIXTURE_UNCHANGED = YES
DEPLOYMENT_PERFORMED = NO
```

If any required gate is not proven, stop with `HARD_STOP`. Do not compensate by deploying or by accepting a tag-based release.

## 9. REQUIRED FINAL REPORT

Return a concise evidence-backed report with these exact fields:

```text
A. PREFLIGHT
REPOSITORY =
BRANCH =
HEAD =
DIRTY_WIP_PRESERVED =

B. ROOT CAUSE
OLD_PIPELINE_BEHAVIOR =
RECURSIVE_DIGEST_PROBLEM =

C. IMPLEMENTATION
FILES_CHANGED =
APPLICATION_BUILD_WORKFLOW =
CONFIG_ONLY_WORKFLOW =
ARTIFACT_MANIFEST_OR_PROVENANCE =
PATH_CLASSIFICATION_RULE =

D. BUILD PATH
APPLICATION_BUILD_PATH_DEFINED =
APPLICATION_ARTIFACT_IMMUTABLE =
ARTIFACT_SOURCE_PROVENANCE =
EXAMPLE_SOURCE_COMMIT =
EXAMPLE_IMAGE_DIGEST =

E. CONFIG-ONLY PATH
CONFIG_ONLY_PATH_DEFINED =
CONFIG_ONLY_BUILD_SKIPPED =
CONFIG_ONLY_IMAGE_PUSH_SKIPPED =
CONFIG_ONLY_DIGEST_VALIDATION =
APPLICATION_CHANGE_CANNOT_BYPASS_BUILD =

F. VALIDATION
WORKFLOW_LINT =
COMPOSE_VALIDATION =
DEPLOY_PREFLIGHT =
TARGETED_TESTS =
GIT_DIFF_CHECK =

G. SAFETY
STAGING_ONLY_SCOPE =
PRODUCTION_UNCHANGED =
CENTRAL_SEAT_FIXTURE_UNCHANGED =
DEPLOYMENT_PERFORMED =
RUNTIME_POLICY_CHANGED =

H. FINAL RESULT
```

Use exactly one final result:

### PASS

```text
RESULT = PASS
P4A_D2_RELEASE_PIPELINE_REPAIR = PASS
READY_FOR_P4A_D3_STAGING_DEPLOYMENT = YES
```

### PARTIAL_PASS

Use only when the separation is implemented but one non-safety validation requires a clearly listed follow-up. Do not mark ready for deployment unless all CI identity and no-build gates are proven.

```text
RESULT = PARTIAL_PASS
P4A_D2_RELEASE_PIPELINE_REPAIR = PARTIAL_PASS
READY_FOR_P4A_D3_STAGING_DEPLOYMENT = NO
FOLLOW_UP_REQUIRED =
```

### HARD_STOP

Use when artifact provenance, no-build evidence, changed-file scope, or Production isolation cannot be proven.

```text
RESULT = HARD_STOP
P4A_D2_RELEASE_PIPELINE_REPAIR = HARD_STOP
READY_FOR_P4A_D3_STAGING_DEPLOYMENT = NO
BLOCKER =
```

P4A-D2 must leave Production untouched and must not perform the final Central Seat cutover. 
