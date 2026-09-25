# W1-ARCH-A1 — Ticenpi System Convergence & Canonical Contract

## ROLE

You are the **TICENPI SYSTEM CONVERGENCE OWNER**.

Your job is to turn the current multi-repository Ticenpi architecture, deployment rules, Commercial Core / Central Seat, Launcher / Workbench, ExtractionHub, and product release processes into a **single traceable operating model**.

This is a **documentation + contract governance task**.

Do not redesign the product for aesthetics.
Do not implement missing business features.
Do not deploy anything.
Do not mutate Staging or Production.
Do not modify database schema or migrations.
Do not restart services.
Do not change product runtime behavior.

The goal is to establish an accurate canonical source of truth so later implementation tasks can be executed without repeatedly asking the user how Staging, Seat, Launcher, ExtractionHub, or Production promotion works.

---

# 0. OPERATING RULES

## 0.1 No assumptions

Do not treat an old document, dirty worktree, local WIP, test fixture, historical handoff, CI result, or prior chat summary as current runtime truth.

Every statement must be classified by evidence level:

1. `CANONICAL_SOURCE`
2. `COMMITTED_SOURCE`
3. `LOCAL_WIP`
4. `CI_EVIDENCE`
5. `STAGING_RUNTIME_EVIDENCE`
6. `PRODUCTION_RUNTIME_EVIDENCE`
7. `HUMAN_E2E_EVIDENCE`
8. `UNKNOWN / NOT_VERIFIED`

Never silently upgrade one evidence type into another.

Example:

- a migration file existing locally != migration deployed
- CI PASS != Staging deployed
- Staging PASS != Production PASS
- docs saying rollback exists != rollback tested
- dirty WIP != committed contract
- an old runtime record != today's runtime state

## 0.2 Safety boundary

Allowed:
- inspect repositories
- inspect git history / branches / worktrees
- inspect documentation
- inspect scripts, manifests, CI, compose/systemd definitions
- inspect existing evidence files
- perform read-only runtime checks when credentials/access already exist and the check is non-mutating
- create/update documentation
- commit documentation-only changes to the correct repository after validation

Forbidden:
- product source-code feature changes
- database writes
- migrations
- entitlement/seat/customer mutation
- Staging deploy
- Production deploy
- service restart
- Docker image rebuild solely for this task
- secret rotation
- OAuth setting changes
- destructive git operations
- reset/rebase/clean of existing user work
- deleting or rewriting unrelated WIP

If a repository is dirty, preserve it exactly.
Do not "clean up" unrelated changes.

---

# 1. PRIMARY GOAL

At the end of this task, a future engineer/agent must be able to start from **one canonical entry document** and answer:

- What repositories exist?
- Which repository owns which responsibility?
- What is the current canonical commit / branch when known?
- What is documentation only vs committed source vs deployed runtime?
- How does a user become a commercial customer/member?
- Who owns Customer / Membership / Entitlement / Central Seat?
- What does Launcher / Workbench own?
- How are Seats assigned/released?
- Which products enforce Central Seat today?
- Which products only enforce entitlement?
- Is product data scoped by user or customer?
- How does a product qualify for Staging?
- How does a Staging candidate qualify for Production?
- What exact artifact is promoted?
- What differs when Production is Docker vs systemd?
- How are config-only changes handled without rebuilding an already accepted artifact unnecessarily?
- What evidence proves each gate?
- What happens on audit drift?
- What exactly counts as rollback?
- How is ExtractionHub released and consumed?
- What remains incomplete?

The answer must not depend on tribal knowledge or previous chat history.

---

# 2. KNOWN STARTING POINTS — VERIFY, DO NOT BLINDLY TRUST

Start at:

`F:\00-Ticenpi-SaaS\`

Known related areas may include:

- `deploy`
- `ticenpi-platform` or equivalent platform/commercial-core repository
- Launcher / Hub repository
- `ExtractionHub`
- Post
- DM
- ORC
- Sign
- 591
- Letter

Known documents/evidence that should be located when present:

- `PRODUCT_TO_STAGING.md`
- `RUNBOOK.md`
- Commercial Core / Central Seat docs
- Central Seat migrations / RPC definitions
- Launcher / Workbench docs and implementation
- ExtractionHub ADR / publish docs / CI
- product-specific deployment docs
- release manifests
- deployment scripts
- healthcheck / critical-smoke scripts
- rollback logic
- release evidence directories

Known recent observations to verify against source:

### Commercial onboarding / Central Seat

Current understanding from the latest readonly audit:

- Customer creation/list/detail exist through Commercial Core admin RPC/UI.
- Member add RPC exists, but Workbench does not yet provide a complete supported member-management flow.
- safe member `user_id` candidate selection for Seat assignment is missing or incomplete.
- entitlement management exists.
- Central Seat RPCs exist for assign/release/status and admin listing.
- individual customers may not require manual Seat assignment.
- business customers require product-specific Seat assignment.
- Seat is customer/product/user scoped.
- a Post Seat does not imply a DM Seat.
- entitlement lifecycle and seat lifecycle are related but not identical.
- Customer/member disable/remove operational paths are not fully surfaced through supported admin UI.
- payment checkout/webhook/plan automation is not yet proven complete.

Treat these as audit claims to validate from current source, not as guaranteed runtime truth.

### Product differences

Current understanding to validate:

- Post has Central Seat-related behavior but product tenant/data scope may still be per-user rather than true shared Customer scope.
- DM currently uses entitlement as its formal gate while Central Seat may still be shadow/observation.
- ORC currently appears user-owned with entitlement gate and no proven Customer-level data scope / Central Seat cutover.
- Sign and 591 may have different Staging and Production delivery forms.
- therefore a single "same Docker digest from Staging to Production" rule cannot be assumed for every product.

### Release-process discrepancies to verify

Recent audit identified possible divergence such as:

- old docs saying deploy does not enforce preflight while actual deploy scripts now may enforce it
- old docs describing all failures as automatic rollback while deploy tooling may have a third state such as `SUCCESS_DECISION_REQUIRED`
- product-specific Production forms may differ from Staging
- candidate commit / CI evidence must not be described as deployed runtime proof
- documentation must identify repo + branch/worktree + commit + environment + verification date

Validate all of these.

## 2.1 FIXED STAGING TEST IDENTITY CONTRACT — MUST VERIFY LIVE STATE

There are two intended fixed ordinary-user Staging test identities for Central Seat positive/negative testing:

- `job0975805890@gmail.com`
- `tcp.ai313@gmail.com`

The intended test model is:

```
One Staging Business Customer
│
├─ both users are active members of the SAME Customer
│
├─ target product has an active entitlement
│
├─ seat_limit is sufficient for the intended positive path
│
├─ job0975805890@gmail.com
│   └─ ASSIGNED Seat
│      └─ expected product access = ALLOW when product Seat enforcement is active
│
└─ tcp.ai313@gmail.com
    └─ UNASSIGNED Seat
       └─ expected product access = DENY when product Seat enforcement is active
