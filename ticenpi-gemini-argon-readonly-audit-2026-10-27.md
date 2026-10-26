# 執行者：Gemini 4 Argon / Google Antigravity

你正在執行一份**獨立於 Claude 的第二意見唯讀總稽核**。

不要讀取 `F:\Claude` 的報告作為分析依據。目的是讓兩個模型獨立調查，之後再做第三方仲裁。

輸出根目錄固定：

`F:\Gemini`

開始前先建立（若不存在）：`F:\Gemini`。這是唯一允許寫入的位置。

## Gemini / Antigravity 工具策略

優先使用目前 IDE / Antigravity 已具備的 read_file、list/search/grep、terminal 唯讀操作、Git 唯讀、browser/network 唯讀與已存在的 MCP servers。

Google Gemini / Antigravity 系列具備 MCP client 能力時，先列出目前已連線 MCP，再判斷是否足夠。

### codebase-memory-mcp

目前官方專案文件列出 Gemini CLI 與 Antigravity 為支援 client，可列為優先候選。若本機已存在，可以使用，但不得讓它在任何被調查 Repo 建 `.codebase-memory` 或其他 artifact。index/cache 必須 relocation 到 `F:\Gemini\_tooling-data`；若做不到，本輪只記錄建議，不使用。

### Graft

確認目前版本是否提供標準 MCP transport，並確認 Antigravity/Gemini MCP client 可以連線後才可使用。不得因為「理論上 MCP 相容」就直接修改全域設定。

## 禁止修改 Gemini / Antigravity 全域設定

本輪不得自行修改 `%USERPROFILE%\.gemini\settings.json`、Antigravity 全域 config、IDE global settings、PATH、Registry。如果需要新增 MCP，只在 `F:\Gemini\TOOLING-RECOMMENDATION.md` 提供可複製設定與驗證步驟，不自動套用。

## 第二意見要求

不要因為 Claude 可能得出某個結論就迎合。你必須獨立回答：AI_Center 的正確邊界、哪些舊案真的重複、哪些 Plugin 化是過度工程、哪些產品應消失在 Navigation、哪些 Open Source 不值得導入、最適合商業化的入口、前端 Experience 最佳方向。證據不足就寫 UNKNOWN。

# Ticenpi 房地產產品生態唯讀總稽核

## 任務目標

目前已有大量約 80% 完成度的房地產相關產品、插件、服務與自動化能力。這一輪**不是施工、不是重構、不是搬移專案**，而是做一次完整的「產品＋技術＋UX＋商業化」唯讀調查。

核心問題：

1. 現在到底有多少產品、插件、系統、共用服務、Automation、Legacy？
2. `F:\AI_Center` 作為新的自動化核心，目前已可取代舊 HUB 的哪些能力？
3. 舊 HUB / 舊案中哪些能力已與 AI_Center 重複，哪些仍是唯一能力必須保留？
4. 哪些跨產品能力值得抽成 Plugin / Shared Service / Library / SDK？
5. 哪些功能不應再以獨立產品或 Menu 出現，而應融入物件、客戶、行銷、成交流程？
6. 如何把目前十多個房地產工具重新組合成一套直覺、具有產品感的商用 Ticenpi Real Estate OS？
7. 如何避免前端變成傳統 ERP / CRM 辦公介面？
8. 現有中央席位、權限、方案能力是否已足以取代 Lago / ZITADEL 類外部系統？
9. Graft、codebase-memory-mcp、Backstage、NocoBase、Twenty、Lago、ZITADEL 等工具到底是「需要」、「只參考」還是「不需要」？
10. 第一階段應該怎麼商業化、怎麼收費、怎麼拓展市場？

# 最高優先級：STRICT READ-ONLY MODE

這一輪是**唯讀稽核**。

## 唯一允許寫入的位置

`F:\Gemini`

除此之外，任何其他位置一律視為 READ ONLY。

## 原始專案絕對禁止修改

以下所有被調查專案：不得新增/修改/刪除檔案；不得格式化或自動修正 lint；不得 npm/pnpm/pip install 到原專案；不得更新 lockfile；不得建立或修改 `.env`、package.json、gitignore；不得建立 worktree、checkout、commit、stash、reset、pull、push、tag、PR；不得改 CI/CD、Docker、Compose、Registry、DNS、OAuth、Credential；不得修改 Production/Staging/Local DB；不得 migration、seed、truncate、update、delete DB records；不得啟動會寫資料的 worker；不得觸發真正發文、LINE、Email、電話、VOIP、付款、Webhook 等 side effect。

