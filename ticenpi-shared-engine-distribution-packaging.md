# TICENPI SHARED ENGINE DISTRIBUTION + PACKAGING OWNER — ZERO-COLLISION PLAN

## 任務目標

目前已完成：

- ExtractionHub CanonicalListing v2
- Shared Semantic Normalizer
- Shared Semantic Comparator
- Shared Presentation Engine
- Formatter Registry / Product Profile Contract
- DM Shadow 第一個 consumer
- YUCT REFERENCE READY
- HBHousing / 住商 REFERENCE READY
- CTHouse adapter / Golden / regression，live gate 因環境 IP 限制暫停

目前真正 blocker：

1. DM 現在透過 `EXTRACTION_CORE_ROOT` 指向本機 ExtractionHub source checkout。
2. Shared Engine 尚未有正式 packaging / version pin / runtime delivery。
3. Post / 591 是 JavaScript consumer，不能各自重寫 Python semantic logic。
4. 尚未證明 DM Docker / Staging 能載入固定版本 Shared Engine。
5. 尚未決定跨語言 consumer 的正式 delivery contract。

本任務唯一目標：

> 找出並實作最小、可版本化、可部署、單一語意來源的 Shared Engine Distribution 方案，先完成 DM 正式 runtime packaging，再產出 Post / 591 的可執行 consumer integration contract。

本任務不要：

- 重寫 Semantic / Presentation Engine
- 重新設計 CanonicalListing
- 修改 Adapter
- 開始永慶 / 台灣房屋
- 直接改 Post / 591 product code
- 部署 Production
- 修改 Production DB
- 建立大型新微服務，除非證據證明必要

---

# 核心架構原則

必須維持：

```
ExtractionHub
├─ CanonicalListing
├─ semantics.py
└─ presentation.py
        ↓
Versioned Shared Engine Delivery
        ↓
┌──────────────┬──────────────┬──────────────┐
↓              ↓              ↓
DM Python      Post JS        591 JS
Consumer       Consumer       Consumer
```

不能變成：

```
DM 自己一套 semantic
Post 自己重寫一套 JS semantic
591 再重寫一套 JS semantic
```

正式原則：

> Single semantic source of truth.
> Product-specific presentation profiles may differ.
> Cross-language consumers must not fork business semantics.

---

# 零碰撞執行策略

允許平行，但只允許 READ-ONLY audit。

可以同時進行：

A. DM packaging / Docker / runtime audit
B. Post JavaScript consumer audit
C. 591 JavaScript consumer audit
D. ExtractionHub package/distribution audit

以上全部：

- 只讀
- 不修改檔案
- 不 commit
- 不 migration
- 不 deploy

四份 audit 完成後：

```
PARALLEL READ
↓
ARCHITECTURE DECISION
↓
SINGLE WRITER IMPLEMENTATION
```

禁止多 Writer 同時實作 shared package / runtime contract。

若有其他 Session 正在修改：

- ExtractionHub packaging
- DM Docker/runtime
- shared engine bridge
- shared artifact contract

且與本任務檔案重疊：

HARD STOP

輸出：

SHARED ENGINE DISTRIBUTION WRITE COLLISION

---

# 工作目錄

主要調查：

F:\00-Ticenpi-SaaS\ExtractionHub
F:\00-Ticenpi-SaaS\TicenpiDM
F:\00-Ticenpi-SaaS\TicenpiPost
F:\00-Ticenpi-SaaS\Ticenpi591

如存在：

F:\00-Ticenpi-SaaS\ticenpi-platform

或其他正式 shared / deploy repo，也要只讀盤點。

不要假設 package / monorepo 結構。

---

# PHASE 0 — GLOBAL PREFLIGHT

對相關 repo 記錄：

- path
- branch
- HEAD
- git status
- git diff
- git diff --cached
- untracked files
- active writer evidence

禁止：

git reset --hard
git clean
git stash
rebase

輸出：

## GLOBAL WRITE MAP

| Repo | Branch | HEAD | Dirty | Active Writer | Writable |

要求：

- Audit phase 全部 read-only
- Implementation phase 只有單一 writer
- Post / 591 本輪預設 read-only

---

# PHASE 1A — READ-ONLY AUDIT: EXTRACTIONHUB DISTRIBUTION

盤點：

