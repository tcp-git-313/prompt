# EXTRACTIONHUB CENTRAL SERVICE — STAGING FIRST
## Central Multi-Source Extraction API + DM Shadow Migration Owner

## 任務目標

把 ExtractionHub 從目前「嵌入 DM Docker 的 pinned wheel / embedded Core」正式提升為獨立的中央共用擷取服務，先完成 Staging，不碰 Production。

最終目標架構：

```
DM ─┐
Post ─┤
591 ──┤
ORC ──┤
未來產品 ─┘
       ↓
ExtractionHub Client
       ↓
ExtractionHub Central API
       ↓
Source Detection
Transport
Adapter Registry
CanonicalListing
Diagnostics
Semantic / Presentation
       ↓
YUCT / CT House / HB Housing / 未來來源
```

本任務不是重寫 ExtractionHub Core。

要保留並沿用目前已完成的：

- CanonicalListing
- Source Detection
- Transport
- Adapter architecture
- YUCT adapter
- CTHouse adapter foundation
- HBHousing adapter foundation
- Diagnostics
- Semantic Comparator
- Presentation Engine
- Clean CI / package identity
- DM Legacy primary + Shadow safety
- Central Seat / platform_admin contract

這次要補的是：

> ExtractionHub 作為獨立 Staging Service 的部署層、API auth boundary、tenant/product caller boundary、multi-source runtime registry、DM client integration、Staging E2E。

---

# 最終 Staging 架構

目標：

```
DM Staging
   │
   │ internal authenticated HTTP
   ▼
ExtractionHub Staging
   │
   ├─ YUCT
   ├─ CTHouse
   ├─ HBHousing
   └─ future adapters
```

優先考慮：

```
Docker internal network
DM backend
→ http://extractionhub-staging:<port>
```

而不是公開依賴瀏覽器直接呼叫 ExtractionHub。

若現有中央部署架構要求公開 hostname，可增加：

```
https://extractionhub-staging.ticenpi.com
```

但產品 backend-to-backend 優先走 internal service/network。

---

# 關鍵原則

1. ExtractionHub 是唯一 Core source of truth。
2. DM 不再長期維護自己的擷取 Core 副本。
3. Embedded wheel 先保留作為 fallback / shadow comparator，不要立刻移除。
4. Central Extraction Service 先作 Shadow，再切 Primary。
5. Legacy scraper 仍為 DM user-visible primary，直到中央服務 live E2E 全通過。
6. CTHouse / HBHousing adapter 不可因為「registry 有名字」就視為正式可用。
7. Production 完全不動。
8. 不把本機 127.0.0.1:8020 直接暴露成正式服務。
9. 不用 dirty worktree build release。
10. 所有 Staging artifact 必須 immutable、可追 source commit / image digest / config。

---

# 已知可信基準

## DM Staging accepted baseline

目前已驗收 Staging：

Release:
20260928-002319

App source:
93ebd87399b9fb6ed3c29ca5a4b8a44d11584ab5

Deploy config:
4802f18880fbcb128618b2145b965879560a3c82

Backend digest:
sha256:fa86fb6ede6b2248815b804d054229b5815c85b93167680560907b97f28fc295

Frontend digest:
sha256:76a9dcd50e9e01b3b85c5c47fe3ce01dd8469db404a9e819f5389a230e657055

Health:
PASS

dm-gates-full:
25/25 PASS

Central Seat:
require

Shadow:
enabled
100% controlled Staging sampling

Accepted evidence:
20260928-002319.ACCEPTED.json

Production:
UNCHANGED

## Shared Engine

Source:
11762508332bb2508538761302a90f8dca5f2586

Wheel SHA:
e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3

## Extraction Core

Accepted DM Staging Core source:
67bad69d8a4f1749220ab8f95160451ed0596beb

Core wheel SHA:
7d4bb57850e8c744ab25f3e3623c723135c8921f0467fa0d0e8547079d4738eb

目前這些 embedded artifacts 已驗證：

- YUCT live UI
- Shadow BOTH_SUCCESS
- Core raw
- Canonical
- NORMALIZED_MATCH
- Legacy primary
- rollback / restore

因此這些是中央 Service 遷移時的 reference baseline。

---