## Git 只允許唯讀指令

允許：`git status`、`git log`、`git branch --show-current`、`git remote -v`、`git diff`、`git ls-files`、`git show`、`git rev-parse`。禁止任何改寫 working tree / index / refs 的 Git 操作。

## Database 只允許唯讀

如果能安全確認唯讀連線，可 inspect schema、list tables、SELECT limited metadata/sample、inspect migration source。若無法保證唯讀，不要連線 DB，只從程式碼與 schema/migration 分析。

## HTTP / Browser

只使用 GET / HEAD / OPTIONS、browser inspect、Network inspect、source/response read。禁止 POST/PUT/PATCH/DELETE、Submit form、會改資料的 login flow、發文、發送訊息、建立 Lead、建立 CRM record、OAuth reauthorization。無法確認 side effect 就視為禁止。

遇到疑似 Production 立即標記：`PRODUCTION — READ ONLY — DO NOT MODIFY`。

# 工具 / MCP 安裝規則

本輪可評估工具，也可使用已存在 MCP / CLI。缺分析能力時，只有以下條件全部成立才允許安裝：

1. 安裝內容完全落在 `F:\Gemini\_tooling`
2. Cache/index/DB 完全落在 `F:\Gemini\_tooling-data`
3. 不修改任何被調查專案
4. 不修改 `%USERPROFILE%`、全域 IDE config、PATH、Registry
5. 不安裝 Windows Service
6. 不建立 startup task
7. 不修改 Git config
8. 不修改現有 MCP config
9. 可完整移除
10. 安裝前記錄來源、License、版本與 Release 資訊

若工具必須修改全域 Agent/IDE/MCP 設定才能使用，本輪禁止自動安裝，只在 `F:\Gemini\TOOLING-RECOMMENDATION.md` 列人工步驟。

# 已知必查專案

```text
F:\AI\_104Recruit
F:\LINE訊息工作台
F:\yuct_api
F:\三節賀卡
F:\批次照片優化
F:\每日房產快訊
F:\周邊生活機能
F:\社區與LINE名片
F:\格局圖
F:\單物件DM
F:\跨房仲案件收集平台
F:\實價查詢
F:\銷售說明書
F:\獨立產品共用
F:\薪資管理系統
F:\購屋說明書
F:\00-Ticenpi-SaaS
F:\591搜尋
F:\AI_Center
F:\AI_CRM
F:\AI_蒐客
```

以上不是完整清單。可在 F:\ 其他合理位置搜尋 Ticenpi、房地產、Real Estate、CRM、Social、LINE、POST、DM、SIGN、HUB、AI、property、listing、lead，但不要掃描無關私人資料。

# 專案分類

每一個資料夾判定 Primary Type：PRODUCT、SYSTEM、PLATFORM、PLUGIN、SHARED SERVICE、LIBRARY、SDK、DATA SOURCE、AUTOMATION、AGENT、INTERNAL TOOL、EXPERIMENT、LEGACY。可再給 Secondary Type。不要把所有資料夾都叫 Project。

# 實際證據優先

不要只相信 README。至少檢查 README*、package.json、lockfile、requirements/pyproject、Docker、Compose、`.env.example`、Next/Vite/TS config、src/app/pages/api/routes/services/workers/jobs/schema/migrations/prisma/docs/.github/workflows。

調查 Framework、Runtime、Git repo/branch/last commit、API、DB、Auth、OAuth、Browser automation、Queue、Cron、Worker、AI provider、LLM routing、Storage、External API、Ports、Deploy target、Production/Staging/local、Shared packages、duplicate code、duplicated business capability。所有重要結論附 Evidence Path；找不到寫 UNKNOWN / NOT VERIFIED / NOT FOUND，禁止猜。

# 01 System Inventory

建立 `01-SYSTEM-INVENTORY.md`。每項至少包含 Name、Path、Primary/Secondary Type、User、Purpose、Framework、UI、API、DB、Auth、AI、Automation、Deployment、Completion、Dependencies、Duplicate、Unique、Recommendation（KEEP/WRAP/MERGE/EXTRACT/RETIRE/UNKNOWN）、Evidence。

# 02 Capability Map

建立 `02-CAPABILITY-MAP.md`，不要以資料夾名稱思考，拆成能力。至少涵蓋：