- package metadata
- pyproject.toml / setup config
- src layout
- dependency boundaries
- semantics.py
- presentation.py
- imports
- versioning
- test packaging
- Docker relevance
- whether modules can be packaged without FastAPI / adapters
- whether pure shared package can be extracted without semantic duplication
- existing CI artifact patterns
- existing wheel / sdist patterns
- existing release identity

回答：

1. Shared Engine 是否能成為獨立 Python package artifact？
2. 是否可從 ExtractionHub build wheel？
3. 是否能固定 commit / version / digest？
4. 是否會把 FastAPI、browser、adapter dependencies 一起拖入？
5. 最小 dependency set 是什麼？
6. 現有 CI 是否能產 artifact？

輸出：

EXTRACTIONHUB DISTRIBUTION AUDIT

---

# PHASE 1B — READ-ONLY AUDIT: DM RUNTIME

盤點：

- DM backend Dockerfile
- compose
- staging compose
- requirements / lock
- build context
- release identity
- image build flow
- `EXTRACTION_CORE_ROOT`
- shared_engine.py bridge
- local dev path behavior
- container filesystem
- runtime import behavior
- Staging deploy path

回答：

1. 現在 DM 為什麼只能靠本機 source checkout？
2. Docker build 時能不能放入 versioned shared artifact？
3. DM image 是否能 pin Shared Engine version / commit / digest？
4. rollback 時 Shared Engine 是否跟 DM release identity 一起回滾？
5. 是否會產生 hidden mutable dependency？

輸出：

DM RUNTIME PACKAGING AUDIT

---

# PHASE 1C — READ-ONLY AUDIT: POST

盤點：

- runtime language / Node version
- package manager
- frontend/backend boundaries
- current caption / layout / price / area formatting
- existing API clients
- shared schema handling
- code generation tooling
- runtime network dependency policy
- build artifact model

不要修改。

回答：

1. Post 需要 semantic compare 嗎，還是只需要 presentation？
2. 哪些 formatter 是 product-specific？
3. 是否已有 OpenAPI / JSON Schema / generated-client workflow？
4. 可以消費哪種跨語言 contract？
5. 如果透過 API，latency / availability / auth 影響是什麼？

輸出：

POST CONSUMER AUDIT

---

# PHASE 1D — READ-ONLY AUDIT: 591

同 Post。

盤點：

- price
- area
- layout
- floor
- age
- address
- parking
- preview formatting
- runtime language
- package tooling
- API/client generation capability

輸出：

591 CONSUMER AUDIT

---

# PHASE 2 — CANDIDATE ARCHITECTURES

至少評估以下方案。

## OPTION A — PYTHON VERSIONED PACKAGE + CROSS-LANGUAGE CONTRACT

ExtractionHub semantic/presentation engine：
→ Python package / wheel

DM：
→ 直接 pin Python package

Post / 591：
→ 不重寫 semantic
→ 透過 stable JSON contract / generated schema consumer

必須說清楚：
Post/591 的 runtime semantic source 怎麼取得。

---

## OPTION B — SHARED SEMANTIC SERVICE/API

Shared Engine 暴露穩定 internal API。

DM / Post / 591：
→ 全部呼叫同一 semantic/presentation service

評估：

- latency
- availability
- deployment coupling
- auth
- failure modes
- offline capability
- version pin
- rollback
- scaling
- whether this is over-architecture

沒有證據必要時，不要選微服務。

---

## OPTION C — LANGUAGE-NEUTRAL SPEC + GENERATED IMPLEMENTATIONS

Single semantic spec：

- JSON Schema
- normalization tables
- formatter definitions
- generated artifacts

Python / JS 都由 spec 產生。

評估：

- 是否真的能避免 semantic drift
- generator complexity
- testing burden
- current project tooling fit

不要人工維護兩份 implementation。

---

## OPTION D — EXISTING PLATFORM CAPABILITY

如果 repo 已有：

- shared artifact registry
- package publishing
- internal API
- generated client
- schema distribution

優先 reuse。

---

# PHASE 3 — DECISION CRITERIA

方案必須用以下 criteria 判斷，不可只憑偏好：

1. Single semantic source of truth
2. No duplicate hand-maintained semantics
3. DM Docker/Staging deployable
4. Version pinning
5. Rollback determinism
6. Low runtime latency
7. Low availability coupling
8. Cross-language support
9. CI testability
10. Security boundary
11. Minimal operational complexity
12. Existing infra reuse
13. Local developer ergonomics
14. Product profile independence

