# TICENPIPOST — FB GROUP FEATURE GAP + MOBILE WEB / LINE LIFF READINESS AUDIT

## ROLE

PRODUCT / ARCHITECTURE AUDITOR

## RECOMMENDED MODEL

LUNA High

## MODE

READ-ONLY INVESTIGATION — NO IMPLEMENTATION

This task is an evidence-driven audit of the currently deployed TicenpiPost product.

The product has been online for some time, but the original launch prioritized only basic functionality. There are older discussions and design documents describing additional Facebook Group capabilities that may be partially implemented, missing, obsolete, or never started.

Your job is to reconstruct the real current state from source code, Git history, project records, prior design documents, deployed behavior, and current official LINE documentation.

You are NOT authorized to implement anything in this task.

Do not modify source code, database data, Supabase settings, deployment manifests, Git history, browser extension settings, Facebook accounts, LINE settings, Staging, or Production.

Do not commit.
Do not push.
Do not deploy.

If any evidence conflicts, report the conflict instead of guessing.

---

# 0. PRIMARY QUESTIONS

Answer these questions with evidence:

1. Which Facebook Group functions were previously discussed?
2. Which of those are actually implemented today?
3. Which are partially implemented?
4. Which are missing?
5. Which older ideas are now obsolete because the current architecture changed?
6. Which existing code paths are present but not reachable from UI?
7. Which features have tests but are not actually wired into production?
8. Which features exist in production but are not documented?
9. What should be implemented next, in what order, and why?
10. Is TicenpiPost currently:
   - a normal responsive website,
   - a SPA,
   - a PWA,
   - an installable Web App,
   - or only a desktop-oriented browser UI?
11. What specifically makes the current mobile web experience difficult?
12. If a LINE Official Account posts a URL to TicenpiPost, what happens today?
13. Is LINE LIFF actually required, or would a normal HTTPS URL in LINE's in-app browser be sufficient?
14. If LIFF is useful, what exact integration problems exist with the current TicenpiPost auth/session/cookie architecture?
15. What is the lowest-risk path from the current product to:
   - mobile-friendly web,
   - LINE Official Account URL launch,
   - optional LIFF integration,
   without rewriting the whole product?

---

# 1. PROJECT / SOURCE LOCATIONS

Primary product repository:

`F:\00-Ticenpi-SaaS\TicenpiPost`

Central deployment / environment repository:

`F:\00-Ticenpi-SaaS\deploy`

Shared commercial/platform repositories may be relevant if auth or entitlement behavior needs to be understood.

Do not assume repo names.
Discover actual dependencies from current source / deployment manifests.

---

# 2. REQUIRED HISTORICAL SOURCES

First reconstruct previous discussions and records.

## 2.1 Current known FB Group design

Read the exact design document:

GitHub:
`https://github.com/tcp-git-313/prompt/blob/main/fb-group-feature-design.md`

Known commit:
`94c103670828a48395602d56fa780ee76e3bfb9f`

Treat this as one historical design source, NOT proof that features were implemented.

Its major ideas include:

- already joined group sync
- GroupSelector
- Group Management
- Facebook real group search
- Browser Worker
- AI category / ranking
- join-status tracking
- queued group joining
- membership questions
- rate limiting / account safety
- evidence / terminal states
- separate discovery vs posting selection UX

Compare every section of this document with real current code.

## 2.2 Claude / project memory

Read and search:

`C:\Users\Tu\.claude\projects\F--\memory\`

Start with:

`MEMORY.md`

Then search the memory folder for:

- 社團
- Facebook group
- fb group
- groups
- GroupSelector
- group management
- group search
- group join
- 社團搜尋
- 加入社團
- AI 分類
- Browser Worker
- LINE
- LIFF
- PWA
- Web App
- mobile
- 手機
- responsive
- app.ticenpi.com
- Post

Do not assume every memory file is still correct.

For each relevant record, identify:
- filename
- claim / decision
- approximate date if available
- whether current source still matches it

## 2.3 Project internal records

Search TicenpiPost for:

- `_charter/`
- ADRs
- `history.md`
- docs
- README files
- handoffs
- architecture notes
- migrations
- tests
- TODO/FIXME
- removed/deprecated code notes

Search Git history for commits containing:

- group
- groups
- facebook
- selector
- discovery
- join
- worker
- mobile
- pwa
- manifest
- service worker
- line
- liff

Do not rely only on current filenames.

## 2.4 Prompt repository records

Search:

`tcp-git-313/prompt`

for related prior prompts/designs.

Especially find any documents about:

- Facebook groups
- Post account binding
- Facebook acquisition
- Browser Worker
- AI recommendations
- PWA/mobile
- LINE/LIFF
- launcher/client execution

Record all relevant sources.

---

# 3. CURRENT IMPLEMENTATION AUDIT — FACEBOOK GROUPS

Perform a real code-level audit.

Do NOT infer implementation from route names alone.

For every capability below, determine:

- IMPLEMENTED
- PARTIAL
- DEAD/UNREACHABLE
- TEST-ONLY
- MISSING
- OBSOLETE
- UNKNOWN

Provide exact evidence:
- file
- function/class
- API route
- DB table
- migration
- frontend route/component
- worker
- queue
- test
- production UI if observable

## 3.1 Joined group sync

Audit:

- how joined groups are acquired
- which Facebook account is used
- whether data comes from Extension / Launcher / server Browser Worker
- group ID
- group name
- canonical URL
- join state
- sync timestamp
- caching
- error handling
- refresh flow
- multi-account behavior

Determine whether current implementation matches the old design.

## 3.2 GroupSelector

Audit:

- current component path
- where it is used
- what data it reads
- whether it only selects already joined groups
- search/filter capabilities
- grouping/tagging capabilities
- whether it is desktop-only / mobile usable

Confirm whether GroupSelector is still intentionally separate from Group Management.

## 3.3 Group Management

Determine whether a dedicated group-management UI currently exists.

Check for:

- already joined tab
- discovery tab
- sync
- labels
- custom groups
- recently synced
- joining state
- question handling

Do not count an internal component as implemented if users cannot reach it.

## 3.4 Facebook real group search

Search for real implementations of:

- `POST /api/groups/search`
- search run/job IDs
- Facebook group search driver
- Browser Worker group search
- selectors / semantic DOM extraction
- result persistence
- real group URL extraction
- privacy/member count parsing
- joined/not-joined detection
- retry/error classification

Verify whether this is:
- implemented,
- partially scaffolded,
- or only in the design doc.

Do NOT execute live Facebook searches on a real user account during this audit unless an existing safe read-only test environment explicitly exists.

## 3.5 AI classification / ranking

Audit whether AI is currently used for group discovery.

Check:

- actual provider/model
- input data
- categories
- relevance score
- reason
- hallucination prevention
- whether AI can create fake group names/URLs

If no implementation exists, say MISSING.

## 3.6 Join group automation

Search for actual implementation of:

- `POST /api/groups/join`
- group join jobs
- account serialization
- queue
- rate limits
- joining
- request pending
- questions required
- checkpoint
- terminal-state verification
- retries
- evidence storage

Do not treat enqueue code alone as successful join functionality.

Do not perform real group-join actions during this audit.

## 3.7 Membership questions

Audit whether the product can:

- detect Facebook join questions
- return them to Post UI
- collect answers
- retry join
- save reusable answer templates
- optionally use AI draft assistance

Classify exact current status.

## 3.8 Account-risk controls

Map current protections used by Facebook operations:

- account lock
- queue
- daily behavior budget
- cold start
- checkpoint detection
- circuit breaker
- delays
- concurrency
- rate limiting
- login-required state

Determine which protections are already shared by group functions and which would need new integration.

---

# 4. CURRENT DATA MODEL AUDIT

Identify actual current tables / models related to:

- joined groups
- group discovery
- group join tasks
- group state
- account-group relationship
- evidence
- retry
- questions/answers
- tags/categories

Compare with historical proposed models:

- GroupDiscoveryResult
- GroupJoinTask

For each proposed field, mark:

- already exists
- equivalent field exists
- not needed anymore
- still missing

Do not propose duplicate tables if an existing generalized task/evidence model can safely serve the requirement.

---

# 5. DEPLOYED PRODUCT AUDIT

Find the actual current Production URL from deployment/runtime configuration.
Do not guess.

Read-only inspect current deployed TicenpiPost UI where possible.

Do not perform destructive or state-changing Facebook actions.

Check:

- current navigation
- group-related pages
- joined-group UI
- mobile viewport
- touch usability
- layout overflow
- modal behavior
- tables/lists
- GroupSelector
- long text handling
- account switch
- login flow
- settings
- back navigation

Record observable differences between:
- desktop width
- phone width

Do not modify Production.

---

# 6. WHAT IS TICENPIPOST TODAY?

Precisely classify the frontend.

Inspect actual source for:

- framework
- SPA routing
- server-side rendering if any
- `manifest.webmanifest`
- `manifest.json`
- service worker
- Workbox
- installability
- offline behavior
- `display: standalone`
- icons
- mobile viewport
- responsive breakpoints
- safe-area CSS
- touch targets
- bottom navigation / mobile navigation
- home-screen install support
- update strategy

Report separately:

WEB_APP_IN_BROWSER =
PWA =
INSTALLABLE =
OFFLINE_CAPABLE =
MOBILE_RESPONSIVE =
MOBILE_UX_READY =
LINE_IN_APP_BROWSER_READY =

Do not use the phrase "not a Web App" without defining what is actually missing.

A browser application can still be a web app without being a PWA.

---

# 7. MOBILE UX GAP AUDIT

Identify why operation is currently difficult on mobile.

Do not give generic advice.

Use source + current UI evidence.

Check at least:

- navigation
- dense desktop layouts
- sidebars
- modals
- tables
- drag/drop
- hover-only interactions
- context menus
- tiny touch targets
- fixed dimensions
- horizontal scrolling
- keyboard opening
- viewport resize
- textarea/editor behavior
- image upload
- Facebook account controls
- GroupSelector
- group search results
- long-running job status
- authentication flows
- error dialogs

Produce a prioritized list:

MOBILE_BLOCKER
MOBILE_MAJOR
MOBILE_MINOR

with source/component evidence.

---

# 8. LINE OFFICIAL ACCOUNT URL / LIFF INVESTIGATION

The user referred to "Line life".
Interpret this as a probable reference to LINE LIFF, but VERIFY the terminology and current LINE platform behavior.

Use current official LINE Developers documentation as the primary source.

Do not rely on stale blogs as authoritative.

Record research date.

## 8.1 Normal URL in LINE Official Account

Determine:

If LINE Official Account sends:

`https://post.ticenpi.com/...`