- Data Acquisition：591、房仲網站、社區、實價、地址、104、公開網路資料、瀏覽器擷取
- Property：ingestion、normalization、community、floor plan、images、location、amenities、pricing/market、documents
- Marketing：DM、銷售說明書、購屋說明書、LINE 名片、賀卡、Social、Copy、Image optimization
- CRM：Contact、Buyer、Seller、Lead、Requirement、Follow-up、Timeline、Deal
- Communication：LINE、Email、Phone、VOIP、Inbox、AI support
- Lead Generation：AI 蒐客、104、跨房仲案件、enrichment
- Operations：Payroll、User、Seat、Permission、Plan、Usage
- Automation/AI：AI_Center、Agent、Workflow、Scheduler、Trigger、Tool Registry、Prompt Registry、Model routing、Approval

依實際調查修正分類。

# 03 Overlap / Unique Capability

建立 `03-CAPABILITY-OVERLAP.md`。每項判定 UNIQUE、DUPLICATED、PARTIALLY DUPLICATED、SAME CAPABILITY/DIFFERENT DOMAIN、UNKNOWN。對重複能力提出 KEEP SEPARATE、REUSE EXISTING、EXTRACT SHARED、LIBRARY、PLUGIN、API SERVICE、WORKER、SDK、FRONTEND PACKAGE、MERGE、RETIRE。

# 04 Shared Capability / Plugin

建立 `04-SHARED-CAPABILITY-STRATEGY.md`。特別調查 Web scraping、Browser automation、URL parser、Property normalization、OCR、PDF parse、Document generation、Image optimization、Geo/Map、Community、Real-price、Social publish、AI gateway、LLM router、Storage、Notifications、Contact matching、Buyer-property matching、QR、Auth、Seat/entitlement。

每項回答：有幾個產品用？是否真重複？行為是否穩定？是否要獨立部署/scaling？是否涉及 Token/Credential/Browser session/Queue？未來其他產業能否復用？最適合 Library / Plugin / Service / Worker / SDK / Package 哪一種？

遵守：`Duplicate twice → observe`；`Duplicate three times → evaluate abstraction`。不要為漂亮架構把所有東西微服務化。

# 05 AI_Center vs 舊 HUB

建立 `05-AI-CENTER-ROLE.md`、`06-HUB-AI-CENTER-MIGRATION.md`。調查 AI_Center 是否應負責 Agent Runtime、Workflow、Task Orchestration、Tool Registry、Prompt Registry、Model Router、Scheduler、Event Consumer、Memory、Approval Center、Human Decision Gateway、Project status、Monitoring。每項分類 CORE / OPTIONAL / NOT RESPONSIBILITY。

AI_Center 不應因為是新系統就吞掉 Property / CRM / Payroll / Social / LINE。優先驗證：`理解 → 決策 → 編排 → Tool Call → 回報 → 人工決策`，Domain Logic 留在 Domain Service。

舊 HUB 每項能力分類 RETIRE NOW / MIGRATE / KEEP TEMPORARILY / KEEP LONG TERM / UNKNOWN。只有確認 AI_Center 等價或更強才能建議退休。

# 06 Developer Intelligence / MCP

評估 codebase-memory-mcp 與 Graft：是否已裝、語言覆蓋、跨 Repo、dependency graph、code context、cache location、Windows、MCP、Gemini/Antigravity 支援、是否互相重複。若無法完全隔離，本輪只寫安裝建議，不實裝。不要兩套都裝，除非證明互補。建立 `07-DEVELOPER-CONTEXT-STRATEGY.md`。

# 07 外部開源評估

建立 `08-OPEN-SOURCE-EVALUATION.md`，逐一評估 Backstage、NocoBase、Twenty CRM、Frappe CRM/ERPNext、Lago、ZITADEL、Freehold、InsulaCRM、Graft、codebase-memory-mcp。

每個必須有 Current problem solved、Existing Ticenpi equivalent、Gap、Benefit、Integration cost、Operational cost、License、SaaS restrictions、Maintenance、Recommendation。Decision 只能是 INSTALL / TEST / REFERENCE ONLY / NOT NEEDED / DEFER。

# Lago / ZITADEL Gap Analysis

目前已有中央席位控制，所以一律先比較。

Billing 建立 `09-BILLING-GAP-ANALYSIS.md`，檢查 Plan、Seat、Feature、Quota、Entitlement、Usage、Credit、Add-on、Billing cycle、Invoice、Metering、Overage。

Identity 建立 `10-IDENTITY-GAP-ANALYSIS.md`，檢查 Organization、Tenant、User、Seat、Role、RBAC、Login、Permission、OAuth、Session，以及未來是否真需要 SSO、MFA、Passkey、SAML、OIDC、Enterprise federation、Delegated administration。只有明確 Gap 才建議 Lago/ZITADEL。