# 已知 Adapter 狀態

## YUCT

READY / reference adapter。

可做中央服務第一個 live adapter。

## CTHouse

Adapter foundation / offline tests / golden 可用。

Live 曾遇到環境-specific access deny / challenge。

因此：

- 不得宣稱 live READY，除非新的 Staging service環境實際成功
- 不得用 proxy / stealth / CAPTCHA bypass
- 若 Staging VPS仍被拒，標記 ENVIRONMENT_SPECIFIC_ACCESS

## HBHousing

Reference adapter 已有較完整 evidence。

歷史上：

- STANDARD_HTTP
- Nuxt JSON state
- Development 5/5
- fresh unseen F01-F10 10/10
- Golden PASS
- regression PASS

但仍需重新確認目前 clean source lineage是否包含該 adapter。

若不在 clean service source，需乾淨整合後才可 publish。

---

# 本任務允許

- 修改 ExtractionHub
- 修改 DM Staging client integration
- 新增 ExtractionHub Dockerfile / compose / manifest / deploy unit
- 新增 health / readiness
- 新增 internal service auth
- 新增 tenant/product caller context
- 建立 hosted CI ARM64 image
- 部署 extractionhub-staging
- 更新 DM Staging，使 Shadow 呼叫 central ExtractionHub
- 做 DM embedded Core vs Central Service comparator
- Staging rollback / restore
- 建立 staging-only service credentials / identity through existing secret management

---

# 本任務禁止

- Production deploy
- Production DB change
- 修改 Post / 591 / Sign / ORC source
- 刪 DM embedded Core
- 刪 Legacy scraper
- 切 Central Service 為 DM user-visible primary
- 用本機 8020當 VPS service
- 使用 dirty source build正式 image
- 直接 copy source 到 VPS
- hot patch running container
- 關閉 Central Seat / auth
- 將 service secret寫入 repo
- 將 user JWT 當 service-to-service authentication

---

# 執行策略

可平行執行 READ-ONLY：

TRACK A — ExtractionHub clean source / adapter inventory
TRACK B — Central service API / auth / tenant contract
TRACK C — deployment / Docker / CI / central manifest
TRACK D — DM client / Shadow migration contract
TRACK E — adapter live readiness matrix

然後：

```
READ-ONLY AUDIT
↓
ARCHITECTURE FREEZE
↓
SINGLE WRITER ExtractionHub Service
↓
CI / ARM64
↓
STAGING DEPLOY
↓
SINGLE WRITER DM Client Integration
↓
DM CI / ARM64
↓
DM STAGING DEPLOY
↓
LIVE DUAL-RUN E2E
↓
ROLLBACK / RESTORE
```

禁止 parallel writers改同一 repo。

---

# PHASE 0 — PREFLIGHT

## ExtractionHub

記錄：

- repo path
- branch
- HEAD
- status
- worktrees
- clean release ancestry
- adapter files
- service API
- current Dockerfile / compose
- CI workflows
- package metadata
- existing 8020 local runtime

## DM

記錄：

- integration branch
- current HEAD
- Staging accepted release
- current Staging commit
- current backend/frontend digests
- current Core lock
- Shadow provider
- accepted rollback target

## Central deploy

盤點：

- services registry
- compose registry
- hostname registry
- Cloudflare / internal routing
- health gate
- ARM64 image policy
- release evidence
- rollback tooling

不要修改。

---

# TRACK A — CLEAN SOURCE / ADAPTER INVENTORY

建立：

| Adapter | Source Exists | In Clean Branch | Tests | Golden | Live Status | Publish Status |

至少：

- yuct
- cthouse
- hbhousing

確認：

1. 哪些 adapter 真正在 clean source lineage。
2. 哪些只存在 dirty worktree。
3. 哪些 registry entry只有 metadata，沒有 runtime implementation。
4. Source detection是否能從 URL自動選 adapter。
5. Adapter registry是否能列出 runtime adapters。
6. canonical model是否一致。
7. diagnostics是否一致。

如果 CTHouse / HBHousing只在 dirty source：

建立 clean integration branch/worktree。

不得直接把 dirty checkout deploy。

---

# TRACK B — CENTRAL SERVICE API CONTRACT

確認現有：