what happens when a user taps it today?

Investigate current official behavior for:

- LINE in-app browser
- external browser
- login/session behavior
- redirects
- cookies/storage
- opening normal HTTPS URLs

Answer whether a normal URL alone is sufficient for the user's desired flow.

## 8.2 LIFF

Research current official LIFF requirements:

- LIFF app registration
- LIFF ID / LIFF URL
- endpoint URL
- HTTPS requirements
- allowed endpoint/domain behavior
- path handling
- LIFF SDK
- `liff.init()`
- `liff.isInClient()`
- `liff.login()`
- current browser / external browser behavior
- LINE Login relationship
- OA relationship
- share functionality
- deep links
- closeWindow behavior where applicable

Do not assume older LIFF behavior is still current.

## 8.3 Current TicenpiPost auth compatibility

Audit actual current auth:

- Supabase
- Google OAuth
- redirect URLs
- session persistence
- host-only cookies / storage
- popup vs redirect
- callback origin
- launcher/session behavior

Then analyze what happens inside:

LINE in-app browser
and
LIFF browser/context.

Specifically investigate risk around:

- Google OAuth inside LINE webview/in-app browser
- third-party or cross-site login transitions
- popup restrictions
- return callback
- session storage/local storage
- cookie size / isolation
- same-site behavior
- browser reopening losing state
- opening external Safari/Chrome
- LINE user identity vs existing Supabase identity
- account linking if LINE Login is introduced

Do not propose replacing Google auth unless evidence shows it is needed.

## 8.4 LINE Official Account integration options

