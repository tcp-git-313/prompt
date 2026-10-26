# AI 格局圖 MVP v0.1 — Luna 建置執行規格

> 目的：建立可實際操作、可驗收、可持續接手開發的 AI 格局圖 MVP。此文件不是概念說明，而是給 Luna 直接執行的工程規格。

## 0. Agent 角色與硬性規則

你是本專案的主要工程執行 Agent。你的工作不是只提出建議，而是實際建立專案、撰寫程式、啟動服務、測試、修錯、留下證據並完成交接。

### 必須遵守
- 每個重要階段都留下：command、log、screenshot、test result、error、fix、final status。
- 每個階段先通過 Build / Test / Evidence Gate，再往下一階段。
- 不確定時先調查與驗證，不得用猜測取代事實。
- 若外部 API Key、第三方服務或權限缺失，標記 `BLOCKED_EXTERNAL_DEPENDENCY`，完成其餘可做項目，不得偽造成功。
- 不得修改無關專案、不得碰 Production、不得刪除既有資料。
- 不得把 API Key、DB 密碼、JWT secret、AI 核心 Prompt、付費邏輯寫入前端或提交 Git。
- 不得用 mock 結果宣稱真實 AI 辨識已通過。
- 不得在 2GB VPS 直接部署大型 Vision/LLM 模型。

## 1. MVP 目標

最終使用流程：

```text
使用者
  ├─ 手動畫格局
  └─ 上傳 / 拍攝格局圖
          ↓
      AI 格局辨識
          ↓
      Draft Floorplan JSON
          ↓
      Schema Validation
          ↓
      原圖 Overlay + AI 描線
          ↓
      使用者人工校正
          ↓
      2D / 3D
          ↓
      Save / Reload
          ↓
      JSON / PNG / PDF
```

## 2. P0 必做功能

### 格局編輯
- 新增 / 移動 / 刪除牆面
- 牆面長度與厚度
- 房間形成與房間名稱
- 門、窗
- 尺寸標示
- Undo / Redo
- Save / Reload
- Mobile 可操作

### 家具
至少提供：
- 客廳：沙發、茶几、電視櫃、電視、單椅
- 餐廳：餐桌、餐椅
- 廚房：冰箱、廚櫃、流理台
- 房間：單人床、雙人床、衣櫃、書桌
- 衛浴：馬桶、洗手台、浴缸或淋浴設備

家具至少支援：新增、拖動、旋轉、刪除、縮放。

## 3. AI 輸入

MVP 接受：JPG、JPEG、PNG、WEBP。

來源包含：
- 手繪格局圖
- 房仲既有格局圖
- 電腦繪製平面圖
- 手機拍攝紙張

## 4. AI 辨識優先順序

P0：
- wall
- door
- window
- room
- room label
- dimension

P1：
- stairs
- column
- balcony
- bathroom fixtures
- furniture

家具辨識不是 MVP 成敗必要條件；牆、門、窗、房間才是核心。

## 5. AI 不得直接覆蓋正式資料

必須走：

```text
Original Image
→ AI Recognition
→ Draft JSON
→ Validation
→ Preview
→ User Confirm
→ Floorplan
```

禁止 `AI → 直接覆蓋正式格局`。

## 6. Confidence

每個 AI 辨識物件支援 `confidence`。

建議 UI：
- >= 0.85：正常
- 0.60–0.84：黃色提醒
- < 0.60：紅色提醒 / 要求人工確認

## 7. Overlay

AI 完成後必須同時顯示原始圖片與 AI 描線，提供 0–100% 透明度控制，使用者可比對原圖與格局結果。

## 8. 真實尺寸校正

若圖片已有尺寸文字，AI 嘗試 OCR。

若沒有尺寸：
1. 使用者點選一面牆。
2. 輸入實際長度，例如 420 cm。
3. 系統計算整張圖 scale。

不得假裝 AI 可以從普通照片準確知道真實尺寸。

## 9. Canonical Floorplan Schema

