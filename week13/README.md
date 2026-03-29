# 💬 Week 13：雲端協作整合 — Slack × Make × Canva 圖文自動推播

**雲端與數位內容管理 ｜ 國立金門大學 工業工程與管理學系**

> **本週課程理念：** Discord 是我們整學期的自動化通知平台，
> 但在真實的職場環境，大多數企業用的是 **Slack**——
> 它有更完整的工作流整合、訊息搜尋、App 生態系。
>
> 今天我們學 Slack 的自動化，
> 再加上 Canva 自動生成圖文卡片，
> 讓你的通知不只是文字，而是**有品牌感的視覺化資訊**。
>
> **更重要的是——這是本學期最後一個新工具，
> 下週就是期末整合展示。**

---

## 🎯 本週學習目標

學完本週，你將能夠：
- ✅ 在 Slack 建立 Workspace、頻道，並設定 Incoming Webhook
- ✅ 比較 Slack 和 Discord 的定位差異，知道職場上何時用哪個
- ✅ 用 Make 把 Notion 情報資料自動推播到 Slack 頻道
- ✅ 了解 Canva API 的功能，用 Make HTTP 模組自動生成圖文卡片
- ✅ 把「圖文卡片 + 文字摘要」組合成有品牌感的 Slack 推播

---

## 🏢 本週實作情境：【工管系對外資訊推播中心】

> 💼 **情境：** 系辦決定把「對內協作」和「對外推播」分開管理：
>
> - **Discord**：對內自動化通知（系辦內部、自動化警報）← 前幾週已建立
> - **Slack**：對外圖文推播（系友、合作企業、教師社群）← 本週建立
>
> **今天的任務：**
> 1. 建立 Slack Workspace 和三個主題頻道
> 2. 把 Week 10 情報局的每日摘要推播到 Slack
> 3. 用 Canva 設計一個「週報圖卡模板」
> 4. Make 自動生成圖卡 + 文字推播到 Slack

---

## ⏰ 第 1 小時：Slack 入門 + Webhook 設定 + 第一則推播

> 🎯 **本小時目標：** 從零開始建立 Slack Workspace，
> 設定 Incoming Webhook，並成功推播第一則訊息。

---

### 💡 觀念一：Slack vs Discord — 定位大不同

很多人以為 Slack 和 Discord 是同類工具，
但它們的設計目標完全不同：

| 比較維度 | Discord | Slack |
|---------|---------|-------|
| **主要用戶** | 遊戲社群、興趣社群 | 企業、新創、遠端團隊 |
| **訊息保留** | 免費版無限制 | 免費版只保留 90 天 |
| **Thread（回覆串）** | 有，但使用率低 | 核心功能，廣泛使用 |
| **App 整合數量** | 約 200+ | **2,500+**（Jira、Salesforce、GitHub 等）|
| **語音頻道** | 非常成熟 | 基本功能 |
| **搜尋功能** | 有限 | 非常強大（全文搜尋）|
| **企業採用率** | 低 | **高**（Fortune 500 有 77% 使用）|
| **免費版限制** | 幾乎無限制 | 訊息保留 90 天、App 整合上限 10 個 |
| **Webhook 設定** | 非常簡單 | 需要建立 Slack App |
| **本課程用途** | 自動化通知（內部）| 圖文推播（對外）|

> [!NOTE]
> **什麼時候用 Discord，什麼時候用 Slack？**
>
> - 測試和學習環境 → **Discord**（設定簡單，免費無限制）
> - 企業實習或工作 → **Slack**（這是業界標準）
> - 對內系統警報 → **Discord**（只需要 Webhook URL）
> - 對外品牌推播 → **Slack**（支援 Block Kit 豐富格式）

---

### 💡 觀念二：Slack 的核心概念

| 概念 | 說明 | 對應 Discord |
|------|------|------------|
| **Workspace** | 一個組織的完整空間 | 伺服器（Server）|
| **Channel** | 頻道（公開或私人）| 頻道（Channel）|
| **Thread** | 在訊息下方的回覆串 | Discord 的 Thread |
| **Incoming Webhook** | 接收外部訊息的 URL | Webhook URL |
| **Block Kit** | Slack 的訊息格式系統（支援按鈕、圖片）| 無對應（Discord 用 Embeds）|
| **Slack App** | 整合外部服務的應用程式 | Bot |

---

### 📌 任務 1-1：建立 Slack Workspace