```

This is the **intended contract**, not proof that Staging currently matches it.

A previous Staging preflight reportedly found both users without usable `customer_members` membership and `product_seat_assignments` empty. That historical finding must not be treated as today's state.

You MUST independently determine the current state from available evidence.

For each identity, verify or determine:

```
AUTH_USER_EXISTS =
PROFILE_EXISTS =
PLATFORM_ROLE =
TEST_ACCESS =
CUSTOMER_ID =
CUSTOMER_TYPE =
MEMBERSHIP_EXISTS =
MEMBERSHIP_STATUS =
TARGET_PRODUCT =
ENTITLEMENT_EXISTS =
ENTITLEMENT_STATUS =
SEAT_LIMIT =
SEAT_ASSIGNMENT =
SEAT_STATUS_RPC_RESULT =
PRODUCT_ACCESS_EXPECTATION =
EVIDENCE_LEVEL =
LAST_VERIFIED =
```

Rules:

- `tcp.a2026i@gmail.com` or any platform-admin identity must NOT be used as the positive/negative ordinary-user Seat fixture because admin bypass can invalidate the test.
- Do not substitute another account merely because it is easier.
- Do not create membership, entitlement, Seat assignment, or customer records in this task.
- Do not change `platform_role`, `test_access`, or any auth metadata.
- If current read-only Staging access is available, perform the safest non-mutating verification.
- If live read-only verification is unavailable, exhaust committed source, evidence files, current docs, git history, CI artifacts, and existing handoffs before writing `NOT_VERIFIED`.
- Clearly distinguish historical evidence from current live Staging evidence.

The canonical docs must state whether the fixed Staging Seat fixture is:

- `READY`
- `PARTIAL`
- `MISSING`
- `NOT_VERIFIED`

and must state the exact missing pieces.

If the fixture is not READY, create a smallest follow-up task named similar to:

`STAGING-SEAT-FIXTURE-A1 — Establish fixed assigned/unassigned Central Seat test identities`

That follow-up task may be a Staging mutation task, but DO NOT execute it in this convergence run.

## 2.2 AUTONOMOUS GAP-RESOLUTION RULE

Do not ask the user to reconstruct architecture or repeat information that can be discovered from the working environment.

When information is missing or contradictory, investigate in this order:

1. current canonical docs
2. committed source
3. deployment manifests/scripts
4. database migration/RPC source
5. git history / branches / worktrees
6. CI workflows, runs, artifacts, and release evidence
7. existing handoff/current-state documents
8. read-only Staging runtime evidence when access exists
9. read-only Production runtime evidence only when necessary and safely available

For a **documentation gap or stale statement**:
- find the evidence
- repair the canonical documentation
- validate cross-links
- commit only documentation owned by this task

For a **missing implementation/runtime/data state**:
- verify that it is actually missing
- document the exact blocker
- identify owner/repo/environment
- produce the smallest next task
- do NOT silently implement or mutate runtime in this run

For an **unknown value after exhaustive search**:
- write `UNKNOWN / NOT_VERIFIED`
- record exactly what was searched
- record what evidence is missing
- do not guess

The agent should continue through discoverable gaps without pausing for user confirmation unless proceeding would require a forbidden mutation, destructive action, unknown secret, or materially ambiguous ownership that cannot be resolved from evidence.

---

# 3. PHASE A — REPOSITORY & EVIDENCE INVENTORY

Perform a readonly inventory of the relevant repositories.

For each repository record:

- canonical repository path
- git remote
- current branch
- HEAD
- default branch if determinable
- dirty/clean
- untracked files relevant to this task
- relevant worktrees
- current deployment/runtime documents
- CI workflow ownership
- deployment ownership
- database/migration ownership
- whether docs appear stale
- latest evidence date found

Do not modify anything in this phase.

Create a working matrix:

| Component | Canonical Repo | Responsibility | Source Status | Staging Delivery | Production Delivery | DB Owner | Seat State | Data Scope | Evidence Level | Last Verified |
|---|---|---|---|---|---|---|---|---|---|---|

Minimum components:

- Commercial Core / Platform
- Launcher / Hub / Workbench
- ExtractionHub
- Post
- DM
- ORC
- Sign
- 591
- Letter
- central deploy/release tooling

Unknown values must be written as `UNKNOWN`, not inferred.

---

# 4. PHASE B — DEFINE OWNERSHIP BOUNDARIES

Produce an explicit ownership contract.

The intended model should be verified and normalized around these layers:

## Layer 1 — PLATFORM / COMMERCIAL CORE

Expected responsibilities to validate:

- Customer
- Customer membership
- Products
- Plans/subscriptions when implemented
- Entitlements
- Central Seat
- Commercial audit trail
- shared entitlement resolver
- server-side admin authorization contract

Platform must be the owner of commercial database contracts if current source supports this.

Launcher must not become a second schema owner.

Product repositories must not invent their own incompatible Customer/Seat semantics.

## Layer 2 — LAUNCHER / WORKBENCH

Expected responsibility:

Launcher is the **operator UI / orchestration interface**, not the source of truth for commercial data.

Document which of these are currently supported:

- create customer
- inspect customer
- update/disable customer
- add member
- suspend/remove member
- set entitlement
- set seat limit
- list assigned seats
- select assignment candidate safely
- assign seat
- release seat
- inspect commercial audit/history

For every item mark:

- `SUPPORTED`
- `PARTIAL`
- `MISSING`

Do not implement missing UI in this task.

## Layer 3 — PRODUCT CONTRACT

Each product must have a standard integration contract describing:

- authentication mechanism
- entitlement gate
- Central Seat gate state:
  - `NOT_USED`
  - `SHADOW`
  - `ENFORCED`
- product provisioning entrypoint
- data scope:
  - `USER`
  - `CUSTOMER`
  - `HYBRID`
  - `UNKNOWN`
- tenant mapping
- Staging delivery type
- Production delivery type
- health endpoint
- critical smoke owner
- rollback method
- release evidence path
- secrets/config owner

Do not force products into identical runtime architecture.
Standardize the **contract**, not necessarily the implementation.

## Layer 4 — RELEASE GOVERNANCE

Define the universal gates while preserving product-specific delivery mechanics.

---

# 5. PHASE C — CANONICAL DOCUMENT SET

Locate the current cross-product deployment documentation owner.

Prefer the existing central deploy/release repository if it is clearly the established cross-product owner.

Do not create duplicate canonical documents in multiple repositories.

The final documentation model must have exactly one primary system entrypoint.

## Required canonical documents

### C1. `TICENPI_SYSTEM_CURRENT_STATE.md`

Create/update a single current-state entry document.

It must contain:

1. purpose and "how to use this document"
2. evidence taxonomy
3. repository ownership map
4. current system component matrix
5. Commercial Core / Central Seat state
6. Launcher / Workbench state
7. ExtractionHub state
8. per-product integration state
9. per-product Staging/Production delivery map
10. current known blockers
11. evidence index
12. explicit "last verified" dates
13. links/paths to the canonical subordinate contracts
14. rules for what must be updated after future releases

This must become the first document future agents read.

### C2. `PRODUCT_TO_STAGING.md`

Audit and repair the existing document instead of blindly replacing it.

It must separate:

#### universal gates

- source identity
- dirty-worktree policy
- CI
- tests
- build/artifact identity
- config identity
- security/auth checks
- health
- critical smoke
- entitlement/seat requirements when applicable
- release evidence
- acceptance decision

from:

#### product-specific delivery profile

A product may be:
- Docker → Docker
- source/package → systemd
- Docker in Staging but systemd in Production
- other documented form

Do not state Docker-specific requirements as universal rules.

Document the config/artifact separation rule.

A config-only correction must not force an unnecessary artifact rebuild unless runtime packaging actually requires it.

Document decision states accurately. If tooling supports a state equivalent to:

- `PASS`
- `ROLLBACK`
- `SUCCESS_DECISION_REQUIRED`

then document all three rather than reducing them to binary success/failure.

### C3. `STAGING_TO_PRODUCTION.md`

This is currently a major missing contract.

Create it.

It must define universal Production promotion gates:

1. accepted Staging identity
2. source/artifact/config freeze
3. Production preflight
4. database change classification
5. backup / restore evidence requirements
6. secrets/config validation
7. OAuth/domain validation
8. entitlement/Seat applicability
9. product-specific delivery mapping
10. Production rollout
11. health
12. critical smoke
13. human E2E when required
14. evidence capture
15. post-deploy audit
16. rollback / roll-forward decision states
17. exact acceptance criteria

Important:

"promote same digest" is valid only for products where Staging and Production actually consume the same immutable artifact type.

For products whose Production runtime is not the same artifact form, define an equivalence/provenance requirement instead:

- same source commit
- same dependency lock / build provenance
- same accepted application version
- controlled config delta
- reproducible build/package proof
- product-specific verification

Do not invent equivalence evidence that current tooling cannot provide. Mark missing proof as a blocker.

### C4. `SYSTEM_OWNERSHIP.md`

Create a compact ownership table:

| Concern | Canonical Owner | UI Owner | Runtime Consumers | Forbidden Duplicate Owner |
|---|---|---|---|

At minimum cover:

- Customer
- Membership
- Entitlement
- Central Seat
- product data tenancy
- product provisioning
- shared extraction
- CI
- deployment
- release manifest
- evidence
- secrets/config
- rollback

### C5. `PRODUCT_INTEGRATION_CONTRACT.md`

Define the mandatory template every product must satisfy.

Required fields:

```
PRODUCT
REPO
CANONICAL_BRANCH
CANONICAL_COMMIT
AUTH
ENTITLEMENT_GATE
SEAT_MODE
DATA_SCOPE
PROVISIONING
STAGING_DELIVERY
PRODUCTION_DELIVERY
DATABASE_OWNER
HEALTH
CRITICAL_SMOKE
CONFIG_OWNER
SECRETS_OWNER
EVIDENCE_PATH
ROLLBACK
LAST_VERIFIED
KNOWN_GAPS
```

Then include one current-state entry for each known product or link to its authoritative product-specific file.

### C6. `STAGING_TEST_IDENTITY_CONTRACT.md`

Create/update a canonical Staging test-identity contract.

It must document:

- the two fixed ordinary-user identities:
  - `job0975805890@gmail.com`
  - `tcp.ai313@gmail.com`
- why platform-admin accounts cannot serve as normal Seat fixtures
- intended same-Customer assigned/unassigned topology
- product-by-product use of the fixture
- current live/historical verification status
- membership status
- entitlement status
- Seat assignment status
- Seat limit
- expected ALLOW/DENY behavior only where Seat enforcement is active
- how shadow-mode products should evaluate the same identities without turning observation into access control
- how to reset/repair the fixture in a separate approved Staging mutation task
- evidence date and evidence level

The contract must not claim the fixture is READY unless current evidence proves all required relationships.

---

# 6. PHASE D — CENTRAL SEAT CONTRACT CONSOLIDATION

Locate all Central Seat / Commercial Core definitions across repositories.

Classify each as:

- canonical source
- generated/copy
- historical migration
- obsolete duplicate
- local WIP
- deployed evidence

Do not delete duplicates automatically.

Produce a source-of-truth map that answers:

```
Commercial DB schema owner = ?
Entitlement resolver owner = ?
Seat RPC owner = ?
Seat lifecycle owner = ?
Launcher caller = ?
Product caller = ?
Migration source = ?
Staging deployed version = ?
Production deployed version = ?
```

If `supabase/` or `docs/commercial-core/` are still untracked in the canonical repository, make that an explicit blocker.

Do not claim the contract is reproducible until canonical source is committed.

Document the intended onboarding sequence based on what source actually supports:

```
Auth user exists
↓
Customer exists
↓
Membership exists
↓
Entitlement exists
↓
Seat assigned when required
↓
Product provisioning
↓
Product access
```

Mark which steps are automatic, UI-supported, RPC-only, manual, or missing.

Do not implement missing flows.

---

# 7. PHASE E — LAUNCHER / WORKBENCH CONTRACT CONSOLIDATION

Document Launcher as an operator surface.

Do not allow Launcher docs to redefine Platform commercial semantics.

Produce a matrix:

| Operation | Backend Contract | Launcher UI | Current State | Required for Launch |
|---|---|---|---|---|

Include:

- create customer
- inspect customer
- update/disable customer
- add member
- suspend/remove member
- entitlement set/update
- seat limit
- list seat
- assignment candidate lookup
- assign
- release
- audit/history

Explicitly identify the safe `user_id` selection gap if it still exists.

Do not workaround missing stable IDs using email ordering, array position, client-side guesses, or unsafe joins.

---

# 8. PHASE F — EXTRACTIONHUB CONSOLIDATION

Determine current actual ExtractionHub form.

Target architecture, if supported by current implementation:

```
ExtractionHub
= versioned shared extraction package / wheel / library
!= another long-running VPS product service
```

Document:

- canonical repository
- package version
- source commit
- clean/dirty source status
- build workflow
- artifact provenance
- product pinning method
- compatibility contract
- smoke/contract tests
- how a product adopts a new version
- how rollback works
- whether any Staging/Production product currently consumes an accepted release

A wheel/artifact built from dirty source cannot be described as a canonical release.

If CI/publish docs/ADR are untracked, mark them as source-governance blockers.

Do not publish a new package in this task.

---

# 9. PHASE G — PRODUCT-SPECIFIC NORMALIZATION

For each product (Post, DM, ORC, Sign, 591, Letter and any other active product discovered), complete the integration contract.

Pay special attention to:

### Post

Verify:
- entitlement behavior
- Central Seat behavior
- auto-bootstrap dependency
- tenant mapping
- whether Customer members share or do not share data
- Staging vs Production delivery
- current accepted artifact/evidence

Do not describe per-user tenant mapping as Customer-shared tenancy.

### DM

Verify:
- formal entitlement gate
- Seat shadow vs enforcement
- design ownership/data scope
- Staging/Production release type
- current release evidence

Do not claim Seat enforcement if it only observes.

### ORC

Verify:
- profile creation behavior
- entitlement gate
- user-owned RLS/data model
- Central Seat gate presence/absence
- Customer-level provisioning presence/absence

Do not treat profile creation as commercial authorization.

### Sign / 591

Verify Staging and Production runtime form separately.

Do not force Docker-to-Docker promotion language if Production uses systemd or another delivery form.

### Letter

Determine actual current commercial/release integration state.
Do not extrapolate from another product.

---

# 10. PHASE H — DOCUMENT DRIFT REPAIR

Search for directly conflicting active documentation.

Examples:

- preflight behavior
- rollback semantics
- environment naming
- canonical Supabase ref
- artifact promotion rules
- old one-off Sign/DM/Post instructions presented as global rules
- old paths
- old ports
- old runtime model

For each conflict:

1. identify current source of truth
2. identify stale statement
3. fix only the documentation necessary to remove active contradiction
4. preserve useful historical docs by labeling them historical rather than rewriting history

Do not mass-edit unrelated documents.

Historical handoffs should remain historical.

---

# 11. PHASE I — VALIDATION

Before committing documentation changes:

## 11.1 Cross-link validation

Every canonical document path must resolve.

No canonical document may point to:
- missing path
- obsolete filename
- local-only temporary path presented as repository truth

## 11.2 Statement validation

For each major system claim, verify it against at least one appropriate source category.

Prefer:
- implementation/source for behavior
- deployment script/manifests for deployment mechanics
- DB migration/RPC source for database contracts
- runtime evidence for deployed state
- E2E evidence for user-visible behavior

## 11.3 Drift validation

Run repository searches for superseded global claims.

Examples:
- "all products use the same digest"
- "deployment never enforces preflight"
- "failure always auto-rolls back"
- obsolete Supabase project refs
- outdated product deployment modes

Report remaining matches and classify whether:
- active bug
- historical text
- test fixture
- harmless example

## 11.4 Git validation

Before any commit:

- show changed files
- confirm all changes are documentation-only
- confirm no source/migration/runtime files changed
- confirm no unrelated user WIP was included

If a repo has pre-existing dirty state, commit only task-owned documentation paths when safe.

Never stage everything with `git add .`.

Use explicit file paths.

---

# 12. COMMIT POLICY

Documentation may span more than one canonical repository.

Prefer the minimum number of repositories.

Allowed commits:
- documentation only
- narrowly scoped
- explicit paths
- no unrelated WIP

Suggested commit naming:

```
docs: establish Ticenpi canonical system contract
docs: normalize product release contracts
docs: consolidate commercial core ownership
```

Do not push unless the environment's established task workflow explicitly authorizes normal branch push for documentation.

If push is not clearly authorized, commit locally and report exact commits.

Do not merge PRs.

---

# 13. REQUIRED FINAL REPORT

Return one concise final report with these exact sections.

## A. RESULT

One of:

```
RESULT = PASS
RESULT = PARTIAL_PASS
RESULT = HARD_STOP
```

PASS means:
- canonical system entrypoint exists
- PRODUCT_TO_STAGING is corrected
- STAGING_TO_PRODUCTION exists
- ownership contract exists
- product integration contract exists
- Central Seat / Launcher / ExtractionHub ownership is documented
- active contradictory documentation found in scope is resolved or explicitly labeled
- changed docs are committed cleanly where allowed

PARTIAL_PASS means:
- useful canonical docs were produced
- one or more evidence/ownership gaps remain and are explicitly identified

HARD_STOP only for a condition where continuing would risk damaging user work or making unsupported claims.

## B. CANONICAL ENTRYPOINT

Report:

```
CANONICAL_SYSTEM_DOC =
REPO =
BRANCH =
COMMIT =
```

## C. DOCUMENTS

Report each canonical document and status:

```
TICENPI_SYSTEM_CURRENT_STATE = PASS/PARTIAL/MISSING
PRODUCT_TO_STAGING = PASS/PARTIAL/MISSING
STAGING_TO_PRODUCTION = PASS/PARTIAL/MISSING
SYSTEM_OWNERSHIP = PASS/PARTIAL/MISSING
PRODUCT_INTEGRATION_CONTRACT = PASS/PARTIAL/MISSING
CENTRAL_SEAT_CONTRACT = PASS/PARTIAL/MISSING
EXTRACTIONHUB_CONTRACT = PASS/PARTIAL/MISSING
STAGING_TEST_IDENTITY_CONTRACT = PASS/PARTIAL/MISSING
```

## D. SYSTEM OWNERSHIP SUMMARY

Return the final owners for:

```
COMMERCIAL_CORE_OWNER =
CENTRAL_SEAT_OWNER =
LAUNCHER_OWNER =
EXTRACTIONHUB_OWNER =
RELEASE_GOVERNANCE_OWNER =
PRODUCT_DATA_SCOPE_OWNER =
```

## E. PRODUCT MATRIX

For each product report:

```
PRODUCT =
ENTITLEMENT =
SEAT =
DATA_SCOPE =
STAGING_DELIVERY =
PRODUCTION_DELIVERY =
LAST_VERIFIED =
BLOCKER =
```

## F. COMMERCIAL ONBOARDING STATE

Report:

```
AUTH_USER =
CUSTOMER =
MEMBERSHIP =
ENTITLEMENT =
SEAT_ASSIGNMENT =
PRODUCT_PROVISIONING =
OFFBOARDING =
PAYMENT_AUTOMATION =
```

Each value must be one of:

```
SUPPORTED
PARTIAL
MISSING
NOT_VERIFIED
```

## G. EXTRACTIONHUB

Report:

```
CANONICAL_REPO =
RELEASE_MODEL =
CLEAN_RELEASE_PROVEN =
PRODUCT_CONSUMERS_PROVEN =
CURRENT_BLOCKERS =
```

## G2. STAGING TEST IDENTITIES

Report exactly:

```
STAGING_SEAT_FIXTURE = READY/PARTIAL/MISSING/NOT_VERIFIED