整個系統必須有自己的 Canonical Schema，禁止資料庫直接綁死第三方 Editor 內部格式。

```json
{
  "version": "1.0",
  "unit": "cm",
  "scale": 1,
  "walls": [],
  "doors": [],
  "windows": [],
  "rooms": [],
  "objects": [],
  "dimensions": [],
  "metadata": {}
}
```

### Wall
```json
{
  "id": "wall_001",
  "start": {"x": 0, "y": 0},
  "end": {"x": 420, "y": 0},
  "thickness": 12,
  "height": 280
}
```

### Door
```json
{
  "id": "door_001",
  "wallId": "wall_001",
  "position": 0.65,
  "width": 90,
  "direction": "left",
  "confidence": 0.93
}
```

### Window
```json
{
  "id": "window_001",
  "wallId": "wall_002",
  "position": 0.5,
  "width": 120,
  "height": 120,
  "sillHeight": 90
}
```

### Room
```json
{
  "id": "room_001",
  "name": "客廳",
  "polygon": [[0,0],[420,0],[420,360],[0,360]],
  "area": 15.12
}
```

### Furniture
```json
{
  "id": "object_001",
  "catalogId": "sofa_001",
  "category": "living_room",
  "position": {"x": 120, "y": 150},
  "rotation": 90,
  "scale": {"x": 1, "y": 1, "z": 1}
}
```

## 10. 架構原則

採用：`Thin Frontend + Protected Backend + External AI`。

```text
Browser
  ├─ 2D Editor
  ├─ Three.js 3D
  └─ Furniture UX
       ↓ HTTPS
Backend API
  ├─ Authentication
  ├─ Projects
  ├─ Upload
  ├─ Storage
  ├─ AI Orchestration
  ├─ Validation
  ├─ Rate Limit
  └─ Export
       ↓
External Vision AI
```

## 11. 2GB VPS 限制

設計目標：2 vCPU / 2GB RAM。

目標：
- Backend idle < 300MB RAM
- 正常整體服務 < 1GB RAM
- 不把大型影像模型放 VPS
- 不以 Swap 當主要解法
- Server 避免大量同步影像處理

## 12. 前後端責任

Browser：2D/3D render、家具拖拉、牆面編輯、camera、orbit controls。

VPS：Auth、API、Project storage、Upload metadata、AI request、AI validation、Export coordination、Rate limit、Audit。

Cloud AI：Image understanding、OCR、牆/門/窗/房間辨識。

## 13. 防逆向與 Secret 管理

假設前端 JavaScript 一定可以被看到。

不得把以下放前端：
- AI API KEY
- DB PASSWORD
- JWT signing secret
- 核心 AI Prompt
- 內部商業規則
- Billing Secret
- Admin API

Minify / Obfuscate 只能增加閱讀成本，不能當安全邊界。

## 14. API

### Project
- `POST /api/v1/projects`
- `GET /api/v1/projects`
- `GET /api/v1/projects/:id`
- `PATCH /api/v1/projects/:id`
- `DELETE /api/v1/projects/:id`

### Floorplan
- `GET /api/v1/projects/:id/floorplan`
- `PUT /api/v1/projects/:id/floorplan`

需有 version / updatedAt / optimistic concurrency。

### Upload
- `POST /api/v1/uploads/floorplan`

檢查：MIME、extension、size、image decode、dimensions；不可只相信副檔名。

### AI
- `POST /api/v1/recognitions`
- `GET /api/v1/recognitions/:id`

回傳至少包含：status、floorplan、warnings、confidence。

## 15. Vision Provider 抽象化

建立 `VisionProvider`：

```text
analyzeFloorplan(image)
```

目錄可規劃：

```text
providers/
  gemini/
  openai/
  mock/
```

Mock 只允許 development / unit test。

## 16. AI Prompt 規則

核心 Prompt 只能存在 Backend。

