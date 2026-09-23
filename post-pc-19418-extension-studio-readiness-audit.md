# POST P-C — 19418 EXTENSION + STUDIO READINESS AUDIT

## ROLE
POST BROWSER ACQUISITION READINESS AUDITOR

## MODEL
LUNA Medium

## MODE
READ-ONLY ONLY — ZERO-MUTATION PARALLEL TASK

Goal:
Resolve as much as possible about the remaining 19418 Launcher/Extension gates without modifying code, browser settings, data, or Git.

Known:
- 9458 detects Extension 0.4.8
- 19418 does not currently show injection/handshake
- manifest includes 19418
- Studio currently reports offline
- Post source must not be changed in this audit

## HARD RULES

DO NOT:
- modify extension source
- modify manifest
- modify Post source
- reload/install/uninstall extension automatically
- change Chrome site access
- modify Launcher
- fake studio_online
- commit
- push
- mutate DB

If user action is required, aggregate all user actions and report once at the end.

## 1. EXTENSION CONTRACT READ-ONLY

Inspect:
- `extension/manifest.json`
- `extension/content.js`
- `extension/background.js`
- `frontend/src/lib/extension-bridge.ts`
- relevant AccountStatusList / binding UI

Confirm:
- 9458 injection match
- 19418 injection match
- production origin match
- handshake mechanism
- extension ID/version exposure mechanism
- whether per-site Chrome access can independently block injection despite manifest match

## 2. 9458 VS 19418 DIFFERENCE

Compare only runtime-visible conditions that could explain:

9458 = detected
19418 = not detected

Check:
- origin/path match
- protocol
- content script run_at
- page CSP relevance
- page frame/top-frame assumptions
- content.js origin guards
- frontend handshake guards
- extension site-access dependency
- stale tab/service worker possibility

Do not modify anything.

Rank causes only by evidence:
PROVEN / PLAUSIBLE / NOT_SUPPORTED.

## 3. STUDIO ONLINE CONTRACT

Read-only inspect the exact Studio detection path:
- local URL/port
- health endpoint
- timeout
- expected process
- session-state mapping
- deep-link trigger

Record the exact user-visible condition required for:
`studio_online=true`

Do not start or alter Launcher automatically.

## 4. MANUAL USER ACTION PACKAGE

If browser/Studio action is required, produce ONE consolidated checklist only.

Example categories:
- start existing Ticenpi Launcher/Studio
- confirm expected local health endpoint responds
- chrome://extensions → version 0.4.8 → enabled → reload
- Chrome extension site access for localhost:19418
- hard reload 19418

Do not ask for actions that are not proven necessary.

## 5. FINAL REPORT

PHASE = 19418 ACQUISITION READINESS AUDIT
RESULT = PASS / USER_ACTION_REQUIRED / HARD_STOP

EXTENSION_MANIFEST_19418 =
CONTENT_SCRIPT_ORIGIN_GUARD =
FRONTEND_HANDSHAKE_GUARD =
SITE_ACCESS_CAN_BLOCK = YES/NO/UNKNOWN

9458_DETECTED =
19418_DETECTED =

MOST_LIKELY_BLOCKER =
BLOCKER_EVIDENCE =

STUDIO_DETECTION_URL =
STUDIO_HEALTH_REQUIREMENT =
STUDIO_ONLINE_CURRENT =

USER_ACTIONS =
1.
2.
3.

SOURCE_CHANGE_REQUIRED = YES/NO

If SOURCE_CHANGE_REQUIRED = YES:
STOP and explain exact evidence.
Do not edit.

No commit.
No push.