`/v1/extract/sync`

以及 request / response contract。

目標 service API 至少：

## POST /v1/extract/sync

Input：

- url
- optional source hint
- caller context
- request id

不要讓產品傳任意 filesystem path。

Output：

- source_id
- canonical listing
- raw snapshot / safe source payload reference
- provenance
- diagnostics
- warnings
- adapter version
- core version

## GET /health

liveness。

## GET /ready

readiness：

- registry loaded
- required config loaded
- service auth ready

## GET /v1/sources

返回 runtime真正已註冊 adapters及狀態。

不能只讀 sites.yaml 就宣稱 supported。

---

# PHASE 1 — SERVICE AUTH / TENANT BOUNDARY

這一段是中央服務必要安全邊界。

產品 backend 不可把 user Supabase JWT直接當 ExtractionHub service credential。

建立：

```
Product Backend
↓
Service Identity
↓
ExtractionHub
↓
Caller Product
Tenant Context
Request Audit
```

優先使用現有 Ticenpi service-to-service contract。

如果沒有既有 contract：

建立最小安全 Staging contract，例如：

- signed service token / internal secret
- caller_product
- tenant_id / customer_id where required
- request id
- timestamp / replay control if existing infra支持

不要自行做複雜 crypto若平台已有方式。

必須防：

- browser直接調 API
- caller偽裝其他 product
- tenant A冒充 tenant B
- anonymous extraction abuse

建立：

## CALLER CONTRACT

Caller:
Product:
Tenant:
User context needed?:
Service credential:
Audit fields:
Rate policy:

---

# PHASE 2 — EXTRACTIONHUB STAGING DEPLOY UNIT

建立正式 service：

```
extractionhubstaging
```

或依中央 registry naming convention使用正式名稱。

需要：

- Dockerfile
- linux/arm64
- immutable GHCR image
- env file
- health
- readiness
- internal port
- external hostname only if architecture需要
- central service registry
- deploy isolation
- rollback snapshot

hostname若需要：

```
extractionhub-staging.ticenpi.com
```

Production hostname這輪只規劃，不建立或不啟用。

---

# PHASE 3 — DOCKER RUNTIME

Docker image必須：

- 從 clean source build
- 不 mount local repo
- 不依賴 F:\
- 不依賴 localhost:8020
- 不使用 editable install
- adapter registry隨 image固定
- source commit / version可查

Runtime identity endpoint / health metadata至少可安全查：

- version
- source commit
- build id
- adapter list
- environment

不要回 secrets。

---

# PHASE 4 — EXTRACTIONHUB CI

Hosted CI至少：

- unit tests
- adapter tests
- canonical tests
- diagnostics
- semantic/presentation
- API contract
- service auth
- tenant isolation
- Docker build
- ARM64 image
- container smoke
- health
- readiness
- runtime adapter list

產出：

- image digest
- source commit
- test run id
- build evidence

---

# PHASE 5 — YUCT CENTRAL SERVICE E2E

先用 YUCT當 reference。

在 extractionhub-staging：

```
POST /v1/extract/sync
YUCT URL
```

驗：

- source detection = yuct
- extraction success
- canonical
- raw
- provenance
- diagnostics
- images
- no credential leak

結果要與 embedded Core reference基本一致。

若不一致：

不要接 DM。

先分類：

- SERVICE_SERIALIZATION
- CONFIG
- REGISTRY
- CORE_VERSION
- TRANSPORT
- OTHER

---

# PHASE 6 — HBHOUSING SERVICE E2E

如果 clean adapter已整合：

用合法公開 HBHousing URL做 Staging live。

驗：

- source detection
- STANDARD_HTTP
- canonical
- provenance
- images
- diagnostics

若成功：

狀態可以提升到：

STAGING_VERIFIED

但 Production publish另做。

---

# PHASE 7 — CTHOUSE SERVICE E2E

如果 clean adapter已整合：

在 Staging VPS做 live。

若成功：

記錄正常結果。

若仍 403 / challenge：

分類：

CTHOUSE_ENVIRONMENT_SPECIFIC_ACCESS

不要：

- proxy
- stealth
- CAPTCHA bypass
- browser fingerprint evasion

Adapter仍可保留 DRAFT / BLOCKED。

