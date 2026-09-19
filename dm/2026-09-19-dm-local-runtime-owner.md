# DM LOCAL RUNTIME OWNER

## Mission
只處理 Ticenpi DM Windows 本機 9421 / 9422 runtime 問題。

非常重要：
Windows localhost 與 VPS localhost 完全不同。
本任務不要把兩者混在一起。

已知：
Windows 127.0.0.1:9421 = OPEN
/api/health:
- environment = local
- release = dev
- databaseTarget = disposable

Windows 127.0.0.1:9422 = CLOSED

使用者指出：
- 原本 Local / Local Docker 頁面正常
- 現在 9421 與預期頁面不同
- 9422 無法開啟

## Goal
回答：
1. Windows 9421 現在到底是哪個 process/container？
2. Windows 9422 原本應該是哪個 service？
3. 為什麼 9422 現在沒有 listener？
4. Local Dev 與 Local Docker 正確 port 設計是什麼？
5. 為什麼 UI 不同？

## Tasks

### 1. Windows Listener
只在本機查：
- 9421
- 9422
- 8000
- 其他 DM 相關 ports

取得：
PID / process / Docker container / WSL process / port mapping

### 2. Local Docker
查：
- docker ps
- docker compose projects
- WSL docker（若存在）

找出 DM：
- container names
- image
- image ID
- compose source path
- port mappings
- environment
- runtime-config

### 3. Local Dev
找出是否另有：
- npm dev
- vite
- next
- python backend
- launcher/dev script

確認各自應使用哪個 port。

### 4. Source Identity
對目前 Local Docker / Local Dev 找：
- repo/worktree
- HEAD SHA
- working tree state
- frontend assets
- runtime-config

回答 UI 來源是：
- CURRENT CANONICAL SOURCE
- STALE IMAGE
- OLD WORKTREE
- DEV SERVER
- OTHER

不能猜 cache。

### 5. 9422 Root Cause
精確判定：
A. 本來就不該存在
B. compose 沒啟動
C. container stopped
D. port mapping 改掉
E. dev server 沒啟動
F. 舊環境殘留認知
G. 其他

必須有 evidence。

### 6. 若 root cause 清楚
只允許修 Local：
- local compose/dev start
- 正確 local service
- local port mapping

禁止：
- VPS
- Production
- Cloudflare
- Tailscale
- Production Docker

## Final Output
WINDOWS_9421 =
...

WINDOWS_9421_OWNER =
...

WINDOWS_9421_SOURCE =
...

WINDOWS_9422 =
OPEN / CLOSED

EXPECTED_9422_OWNER =
...

9422_ROOT_CAUSE_PROVEN =
YES / NO

9422_ROOT_CAUSE =
...

LOCAL_DEV_PORT =
...

LOCAL_DOCKER_PORT =
...

LOCAL_DEV_VS_DOCKER_DESIGN =
...

UI_DIFFERENCE_ROOT_CAUSE =
...

LOCAL_FIX_APPLIED =
YES / NO

PRODUCTION_TOUCHED =
NO

LOCAL_RUNTIME_STATUS =
PASS / BLOCKED

完成後停止。
