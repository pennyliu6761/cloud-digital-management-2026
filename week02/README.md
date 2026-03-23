## ⏰ 第 2 小時：API 串接初體驗 — Make 視覺化工作流 × Discord 推播警報

> 🎯 **本小時目標：** 讓系統主動通知人，而不是讓人主動去看系統。
> 理解 API 與 Webhook 的運作邏輯，完成第一個「條件觸發式」自動化通知。

> [!WARNING]
> **📢 工具異動說明**
> LINE Notify 已於 2025 年 3 月 31 日正式停止服務。
> 本課程改用 **Discord Webhook** 作為推播通知工具。
> 沒有 Discord 帳號的同學，可使用本節末尾的「備案：Gmail 通知」完成相同練習。

---

### 💡 觀念講解：API、Webhook 與 Make 是什麼？

#### API 是什麼？

**API（Application Programming Interface）** 是讓不同軟體互相溝通的橋樑。
```
你（軟體 A）  →  API（服務生）  →  Discord 伺服器（廚房）
   下訂單         傳遞需求              執行出餐
```

#### Webhook vs 傳統 API 輪詢

| 機制 | 運作方式 | 比喻 |
|------|---------|------|
| **傳統輪詢** | 每隔 5 分鐘主動問「有新資料嗎？」 | 你一直打電話問餐廳「好了嗎？」 |
| **Webhook 推送** | 有新資料時，系統主動通知你 | 餐廳好了主動打電話叫你去取 |

#### Discord Webhook 是最純粹的 Webhook 範例

> 你只需要一個 **URL**，任何程式只要對這個 URL 送出資料，訊息就會出現在 Discord 頻道。
> **URL 本身就是那把鑰匙** — 沒有帳號驗證、沒有 Token 管理，這是 Webhook 概念最直觀的體現。

#### Make 是什麼？

Make 把複雜的 API 程式碼變成**視覺化拖拉節點**，讓不懂程式的人也能串接任何系統。
```
[ Google Sheets ] ──→ [ Filter 過濾器 ] ──→ [ Discord Webhook ]
   監控新報名            只讓 VIP 通過           發送警報訊息
```

---

### 📌 任務 2-1：建立 Discord 伺服器與取得 Webhook URL

**工具：** Discord（完全免費，無需信用卡）

#### 步驟一：建立 Discord 伺服器