這不應阻塞 YUCT / HBHousing中央服務。

---

# PHASE 8 — DM CENTRAL CLIENT

建立 DM 的 ExtractionHub client。

位置依 repo架構。

Client責任：

- service endpoint
- service auth
- caller_product=dm
- tenant context
- timeout
- retries只針對安全 idempotent情況
- typed errors
- correlation id
- no token logging

不能讓 frontend直接呼 ExtractionHub。

流程：

```
DM Backend
↓
ExtractionHub Client
↓
ExtractionHub Staging Service
```

---

# PHASE 9 — DM SHADOW DUAL-RUN

先不要取代 embedded Core。

建立中央服務 Shadow：

```
YUCT request
↓
Legacy primary
↓
embedded Core shadow
↓
Central Service shadow
```

或若避免三路過度複雜：

至少：

```
Legacy primary
↓
Central Service shadow
```

但保留 embedded Core作 fallback reference。

建議最安全：

```
Legacy = user visible
Embedded Core = reference
Central Service = new shadow
```

只在 controlled Staging 100% sample。

比較：

- embedded canonical
- service canonical
- semantic status
- diagnostics
- latency

---

# PHASE 10 — DM GENERIC SOURCE ROUTE

不要再把 DM前端永久寫死成 YUCT-only。

但本輪先做 Staging controlled generic import path。

由 ExtractionHub source detection決定來源。

DM backend：

```
URL
↓
ExtractionHub source detection
↓
adapter
↓
CanonicalListing
↓
DM mapper
```

前端不要自己硬編：

- ychouse
- cthouse
- hbhousing

只做安全 URL validation + backend source result。

若產品需求仍要限制來源：

限制應來自 registry / entitlement / feature flag，不應散落 hardcode。

---

# PHASE 11 — SOURCE REGISTRY CONTRACT

建立或收斂為單一 registry。

每個 source至少：

- source_id
- domains
- adapter status
- runtime_available
- staging_verified
- production_verified
- transport
- feature capabilities
- known limitations

不要把：

`sites.yaml有source`

等同：

`runtime adapter ready`

UI只能顯示 runtime真正可用來源。

---

# PHASE 12 — DM STAGING TEST MATRIX

完成 client後，build新 DM candidate。

必須保持：

- DM_REQUIRE_AUTH=1
- DM_SEAT_POLICY=require
- Legacy primary
- Central Seat
- platform_admin
- current accepted UI fixes
- Shadow Console

跑：

- backend
- frontend
- auth
- entitlement
- Seat
- platform_admin
- Shadow
- service client
- cookie
- tenant
- dm-gates-full
- build

---

# PHASE 13 — DM STAGING DEPLOY

正式 CI：

- ARM64 backend
- ARM64 frontend
- immutable digests

中央 deploy：

- only dmruntimestaging
- no unrelated service changes
- health
- release identity
- gates

---

# PHASE 14 — LIVE DM YUCT VIA CENTRAL SERVICE

從：

https://dm-staging.ticenpi.com/

真人 UI匯入 YUCT。

要求：

- Legacy user result正常
- Central Service run成功
- Shadow Console可看
- canonical一致
- NORMALIZED_MATCH / EXACT合理
- embedded Core vs service無重大 drift

---

# PHASE 15 — LIVE DM HBHOUSING

如果 HBHousing service adapter已 Staging verified：

從 DM Staging generic import貼 HBHousing URL。

驗：

- source detection
- central extraction
- canonical
- DM mapper
- UI result
- Shadow record

若 DM產品 UI還不適合正式顯示：

至少 admin controlled import / preview path可測。

---

# PHASE 16 — LIVE DM CTHOUSE

同上。

若 VPS access denied：

UI / API要給 typed error，例如：

SOURCE_TEMPORARILY_UNAVAILABLE

不要顯示 generic 500。

---

# PHASE 17 — SERVICE FAILURE SAFETY

模擬：

- ExtractionHub unavailable
- timeout
- 5xx
- adapter error

要求：

YUCT Legacy primary仍可回 user結果。

Central service failure不得破壞現有 DM使用。

若未來 non-YUCT只有 central adapter：

要有明確 source unavailable UX。

---