> [!NOTE]
> **課前任務確認：** 你應該已經建立好 Slack 帳號。

1. 前往 [slack.com](https://slack.com/) → 點擊「**Create a new workspace**」
2. 用 Google 帳號登入
3. Workspace 名稱：`工管系 2026 春季`
4. 第一個頻道名稱：`一般`（保留預設即可）
5. 邀請 1-2 位同學（輸入 Email 或跳過）

完成後建立以下三個頻道：

| 頻道名稱 | 說明 | 類型 |
|---------|------|------|
| `#產業情報週報` | 每週自動推播產業動態摘要 | 公開 |
| `#職缺推薦` | AI 優質職缺自動推播 | 公開 |
| `#系統狀態` | Make 劇本執行狀態 | 私人 |

<!-- 📸 截圖：Slack Workspace 建立完成，顯示三個頻道 -->

---

### 📌 任務 1-2：建立 Slack App 並取得 Webhook URL

Slack 的 Webhook 比 Discord 稍微複雜，需要建立一個 App：

1. 前往 [api.slack.com/apps](https://api.slack.com/apps)
2. 點擊「**Create New App**」
3. 選擇「**From scratch**」
4. App Name：`情報推播機器人`
5. 選擇你的 Workspace：`工管系 2026 春季`
6. 點擊「**Create App**」

**啟用 Incoming Webhooks：**

1. 在左側選單選「**Incoming Webhooks**」
2. 把「Activate Incoming Webhooks」切換為「**On**」
3. 點擊頁面最下方「**Add New Webhook to Workspace**」
4. 選擇要推播的頻道：`#產業情報週報`
5. 點擊「**Allow**」
6. 複製出現的 Webhook URL（格式：`https://hooks.slack.com/services/T.../B.../xxx`）
7. 重複步驟 3-6，為 `#職缺推薦` 再建立一個 Webhook

<!-- 📸 截圖：Slack App 的 Incoming Webhooks 設定頁面，顯示兩個 Webhook URL -->

---

### 📌 任務 1-3：用 Hoppscotch 測試 Slack Webhook

在搬到 Make 之前，先在 Hoppscotch 確認 Webhook 格式正確：

1. 方法：**POST**
2. URL：你的 Slack Webhook URL
3. Headers：`Content-Type: application/json`
4. Body（JSON）：
```json
    {
      "text": "🎉 Hello from Hoppscotch！這是工管系情報推播機器人的第一則訊息。"
    }
```

5. 點擊「**Send**」
6. 確認 Slack `#產業情報週報` 頻道收到訊息

<!-- 📸 截圖：Slack 頻道收到測試訊息 -->

---

### 📌 任務 1-4：Slack Block Kit — 豐富格式訊息

Slack 支援「**Block Kit**」格式，讓訊息不只是純文字，
可以包含標題、分隔線、按鈕、圖片等元素：

在 Hoppscotch 測試以下 Block Kit 格式：
```json
{
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": "📊 工管系產業情報週報",
        "emoji": true
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*本週重點情報摘要*\n\n• 台積電宣布在金門設立封測廠，預計創造 500 個就業機會\n• 勞動部修訂學生實習薪資規定，最低時薪調整為 176 元\n• 工業 4.0 導入率調查：台灣製造業 AI 採用率達 43%"
      }
    },
    {
      "type": "divider"
    },
    {
      "type": "section",
      "fields": [
        {
          "type": "mrkdwn",
          "text": "*📅 日期*\n2026/05/05"
        },
        {
          "type": "mrkdwn",
          "text": "*🤖 資料來源*\nParseHub × Gemini AI"
        }
      ]
    },
    {
      "type": "actions",
      "elements": [
        {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "查看完整情報知識庫",
            "emoji": true
          },
          "url": "https://你的Notion頁面連結",
          "style": "primary"
        }
      ]
    }
  ]
}
```

<!-- 📸 截圖：Slack 頻道收到 Block Kit 格式的豐富訊息，包含標題、內文、按鈕 -->

> [!NOTE]
> **Block Kit 的三個最常用區塊：**
>
> | Block 類型 | 說明 | 類比 |
> |-----------|------|------|
> | `header` | 頂部大標題 | HTML 的 `<h1>` |
> | `section` | 正文區塊，支援 Markdown | HTML 的 `<p>` |
> | `actions` | 按鈕區塊 | HTML 的 `<button>` |

> [!TIP]
> **🏆 第 1 小時 Checkpoint 完成！**
>
> - ✅ 建立 Slack Workspace 和三個主題頻道
> - ✅ 建立 Slack App，取得兩個 Webhook URL
> - ✅ 用 Hoppscotch 測試純文字和 Block Kit 格式的推播
> - ✅ 理解 Slack Block Kit 的基本結構

---

### 🎯 第 1 小時學生練習題

> [!NOTE]
> **⏱️ 練習時間：15 分鐘｜個人作業**

---

#### 【練習 1-A】Block Kit 設計（基礎）

設計一則「職缺推薦」的 Block Kit 訊息，
要包含以下資訊：

- Header：`⭐ 本週 AI 優質職缺推薦`
- 職缺名稱（粗體）
- 公司、地點、薪資（三欄 Fields）
- AI 摘要（引用格式，用 `>` 開頭）
- 兩個按鈕：「查看職缺」（連結到 Notion）、「加入追蹤清單」

在 Hoppscotch 測試，截圖 Slack 收到的訊息效果。

> [!TIP]
> 可以用 [Slack Block Kit Builder](https://app.slack.com/block-kit-builder)
> 這個官方工具視覺化地設計你的 Block Kit，
> 設計好後複製 JSON 貼到 Hoppscotch。

---

#### 【練習 1-B】Slack Markdown 語法（進階）

Slack 使用自己的 Markdown 語法（`mrkdwn`），和標準 Markdown 略有不同：

| 效果 | Slack mrkdwn | 標準 Markdown |
|------|-------------|--------------|
| 粗體 | `*文字*` | `**文字**` |
| 斜體 | `_文字_` | `*文字*` |
| 刪除線 | `~文字~` | `~~文字~~` |
| 程式碼 | `` `程式碼` `` | `` `程式碼` `` |
| 超連結 | `<URL\|文字>` | `[文字](URL)` |
| 使用者 @mention | `<@使用者ID>` | 無 |
| 頻道 @mention | `<#頻道ID>` | 無 |

設計一則訊息，使用以上七種語法各至少一次，
在 Hoppscotch 測試並截圖結果。

---

#### 【練習 1-C】觀念問答（思考題）

請回答，貼到課堂共用文件：

> ❓ 如果你在實習的企業，
> 主管要求你用 Slack 整合一套「客戶問題自動分類系統」——
> 客戶透過 Slack 提問，AI 自動分析問題類型，
> 分派給對應的工程師，並追蹤回覆時間。
>
> 參考這學期學過的工具，你會怎麼設計這套系統？
> 請說明每個工具在系統中的角色，並畫出資料流程圖。

---

## ⏰ 第 2 小時：Make 整合 Slack — 把情報局接進 Slack

> 🎯 **本小時目標：** 用 Make 把 Week 10 情報局的每日摘要
> 和 Week 08 的優質職缺，
> 自動推播到對應的 Slack 頻道（Block Kit 格式）。

---

### 📌 任務 2-1：修改情報局每日彙整劇本

**原本（Week 10）：** Discord `#每日彙整` 頻道
**新增（Week 13）：** 同時也推播到 Slack `#產業情報週報`

開啟 Week 10 建立的「每日彙整劇本」：

1. 在最後的 Discord 推播節點之後，
   新增一個「**HTTP → Make a request**」
2. 設定：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | URL | 你的 Slack `#產業情報週報` Webhook URL |
    | Method | `POST` |
    | Body type | `Raw` |
    | Content type | `application/json` |

3. Body（Block Kit 格式）：
```json
    {
      "blocks": [
        {
          "type": "header",
          "text": {
            "type": "plain_text",
            "text": "📰 工管系產業情報日報 — {{formatDate(now; \"MM/DD\")}}"
          }
        },
        {
          "type": "section",
          "text": {
            "type": "mrkdwn",
            "text": "{{ai_daily_summary}}"
          }
        },
        {
          "type": "divider"
        },
        {
          "type": "section",
          "fields": [
            {
              "type": "mrkdwn",
              "text": "*📊 今日情報數*\n{{total_count}} 則"
            },
            {
              "type": "mrkdwn",
              "text": "*🤖 AI 分析引擎*\nGemini 2.0 Flash"
            },
            {
              "type": "mrkdwn",
              "text": "*📅 更新時間*\n{{formatDate(now; \"HH:mm\")}}"
            },
            {
              "type": "mrkdwn",
              "text": "*⚡ 資料來源*\nParseHub 自動採集"
            }
          ]
        },
        {
          "type": "actions",
          "elements": [
            {
              "type": "button",
              "text": {
                "type": "plain_text",
                "text": "🕵️ 查看完整情報知識庫"
              },
              "url": "{{notion_knowledge_base_url}}",
              "style": "primary"
            },
            {
              "type": "button",
              "text": {
                "type": "plain_text",
                "text": "📊 即時儀表板"
              },
              "url": "{{looker_studio_url}}"
            }
          ]
        }
      ]
    }
```

<!-- 📸 截圖：Slack #產業情報週報 收到 Block Kit 格式的日報，包含按鈕 -->

---

### 📌 任務 2-2：修改職缺 AI 分析劇本推播到 Slack

開啟 Week 08 建立的「AI 職缺分析劇本」：

在 Router 的「高相關（ie_score ≥ 4）」支線，
Discord 推播之後，新增一個 Slack 推播節點：
```json
{
  "blocks": [
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "⭐ *{{job_title}}* — {{company_name}}"
      },
      "accessory": {
        "type": "button",
        "text": {
          "type": "plain_text",
          "text": "查看職缺"
        },
        "url": "{{job_url}}",
        "style": "primary"
      }
    },
    {
      "type": "section",
      "fields": [
        {
          "type": "mrkdwn",
          "text": "*📍 地點*\n{{location}}"
        },
        {
          "type": "mrkdwn",
          "text": "*💰 薪資*\n{{salary}}"
        },
        {
          "type": "mrkdwn",
          "text": "*🎯 AI 評分*\n{{ie_score}}/5 ⭐"
        },
        {
          "type": "mrkdwn",
          "text": "*📊 技能等級*\n{{ai_level}}"
        }
      ]
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*🤖 AI 分析：* {{ai_reason}}\n*🔑 核心技能：* {{ai_skills}}"
      }
    },
    {
      "type": "divider"
    }
  ]
}
```

<!-- 📸 截圖：Slack #職缺推薦 收到 Block Kit 格式的職缺推薦訊息 -->

---

### 📌 任務 2-3：建立 Slack 系統狀態通知

把所有 Make 劇本的「系統錯誤」通知，
從 Discord `#系統錯誤警報` 同步到 Slack `#系統狀態`：

**通用錯誤通知模板（Block Kit）：**
```json
{
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": "🚨 系統錯誤警報"
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*錯誤發生時間：* {{formatDate(now; \"YYYY/MM/DD HH:mm\")}}\n*錯誤劇本：* {{scenario_name}}\n*錯誤訊息：* `{{error_message}}`"
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "> ⚠️ 請登入 Make 查看執行記錄並手動處理失敗的資料。"
      }
    }
  ]
}
```

在 Week 07、08、10 的所有錯誤處理節點後，
各自新增這個 Slack 通知。

<!-- 📸 截圖：Make 劇本出現錯誤時，Slack #系統狀態 頻道收到警報 -->

> [!TIP]
> **🏆 第 2 小時 Checkpoint 完成！**
>
> - ✅ 情報局每日彙整同時推播到 Discord 和 Slack（Block Kit 格式）
> - ✅ 優質職缺推薦同時推播到 Discord 和 Slack（含職缺詳情和按鈕）
> - ✅ 系統錯誤通知整合到 Slack `#系統狀態`

---

### 🎯 第 2 小時學生練習題

> [!NOTE]
> **⏱️ 練習時間：20 分鐘｜個人作業**

---

#### 【練習 2-A】週報 Block Kit 設計（基礎）

設計一個「每週職缺市場週報」的 Block Kit，
彙整以下資訊推播到 Slack `#職缺推薦`：

- 本週新增職缺總數
- AI 優質職缺數（評分 ≥ 4）
- 最熱門的三個技能關鍵字
- 平均薪資下限
- 本週最高評分職缺（職缺名稱 + 公司 + 評分）

每週五下午 5:00 自動推播。
截圖 Slack 收到的週報訊息。

---

#### 【練習 2-B】Slack 互動按鈕（進階）

Block Kit 的按鈕除了連結外部網址，
還可以觸發 Slack 的 Interactivity（互動功能）。

但這需要建立更複雜的 Slack App（需要 HTTPS 伺服器接收互動）。

**替代方案：** 使用 Slack 的 Workflow Builder（免費版有限制）
或用「**連結按鈕**」指向你的 Notion 頁面，
讓使用者點擊後在 Notion 完成動作（例如：標記職缺為「感興趣」）。

設計一個「職缺追蹤流程」：
1. Slack 推播職缺，附帶「加入追蹤清單」按鈕
2. 按鈕連結到你的 Notion 頁面（一個職缺追蹤 Database）
3. 使用者手動在 Notion 新增這筆職缺

截圖流程，並說明這個「半自動化」方案的優缺點，
以及「完全自動化」需要什麼額外的技術。

---

#### 【練習 2-C】觀念問答（思考題）

請回答：

> ❓ 這週你同時向 Discord 和 Slack 推播相同的訊息，
> 但用了不同的格式（Discord 用純文字，Slack 用 Block Kit）。
>
> 如果未來你要再加入第三個通知平台（例如 LINE 或 Teams），
> 你現在的 Make 劇本架構需要做什麼修改？
>
> 從「系統設計」的角度，
> 如何設計一套「一次設定，多平台推播」的架構，
> 讓新增一個通知平台只需要改最少的地方？

---

## ⏰ 第 3 小時：Canva 圖文自動化 + 整學期系統整合回顧

> 🎯 **本小時目標：** 了解 Canva 的自動化能力，
> 用 Make + Canva 自動生成帶資料的圖文卡片，
> 並整合推播到 Slack。
> 最後進行整學期系統的完整回顧，為期末發表做準備。

---

### 💡 觀念：Canva 的自動化能力

Canva 不只是設計工具，它提供兩種自動化方式：

**方式一：Canva Bulk Create（批次建立）**
- 上傳 CSV 資料，自動把資料填入設計模板
- 一次生成多份個人化設計（例如：100 份員工識別證）
- **免費功能，不需要 API**

**方式二：Canva Connect API**
- 程式化操作 Canva，自動生成設計並下載
- 需要申請 API 存取權限（目前僅開放給企業合作夥伴）
- **本課程不使用（需要審核才能申請）**

> [!NOTE]
> **本週使用 Canva Bulk Create（完全免費）：**
> 我們設計一個「週報圖卡模板」，
> 用 Bulk Create 把本週的情報資料填入，
> 下載後上傳到 Slack。
> 這個步驟目前需要手動下載再上傳，
> 但你會理解「資料驅動設計」的概念。

---

### 📌 任務 3-1：設計 Canva 週報圖卡模板

1. 前往 [canva.com](https://www.canva.com/) 登入
2. 點擊「**建立設計**」→ 選擇「**Instagram 貼文（1:1）**」（1080×1080 px）
3. 設計以下版面（參考配置）：
```
    ┌─────────────────────────┐
    │  工管系產業情報週報       │  ← 標題（白色文字）
    │  2026 / {{週次}}         │
    ├─────────────────────────┤
    │                         │
    │  📊 本週重點              │  ← 主體
    │                         │
    │  {{重點一}}              │
    │  {{重點二}}              │
    │  {{重點三}}              │
    │                         │
    ├─────────────────────────┤
    │  職缺: {{職缺數}} 則      │  ← 數據區
    │  優質: {{優質數}} 則      │
    │  平均薪資: {{薪資}} 元    │
    ├─────────────────────────┤
    │  國立金門大學 工管系      │  ← 頁尾
    │  Powered by AI × Make   │
    └─────────────────────────┘
```

4. 設計風格：
    - 背景：深藍色漸層（和 GitHub 網站配色一致）
    - 標題字體：粗體、白色
    - 數據區：橙色強調色
    - 整體：科技感、簡潔

5. 把所有**動態文字**（例如「{{重點一}}」）設定為 Canva 的「**Bulk Create 變數**」：
    - 選取文字 → 右鍵 → 「**連結至資料**」→ 輸入欄位名稱

<!-- 📸 截圖：Canva 設計完成的週報圖卡模板，顯示動態變數欄位 -->

---

### 📌 任務 3-2：用 Bulk Create 填入資料

1. 點擊左側工具列「**Apps**」→ 搜尋「**Bulk Create**」
2. 選擇「**手動輸入資料**」或「**上傳 CSV**」
3. 填入以下測試資料：

    | 欄位名稱 | 值 |
    |---------|---|
    | 週次 | W19 |
    | 重點一 | 台積電擴大工管人才需求，開放 20 個實習名額 |
    | 重點二 | 勞動部修訂學生實習補助，金額提升至每月 5,000 元 |
    | 重點三 | AI 導入製造業調查：台灣製造業 AI 採用率達 43% |
    | 職缺數 | 47 |
    | 優質數 | 12 |
    | 薪資 | 36,500 |

4. 點擊「**生成設計**」，Canva 自動把資料填入模板
5. 下載為 PNG 格式

<!-- 📸 截圖：Canva Bulk Create 介面，顯示資料填入後的預覽圖 -->
<!-- 📸 截圖：下載完成的週報圖卡（PNG），資料正確填入 -->

---

### 📌 任務 3-3：把圖卡上傳到 Slack

目前這個步驟需要手動操作（因為 Canva Connect API 還未開放），
但可以整合在 Make 的工作流程中：

**半自動化方案（本課程使用）：**
```
Make 每週五 5:00
    ↓
從 Google Sheets 取得本週統計數字
    ↓
把數字整理成 CSV 格式（Tools → Create CSV）
    ↓
推播 Slack 訊息：
「本週 Canva 圖卡資料已準備好，請到以下連結下載 CSV：
 [Google Sheets 連結]
 完成設計後上傳到此頻道。」
    ↓
設計師手動操作 Canva Bulk Create
    ↓
手動上傳圖卡到 Slack
```

**Make 的 Slack 推播設定（提醒設計師下載資料）：**
```json
{
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": "🎨 本週圖卡設計提醒"
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "本週週報圖卡的資料已自動準備完成，請按以下步驟操作：\n\n1️⃣ 點擊下方按鈕下載本週資料 CSV\n2️⃣ 在 Canva 使用 Bulk Create 填入資料\n3️⃣ 下載 PNG 後上傳到此頻道"
      }
    },
    {
      "type": "section",
      "fields": [
        {
          "type": "mrkdwn",
          "text": "*本週職缺數*\n{{job_count}} 則"
        },
        {
          "type": "mrkdwn",
          "text": "*優質職缺*\n{{quality_count}} 則"
        },
        {
          "type": "mrkdwn",
          "text": "*平均薪資*\nNT$ {{avg_salary}}"
        },
        {
          "type": "mrkdwn",
          "text": "*本週情報*\n{{intel_count}} 筆"
        }
      ]
    },
    {
      "type": "actions",
      "elements": [
        {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "📥 下載本週資料"
          },
          "url": "{{google_sheets_url}}",
          "style": "primary"
        },
        {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "🎨 開啟 Canva 模板"
          },
          "url": "{{canva_template_url}}"
        }
      ]
    }
  ]
}
```

<!-- 📸 截圖：Slack 收到圖卡設計提醒，包含數據和兩個按鈕 -->

---

### 📌 任務 3-4：整學期系統整合回顧

這是本學期最後一個新工具課，
花 15 分鐘做一次完整的系統回顧，
確認你的期末展示系統都正常運作。

**完整系統檢查清單：**

| 系統元件 | 狀態檢查 | 問題記錄 |
|---------|---------|---------|
| Make 爬蟲劇本（Week 07）| 手動執行一次，確認 ParseHub 正常 | |
| Make AI 分析劇本（Week 08）| 確認 Gemini 分析正常回傳 JSON | |
| Make 情報局劇本（Week 10）| 確認 Notion API 可以建立頁面 | |
| Make 截止日提醒劇本（Week 12）| 確認 Notion Query API 正常 | |
| Notion 情報知識庫（Week 10）| 確認有最新情報資料 | |
| Notion 專題管理中心（Week 12）| 確認任務狀態是最新的 | |
| GitHub 展示網站（Week 11）| 確認 Vercel 部署正常，網址可以開啟 | |
| Looker Studio 儀表板（Week 09）| 確認資料有更新 | |
| Discord 各頻道（Week 01-12）| 確認各頻道功能正常 | |
| Slack 各頻道（Week 13）| 確認 Webhook 正常 | |

**系統整合架構圖（最終版）：**
```
【資料採集層】
ParseHub（週一自動爬蟲）
    └→ Google Sheets raw_data

【AI 分析層】
Gemini 2.0 Flash
    ├→ 職缺分析（等級/評分/技能）
    └→ 情報摘要（分類/重要程度/關鍵字）

【自動化核心】
Make（十多個劇本）
    ├→ 資料清洗 → cleaned_data
    ├→ 結果回填 → analyzed_data
    └→ 知識庫建立 → Notion Database

【知識庫層】
Notion
    ├→ 情報知識庫（Week 10）
    └→ 專題管理中心（Week 12）

【視覺化層】
Looker Studio
    └→ 嵌入 GitHub 網站 / Notion

【展示層】
GitHub + Vercel
    └→ 對外展示網站（期末用）

【通知層】
Discord（內部）+ Slack（對外）
    ├→ 緊急情報 / 系統錯誤
    ├→ 截止日提醒 / 任務完成
    └→ 每日情報 / 職缺推薦 / 週報圖卡
```

<!-- 📸 截圖：Slack 週報頻道的完整訊息流，顯示多週的自動推播記錄 -->

> [!TIP]
> **🏆 第 3 小時 Checkpoint 完成！**
>
> - ✅ 設計 Canva 週報圖卡模板（含動態變數）
> - ✅ 用 Bulk Create 填入真實資料，下載 PNG 圖卡
> - ✅ Make 劇本自動提醒設計師下載資料和開啟 Canva 模板
> - ✅ 完成整學期系統整合回顧，確認所有元件正常運作

---

### 🎯 第 3 小時學生練習題

> [!NOTE]
> **⏱️ 練習時間：20 分鐘｜個人 + 分組**

---

#### 【練習 3-A】Canva 圖卡第二版（個人，基礎）

修改你的 Canva 週報圖卡模板，改成「職缺推薦卡」：

設計一個 **Instagram Story 尺寸（9:16）** 的職缺推薦卡，
包含以下動態欄位：
- 職缺名稱（大標題）
- 公司名稱
- 地點和薪資
- AI 核心技能（三個標籤）
- AI 評分（用星星視覺化）
- 底部：「工管系職缺情報站」品牌標識

完成設計後，用三筆假的職缺資料做 Bulk Create，
截圖三張不同的職缺推薦卡。

---

#### 【練習 3-B】期末整合挑戰（分組，2～3 人）

**完成你們期末專題系統的最後整合：**

確認以下所有元件都已完成並正常運作：

1. **GitHub 展示網站**：
    - `index.html`（首頁）已有真實的系統介紹
    - `project.html`（期末專題）已填入真實內容
    - 嵌入 Looker Studio 儀表板
    - Vercel 部署正常，網址可以分享

2. **Notion 系統**：
    - 情報知識庫有最新資料
    - 專題管理中心的任務狀態是最新的

3. **自動化系統**：
    - 至少三個 Make 劇本正常運作
    - Discord 和 Slack 都能收到推播

4. **視覺化**：
    - Looker Studio 儀表板有真實資料

5. **Canva 圖卡**：
    - 設計一張你們組的「系統成果海報」
    - 用 Bulk Create 填入真實的成果數據
    - 上傳到 Slack `#職缺推薦` 頻道

繳交：Vercel 網址（期末展示用）

---

#### 【練習 3-C】期末發表準備（重要）

> ❓ **期末發表（Week 14）距離現在只剩一週，請完成以下準備：**
>
> **報告時間：10 分鐘（含換場）**
>
> **建議結構（可調整）：**
> - 0:00-2:00：問題背景（Before：沒有這套系統時的痛點）
> - 2:00-5:00：系統架構說明（工具地圖 + 資料流程圖）
> - 5:00-8:00：Live Demo（最精彩的三個功能，現場展示）
> - 8:00-9:30：成果數據（量化成效）
> - 9:30-10:00：學習反思（最大的挑戰 + 未來改進方向）
>
> **Live Demo 建議（選最穩定的功能）：**
> 1. 填寫表單 → Discord 即時通知（Week 01-02）
> 2. Notion 情報知識庫（展示 AI 自動填入的情報）
> 3. GitHub 網站（展示 Vercel 部署的成果頁面）
>
> **請回答以下問題並貼到課堂共用文件：**
> 1. 你們的 Live Demo 選擇展示哪三個功能？為什麼？
> 2. 成果數據有哪些？（例如：系統節省了多少時間？處理了多少筆資料？）
> 3. 如果再給你們一個月，你們最想新增什麼功能？

---

## 🏆 本週總結

### 📊 本週技能地圖
```
【Slack 設定】
    ├── Workspace 和頻道建立
    ├── Slack App + Incoming Webhook 設定
    ├── Block Kit：header、section、fields、actions
    └── Slack mrkdwn 語法

        ↓ 設定好推播管道

【Make 整合 Slack】
    ├── 情報局每日摘要 → Slack Block Kit
    ├── AI 優質職缺推薦 → Slack Block Kit（含按鈕）
    └── 系統錯誤警報 → Slack 系統狀態頻道

        ↓ 豐富視覺化輸出

【Canva 圖文自動化】
    ├── Bulk Create：用 CSV 批次生成個人化設計
    ├── 動態變數設定：連結資料到設計元素
    ├── 週報圖卡模板設計（1:1 尺寸）
    └── 半自動化流程：Make 提醒 → 手動 Canva → 上傳 Slack

        ↓ 整學期整合

【系統回顧】
    └── 十多個 Make 劇本 × 六個資料層 × 兩個通知平台
```

### 📊 整學期工具使用統計

| 工具 | 首次學習 | 本學期使用次數 | 主要角色 |
|------|---------|-------------|---------|
| Google Workspace | 暖身 | 全學期 | 資料骨幹 |
| Make | Week 01 | 13 週 | 自動化核心 |
| Discord | Week 01 | 12 週 | 內部通知 |
| Gemini API | Week 08 | 3 週 | AI 分析引擎 |
| Notion | Week 04 | 4 週 | 知識庫 + 架站 |
| ParseHub | Week 07 | 2 週 | 資料採集 |
| Looker Studio | Week 01 | 3 週 | 視覺化 |
| GitHub + Vercel | Week 11 | 2 週 | 展示平台 |
| Slack | Week 13 | 1 週 | 對外推播 |
| Canva | Week 13 | 1 週 | 圖文設計 |

### 🤔 課後思考問題

1. **工具選擇的哲學：** 這學期你學了十多個工具，
   但在真實工作中，你不可能同時用這麼多工具。
   如果只能保留三個工具（除了 Google Workspace），
   你會選哪三個？理由是什麼？

2. **自動化的邊界：** 你建立的系統中，
   哪些部分「應該」維持人工操作，而不應該自動化？
   「完全自動化」在什麼情況下反而是壞事？

3. **技能的可轉移性：** 假設三年後 Make 停止服務，
   你的所有 Make 劇本都失效了。
   但你學到的「自動化思維」還在。
   請說明這個思維如何幫助你快速遷移到其他平台（例如 n8n 或 Zapier）。

### 📚 本週自學資源

| 資源 | 用途 |
|------|------|
| [Slack Block Kit Builder](https://app.slack.com/block-kit-builder) | 視覺化設計 Block Kit 訊息 |
| [Slack API 文件](https://api.slack.com/messaging/composing/layouts) | Block Kit 完整語法說明 |
| [Canva Bulk Create 教學](https://www.canva.com/help/bulk-create/) | 官方 Bulk Create 說明 |
| [n8n（Make 的開源替代品）](https://n8n.io/) | 如果未來 Make 變貴，這是替代方案 |

---

### 📝 下週預告

> 🔜 **Week 14：期末整合專題 — 成果發表與系統展示**
>
> **這不是一堂課，而是一場發表會。**
>
> 每組 10 分鐘，展示這學期建立的完整自動化系統：
> - 2 分鐘：問題背景
> - 3 分鐘：系統架構和工具說明
> - 3 分鐘：Live Demo
> - 2 分鐘：成果數據和反思
>
> **期末當天需要準備：**
> 1. GitHub 展示網站的 Vercel 網址（投影在大螢幕）
> 2. 系統架構圖（在 GitHub 網站或 Notion 頁面）
> 3. Live Demo 的系統已啟動並正常運作
> 4. 備用方案（萬一 Demo 失敗的截圖或錄影）
>
> **評分標準：**
>
> | 項目 | 比重 |
> |------|------|
> | 系統整合完整度（用了幾個工具、幾個資料層）| 30% |
> | 自動化程度（有多少步驟不需要人工介入）| 25% |
> | 實際解決的問題（有沒有真實的應用價值）| 25% |
> | 展示呈現（網站品質、說明清晰度）| 20% |
>
> **加油！你們已經完成了 13 週的學習，期末發表是展示成果的時刻。**

---

*雲端與數位內容管理 ｜ 國立金門大學 工業工程與管理學系 ｜ 2026 春季學期*
*Week 13 of 14*
