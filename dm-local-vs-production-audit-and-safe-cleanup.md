# DM Local vs Production Audit + Safe Cleanup

## 任務

對 TicenpiDM 做一次「本機完整狀態 vs Production 已部署版本」的唯讀比對，先把目前本機 Git HEAD + 所有未提交 WIP 與 Production 版本分類清楚，再清除「已確認不需要」的舊檔／重複檔／過時 WIP。

核心目標：

> 先查清楚，再清理；絕對不能因為清理而把本機最新功能、尚未提交的 UI WIP 或 Production 已有但本機仍需要的內容弄掉。

---

## 專案

本機主工作區：

`F:\00-Ticenpi-SaaS\TicenpiDM`

目前本機 Local Dev：

- Frontend: `http://127.0.0.1:9422/`
- Backend: `http://127.0.0.1:8000/`

Production 版本：

`93ebd87399b9fb6ed3c29ca5a4b8a44d11584ab5`

注意：

- Production SHA 是本任務目前的比較基準。
- 不要自行改成其他 Production SHA。
- 如果實際部署紀錄顯示 Production SHA 已不同，先停止清理並報告實際 SHA，不可猜測。

---

# 第一原則：禁止破壞性 Git 操作

整個任務期間禁止：

- `git reset --hard`
- `git reset`
- `git restore`
- `git checkout` 用於覆蓋工作檔
- `git clean`
- `git stash`
- 強制 checkout 其他 branch
- rebase
- force push
- 直接把 Production checkout 覆蓋本機
- 用舊 release worktree 覆蓋目前主工作區

絕對不要因為「整理乾淨」而消滅未提交內容。

---

# 第二原則：先盤點，不要一開始刪

先取得完整證據：

1. `git branch --show-current`
2. `git rev-parse HEAD`
3. `git status --short`
4. `git diff --stat`
5. `git diff --name-status`
6. `git diff --cached --name-status`
7. 必要時檢查：
   - `git diff`
   - `git diff --cached`
   - 最近 commit history
   - Production commit diff
   - relevant file history

如果本機有大量未提交檔案，例如 49 個，不能把「未提交」直接等同於「不要」。

---

# 第三原則：建立五類分類

將本機相對於 Production 的差異逐檔分類：

### A. Production 已有，而且本機只是相同內容

結論：

- 不需要保留一份「重複副本」。
- 但如果只是 Git working tree 的正常修改，不可自行刪除。
- 只有確認它是純粹生成物／重複副本／明確無用途檔案，才可清除。

### B. 本機真正新增，而且 Production 沒有

例如：

- 新 UI
- 新功能
- 新測試
- 新樣式
- 新資料結構
- 本機正在開發中的功能

結論：

> 一律保留。

即使未 commit，也不能刪。

### C. 本機是舊版本／過時 WIP，而且 Production 已經有更新版本

這是最重要的清理候選。

但必須確認：

- Production 的對應功能確實存在
- 本機版本沒有額外的新功能
- 本機差異不是尚未完成的新需求
- 刪除後不會影響 Local Dev
- 不會只是因為檔案名稱相同就誤判

只有全部確認後才可標記為「可清除」。

### D. 無關的暫存／生成／垃圾檔

例如：

- build output
- cache
- log
- temporary file
- duplicated export
- 明確可重新產生的暫存檔

確認不屬於 source of truth 後才可清除。

### E. 無法判定

例如：

- 同時包含新舊功能
- backend / frontend 相互依賴
- schema / migration 不確定
- Production 與 Local 的行為不同但原因不明

結論：

> 保留，不刪。

---

# 第四原則：特別檢查目前已知的 49 個未提交檔案

如果目前仍有約 49 個 dirty files，不要用「49 個都是垃圾」處理。

特別檢查：

- `normalize.js`
- `design-doc.v1.json`
- `CardLayoutPicker`
- `App.vue`
- `BuildStamp`
- backend WIP
- 照片旋轉相關
- 字型相關
- `fontFamily`
- `fontSize`
- `rotation`
- `photoAdjust.rotation`
- `textStyles`
- face-target / front-back 相關功能
- Extraction Shadow / Console 相關功能
- 任何最近 UI WIP

這些名稱只是「需要檢查」，不是代表一定保留或一定刪除。

必須以實際 diff、引用關係、Production commit 與 runtime 行為判定。

---

# 第五原則：Production 不等於 Local 最新

不要做以下錯誤推論：

- Production 有某功能 → Local 對應檔案就是舊的
- Local 未 commit → 一定不要
- Production commit 比 Local HEAD 新 → Local 一定比較舊
- 檔名相同 → 可以直接覆蓋
- Production 有 admin console → Local 的 App.vue 就可以直接刪
- Production 沒有某檔 → Local 就一定是新功能

必須比較實際內容。

---

# 第六原則：不要只比較 Git HEAD

真正要比較的是：

> Local Working Tree 實際內容

也就是：

**Local HEAD + staged changes + unstaged changes + untracked files**

而不是只拿 Local HEAD 跟 Production SHA 比。

這一點非常重要。

---

# 第七原則：建立報告

先輸出：

## 1. 版本關係

包含：

- current branch
- local HEAD
- Production SHA
- ahead / behind
- merge-base
- 是否存在 release worktree
- release worktree SHA（如果存在）
- 是否與目前 Local 主工作區不同

## 2. Dirty files

列出：

