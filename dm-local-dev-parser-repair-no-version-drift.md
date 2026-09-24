# DM LOCAL DEV STARTUP — POWERSHELL PARSER REPAIR ONLY / NO VERSION DRIFT

## OWNER
DM LOCAL DEV STARTUP REPAIR OWNER

## GOAL

修好 DM 本機開發啟動流程，讓：

- Frontend: http://localhost:9422
- Backend: http://localhost:8000

可以正常啟動。

本任務只處理 PowerShell 啟動腳本的 parser / encoding / mojibake 問題。

**不要改 DM UI、不要改產品功能、不要回退版本、不要動 Docker / Staging / Production。**

---

# WORKING REPO

`F:\00-Ticenpi-SaaS\TicenpiDM`

Expected branch:

`dm/integration-20260920`

Known HEAD before this repair session:

`561bd8c76fec5c0fb86cb9cf4f56734d28367b8a`

目前 repo 有既有 dirty WIP，必須完整保留。

---

# CURRENT OBSERVED FAILURE

使用：

`scripts\start_local_dev.ps1`

啟動時，先前已遇到：

1. `start_local_dev.ps1` 第 133 行附近 mojibake 導致：
   - UnexpectedToken
   - TerminatorExpectedAtEndOfString
   - MissingEndCurlyBrace
   - MissingEndParenthesisInSubexpression

2. 使用者已手動把該區塊改成：

```powershell
$agnesSourceLabel = switch ($agnesSource) {
    'process' { 'process environment' }
    'central' { 'central environment' }
    default   { 'project .env' }
}
Write-Host "AGNES_API_KEY source = $agnesSourceLabel"
```

並把 `start_local_dev.ps1` 存成 Windows PowerShell 5.1 可讀的 UTF-8 BOM。

該檔目前 Parser.ParseFile 已回報 0 errors。

3. 繼續啟動後，現在阻塞在：

`scripts\local_source_lib.ps1`

已觀察到的壞點包括：

- line 105 左右：mojibake throw string，疑似 missing closing quote
- line 107：畫面曾出現 `$*.IsReadOnly`，需確認原始實際內容是否應為 `$_.IsReadOnly`
- line 112：mojibake comment 疑似把 `function Import-DmCentralSecret {` 吃進同一行
- line 114：畫面曾出現 `$*.Value`，需確認是否應為 `$_.Value`
- 後續 line 154 被連帶報 `TerminatorExpectedAtEndOfString`，很可能只是前方字串未閉合造成的 cascading parser error

目前：

```powershell
git status --short scripts/local_source_lib.ps1
git diff -- scripts/local_source_lib.ps1
```

都沒有輸出。

也就是說，`local_source_lib.ps1` 目前 working tree 與 HEAD 一致。

但這不代表要回退整個 repo，也不代表要回退 UI。

---

# CRITICAL RULE — NO VERSION ROLLBACK

嚴禁執行：

- `git reset`
- `git reset --hard`
- `git checkout .`
- `git restore .`
- `git restore --source=<old commit> .`
- `git revert`
- `git cherry-pick`
- branch switch
- worktree replacement
- stash pop / stash apply
- 清除 untracked files
- 覆蓋整個 repo
- 回退 DM UI V3 / V3.1
- 回退任何既有 dirty WIP

**本任務不是版本回退。**

Git history 只能作為唯讀參考，用來確認原始 intended logic。

如果需要看舊 commit，只能用：

- `git show`
- `git log`
- `git diff <commit>..<commit> -- <specific file>`

不得把舊版直接 restore 進 working tree。

---

# ALLOWED WRITE SCOPE

原則上只允許修改這兩支：

1. `scripts/start_local_dev.ps1`
2. `scripts/local_source_lib.ps1`

如果第一支目前 Parser 已通過，優先不要再改。

除非驗證證明仍有必要，否則本輪應主要只修：

`scripts/local_source_lib.ps1`

禁止修改：

- `src/frontend/**`
- `src/backend/**`
- DM UI files
- `DmEditor.vue`
- `A4Preview.vue`
- `preview.js`
- UI tests
- Docker files
- compose
- deployment scripts
- release worktree
- VPS files
- Staging
- Production
- Supabase schema
- ExtractionHub

如果發現必須超出這兩支腳本才能修，HARD STOP，先回報，不要自行擴大。

---

# PRE-FLIGHT

開始前先回報：

```text
Repo:
Branch:
HEAD:
git status --short:
```

並額外只針對兩支腳本回報：

