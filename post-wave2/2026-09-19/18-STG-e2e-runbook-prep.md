# STG-PREP-D — Staging E2E Runbook Preparation

- 抬頭：POST STAGING E2E READINESS OWNER
- 模型：MiMo-V2.5
- 預計工程大小：中型
- 模式：READ ONLY / TEST PLANNING

---

你現在是：

POST STAGING E2E READINESS OWNER

這個任務與 W2INT-v2 平行。

不是現在執行真實 Staging。
不是 Deployment。

目標：

在 Staging artifact 尚未 deploy 前，把正式驗收要跑的 E2E matrix、前置資料、測試帳號條件、expected result 全部準備好。

## HARD RULES

READ ONLY。

禁止：

- 修改 source
- commit
- push
- Production
- Staging mutation
- real Facebook publish
- real delete/comment/relist
- Supabase write
- entitlement write
- OAuth config write

## AUTHORITATIVE ARCHITECTURE

必須依目前已凍結架構準備：

### Web identity

Google OAuth / Supabase JWT

### Commercial auth

JWT
→ Membership
→ Entitlement
→ Product Access

### Studio

Studio/Launcher heartbeat只是 Desktop capability state。

studio_offline / account_mismatch 不阻擋一般 Web access。

### Local

Google OAuth保留。
Commercial entitlement bypass。
Studio gate bypass。
Facebook first bind走 Chrome Extension direct。
Recurring sync走 device token。

### Production/Staging Web

Launcher不應是 Web prerequisite。

### Publish safety

Master：
TICENPI_AUTO_PUBLISH

Actions：
- POST
- COMMENT
- MARKETPLACE
- RELIST
- DELETE

全部 default OFF。
只有 master + specific action同時啟用才允許 real write。

## GOAL

建立正式 Staging E2E test matrix。

至少分成以下獨立 tracks，之後可平行執行：

### Track 1 — Health / Deployment

- image digest identity
- /api/health
- config_drift
- critical smoke
- container readiness
- worker/redis/postgres

### Track 2 — Google / Membership / Entitlement

- Google login success
- JWT identity
- correct tenant
- entitled user access
- not_entitled blocked
- no tenant blocked
- tenant isolation
- X-Tenant-Key production-like disabled behavior

### Track 3 — Studio informational behavior

- Studio online
- Studio offline
- account mismatch
- /api/session/state reports real state
- Web shell remains usable when Studio offline/mismatch

### Track 4 — Mobile Web

- Safari/mobile viewport
- Google OAuth return
- Web shell usable without Launcher
- entitled access
- navigation/core flows

### Track 5 — Extension / Facebook binding

- Local direct extension first bind
- EXTENSION_ID race/DOMContentLoaded behavior
- recurring device token sync
- production path remains Studio/content bridge if still required by current architecture

不得現在執行。

### Track 6 — Publish Safety

先做 safe/dry-run：

- master OFF
- master ON + actions OFF
- individual action isolation
- unknown action fail closed

對真實 Facebook write E2E：

只列出受控測試條件。

不得現在執行。

### Track 7 — Marketplace / Relist

列出：

- source prerequisites
- expected queue behavior
- dry-run behavior
- controlled real E2E gate

## TEST DATA REQUIREMENTS

整理需要哪些測試身份：

- entitled Google user
- non-entitled Google user
- second tenant user
- local user
- FB test account
- FB test group/page/listing targets

不要建立帳號。
不要輸出任何 secret/token。

## PASS / FAIL EVIDENCE

為每個 track定義：

- command/UI action
- expected HTTP/UI result
- evidence to capture
- hard stop condition

## PARALLELISM

指出 Staging deploy後哪些 tracks可同時執行。

目標是最大化平行：

Health
Auth
Mobile
Studio state
Extension
Publish dry-run

只有真正有 dependency的才串行。

## OUTPUT

```
TASK=STG_E2E_READINESS

MODE=READ_ONLY

TRACKS_DEFINED=

TEST_IDENTITIES_REQUIRED=

FB_TEST_ASSETS_REQUIRED=

HEALTH_TRACK=

AUTH_ENTITLEMENT_TRACK=

STUDIO_STATE_TRACK=

MOBILE_TRACK=

EXTENSION_BIND_TRACK=

PUBLISH_SAFETY_TRACK=

MARKETPLACE_RELIST_TRACK=

REAL_EXTERNAL_ACTIONS_PLANNED=
YES/NO

REAL_EXTERNAL_ACTIONS_EXECUTED=
NO

PARALLEL_TRACKS_AFTER_DEPLOY=

SERIAL_DEPENDENCIES=

MISSING_TEST_DATA=

MISSING_ENVIRONMENT_INFO=

HARD_STOP_CONDITIONS=

MUTATION_PERFORMED=NO

READY_TO_RUN_AFTER_STAGING_DEPLOY=
YES/NO
```