輸出：

## ARCHITECTURE DECISION MATRIX

| Criterion | A | B | C | D |

最後只能選一個 primary architecture。

若證據不足：

HARD STOP

不要硬選。

---

# PHASE 4 — REQUIRED ARCHITECTURE DECISION

產出：

SHARED ENGINE DISTRIBUTION ADR

至少包含：

Chosen architecture:
Why:
Rejected alternatives:
Versioning:
Artifact identity:
Runtime loading:
Rollback:
Cross-language contract:
Failure behavior:
Security:
CI gates:
Migration path:

重要：

不能以「先複製一份 JS」當正式方案。

---

# PHASE 5 — SINGLE WRITER IMPLEMENTATION

只有 ADR 完成後才開始寫。

第一個 implementation scope：

1. Shared Engine 的正式 versioned delivery
2. DM runtime packaging
3. DM local compatibility
4. DM Docker validation
5. Staging-ready identity
6. Post / 591 contract artifact 或 integration boundary

不直接接 Post / 591。

---

# PHASE 6 — SHARED ENGINE ARTIFACT IDENTITY

Shared Engine 必須有可追蹤 identity。

至少包含：

- semantic contract version
- presentation contract version
- source commit
- package/artifact version
- build identity
- optional digest if available

DM runtime 必須能回答：

```
Shared Engine Version:
Source Commit:
Artifact Identity:
Loaded From:
```

禁止：

只靠「目前資料夾裡的最新版」。

---

# PHASE 7 — DM PACKAGING

目標：

移除「Production/Staging runtime 依賴開發機 source path」這個問題。

Local dev 可以保留 developer override，但：

Staging / Production candidate 必須：

- build-time 或 immutable-runtime delivery
- pin version
- verify loaded identity
- fail closed on incompatible version
- rollback deterministic

不要把：

`F:\00-Ticenpi-SaaS\ExtractionHub`

這種 host path 當正式 runtime dependency。

---

# PHASE 8 — DM COMPATIBILITY GATE

DM bridge 必須驗：

- expected semantic contract version
- expected presentation contract version
- loaded artifact identity
- formatter registry compatibility

若不相容：

不要默默 fallback 到另一份 logic。

必須：

SHARED_ENGINE_INCOMPATIBLE

並依 Shadow mode 原則：

Legacy user path 不受影響。

---

# PHASE 9 — LOCAL + DOCKER VALIDATION

至少驗：

## Local

- Shared Engine import
- semantic compare
- presentation format
- DM Shadow bridge
- admin profile
- Legacy response unaffected

## Docker

DM container：
- 不依賴 host ExtractionHub source checkout
- 能載入 pinned Shared Engine
- identity 可查
- required tests pass

必要案例：

Legacy：
3房2廳2衛

Core：
3房(室)2廳2衛

Canonical：
3/2/2

Status：
NORMALIZED_MATCH

DM compact：
3房2廳2衛

DM slash：
3/2/2

Formatter switch：
Comparator unchanged

---

# PHASE 10 — STAGING READINESS ONLY

本輪預設不要部署 Staging。

只做到：

STAGING READY

需要證明：

- Docker image contains / can resolve immutable Shared Engine
- version identity exposed
- compose/env contract documented
- no host-path dependency
- rollback path clear
- CI artifact reproducible

若使用 external artifact registry：

必須證明：
- availability
- immutable version
- checksum/digest
- auth handling

---

# PHASE 11 — POST / 591 CROSS-LANGUAGE CONTRACT

根據 ADR，產出可直接進下一輪實作的 contract。

至少定義：

- canonical input JSON
- semantic compare request/response if consumer needs it
- presentation request/response
- formatter IDs
- profile schema
- version negotiation
- error contract
- compatibility policy

Post / 591 不得自行發明：

- layout normalize rules
- price normalize rules
- area normalize rules
- semantic match status

如果是 generated contract：

必須證明 generator source 是單一。

如果是 API：

必須定義 timeout / failure / fallback。

---

# PHASE 12 — VERSIONING POLICY

至少定義：

SEMANTIC_CONTRACT_VERSION
PRESENTATION_CONTRACT_VERSION
FORMATTER_REGISTRY_VERSION

規則：

Patch：
- bug fix
- no semantic breaking change

Minor：
- additive formatter / field support

Major：
- semantic-breaking behavior

實際 version scheme 依現有 repo convention。