```powershell
git status --short -- scripts/start_local_dev.ps1 scripts/local_source_lib.ps1
git diff -- scripts/start_local_dev.ps1 scripts/local_source_lib.ps1
```

必須辨識使用者剛才手動修改的 `start_local_dev.ps1`。

不要覆蓋該修正。

如果 branch 不是 `dm/integration-20260920`，HARD STOP。

如果 HEAD 與已知 HEAD 不同，不要自動回退；只回報差異並確認是否仍是同一工作線。若只是有新 commit，先唯讀檢查再決定。

---

# PHASE 1 — PARSER DIAGNOSIS ONLY

對兩支腳本分別執行 PowerShell Parser.ParseFile。

使用 Windows PowerShell 5.1 compatibility。

輸出：

- parser error count
- line
- column
- ErrorId
- message
- extent text

例如：

```powershell
$errors = $null
[System.Management.Automation.Language.Parser]::ParseFile(
  $path,
  [ref]$null,
  [ref]$errors
) | Out-Null
```

先建立 baseline。

不要一看到第一個錯誤就直接改大量檔案。

---

# PHASE 2 — INSPECT THE ACTUAL DAMAGED REGION

針對 `scripts/local_source_lib.ps1`：

先檢查約 line 90–170。

重點確認：

1. line 105 的 throw string 是否少 closing quote
2. line 107 是否真的存在錯誤變數 `$*`，還是終端顯示 / markdown 轉義造成
3. `function Import-DmCentralSecret` 是否被錯誤併入 comment
4. `Where-Object { $_.Value }` 是否被破壞
5. here-string `@' ... '@` 是否完整閉合
6. `Get-DmPortProcess` 本身是否其實正常，只是被前面 parser error 連帶報錯
7. 是否存在其他 mojibake string literal 破壞引號
8. mojibake 是否只在 comments，還是進入 executable string/token

區分：

- harmless mojibake comment
- parser-breaking mojibake executable code

只修會影響 parser / startup 的部分。

不要為了「順便美化」整支檔案。

---

# PHASE 3 — USE GIT HISTORY AS READ-ONLY REFERENCE

已知：

```text
9f302de chore(scripts): load AGNES_API_KEY from the central secrets file first, .env only as fallback
9e781f4 feat(dm): enforce one canonical source for Local Dev (9422/8000) and Local Docker (19421)
```

可以唯讀比較這兩個 commit 對：

- `scripts/local_source_lib.ps1`
- `scripts/start_local_dev.ps1`

的差異。

目的只有：

- 確認 `Import-DmCentralSecret` intended logic
- 確認 AGNES_API_KEY 的 fallback semantics
- 確認被 mojibake 破壞前的正確 PowerShell 語法
- 確認 `$_` 等 pipeline variables
- 確認 function boundaries / newlines

禁止直接 checkout / restore 舊 commit。

---

# AGNES BEHAVIOR MUST NOT CHANGE

本任務不是重新設計 AGNES secret handling。

應保留目前 intended contract：

1. 如果 process environment 已有 `AGNES_API_KEY`
   → 使用 process value

2. 否則如果 central secrets file 有該 key
   → 只載入 `AGNES_API_KEY`
   → 不載入整份 secrets file

3. 否則
   → 讓既有 project `.env` / env loader fallback 機制繼續處理

不得：

- 印出 secret value
- hardcode API key
- 改 secret name
- 改 Staging / Production secrets
- 改 central secrets file
- 把全部 secret import 進 process

本輪只確保本機腳本可 parse 且保留既有語意。

---

# ENCODING RULE

目標執行環境包含：

`Windows PowerShell 5.1 / powershell.exe`

所以兩支 `.ps1` 必須以 PowerShell 5.1 安全的 encoding 儲存。

優先：

`UTF-8 with BOM`

但只對實際修改過的腳本做必要 encoding normalize。

不要把整個 repo 批次轉碼。

不要因 encoding normalize 造成整支檔案大量無意義 diff。

若現有檔案已經是可安全解析 encoding，就保持。

---

# MINIMAL REPAIR PRINCIPLE

修改目標是：

`Parser errors = 0`

而不是重構。

請遵守：

- 不改函式名稱
- 不改 port
- 不改 frontend/backend啟動方式
- 不改 canonical source checking
- 不改 fingerprint semantics
- 不改 Local Docker logic
- 不改 AGNES contract
- 不改 repo identity guard
- 不改工作目錄判斷
- 不改既有安全檢查