| 檔案 | Git 狀態 | 分類 | 是否保留 | 理由 |
|---|---|---|---|---|

## 3. 功能差異

至少檢查：

- UI
- A4 editor
- sticker / photo
- rotation
- text styles
- font
- layout data
- face-target
- front/back
- Extraction Shadow
- admin console
- backend
- tests
- config
- build/deploy scripts

## 4. 最終清理候選

分成：

### SAFE_TO_DELETE

只有「證據充分」才能放這裡。

### KEEP_LOCAL_NEW

本機真正的新功能。

### KEEP_WIP

雖未完成，但不能刪。

### KEEP_PRODUCTION_DEPENDENCY

Production 有關聯，但本機仍需要。

### UNKNOWN_KEEP

無法判定，一律保留。

---

# 第八原則：清理規則

只有以下條件全部成立，才可以刪：

1. 已完成 Local vs Production 實際內容比對。
2. 確認不是 Local-only 新功能。
3. 確認不是未完成 WIP。
4. 確認不是測試或必要 dependency。
5. 確認不是 runtime / deploy / config 必需檔。
6. 確認不是 Production 尚未同步但本機正在保留的功能。
7. 有明確證據說明「這個檔案已經沒有用途」。
8. 刪除後 Local Dev 不會因此失效。

如果任何一項不確定：

> 不刪。

---

# 第九原則：清理前必須建立可回復方案

在真正刪檔之前：

1. 建立清理前 inventory。
2. 將準備刪除的檔案完整列出。
3. 對每個檔案寫出刪除理由。
4. 不要使用 destructive Git command。
5. 優先把真正不要的檔案移到一個本機 quarantine / archive 目錄，而不是直接永久刪除。

例如：

`_cleanup_quarantine\YYYYMMDD-HHMMSS\...`

但：

- 不要把 quarantine 內容加入 Git。
- 不要把 quarantine 當成 source code。
- 不要修改 Production。
- 不要修改 release worktree。

如果檔案只是 Git ignored/generated garbage，可直接刪除，但仍需在報告中列出。

---

# 第十原則：清理後驗證

清理後必須：

1. `git status --short`
2. `git diff --stat`
3. frontend tests
4. backend tests（如果 backend 有變動）
5. build
6. 啟動 Local Dev
7. 確認：
   - `http://127.0.0.1:9422/`
   - `http://127.0.0.1:8000/`
8. 做基本 browser smoke
9. 確認 A4 editor 可正常開啟
10. 確認現有 UI WIP 沒被誤刪

---

# 第十一原則：不要自行 commit / push / deploy

本任務預設：

- 不 commit
- 不 push
- 不 Docker
- 不 Staging
- 不 Production
- 不修改 Supabase schema
- 不修改 VPS

只整理 Local 工作區。

---

# 最重要的決策規則

請遵守：

> **「比對完成」≠「全部同步 Production」**
>
> **「Production 已有」≠「Local 就可以刪」**
>
> **「Local 未 commit」≠「不要」**
>
> **只有確定沒有價值、沒有依賴、不是新功能、不是 WIP 的檔案才能清除。**

---

# 執行流程

## PHASE 1 — READ ONLY AUDIT

先完整盤點。

此階段：

- 不刪
- 不改
- 不 commit
- 不 push

最後輸出完整分類。

## PHASE 2 — SAFE CLEANUP

只有 SAFE_TO_DELETE 才清除。

KEEP_LOCAL_NEW / KEEP_WIP / UNKNOWN_KEEP 一律保留。

## PHASE 3 — VALIDATION

跑測試、build、Local Dev smoke。

## PHASE 4 — FINAL REPORT

最後輸出：

### Before

- Local HEAD
- Production SHA
- dirty file 數量
- untracked file 數量

### Classification

- Production 已有：
- Local-only 新功能：
- WIP：
- 可清除：
- Unknown：

### Cleanup

逐檔列出實際清掉的檔案與理由。

### Preserved

列出特別保留的 Local-only / WIP 功能。

### Validation

- tests:
- build:
- frontend:
- backend:
- Local 9422:
- Local 8000:

### Git state

- branch:
- HEAD:
- remaining dirty files:
- untracked files:
- commit/push: NO

---

# HARD STOP

遇到以下任一情況立即停止，不要自行決定：

- Production SHA 與指定 SHA 不一致
- 無法取得 Production source
- 無法判定某檔案是不是新功能
- 發現 Local WIP 與 Production 功能互相重疊但行為不同
- 發現可能是 schema / migration / auth / entitlement 相關
- 發現可能影響 Production
- 發現 release worktree 與 Local 主工作區內容不同且用途不明
- 刪除某檔可能造成 runtime failure
- 測試或 build 因清理而失敗

此時只報告，不刪。

---

# 絕對不要做

不要：

- 為了讓 Git status 乾淨而刪 WIP
- 為了跟 Production 一樣而覆蓋 Local
- 把 Local 所有 dirty files commit
- 把 Production checkout 複製回 Local
- 把 release worktree 當成 Local 最新版本
- 把舊版本當成唯一 source of truth
- 用 reset / restore / clean 解決混亂
- 因為檔案名稱相同就判定內容相同

---

# 成功條件

成功不是「git status 變乾淨」。

真正成功是：

> 已經知道 Local 哪些是最新功能、哪些是 WIP、哪些 Production 已經有、哪些是真的垃圾；只清除真正不要的內容，而且 Local 現有可用功能與 WIP 都沒有被破壞。