ASSIGNED_TEST_USER = job0975805890@gmail.com
ASSIGNED_AUTH_USER_EXISTS =
ASSIGNED_MEMBERSHIP =
ASSIGNED_CUSTOMER =
ASSIGNED_ENTITLEMENT =
ASSIGNED_SEAT =
ASSIGNED_SEAT_STATUS =
ASSIGNED_EXPECTED_ACCESS =

UNASSIGNED_TEST_USER = tcp.ai313@gmail.com
UNASSIGNED_AUTH_USER_EXISTS =
UNASSIGNED_MEMBERSHIP =
UNASSIGNED_CUSTOMER =
UNASSIGNED_ENTITLEMENT =
UNASSIGNED_SEAT =
UNASSIGNED_SEAT_STATUS =
UNASSIGNED_EXPECTED_ACCESS =

SAME_CUSTOMER_PROVEN =
TARGET_PRODUCT =
SEAT_LIMIT =
LIVE_STAGING_VERIFIED =
LAST_VERIFIED =
MISSING_PIECES =
NEXT_FIXTURE_TASK =
```

Do not return expected ALLOW/DENY as proven runtime behavior unless the product is actually in Seat enforcement mode and the corresponding evidence exists.

## H. EVIDENCE QUALITY

Report:

```
COMMITTED_SOURCE_VERIFIED =
CI_VERIFIED =
STAGING_RUNTIME_VERIFIED =
PRODUCTION_RUNTIME_VERIFIED =
HUMAN_E2E_VERIFIED =
```

Do not write YES unless verified during this task or backed by clearly dated evidence that is explicitly labeled as historical evidence.

## I. CHANGES

Report:

```
CHANGED_FILES =
COMMITS =
SOURCE_CODE_MODIFIED = NO
DATABASE_MUTATED = NO
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO
```

## J. REMAINING BLOCKERS

Only concrete unresolved blockers.

No vague recommendations.

## K. NEXT TASKS

Create the smallest independent implementation tasks needed after documentation convergence.

Prioritize in this order:

1. canonical source / ownership blockers
2. Launcher commercial-management blockers
3. Central Seat cutover blockers
4. product Customer data-scope blockers
5. ExtractionHub release blockers
6. Staging→Production tooling gaps

Each next task must have:
- task name
- owner/repo
- exact scope
- dependencies
- whether it can run in parallel
- mutation level: docs/source/staging/production

Do not execute those implementation tasks in this run.

---

# 14. SUCCESS CONDITION

The work is successful when the following statement is true:

> A new engineer or AI agent can begin at one canonical Ticenpi system document, determine the authoritative owner and current evidence level for Commercial Core, Central Seat, Launcher, ExtractionHub, every product, Staging, Production, release promotion, rollback, and the fixed assigned/unassigned Staging Seat test identities, and can identify the next safe task without asking the user to reconstruct the architecture from chat history.

Do not stop after analysis.

If the documentation is incorrect or incomplete, repair the documentation within the allowed documentation-only scope, validate it, and commit the result safely.