如果某個 malformed 中文錯誤訊息會破壞 parser，可直接換成簡短 English ASCII message。

例如：

```powershell
throw "Source fingerprint mismatch: actual=$overall expected=$($Identity.Overall)"
```

但不要改該條件本身。

同理，壞掉的中文 comment 可：

- 保留不影響 parser 的亂碼 comment，或
- 只在必要時換成簡短 English comment

不要花本輪大量時間做 comment translation。

---

# KNOWN USER MANUAL FIX — PRESERVE IT

`scripts/start_local_dev.ps1` 目前使用者手動修成：

```powershell
$agnesSourceLabel = switch ($agnesSource) {
    'process' { 'process environment' }
    'central' { 'central environment' }
    default   { 'project .env' }
}
Write-Host "AGNES_API_KEY source = $agnesSourceLabel"
```

而且 Parser 已經通過。

不要把它改回原本嵌套：

```powershell
Write-Host "... $(switch (...))"
```

不要重新引入中文 mojibake。

若無必要，完全不要再修改這區。

---

# VALIDATION ORDER

## Gate 1 — Parser

兩支都必須：

`Parser error count = 0`

對：

- `scripts/start_local_dev.ps1`
- `scripts/local_source_lib.ps1`

分別驗證。

只要任一支不是 0，不能進下一步。

---

## Gate 2 — Diff Scope

執行：

```powershell
git diff -- scripts/start_local_dev.ps1 scripts/local_source_lib.ps1
```

確認：

- 只有 parser / encoding / safe diagnostic repair
- 沒有 port change
- 沒有功能重構
- 沒有 secret value
- 沒有其他檔案被修改

再執行：

```powershell
git status --short
```

確認既有 UI dirty WIP 都還在，且沒有被 reset / overwrite。

---

## Gate 3 — Actual Local Startup

執行：

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File "F:\00-Ticenpi-SaaS\TicenpiDM\scripts\start_local_dev.ps1"
```

或使用 repo 既有 bat launcher。

只驗證本機。

需要確認：

- backend process 可啟動
- frontend process 可啟動
- 8000 listen
- 9422 listen

---

## Gate 4 — HTTP Smoke

確認：

`http://localhost:9422/`

能正常回應。

並用 repo 既有 backend health endpoint 驗證 8000。

如果不知道 health path：
- 先從 repo 讀現有啟動 / smoke script
- 不要猜
- 不要建立新 endpoint

---

# IF STARTUP FAILS AFTER PARSER IS FIXED

如果 Parser 已 0，但啟動出現新的 runtime error：

1. 停止
2. 記錄完整錯誤
3. 判斷是否仍屬這兩支 startup scripts
4. 如果是明確、局部、無風險的 startup bug，可最小修正
5. 如果牽涉 Python dependency、Node dependency、UI code、backend code、Docker、auth 或 schema，HARD STOP

不要把「修 parser」擴張成另一個工程。

---

# NO COMMIT / NO PUSH

本輪完成後：

- 不 commit
- 不 push
- 不 deploy

保留 working tree 給使用者驗收。

---

# ACCEPTANCE CRITERIA

PASS only if:

1. `start_local_dev.ps1` Parser errors = 0
2. `local_source_lib.ps1` Parser errors = 0
3. 沒有 git reset / restore / revert / checkout rollback
4. 既有 dirty UI WIP 完整保留
5. AGNES_API_KEY semantics 未改
6. 沒有 secret value 被印出或寫入 repo
7. backend 8000 successfully starts
8. frontend 9422 successfully starts
9. local HTTP smoke passes
10. Docker changed = NO
11. Staging changed = NO
12. Production changed = NO
13. no commit
14. no push

---

# FINAL REPORT FORMAT

只回報：

```text
DM LOCAL DEV STARTUP REPAIR

Repo:
Branch:
HEAD:

Root cause:
- ...

Files modified:
- ...

Parser:
- start_local_dev.ps1: 0 errors
- local_source_lib.ps1: 0 errors

Behavior preserved:
- ports unchanged
- AGNES secret semantics unchanged
- canonical source checks unchanged
- UI WIP preserved

Startup:
- Backend 8000: PASS/FAIL
- Frontend 9422: PASS/FAIL
- HTTP smoke: PASS/FAIL

git reset/restore/revert used: NO
Docker changed: NO
Staging changed: NO
Production changed: NO
Committed: NO
Pushed: NO

Remaining issue:
- none / exact blocker
```
