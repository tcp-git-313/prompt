# FB 社團功能設計

## 1. 目標

本設計針對 TicenpiPost 的 Facebook 社團功能，目標是把「已加入社團清單、搜尋新社團、AI 分類、選擇加入、加入終態追蹤」整合成一個直覺流程。

核心原則：

- 使用者盡量不需要跳到 Facebook 手動找社團。
- Facebook 真實搜尋結果才是資料來源，AI 只做分類、排序與推薦。
- 不在前端嵌入 Facebook 搜尋框；由專案內搜尋框控制後端 Browser Worker 執行真實 Facebook 搜尋。
- 加入社團屬於寫入行為，必須排隊、限速、終態追蹤，不可一次大量連續操作。
- 遇到登入、Checkpoint、驗證、加入問題等情況要停下並回報，不可猜測或硬繞。

---

## 2. 既有基礎

目前 TicenpiPost 已具備：

1. 可由指定 Facebook 帳號同步「已加入社團」。
2. 已同步社團可取得真實社團名稱與 URL。
3. 前端 GroupSelector 已能載入與選擇社團。
4. 後端已有 Playwright Worker、帳號鎖、Queue 與帳號狀態機制。

因此新功能不需要重做「已加入社團清單」，而是在既有功能上增加「探索新社團」與「加入社團」。

---

## 3. 前端資訊架構

社團管理建議分成兩個 Tab：

### 3.1 已加入社團

顯示：

- 社團名稱
- 社團 URL
- 分類標籤
- 是否可發文
- 最近同步時間
- 搜尋／篩選
- 使用者自訂分組

保留：

- 從 Facebook 重新同步
- 快取載入
- 手動新增 URL 作為 fallback

### 3.2 探索新社團

介面示意：

~~~text
社團管理

[ 已加入社團 10 ] [ 探索新社團 ]

搜尋 Facebook 社團
[ 淡水 房屋 買賣                        ] [搜尋]

結果：

☐ 淡水買屋賣屋交流區
  公開社團 / 8.2萬成員 / 尚未加入

☐ 淡水竹圍房屋交流
  私人社團 / 1.6萬成員 / 需審核

✓ 淡水房仲交流平台
  已加入

[加入已選社團]
~~~

---

## 4. Facebook 社團搜尋

### 4.1 使用者流程

~~~text
專案內輸入搜尋詞
↓
POST /api/groups/search
↓
建立背景任務
↓
T2 / Browser Worker 使用該帳號 Cookie
↓
打開 Facebook 社團搜尋
↓
擷取真實搜尋結果
↓
回傳前端
~~~

### 4.2 搜尋結果資料

每筆至少包含：

~~~text
group_id
name
url
member_count
privacy
join_status
source_account_id
searched_at
~~~

join_status 建議：

- JOINED
- NOT_JOINED
- REQUEST_PENDING
- QUESTIONS_REQUIRED
- UNAVAILABLE
- UNKNOWN

不要只存社團名稱；URL 必須是主要識別資料之一。

---

## 5. Site / DOM 操作原則

Facebook DOM 可能改版，因此：

- 優先 accessible role、可見文字、穩定語意。
- 使用候選鏈，不依賴單一亂碼 CSS class。
- 選擇器外部化。
- 每次搜尋／加入後做終態確認。
- 搜尋結果與加入結果都保存必要 evidence。

搜尋失敗時要區分：

- SELECTOR_DRIFT
- LOGIN_REQUIRED
- CHECKPOINT
- RATE_LIMITED
- EMPTY_RESULT
- NETWORK_ERROR

不可全部包成「搜尋失敗」。

---

## 6. AI 的角色

AI 不應自己生成一份「可能存在的 Facebook 社團清單」。

正確流程：

~~~text
Facebook 真實搜尋結果
↓
AI 分類 / 排序 / 標籤
↓
使用者選擇
~~~

AI 可做：

- 房地產高度相關
- 地區生活
- 買賣交流
- 租屋
- 投資
- 求職
- 二手商品
- 明顯不相關

也可給：

- relevance_score
- category
- reason

但 Facebook URL、社團名稱、加入狀態必須來自實際搜尋，不可由 AI 虛構。

---

## 7. 加入社團流程

不要搜尋完就直接全部加入。

正確流程：

~~~text
使用者勾選
↓
POST /api/groups/join
↓
每個社團建立獨立 Queue Job
↓
同一 Facebook 帳號串行或依風險策略執行
↓
打開社團
↓
執行加入
↓
判斷終態
↓
更新 UI
~~~