# NocoBase / Control Plane

不要把 NocoBase 預設當 Customer Frontend。優先評估 Internal Admin、Operations Console、Workflow、Backoffice、Human approval、Data management、Configuration。客戶前端與 Control Plane 分離。建立 `11-CONTROL-PLANE-STRATEGY.md`。

# Twenty / Frappe

比較 `F:\AI_CRM` vs Twenty/Frappe/ERPNext。回答現有 AI_CRM 已有什麼、哪些 Data Model/UX/Workflow 值得借、是整合、參考還是不需要。

# Shared Domain Model

建立 `12-SHARED-DOMAIN-MODEL.md`、`13-ID-STRATEGY.md`。研究 Organization、Workspace、User、Team、Contact、Lead、Buyer、Seller、Agent、Property、Listing、Community、Address、BuyerRequirement、PropertyMatch、Campaign、Creative、Post、Conversation、Message、Call、Task、Activity、Deal、Transaction、Contract、Document、Media、Integration、Account、Plan、Seat、Entitlement、Usage。

回答同一物件如何跨 DM/說明書/LINE/CRM/Social；同一客戶如何跨 CRM/LINE/VOIP/買方匹配。研究 organization_id、workspace_id、user_id、contact_id、property_id、listing_id、conversation_id、campaign_id、deal_id。Shared ID ≠ Mega DB。

# Event Architecture

建立 `14-EVENT-ARCHITECTURE.md`。研究 PropertyCreated、PropertyUpdated、LeadCreated、ContactCreated、MessageReceived、CampaignPublished、BuyerRequirementCreated、PropertyMatched、TaskDue、DealWon，以降低 hard-code coupling。

# Product IA / Feature Placement

建立 `15-PRODUCT-INFORMATION-ARCHITECTURE.md`、`16-FEATURE-PLACEMENT.md`。禁止把資料夾名稱直接變 Menu。批次照片優化、實價、生活機能、LINE 名片、DM 等都要判斷應位於 Independent Product / Main Navigation / Sub-page / Context Action / Quick Action / Plugin / Background Service / AI Tool / Admin only。

# Product Experience / UX

建立 `17-PRODUCT-EXPERIENCE.md`、`18-VISUAL-DIRECTION.md`、`19-UX-PROTOTYPE-SPEC.md`。

不要做普通 CRM/ERP：Sidebar + 20 Menu + KPI Cards + Table + Chart + Filter + Form。先從房仲 Journey 開始。

Listing Journey：拿到物件 → 建立 → 補資料 → 圖片 → 格局 → 實價 → 社區 → 生活機能 → 文案 → DM/說明書 → Social → 詢問 → CRM → Follow-up → Deal。

Buyer Journey：買方需求 → Requirement → Search → Match → 推薦 → LINE → Follow-up → 帶看 → Deal。

Prospecting：找名單 → Enrichment → CRM → Qualification → Contact → Follow-up。

UX 優先 `Object + Action + Next Best Action`，不是 `Module + CRUD + Menu`。

至少提出三套明顯不同方向：AI Workspace、Visual Real Estate Journey、Action/Mission Workspace。視覺希望生動、活潑、高質感、新奇、房屋圖片導向、地圖/空間感、Motion、Micro interaction、Timeline、Smart actions、AI suggestion、Command bar、Drag/Swipe/Context actions，但不能為炫技犧牲效率。

# AI Experience

研究 `Ask Ticenpi`：今天有哪些客戶一定要追？這間房有哪些買方？幫物件產生完整行銷素材；整理今天 LINE 買方需求；補齊新物件並指出缺項。AI_Center 應是跨產品 orchestration，不是另一個獨立 Menu。

# Commercial Product / Business / GTM

建立 `20-COMMERCIAL-PRODUCT.md`、`21-BUSINESS-MODEL.md`、`22-GO-TO-MARKET.md`。

不要賣「591 + DM + CRM + LINE + AI + 照片」功能列表，要找一句產品承諾。研究 Acquisition Wedge：房屋網址/地址 → 自動取得資料 → 圖片 → 社區 → 生活機能 → 實價 → 文案 → DM → Social，判斷是否適合作為 Free Acquisition Product。

研究 Free / Agent / Pro / Team / Brokerage / Enterprise，以及 Base subscription / Seat / Usage / Credits / AI credits / Add-on，但先做價值與成本分析，不要直接定死價格。

第一市場不能同時打個人房仲、小型店、加盟店、建商、代銷，必須提出 Primary beachhead / Secondary / Later。