# PHASE 18 — OBSERVABILITY

ExtractionHub Staging需要：

- request id
- caller product
- source id
- adapter version
- latency
- outcome
- typed error
- safe diagnostics

不要 log：

- credentials
- bearer
- cookie
- signed URL secret query
- PII beyond operational need

DM與ExtractionHub correlation id要能串接。

---

# PHASE 19 — EXTRACTIONHUB STAGING ROLLBACK

驗：

```
new ExtractionHub release
↓
rollback previous service release
↓
health
↓
restore new release
↓
health
```

DM client在 service rollback期間不應崩潰。

---

# PHASE 20 — DM STAGING ROLLBACK

如果 DM client integration有新 release：

執行：

```
new DM candidate
↓
rollback accepted previous DM release
↓
health
↓
restore new candidate
↓
health
```

確認：

- Seat
- auth
- Legacy
- Shadow

---

# PHASE 21 — PRIMARY CUTOVER DECISION

本任務最後不要直接切 Primary。

只輸出 evidence：

## YUCT

Central Service：
PASS / FAIL

Embedded Core parity：
PASS / FAIL

Ready for primary：
YES / NO

## HBHousing

Staging verified：
YES / NO

## CTHouse

Staging verified：
YES / BLOCKED

只有下一個明確任務才可把：

```
DM embedded Core
→ fallback only

Central ExtractionHub
→ primary
```

---

# PHASE 22 — PRODUCTION PLAN ONLY

本輪 Production：

NO DEPLOY。

只建立 promotion plan：

```
ExtractionHub Staging ACCEPTED
↓
DM Staging central-client ACCEPTED
↓
Production approval
↓
ExtractionHub Production service
↓
DM Production client
↓
controlled rollout
```

---

# FINAL REPORT FORMAT

# EXTRACTIONHUB CENTRAL SERVICE — STAGING REPORT

## 1. ExtractionHub Runtime

Service:
URL/internal endpoint:
Release:
Commit:
Image digest:
Health:
Ready:

## 2. Service Auth

Caller auth:
Product identity:
Tenant context:
Anonymous:
DENY

Cross-tenant:
DENY

## 3. Runtime Adapters

| Source | Runtime | Staging Live | Status |

YUCT:
CTHouse:
HBHousing:

## 4. YUCT

Extraction:
Canonical:
Images:
Diagnostics:
Embedded parity:

## 5. HBHousing

Extraction:
Canonical:
Images:
Diagnostics:
Status:

## 6. CTHouse

Extraction:
HTTP:
Classification:
Status:

## 7. DM Client

Commit:
Service endpoint:
Service auth:
Timeout:
Typed errors:

## 8. DM Generic Source Route

YUCT:
HBHousing:
CTHouse:

## 9. DM Shadow

Legacy primary:
Embedded Core:
Central Service:
Comparator:
Console:

## 10. Central Seat / Auth

Seat=require:
Assigned:
Unassigned:
platform_admin:
Auth:

## 11. CI / Images

ExtractionHub CI:
ExtractionHub ARM64:
DM CI:
DM ARM64:

## 12. Staging Deploys

ExtractionHub release:
DM release:

Health:
Gates:

## 13. Rollback

ExtractionHub rollback:
Restore:

DM rollback:
Restore:

## 14. Production

ExtractionHub Production:
NOT DEPLOYED

DM Production:
UNCHANGED

Production DB:
UNCHANGED

## 15. Remaining Items

Only real blockers.

## 16. Final Result

只能：

EXTRACTIONHUB CENTRAL SERVICE STAGING READY

或

EXTRACTIONHUB CENTRAL SERVICE STAGING BLOCKED

---

# HARD STOP

不要：

- 切 Central Service primary
- 移除 embedded Core
- 移除 Legacy scraper
- 部署 Production
- 修改 Production DB
- 修改 Post / 591 / Sign / ORC
- 使用 dirty adapter source
- 使用 localhost:8020取代正式 service
- 暴露 unauthenticated extraction API
- 使用 user JWT取代 service identity
- 為 CTHouse繞過反爬限制

本任務做到：
ExtractionHub Staging service + DM Staging central client + multi-source Staging E2E + rollback/restore，
然後停止並交報告。