不要另造不必要 release system。

---

# PHASE 13 — CI GATES

至少建立或補強：

1. Shared Engine unit tests
2. package/artifact build
3. artifact identity test
4. DM consumer compatibility
5. DM Docker import test
6. semantic golden tests
7. formatter golden tests
8. YUCT regression
9. CTHouse regression
10. HBHousing regression
11. git diff --check

若使用 generated cross-language contract：

加：

12. generated artifact drift check

若使用 API：

加：

12. API contract test

---

# PHASE 14 — SECURITY

不得：

- 動態 eval formatter
- 任意 code upload
- runtime import arbitrary path in Staging/Production
- unsigned/unverified mutable artifact
- silent version downgrade
- secret 放進 package metadata

Shared Engine artifact 必須只包含需要的 code / schema。

不要把：

JWT
cookie
API key
private config

封入 artifact。

---

# PHASE 15 — PERFORMANCE

比較至少：

- current local source import
- chosen packaged/runtime delivery

量測：

- import/startup overhead
- normalize
- compare
- format
- DM Shadow added latency

若選 API：

另外：

- network latency
- timeout
- error rate behavior

若 chosen solution 顯著增加 runtime coupling：

必須明確回報。

---

# PHASE 16 — DOCUMENTATION

沿用既有 SSOT。

ExtractionHub：
- _charter/current-state.md
- _charter/history.md
- architecture/ADR if existing convention

DM：
- existing current-state / history / deploy docs

Post / 591：
本輪只讀，不修改其 docs，除非 repo policy 明確允許 read-only audit report 存在 shared architecture repo。

不要建立第二套 SSOT。

---

# PHASE 17 — CLOSEOUT COLLISION CHECK

結束前再次檢查：

- active writers
- git status
- git diff
- staged changes
- unrelated modifications

確認：

Post writes = NONE
591 writes = NONE
Production writes = NONE
Production DB writes = NONE

---

# FINAL REPORT FORMAT

# TICENPI SHARED ENGINE DISTRIBUTION + PACKAGING REPORT

## 1. Baseline

| Repo | Branch | HEAD | Dirty | Writes |

## 2. Parallel Safety

Read-only audits:
PASS / FAIL

Parallel writers:
0

Single writer:
YES / NO

Collision:
NONE / FOUND

## 3. Distribution Audits

ExtractionHub:
DM:
Post:
591:

## 4. Architecture Decision

Chosen:
Reason:

Rejected:
A:
B:
C:

## 5. Shared Engine Identity

Semantic contract:
Presentation contract:
Formatter registry:
Source commit:
Artifact version:
Artifact identity/digest:

## 6. DM Packaging

Local source path required:
YES / NO

Docker packaged:
PASS / FAIL

Pinned:
PASS / FAIL

Runtime identity:
PASS / FAIL

Rollback deterministic:
PASS / FAIL

## 7. DM Validation

Legacy path unchanged:
PASS / FAIL

NORMALIZED_MATCH example:
PASS / FAIL

Presentation switch:
PASS / FAIL

Comparator unchanged:
PASS / FAIL

## 8. Cross-Language Contract

Post:
READY / BLOCKED

591:
READY / BLOCKED

Single semantic source:
PASS / FAIL

Duplicate implementation:
NONE / FOUND

## 9. CI

Shared unit:
Package build:
Artifact identity:
DM compatibility:
DM Docker:
YUCT:
CTHouse:
HBHousing:
Full tests:
git diff --check:

## 10. Performance

Import/startup:
Normalize:
Compare:
Format:
DM added latency:

## 11. Staging Readiness

Immutable delivery:
PASS / FAIL

Host path dependency:
NONE / FOUND

Version identity:
PASS / FAIL

Rollback:
PASS / FAIL

Result:
READY / BLOCKED

## 12. Remaining Gaps

NONE
或逐項列出。

## 13. Final Result

只能：

TICENPI SHARED ENGINE DISTRIBUTION READY

或

TICENPI SHARED ENGINE DISTRIBUTION BLOCKED

---

# HARD STOP

完成本輪後不要：

- 部署 Production
- 修改 Production DB
- 直接修改 Post
- 直接修改 591
- 開始永慶 Adapter
- 開始台灣房屋 Adapter
- 發布 Source Registry
- 切 Extraction Core Primary
- 刪除 Legacy scraper

如果 Staging-ready 已證明，等待使用者核准下一步。