# Migration Roadmap

建立 `23-MIGRATION-ROADMAP.md`。原則：`Reuse > Wrap > Integrate > Extract > Refactor > Rewrite`。每 Phase 至少 Goal、Scope、Systems affected、Dependencies、Data/API/User impact、Risk、Rollback、Acceptance criteria、Owner decision。本輪只規劃，不施工。

# 必須輸出的 Mermaid

CURRENT-SYSTEM-MAP.mmd、TARGET-PLATFORM-ARCHITECTURE.mmd、AI-CENTER-ARCHITECTURE.mmd、SHARED-CAPABILITY-MAP.mmd、CUSTOMER-UX-FLOW.mmd、DATA-FLOW.mmd。

# 最終輸出目錄

所有檔案只能在 `F:\Gemini`：

```text
00-EXECUTION-STATE.md
01-SYSTEM-INVENTORY.md
02-CAPABILITY-MAP.md
03-CAPABILITY-OVERLAP.md
04-SHARED-CAPABILITY-STRATEGY.md
05-AI-CENTER-ROLE.md
06-HUB-AI-CENTER-MIGRATION.md
07-DEVELOPER-CONTEXT-STRATEGY.md
08-OPEN-SOURCE-EVALUATION.md
09-BILLING-GAP-ANALYSIS.md
10-IDENTITY-GAP-ANALYSIS.md
11-CONTROL-PLANE-STRATEGY.md
12-SHARED-DOMAIN-MODEL.md
13-ID-STRATEGY.md
14-EVENT-ARCHITECTURE.md
15-PRODUCT-INFORMATION-ARCHITECTURE.md
16-FEATURE-PLACEMENT.md
17-PRODUCT-EXPERIENCE.md
18-VISUAL-DIRECTION.md
19-UX-PROTOTYPE-SPEC.md
20-COMMERCIAL-PRODUCT.md
21-BUSINESS-MODEL.md
22-GO-TO-MARKET.md
23-MIGRATION-ROADMAP.md
TOOLING-RECOMMENDATION.md
LICENSE-REVIEW.md
DECISIONS-REQUIRED.md
CURRENT-SYSTEM-MAP.mmd
TARGET-PLATFORM-ARCHITECTURE.mmd
AI-CENTER-ARCHITECTURE.mmd
SHARED-CAPABILITY-MAP.mmd
CUSTOMER-UX-FLOW.mmd
DATA-FLOW.mmd
EXECUTIVE-BRIEF.md
MASTER-PLAN.md
FINAL-REPORT.md
```

# EXECUTIVE-BRIEF

白話回答：現在找到多少產品/Plugin/Shared Service/Automation/Legacy？最大重複？AI_Center 已可取代舊 HUB 哪些？哪些唯一能力不能刪？最值得共用化 5～10 項？哪些功能不應在前端成為 Menu？最後客戶登入看到什麼？哪些產品不用重寫？哪些外部開源值得用/不需要？第一個最好賣功能？Free 給什麼？Paid 怎麼升級？第一批客戶？下一輪只能施工一件事該做什麼？

# MASTER PLAN

至少包含 Executive Summary、Current Ecosystem、Current Problems、Capability Map、Duplicate/Unique、AI_Center Target Role、Legacy HUB Retirement、Shared Capability Architecture、Plugin Strategy、Domain Model、Unified Product Architecture、Customer Experience、Control Plane、AI Experience、Open Source Decisions、SaaS Foundation、Commercial Product、Pricing Hypothesis、GTM、Migration、Risks、Recommended Build Order、Do Not Build Yet、Owner Decisions Required。

# STOP CONDITIONS

若任何步驟需要修改原專案、寫 DB、更新 dependency、修改 IDE/Agent 全域 config、改 Production/Staging、登入第三方並產生 side effect、取得新的敏感 credential 或 destructive operation，立即停止該步驟，記錄 `BLOCKED_BY_READ_ONLY_POLICY`，改用靜態證據完成最大程度分析。

# 成功標準

1. 沒有任何原專案被修改。
2. 所有調查產物只存在 `F:\Gemini`。
3. 清楚說明所有現有能力。
4. 清楚說明重複與唯一能力。
5. 提出 AI_Center / 舊 HUB 收斂方案。
6. 提出共用能力與 Plugin 策略。
7. 提出不混亂的 Customer UX。
8. 提出商業模式與市場入口。
9. 明確判斷哪些開源工具需要、哪些不需要。
10. MASTER PLAN 足以交給下一輪 Coding Agent 施工。