要求 AI：
- 只輸出結構化 JSON
- 不知道就 null
- 不幻想牆面
- 不自行補不存在的房間
- 座標使用 image pixel 或 normalized coordinate
- 每個物件回傳 confidence

## 17. AI JSON Validation

所有 AI response 必須經 Zod / JSON Schema / Ajv 驗證。

禁止直接 `JSON.parse` 後信任 AI。

必須處理：invalid JSON、timeout、rate limit、API error、empty response、partial detection、unsupported image。

## 18. Editor 選型

第一優先評估 `openPlan3D`，但在採用前必須重新驗證：
- License
- Repository 狀態
- Build status
- Dependency status
- Data structure
- 2D/3D
- Furniture
- JSON Import/Export
- Mobile behavior
- Bundle size
- Extensibility

另外比較 `floorplan-3d` 等候選。

若第三方專案無 License、License 不允許商業使用、長期無維護、build 不通、重大安全問題，只能 reference，不得作正式核心。

## 19. Adapter Layer

若使用 openPlan3D：

```text
Canonical Floorplan JSON
        ↓
OpenPlanAdapter
        ↓
openPlan3D
```

DB 不得直接只存第三方內部 JSON。

## 20. Furniture Library

原則：Low Poly、小檔案、手機優先、Lazy Load。

每件家具至少：thumbnail、2D footprint、3D model、dimensions、category。

Asset budget：
- 一般模型 < 500KB
- 特殊模型 < 1MB

不要載入未使用分類全部 GLB。

## 21. 圖片處理

高解析圖片前端優先做：
- EXIF orientation
- resize
- optional compression

AI 分析圖建議最長邊 1600–2400px，原圖可另存。

## 22. Storage / DB

MVP 優先：Local filesystem + SQLite。

不要無必要導入 PostgreSQL / Redis / Kafka / RabbitMQ。

至少資料表：users、projects、project_versions、uploads、recognitions、exports。

Floorplan 保存保留最近至少 20 個版本。

Autosave debounce 2–5 秒，不得每個 mousemove 打 API。

## 23. Export

P0：JSON、PNG。

P0/P1 邊界：PDF 優先完成，DXF 可列 P1。

PNG 至少 2048px，可白底或透明。

PDF 至少支援 A4、案件名稱、格局圖、基本尺寸。

## 24. Mobile / Responsive

至少實測：390、430、768、1440 px。

不得出現：
- page-level horizontal overflow
- toolbar 無法使用
- canvas 超出
- modal 無法關閉
- 主要 touch target 太小

手機至少可：pan、zoom、select、move furniture、save。

## 25. AI Import UX

```text
Upload
→ Analyze
→ Original Image
→ AI Overlay
→ Confidence Warnings
→ Manual Correction
→ Scale Calibration
→ Confirm
```

## 26. Security

必須完成：HTTPS、security headers、input validation、rate limit、upload validation、auth、CSRF strategy、CORS policy、secret isolation。

Upload 建議上限 15MB，必須真正 decode 驗證 image；MVP 不接受 SVG upload。

AI endpoint 要有限流，例如 5 次/min/user，參數放設定檔。

另需：per-user quota、daily limit、server-side usage log，防止 AI API 費用被刷。

Log 不可記錄 API Key、password、完整 JWT。

## 27. Repository 建議結構

```text
floorplan-ai/
├── apps/
│   ├── web/
│   └── api/
├── packages/
│   ├── floorplan-schema/
│   ├── floorplan-engine/
│   ├── editor-adapter/
│   └── shared/
├── assets/furniture/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── e2e/
│   └── fixtures/
├── docs/
├── evidence/
├── scripts/
├── .env.example
├── README.md
└── docker-compose.yml
```

## 28. 建議技術棧

優先建議：
- Frontend：React + TypeScript + Vite
- Backend：Node.js + TypeScript + Fastify
- Schema：Zod
- DB：SQLite
- 3D：Three.js
- Test：Vitest + Playwright

如果 Editor 技術限制需要調整，必須記錄原因。

## 29. Docker / Env