Compare at least:

OPTION A:
LINE OA sends normal `https://post.ticenpi.com` URL

OPTION B:
LINE OA sends a LIFF URL wrapping/reusing current TicenpiPost frontend

OPTION C:
Build a dedicated mobile/LIFF entry surface that routes into the same backend

OPTION D:
Full PWA + normal URL, LIFF only for LINE-specific capabilities

For each report:

- changes required
- auth impact
- mobile UX impact
- backend impact
- deployment impact
- user friction
- risk
- whether current Facebook/Launcher flows work inside that environment

Do not rank with an arbitrary numeric score.

Instead state factual tradeoffs and conditions.

---

# 9. IMPORTANT LAUNCHER / EXTENSION CONSTRAINT

Current Post Facebook acquisition architecture may use:

- Ticenpi Launcher / Studio
- Chrome Extension fallback
- local desktop process
- browser-specific capabilities

Investigate whether these are usable when TicenpiPost is opened from LINE on a phone.

This is critical.

For mobile + LINE:

Determine:

- can Launcher exist on mobile?
- can Chrome Extension fallback exist in LINE's in-app browser?
- which Facebook functions therefore cannot work on mobile under current architecture?
- which functions are read-only and can still work?
- whether mobile/LINE should be:
  - management/view-only,
  - full control,
  - or delegated task control while desktop agent executes Facebook actions

Do not redesign yet.
Report actual constraints.

---

# 10. HISTORICAL GAP MATRIX

Build a matrix with one row per previously discussed feature.

Required columns:

FEATURE
SOURCE_RECORD
DISCUSSION_DATE_IF_KNOWN
ORIGINAL_INTENT
CURRENT_STATUS
CURRENT_EVIDENCE
LIVE_REACHABLE
TEST_COVERAGE
BLOCKER
STILL_RELEVANT
NOTES

Include at least all sections from `fb-group-feature-design.md`.

Also include additional group/mobile/LINE ideas found from prior records that are not in that document.

---

# 11. "WHAT CAN WE STILL DO?" ROADMAP

After the audit, produce a phased implementation roadmap.

Do NOT implement.

The roadmap should minimize rework and preserve current Production.

Structure:

## Phase 0 — correctness / documentation reconciliation

Only items needed to make current architecture understandable and safe.

## Phase 1 — mobile usability baseline

Make current core Post flows usable on phone without requiring PWA/LIFF yet.

## Phase 2 — FB Group management missing basics

Only gaps that build on existing joined-group functionality.

## Phase 3 — real group discovery

Facebook real search + persistence + UI.

## Phase 4 — AI classification

Only after real search data exists.

## Phase 5 — join workflow

Queue + terminal states + questions + risk controls.

## Phase 6 — LINE URL / LIFF integration

Choose based on findings:
- normal URL,
- LIFF,
- or hybrid.

## Phase 7 — optional PWA/installability

Only if it solves a real product need.

For each phase report:

- purpose
- exact components
- dependencies
- migrations
- APIs
- frontend changes
- tests
- staging gate
- production gate
- risk
- estimated complexity: S / M / L / XL

Do not provide calendar estimates unless there is real evidence for them.

---

# 12. DO NOT CONFUSE THESE CONCEPTS

Explicitly distinguish:

### Web App
A web application accessed in a browser.

### Responsive Web App
A web app whose UI adapts correctly to phone/tablet.

### PWA
A web app with web app manifest/service worker/installability and related capabilities.

### LINE in-app browser
LINE opening an ordinary web URL internally.

### LIFF
LINE Front-end Framework context/app registered in LINE Developers.

The final report must use these terms correctly.

---

# 13. SECURITY / PRIVACY REVIEW

For any proposed mobile / LINE flow, inspect whether it changes:

- authentication trust boundary
- Supabase session handling
- tenant isolation
- Customer/Seat authorization
- Facebook cookies/tokens
- device tokens
- Launcher bind tokens
- extension bridge
- account ownership

Do not copy secrets into the report.

Do not propose storing Facebook credentials in LINE local storage.

---

# 14. CURRENT CENTRAL SEAT CONTEXT