1. 前往 [discord.com](https://discord.com) 或開啟 Discord App，登入帳號
2. 點擊左側伺服器列表最下方「**+**」→「**新增一個伺服器**」→「**建立自己的**」→「**我和我的好友**」
3. 伺服器名稱輸入：`金門聚落文化營 戰情指揮部`
4. 點擊「**建立**」

<img width="376" height="349" alt="image" src="https://github.com/user-attachments/assets/c6ff117d-0613-413f-ad42-d961562da491" />
<img width="434" height="544" alt="image" src="https://github.com/user-attachments/assets/9df92433-89d5-44e5-a358-baaaeb974891" />
<img width="433" height="367" alt="image" src="https://github.com/user-attachments/assets/c16a9730-5264-4a13-86fa-556b5866f954" />
<img width="438" height="405" alt="image" src="https://github.com/user-attachments/assets/3fdd725f-692b-495c-99f7-856ec08267b3" />

---

#### 步驟二：建立專屬通知頻道

1. 左側頻道列表，點擊「**+**」→ 選擇「**文字頻道**」
2. 頻道名稱輸入：`vip-報名警報`
3. 點擊「**建立頻道**」

<img width="478" height="524" alt="image" src="https://github.com/user-attachments/assets/195155a3-6439-43e6-935c-ce2e93c94246" />
<img width="747" height="606" alt="image" src="https://github.com/user-attachments/assets/695fb1e0-af36-4ae1-b4ef-df3d9b7acba1" />

---

#### 步驟三：取得 Webhook URL（核心步驟）

1. 對 `vip-報名警報` 頻道點擊右側「**⚙️ 編輯頻道**」
2. 左側選「**整合**」→「**Webhook**」→「**建立 Webhook**」
3. 設定如下：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | 名稱 | `營隊戰情機器人` |
    | 頻道 | `vip-報名警報` |

4. 點擊「**複製 Webhook 網址**」

> [!WARNING]
> 立即將這串 URL 貼到記事本備用！格式如下：
> ```
> https://discord.com/api/webhooks/1234567890/xxxxxxxxxxxxxxxxxxxxxxxx
> ```

<img width="447" height="266" alt="image" src="https://github.com/user-attachments/assets/3865758e-7024-46b8-8e55-c3f943a98117" />

> [!TIP]
> **這個 URL 就是你的「推播管道入口」**
> 任何人只要能對這個 URL 發送 HTTP POST 請求，訊息就會出現在頻道。
> Webhook 的核心概念：**用 URL 取代複雜的身份驗證流程**。

---

### 📌 任務 2-2：建立 Make 帳號與新劇本

**工具：** [Make.com](https://www.make.com/)（免費方案每月 1,000 次操作）

1. 前往 Make.com，點擊「**Get started free**」
2. 用 Google 帳號登入（不需要信用卡）
3. 點擊左側「**Scenarios**」→ 右上角「**+ Create a new scenario**」

<!-- 📸 截圖：Make 空白畫布 -->

> [!NOTE]
> **Make 的核心概念**
> - 每個劇本（Scenario）是一條自動化流水線，由左到右依序執行
> - 每個圓圈節點（Module）代表一個動作
> - 資料像水一樣從左邊流向右邊

---

### 📌 任務 2-3：串接 Google Sheets → Discord 自動警報

#### 步驟一：設定觸發器（Google Sheets 監控新報名）

1. 點擊畫布中央「**+**」→ 搜尋並選擇「**Google Sheets**」
2. 選擇觸發動作：「**Watch Rows**」
3. 點擊「**Create a connection**」→ 授權你的 Google 帳號
4. 設定如下：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | Spreadsheet | `金門聚落文化營_報名總表` |
    | Sheet | `表單回覆 1` |
    | Limit | `2` |

5. 點擊「**OK**」→「**Choose manually**」→ 選擇最後一筆現有資料 → 儲存

<!-- 📸 截圖：Google Sheets Watch Rows 節點設定完成 -->

---

#### 步驟二：設定過濾器（只有 VIP 才發警報）

1. 滑鼠移到 Google Sheets 節點右邊，**右鍵點擊連接線**
2. 選擇「**Add a filter**」
3. 設定過濾條件：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | Label | `僅限 VIP 警報` |
    | Condition | `是否為VIP貴賓？` → `Equal to (text)` → `是` |

<!-- 📸 截圖：Filter 設定畫面 -->

> [!NOTE]
> **這就是商業邏輯！**
> 不需要寫 `if (vip === "是") { sendAlert() }`，
> 用拖拉的方式就完成了條件判斷。

---

#### 步驟三：設定動作（Discord Webhook 推播）

1. 點擊過濾器右邊「**+**」
2. 搜尋「**Discord**」→ 選擇「**Send a Message by Webhook Bot**」

> [!NOTE]
> 找不到 Discord 模組？可改用通用方式：
> 搜尋「**HTTP**」→「**Make a request**」→ Method 選 `POST`，URL 貼入 Webhook URL，Body type 選 `Raw`，Content type 選 `JSON`，Request content 填入訊息 JSON。效果完全相同，且適用所有支援 Webhook 的服務。

3. 設定如下：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | Webhook URL | 貼上任務 2-1 複製的 Discord Webhook URL |
    | Username | `營隊戰情機器人` |

4. **Content 欄位**組裝以下訊息（`{{ }}` 從左邊 Google Sheets 帶入動態變數）：
```
    🚨 **【VIP 報名警報】** 🚨

    有貴賓剛剛完成報名，請相關人員注意！

    ▸ **姓名：** {{姓名}}
    ▸ **場次：** {{參加場次}}
    ▸ **信箱：** {{Email信箱}}
    ▸ **電話：** {{聯絡電話}}
    ▸ **報名時間：** {{Timestamp}}

    請於 30 分鐘內完成接待確認。
```

> [!TIP]
> Discord 支援 Markdown 語法，`**文字**` 顯示為粗體，讓警報訊息排版更清晰易讀。

<!-- 📸 截圖：Make 劇本完整畫布（Google Sheets + Filter + Discord 三個節點） -->

---

#### 步驟四：壓力測試！

1. 點擊 Make 畫布左下角「**Run once**」
2. Google Sheets 節點出現轉圈雷達 → **系統正在監聽**
3. 用另一個瀏覽器填寫報名表，「是否為 VIP 貴賓？」**勾選「是」** 並送出
4. 回到 Make，節點上的數字從 0 變成 1
5. 打開 Discord `vip-報名警報` 頻道——

**🔔 機器人的警報訊息到了！**

<!-- 📸 截圖：Discord 頻道收到警報訊息的畫面 -->

---

### 📌 備案：改用 Gmail 通知（無 Discord 帳號適用）

將步驟三的動作模組換成 Gmail：

1. 搜尋「**Gmail**」→「**Send an Email**」
2. 授權 Google 帳號後設定：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | To | 你自己或團隊負責人的 Email |
    | Subject | `🚨【VIP 報名警報】{{姓名}} 剛剛完成報名` |
    | Content | 同 Discord 的訊息內容 |

> [!NOTE]
> **Gmail 與 Discord Webhook 的本質差異**
>
> | | Discord Webhook | Gmail（OAuth）|
> |---|---|---|
> | 驗證方式 | URL 即金鑰，無需帳號 | OAuth 授權，綁定 Google 帳號 |
> | 設定複雜度 | 極低，貼 URL 就能用 | 中等，需授權流程 |
> | 安全風險 | URL 外洩即可被任何人使用 | Token 有效期限，較安全 |
> | 教學價值 | 展示最純粹的 Webhook 概念 | 展示 OAuth 授權概念 |

---

> [!NOTE]
> **💡 延伸思考：Make 的 Router（路由器）**
>
> 如果你想要：
> - A 場次警報 → Discord 的 `#a場-工作人員` 頻道
> - B 場次警報 → Discord 的 `#b場-工作人員` 頻道
>
> 只需要為每個頻道各建一個 Webhook URL，在兩個節點之間插入「**Router**」模組，
> 分成兩條支線，每條設定不同過濾條件和不同 Webhook URL。
>
> 這就是**多路分流自動化**的基礎架構！

> [!TIP]
> **🏆 第 2 小時 Checkpoint 完成！**
>
> 你剛剛完成了：
> - ✅ 理解 API 與 Webhook 的核心概念（URL 即端點）
> - ✅ 在 Make 建立第一個「觸發 → 條件 → 動作」自動化劇本
> - ✅ 系統能在 VIP 報名的 30 秒內，自動推播警報到 Discord
>
> 這是「**事件驅動架構（Event-Driven Architecture）**」的基礎概念——
> 大型科技公司的後端系統也是這樣運作的！

---

### 🎯 第 2 小時學生練習題

> [!NOTE]
> **⏱️ 練習時間：15 分鐘｜個人作業**
>
> 完成後截圖存檔，下課前上傳至課堂共用 Google 試算表。

---

#### 【練習 2-A】Discord 警報訊息改版（基礎）

請修改你的 Make 劇本，將 Discord 的 **Content 欄位**改版成以下格式：

1. 訊息開頭加上一行由 `═` 組成的分隔線（至少 20 個字元）
2. 加入「**報名序號**」，使用 Make 內建變數 `{{1. Row Number}}`
3. 訊息結尾加上：`請在 2 小時內完成接待安排，逾時請升級為 Level 2 警報。`

改版完成後，送出一筆新的 VIP 報名表單，**截圖 Discord 頻道收到的新格式訊息**。

> [!TIP]
> Discord Markdown 小提醒：
> - `**粗體**` → **粗體**
> - `__底線__` → 底線
> - `> 引言` → 引言縮排區塊
> - 在 Content 欄位中可以直接使用這些語法

---

#### 【練習 2-B】多條件過濾器（進階）

課堂範例只有一個過濾條件（VIP = 是）。請修改你的 Make 劇本，讓過濾器**同時滿足兩個條件**才發警報：

- 條件一：`是否為VIP貴賓？` 等於 `是`
- 條件二：`參加場次` 等於 `C場：週日全天`

> [!TIP]
> 在 Make 的 Filter 設定中，點擊「**Add AND rule**」可新增第二個條件，
> 兩個條件同時為真才會繼續執行。

設定完成後，分別送出兩筆測試資料：

| 測試 | VIP | 場次 | 預期結果 |
|------|-----|------|---------|
| 測試一 | 是 | A場：週六上午 | ❌ 不應觸發通知 |
| 測試二 | 是 | C場：週日全天 | ✅ 應觸發通知 |

**截圖 Make 的執行記錄（Operations History）**，說明哪一筆被過濾掉、哪一筆通過。

<!-- 📸 截圖：Make Operations History 顯示兩筆測試的執行結果 -->

---

#### 【練習 2-C】Webhook vs OAuth 概念比較（思考題）

請用自己的話（每題 50～100 字）回答以下兩題，寫在課堂共用文件：

**問題一：安全性分析**

> Discord Webhook 和 Gmail（OAuth）都能讓 Make 發出通知，但驗證方式完全不同。
> 如果你的 Discord Webhook URL 不小心被別人拿到，會發生什麼事？
> 相比之下，Gmail OAuth 的設計如何降低這個風險？

**問題二：Limit 參數的影響**

> 假設你在 Make 的 Google Sheets 節點，把 `Limit` 從 `2` 改成 `100`，會發生什麼事？
> 什麼情況下這樣設定是合理的？什麼情況下會造成問題？