提供 Dockerfile 與 docker-compose.yml，但 2GB VPS 避免切太多 container。

`.env.example` 至少：

```text
PORT=
DATABASE_PATH=
AI_PROVIDER=
AI_API_KEY=
UPLOAD_DIR=
MAX_UPLOAD_MB=
AI_DAILY_LIMIT=
```

`.env` 禁止提交 Git。

## 30. 執行 Phase 與 Gate

### Phase 0 — Preflight
檢查 OS、Node、Docker、Git、RAM、Disk、Ports、Existing Services、Workspace。

產出 `docs/00-PREFLIGHT.md`。

### Phase 1 — Editor Research
比較 openPlan3D、floorplan-3d 與必要候選。

產出 `docs/01-EDITOR-SELECTION.md`。

### Phase 2 — Canonical Schema
完成 Schema、Validator、Fixtures、Unit Tests。

### Phase 3 — Manual Editor MVP
完成 blank project、wall、door、window、room、furniture、save/load、undo/redo。

### Phase 4 — 2D/3D
完成 2D editor、3D preview、wall extrusion、door/window representation、furniture、camera。

### Phase 5 — Furniture
完成 Living / Dining / Kitchen / Bedroom / Bathroom，驗證 lazy loading 與 asset size。

### Phase 6 — Upload
完成 upload、validation、resize、preview、storage。

測 valid JPG、PNG、oversized、fake extension、corrupt image。

### Phase 7 — AI Adapter
完成 VisionProvider，再接真實 provider。無 Key 時標 `BLOCKED_EXTERNAL_DEPENDENCY`。

### Phase 8 — AI Recognition
完成 image → AI → raw result → schema validation → canonical floorplan。

保存 raw response、normalized result、validation warnings；不得保存 secret。

### Phase 9 — Overlay Correction
完成 original image、AI overlay、opacity、manual correction、confidence display。

### Phase 10 — Scale Calibration
完成選牆、輸入真實長度、計算 scale、重新計算尺寸。

### Phase 11 — Export
完成 JSON、PNG、PDF。

### Phase 12 — Security
至少跑 dependency audit、secret scan、upload abuse、authorization、rate-limit、CORS。

### Phase 13 — Performance
記錄 2GB 環境：Idle RAM、Editing RAM、AI Upload RAM、Export RAM、Peak RAM、CPU。

每一 Phase 完成後，必須：Build PASS、Unit/Relevant Test PASS、手動驗證 PASS、文件更新、Evidence 落地，才能進下一階段。

## 31. Performance Cases

前端：
- 20 / 50 / 100 walls
- 10 / 30 / 50 furniture

記錄 FPS、interaction latency、browser memory。

## 32. AI Dataset

至少 10 張：
- 3 張清楚電腦格局
- 3 張手機拍攝格局
- 3 張手繪格局
- 1 張故意非常差

若無真實圖，可先建 fixture，但不得偽造真實 AI 準確率。

記錄：牆面辨識、門、窗、房間名稱、尺寸 OCR、人工修正數量。

AI 目標不是 100%，而是實際節省描圖時間。

## 33. E2E 必測

### Case A — Manual
建立空白格局 → 4 面牆 → 房間 → 門 → 窗 → 沙發 → 保存 → Reload，內容必須完整保留。

### Case B — AI
Upload → AI Recognition → Preview → 修牆 → Scale Calibration → Save → 3D → PNG Export。

### Case C — Mobile
390px 完成建立格局、家具、縮放、保存。

### Failure Tests
- AI timeout
- AI invalid JSON
- DB unavailable
- corrupt upload
- project not found
- unauthorized request
- rate limit exceeded

## 34. 驗收標準

### PASS
全部 P0 通過，且：
- 無 Critical security issue
- 無資料毀損
- 無 secret exposure
- 無 major mobile blocker

### CONDITIONAL PASS
主要功能完成，但受外部 Key / 第三方服務 / 非阻斷問題影響；必須明確列出。