### 7.1 加入終態

至少：

- JOINED
- REQUESTED
- QUESTIONS_REQUIRED
- ALREADY_JOINED
- REJECTED_OR_UNAVAILABLE
- CHECKPOINT
- FAILED
- UNKNOWN

HTTP 202 / job enqueue 不代表加入成功。

---

## 8. 加入問題

如果 Facebook 社團要求回答入社問題：

不要讓 AI 自動亂填。

建議：

~~~text
QUESTIONS_REQUIRED
↓
把問題帶回 TicenpiPost UI
↓
使用者填寫答案
↓
重新排入加入任務
~~~

可以提供：

- 儲存常用答案模板
- AI 協助草擬文字

但最終答案應由使用者確認。

---

## 9. 帳號風險控制

加入社團屬於 Facebook 寫入動作，必須沿用 TicenpiPost 帳號安全機制。

至少：

- Account Lock
- 每日行為預算
- 冷啟限制
- Checkpoint Detection
- Circuit Breaker
- 延遲與分散執行
- 不在工程壓測時大量操作真實帳號

不能設計：

「使用者選 50 個 → 立刻連按 50 次加入」。

---

## 10. API 建議

### 搜尋

POST /api/groups/search

輸入：

~~~json
{
  "account_id": "...",
  "query": "淡水 房屋"
}
~~~

回 202 + run_id。

### 搜尋結果

GET /api/groups/search/{run_id}

### 加入

POST /api/groups/join

~~~json
{
  "account_id": "...",
  "group_urls": [
    "https://www.facebook.com/groups/..."
  ]
}
~~~

### 加入工作狀態

GET /api/groups/join/{run_id}

---

## 11. 狀態資料模型

建議至少：

### GroupDiscoveryResult

~~~text
id
account_id
query
group_id
name
url
member_count
privacy
join_status
ai_category
ai_relevance_score
searched_at
~~~

### GroupJoinTask

~~~text
id
account_id
group_url
status
questions[]
answers[]
attempt_count
last_error
evidence
created_at
updated_at
~~~

---

## 12. UI 建議

### 已加入

- 搜尋
- 標籤
- 分組
- 重新同步
- 最近同步

### 探索

- 搜尋框
- 結果列表
- AI 類別
- 已加入標記
- 多選
- 加入已選

### 加入任務

顯示：

~~~text
淡水買屋賣屋交流區
已送出申請

竹圍生活大小事
需要回答 3 個問題
[處理]

淡水投資房產
已加入
~~~

---

## 13. 與現有 GroupSelector 的關係

不要把 GroupSelector 改成搜尋新社團的全部功能。

建議：

- GroupSelector：負責「這次發文要選哪些已加入社團」
- Group Management：負責同步、探索、加入、分類
- GroupSelector 從 Group Management 的已加入資料來源讀取

這樣發文 UI 不會變得過度複雜。

---

## 14. 執行順序

Phase 1：
- 整理已加入社團的資料模型與快取
- 加上探索頁 UI

Phase 2：
- fb_group_search driver
- 搜尋結果 API
- 真實帳號唯讀驗證

Phase 3：
- AI 分類與排序

Phase 4：
- fb_group_join driver
- Queue
- 加入終態

Phase 5：
- 入社問題回填 UI
- Retry / Review Queue

Phase 6：
- 風險控制與 Regression

---

## 15. 驗收標準

1. 可在 TicenpiPost 內輸入搜尋詞並取得 Facebook 真實社團結果。
2. 搜尋結果包含真實 URL，不以名稱作唯一識別。
3. 已加入社團能正確標示。
4. AI 只能分類真實結果，不生成虛構社團。
5. 使用者可多選後加入。
6. 加入任務為 Background Queue。
7. 可辨識已加入／申請中／需回答問題／Checkpoint／失敗。
8. 不把 enqueue 成功視為加入成功。
9. 不大量並行操作同一 Facebook 帳號。
10. 發文用 GroupSelector 與社團管理功能保持分離。

---

## 16. 核心結論

最終體驗：

~~~text
搜尋 Facebook 社團
↓
看到真實結果
↓
AI 幫忙分類
↓
使用者勾選
↓
系統排隊加入
↓
終態追蹤
↓
加入成功後自動出現在發文 GroupSelector
~~~

Facebook 是資料來源；
AI 是整理工具；
Queue 與終態驗證負責可靠性；
帳號風險控制優先於大量自動化。