Central Seat is actively being completed in parallel.

Do not change it.

For this audit only determine whether future mobile/LIFF flows need to preserve:

User
→ Customer
→ Entitlement
→ Product Seat
→ Product access

Do not create a second LINE-specific entitlement system.

LINE identity, if later introduced, must not silently become a second commercial identity SSOT without an explicit account-linking design.

---

# 15. LIVE ACTION SAFETY

During this investigation:

ALLOWED:
- read source
- read Git history
- read docs
- read tests
- read database schema
- read deployment manifests
- GET/health/read-only UI inspection
- official web documentation research
- local non-mutating tests

NOT ALLOWED:
- join a Facebook group
- leave a Facebook group
- post to Facebook
- change account cookies
- modify Facebook account state
- create/delete customers
- change Seat
- modify Supabase settings
- modify Production/Staging
- commit/push code
- deploy
- modify LINE settings

If a real Facebook write would be required to prove something:
mark it UNPROVEN and describe the future Staging acceptance test.

---

# 16. REQUIRED FINAL REPORT

Return one structured audit report.

Start with:

AUDIT_RESULT = COMPLETE / PARTIAL / BLOCKED
EVIDENCE_AS_OF =
POST_REPO_COMMIT =
PRODUCTION_RUNTIME_IDENTITY =
STAGING_RUNTIME_IDENTITY =

Then:

## A. CURRENT PRODUCT SUMMARY

CURRENT_FRONTEND_TYPE =
PWA =
MOBILE_READY =
LINE_NORMAL_URL_READY =
LIFF_READY =

## B. FB GROUP FEATURE STATUS

For every historical feature:
IMPLEMENTED / PARTIAL / MISSING / DEAD / OBSOLETE / UNKNOWN

## C. HISTORICAL RECORDS FOUND

List:
- memory files
- ADRs
- docs
- Git commits
- prompt docs
- tests

with paths/URLs and relevance.

## D. CURRENT USER-REACHABLE GROUP FLOW

Show the actual current flow from UI to backend/worker.

## E. MISSING / PARTIAL FUNCTIONS

Evidence-backed gap list.

## F. MOBILE WEB AUDIT

BLOCKERS =
MAJOR =
MINOR =

## G. LINE OFFICIAL ACCOUNT NORMAL-URL FINDINGS

NORMAL_URL_BEHAVIOR =
AUTH_IMPACT =
MOBILE_IMPACT =
FACEBOOK_EXECUTION_LIMITS =

## H. LIFF FINDINGS

LIFF_REQUIRED_FOR_BASIC_URL_FLOW = YES/NO/CONDITIONAL
LIFF_BENEFITS =
LIFF_REQUIRED_CHANGES =
AUTH_RISKS =
CURRENT_BLOCKERS =

Cite current official LINE documentation URLs and research date.

## I. LAUNCHER / EXTENSION ON MOBILE

LAUNCHER_AVAILABLE_ON_PHONE =
EXTENSION_AVAILABLE_IN_LINE_BROWSER =
FULL_FB_CONTROL_FROM_PHONE =
READ_ONLY_OR_MANAGEMENT_FEATURES_AVAILABLE =

## J. RECOMMENDED PHASED ROADMAP

Phase 0
Phase 1
Phase 2
...

For every phase:
SCOPE
DEPENDENCIES
RISK
COMPLEXITY
STAGING_GATE

## K. DO NOW / DO LATER / DO NOT DO

Evidence-based buckets.
Do not use arbitrary rankings.

## L. OPEN QUESTIONS

Only questions that genuinely cannot be answered from source/docs/runtime.

## M. FINAL CONCLUSION

Summarize:
1. what already exists,
2. what was discussed but never built,
3. what should be finished before LINE integration,
4. whether normal LINE OA URL is sufficient,
5. whether LIFF is worth adding,
6. whether PWA is actually necessary.

---

# 17. NO IMPLEMENTATION

This task ends after the audit.

Do not edit code.

Do not create a feature branch.

Do not commit.

Do not push.

Do not deploy.

If the audit identifies implementation work, only describe the next tasks.

The user will decide which implementation prompt to run next.