### FAIL
只要出現以下任一：
- 保存會丟資料
- AI 非法格局可使系統 crash
- API Key 出現在 frontend
- 2GB VPS OOM
- Editor 無法使用
- Mobile 主流程不能完成

## 35. 必要文件

建立：

```text
docs/
  00-PREFLIGHT.md
  01-EDITOR-SELECTION.md
  02-ARCHITECTURE.md
  03-DATA-SCHEMA.md
  04-AI-PIPELINE.md
  05-SECURITY.md
  06-PERFORMANCE.md
  07-TEST-PLAN.md
  08-TEST-RESULT.md
  09-DEPLOYMENT.md
  10-KNOWN-ISSUES.md
  EXECUTION-LOG.md
```

另建立：

```text
evidence/
  screenshots/
  logs/
  test-results/
  performance/
  exports/
```

## 36. README / 啟動

README 必須包含 Requirements、Install、Development、Environment、Build、Test、Run、Docker、AI Setup、Folder Structure、Troubleshooting。

目標一鍵啟動：

```bash
npm install
npm run dev
```

一鍵測試：

```bash
npm run test
npm run test:e2e
npm run test:security
```

## 37. Final Report

完成後建立 `FINAL-REPORT.md`：

```text
# Final Status
PASS / CONDITIONAL PASS / FAIL

# Completed
# Not Completed
# Architecture
# AI
# Security
# Performance
# Tests
# Known Issues
# External Dependencies
# Deployment Readiness
# Recommended Next Steps
```

## 38. Production Gate

MVP 完成不等於可直接 Production。Production 前另驗：Security、Backup、TLS、Monitoring、Domain、AI Cost、Rate Limit、Privacy、Data Retention。

## 39. Definition of Done

必須能完整走完：

```text
開網站
→ 建立案件
→ 上傳手繪格局
→ AI 產生 Draft
→ 原圖 Overlay
→ 調整牆面
→ 調整門窗
→ 設定尺寸
→ 加入家具
→ 切換 3D
→ 保存
→ Reload
→ 資料仍存在
→ 輸出 PNG / PDF
```

## 40. 工程優先順序

```text
可用
> 資料正確
> 安全
> 手機可操作
> 低資源
> 漂亮
> 高階 3D
```

第一版不要把時間大量花在：超寫實光影、大量 3D 模型、AI 自動裝潢、VR、多人即時協作、材質商城。

## 41. P1 後續

可列入：DXF Import/Export、SVG、更多家具、材質、地板、牆色、自動家具配置、AI 室內設計、多樓層、陽台、樓梯、柱樑、面積統計、坪數換算、房仲 DM / 案件系統整合。

## 42. 執行順序（不可跳）

```text
Preflight
→ Editor 技術驗證
→ Canonical Schema
→ Manual Editor
→ Save/Reload
→ 2D/3D
→ Furniture
→ Upload
→ AI Adapter
→ AI Recognition
→ Overlay
→ Calibration
→ Export
→ Mobile
→ Security
→ Performance
→ E2E
→ Final Report
```

## 43. 問題處理規則

遇到失敗：

```text
調查
→ 重現
→ Root Cause
→ 最小修正
→ Regression Test
```

禁止為了讓 CI 綠燈直接刪測試、跳過驗證或把真實失敗改成 mock PASS。

## 44. 最終交付

至少包含：
- 可執行原始碼
- 可正常啟動 Web UI
- 手動畫格局
- 家具配置
- 2D / 3D
- Canonical Floorplan JSON
- 圖片 Upload
- AI Adapter
- 真實 AI 辨識，或清楚的 External Block 狀態
- Overlay 校正
- 比例校正
- 儲存 / 讀取 / 版本
- PNG / PDF
- Mobile
- Security Tests
- Performance Report
- E2E
- docs / evidence
- FINAL-REPORT.md

任何未完成項目必須明確列出；不得用「應該可以」「理論上」「看起來正常」代替測試證據。
