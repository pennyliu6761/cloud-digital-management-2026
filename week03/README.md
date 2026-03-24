## ⏰ 第 3 小時：AI 賦能雲端 — Gemini 視覺辨識 × 收據自動核銷

> 🎯 **本小時目標：** 把 AI 的「眼睛」接入工作流。
> 讓 Gemini Vision 模型自動閱讀上傳的收據圖片，提取金額與姓名，
> 自動回填試算表，並推播人工覆核通知到 Discord——徹底取代人工核對。

---

### 💡 觀念講解：什麼是 Prompt Engineering？

**Prompt** 就是你給 AI 的「任務說明書」。同樣的 AI，不同的 Prompt 會得到截然不同的結果。

| Prompt 品質 | 範例 | AI 可能的回應 |
|------------|------|-------------|
| 太模糊 | `幫我看這張圖` | `這是一張收據，顯示了消費資訊...` |
| 精確指令 | `只以 JSON 格式回覆，提取 name 和 amount，不要加任何其他文字` | `{"name": "王大明", "amount": 1500}` |

> **機器可以閱讀 JSON，才能讓後續自動化流程繼續執行。**
> 如果 AI 回覆的是自然語言，Make 的下一個節點就無法解析它。

---

### 📌 任務 3-1：在表單加入收據上傳功能

1. 開啟 `金門聚落文化營 2026 — 線上報名表`
2. 新增題目，題型選「**檔案上傳**」，設定如下：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | 題目 | `請上傳匯款收據截圖（圖片格式）` |
    | 允許的檔案類型 | 勾選「**圖片**」|
    | 最大檔案數量 | 1 |
    | 最大檔案大小 | 10 MB |

<img width="833" height="511" alt="image" src="https://github.com/user-attachments/assets/ea2b303b-0a2e-4dfb-b563-c6f73dd3cb4f" />

3. **建立測試資料：**
   - 在網路上搜尋「匯款收據截圖範例」存成圖檔，或直接拍一張任何收據
   - 填寫一次表單並上傳這張圖片，送出

4. 確認試算表出現一個 Google Drive 連結，格式類似：
```
    https://drive.google.com/open?id=xxxxx
```

<img width="1102" height="334" alt="image" src="https://github.com/user-attachments/assets/0212b5a2-e4cc-4f4f-ad65-6d2b267502fb" />


---

### 📌 任務 3-2：取得 Google AI Studio API Key

**工具：** [Google AI Studio](https://aistudio.google.com/)（免費，每分鐘 15 次請求）

1. 前往 Google AI Studio，用 Google 帳號登入
2. 點擊左側「**Get API key**」
3. 點擊「**Create API key**」→ 選擇 Google Cloud 專案（若無，點擊建立新專案）
4. 立刻複製這串 API Key 貼到記事本備用：
```
    我的 Gemini API Key：
    AIzaSy_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

<!-- 📸 截圖：AI Studio 介面（API Key 遮住後八碼） -->

5. **先在 AI Studio 內測試辨識能力：**
   - 點擊左側「**Stream realtime**」
   - 點擊「**+**」→「**Upload file**」上傳你的收據圖片
   - 輸入以下 Prompt 測試：
```
    你是一位專業會計助理。請辨識這張收據或匯款截圖，
    提取「匯款人姓名」和「總金額（純數字）」。
    請只回覆 JSON 格式，不要加任何其他文字：
    {"name": "姓名", "amount": 數字}
```

   - 確認 AI 在 2～3 秒內正確回應 JSON

<!-- 📸 截圖：AI Studio 成功辨識收據並回傳 JSON 的畫面 -->

---

### 📌 任務 3-3：Make 串接 AI 視覺辨識完整流程

> [!NOTE]
> 建議新建一個劇本專門處理「收據辨識核銷流程」，
> 與第 2 小時的「VIP 警報流程」分開管理，避免混淆。

整個流程的資料流如下：
```
[ Google Sheets ]
  Watch Rows（監控含圖片的新報名）
        │
        ▼
[ Google Drive ]
  Download a File（下載收據圖片）
        │
        ▼
[ Google AI Studio ]
  Generate Content（Gemini 分析圖片）
        │
        ▼
[ JSON ]
  Parse JSON（解析 AI 回傳的文字）
        │
        ▼
[ Google Sheets ]            [ Discord + Gmail ]
  Update a Row          →    覆核通知（同步推播）
（回填 AI 判讀結果）
```

<!-- 📸 截圖：Make 完整劇本畫布（所有節點一起） -->

---

#### 步驟一：Google Sheets 監控新報名

設定同第 2 小時的 Watch Rows，**不需要過濾器**，所有新報名都要處理。

| 設定項目 | 填入內容 |
|---------|---------|
| Spreadsheet | `金門聚落文化營_報名總表` |
| Sheet | `表單回覆 1` |
| Limit | `2` |

---

#### 步驟二：Google Drive 下載圖片

1. 新增模組：「**Google Drive**」→「**Download a File**」
2. **File ID** 欄位：從左邊傳來的 Google Sheets 變數中，找到「請上傳匯款收據截圖」那一欄的變數帶入

<!-- 📸 截圖：Google Drive Download a File 節點設定，File ID 對應欄位 -->

> [!NOTE]
> Google 表單上傳的圖片連結格式是 `https://drive.google.com/open?id=FILE_ID`，
> Make 的 Google Drive 模組會自動解析 ID 並下載圖片實體，不需要手動處理。

---

#### 步驟三：呼叫 Gemini AI 分析圖片

1. 新增模組：「**Google AI Studio (Gemini)**」→「**Generate Content**」
2. 建立 Connection：貼上你的 API Key → 點擊「**Save**」
3. 設定如下：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | Model | `gemini-1.5-flash` |
    | Role | `User` |

4. **Text（Prompt）欄位**，輸入以下內容：
```
    你是一位專業的會計核銷助理。
    請仔細辨識附圖中的收據、發票或匯款截圖。
    請提取以下兩項資訊：
    1. 「匯款人姓名」或「消費者姓名」
    2. 「總金額」（純數字，不含貨幣符號）

    回覆規則（極重要）：
    - 只能回覆嚴格的 JSON 格式
    - 不得加上任何解釋文字、Markdown 符號或其他內容
    - 若無法辨識，回覆 {"name": "無法辨識", "amount": 0}

    正確回覆範例：
    {"name": "陳小明", "amount": 2500}
```

5. **Media（附加圖片）：**
   - 點擊「**Add item**」
   - `File` 欄位：選擇上一步 Google Drive 下載的 `Data`
   - `MIME type`：輸入 `image/jpeg`（或依圖片類型選 `image/png`）

<!-- 📸 截圖：Gemini 模組設定畫面，特別是 Prompt 和 Media 設定 -->

---

#### 步驟四：解析 AI 回傳的 JSON

1. 新增模組：「**JSON**」→「**Parse JSON**」
2. **JSON string** 欄位：選擇上一步 Gemini 模組回傳的 `Text` 變數

<!-- 📸 截圖：Parse JSON 節點設定 -->

> [!NOTE]
> Gemini 回傳的是純文字字串，Parse JSON 把它轉成 Make 可以讀取的結構化欄位。
> 轉換後你就能分別取用 `name` 和 `amount` 兩個獨立變數。

---

#### 步驟五：將 AI 判讀結果回填試算表

1. 先回到 Google 試算表，手動新增兩欄：
   - `AI判讀姓名`
   - `AI判讀金額`

2. 新增模組：「**Google Sheets**」→「**Update a Row**」
3. 設定如下：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | Spreadsheet | `金門聚落文化營_報名總表` |
    | Sheet | `表單回覆 1` |
    | Row Number | 第一個 Watch Rows 模組傳來的 `Row number` 變數 |
    | AI判讀姓名 | Parse JSON 的 `name` 變數 |
    | AI判讀金額 | Parse JSON 的 `amount` 變數 |

<!-- 📸 截圖：Update a Row 節點設定，顯示欄位對應 -->

---

#### 步驟六：推播人工覆核通知

這個步驟同時設定 Discord 和 Gmail 兩個通知，**使用 Router 模組分流**：

1. 新增模組：「**Flow Control**」→「**Router**」
2. Router 分成兩條支線：
   - 支線 1 → Discord
   - 支線 2 → Gmail

**支線 1：Discord 通知**

新增「**Discord**」→「**Send a Message by Webhook Bot**」，Message 欄位輸入：
```
💰 **【收據辨識通知】**

系統已自動辨識新報名者的匯款收據，請覆核。

▸ **報名者：** {{姓名}}
▸ **AI 判讀姓名：** {{name}}
▸ **AI 判讀金額：** {{amount}} 元

⚠️ AI 有一定機率判讀錯誤，人工確認為必要程序。
請開啟試算表確認是否正確。
```

**支線 2：Gmail 備案通知**

新增「**Gmail**」→「**Send an Email**」，設定如下：

| 設定項目 | 填入內容 |
|---------|---------|
| To | 會計人員或負責人 Email |
| Subject | `【收據辨識】{{姓名}} 的匯款待覆核 — AI 判讀金額 {{amount}} 元` |
| Content | 同 Discord 訊息內容 |

<!-- 📸 截圖：Router 節點分成兩條支線的畫布截圖 -->

---

### 🎯 終極壓力測試！

1. 確認你已有一筆含收據圖片的測試報名資料
2. 點擊 Make 左下角「**Run once**」
3. 觀察資料依序流經每個節點：
```
    📋 Google Sheets 抓到資料
         ↓
    📥 Google Drive 下載圖片
         ↓
    🧠 Gemini AI 分析圖片（約 3～8 秒）
         ↓
    🔍 JSON 解析完成
         ↓
    📝 試算表自動更新
         ↓
    🔔 Discord 收到通知  +  📧 Gmail 收到備案通知
```

4. 打開 Google 試算表——`AI判讀姓名` 和 `AI判讀金額` 兩欄自動填入了圖片裡的資訊！

<!-- 📸 截圖：試算表 AI 判讀欄位自動填入資料 -->
<!-- 📸 截圖：Discord 頻道收到覆核通知 -->
<!-- 📸 截圖：Gmail 收到備案通知 -->

---

> [!WARNING]
> **🚨 AI 幻覺防呆機制（重要）**
>
> AI 不是 100% 正確，它有時會「幻覺」——例如把 1,500 看成 15,000。
>
> **實務上的防呆設計：**
> 1. 永遠讓 Discord / Gmail 通知人工覆核（我們剛剛已經做了）
> 2. 在試算表加入「人工確認」欄位，未勾選代表待審核
> 3. 金額差異超過 10% 自動標記紅色（可用試算表條件格式設定）
>
> **黃金原則：自動化負責苦力，人類負責最終決策。**
> 就算是銀行的 AI 過帳系統，最後也需要人工授權才能真正撥款。

> [!TIP]
> **🏆 第 3 小時 Checkpoint 完成！**
>
> 你剛剛串接完成的系統：
> - ✅ 表單上傳圖片 → AI 自動辨識內容
> - ✅ 辨識結果自動回填試算表，不需要手動輸入
> - ✅ Discord + Gmail 雙管道推播，確保覆核通知不漏接
> - ✅ AI 幻覺防呆機制，讓人工保有最終決策權
>
> 這套系統如果對外發包，報價至少 **NT$ 80,000**。你用零成本、零程式碼完成了。

---

### 🎯 第 3 小時學生練習題

> [!NOTE]
> **⏱️ 練習時間：20 分鐘｜個人 + 分組討論**

---

#### 【練習 3-A】Prompt 改版挑戰（個人，基礎）

課堂範例只提取「姓名」和「金額」兩個欄位。請修改 Prompt，讓 AI 額外提取第三個欄位：`bank_last5`（銀行帳號後五碼，若圖片中無法辨識則填入 `"無"`）。

新的 JSON 輸出格式應為：
```json
{"name": "陳小明", "amount": 2500, "bank_last5": "88765"}
```

需要同步更新的地方：
1. Make 劇本中 Gemini 模組的 Prompt 文字
2. Google 試算表新增 `AI判讀帳號後五碼` 欄位
3. Make 劇本中 Update a Row 模組新增對應欄位
4. Discord 通知訊息中加入 `▸ **AI 判讀帳號後五碼：** {{bank_last5}}`

完成後重新執行，截圖試算表顯示三個 AI 判讀欄位都有資料。

---

#### 【練習 3-B】Prompt 對照實驗（個人，進階）

用同一張收據圖片，在 Google AI Studio 分別測試三種 Prompt，截圖三個不同的回應結果：

| | Prompt | 目的 |
|---|---|---|
| **A（模糊版）** | `請幫我看這張圖片裡有什麼資訊` | 觀察 AI 自由發揮的結果 |
| **B（半精確版）** | `請提取這張收據的姓名和金額` | 觀察沒有格式限制的結果 |
| **C（嚴格版）** | 課堂使用的完整 Prompt | 觀察有嚴格規範的結果 |

在課堂共用文件中說明：三個 Prompt 的回應有何差異？為什麼只有 C 適合接入自動化流程？

---

#### 【練習 3-C】期末延伸構想（分組，2～3 人，挑戰）

你們小組要提出一個「第 14 週期末專題」的初步構想，需要：

1. **選定一個真實情境**（工工系的痛點、金門大學的行政流程、或金門縣的在地問題）
2. **說明使用哪些工具**（至少用到這三週學過的 3 種以上工具）
3. **畫出資料流程圖**（從使用者第一個動作到系統最後輸出，每個節點用箭頭連起來，可手繪拍照）

**下週課程最後 10 分鐘，每組 2 分鐘說明構想，老師給初步回饋。**

> [!NOTE]
> **範例構想（僅供參考，不要照抄）**
>
> **情境：** 工工系系辦用紙本管理「借用設備申請」，常搞不清楚設備被誰借走了。
>
> **工具：** Google 表單 → 試算表 → AppSheet（手機確認歸還）→ Make（逾期未還推播 Discord）→ Looker Studio（設備使用率儀表板）
>
> **資料流：** 學生填表 → 試算表新增紀錄 → AppSheet 顯示待確認清單 → 管理員確認借出 → Make 監控逾期 → 自動推播 Discord 通知

---

## 🏆 第一週總結

### 📊 你今天打造的完整系統
```
[ Google 表單 ] ──→ [ 試算表 (SSOT) ] ──→ [ AppSheet App ]
       │                    │                      │
       │ AutoCrat           │ Make 監控             │ 手機報到
       ↓                    ↓                      ↓
[ PDF 憑證 ]        [ VIP 警報流程 ]        [ Looker Studio ]
  寄到信箱           Discord 推播             即時儀表板
                           │
                  [ Google Drive 圖片 ]
                           ↓
                  [ Gemini AI 視覺辨識 ]
                           ↓
              [ 自動回填 + Discord / Gmail 覆核通知 ]
```

### 💰 價值換算

| 功能項目 | 外包報價估算 | 你的成本 |
|---------|-----------|---------|
| 自動化報名 + PDF 憑證系統 | NT$ 30,000 | 免費 |
| 手機報到 App + 儀表板 | NT$ 70,000 | 免費 |
| Discord 自動警報系統 | NT$ 15,000 | 免費 |
| AI 收據辨識核銷 | NT$ 80,000 | 免費 |
| **合計** | **NT$ 195,000+** | **NT$ 0** |

### 🤔 課後思考問題

1. **系統瓶頸：** 如果同時有 500 人在同一分鐘送出表單，這套系統哪個環節最可能出問題？如何改善？

2. **AI 倫理：** AI 幫你自動辨識收據，如果判讀錯誤導致民眾被多收費，責任在誰？系統設計者應該如何降低這個風險？

3. **延伸應用：** 除了活動報名，這套「圖片上傳 → AI 辨識 → 自動入庫」的流程，你能想到哪些其他的應用場景？

### 📚 課後自學資源

| 資源 | 用途 |
|------|------|
| [Google AI Studio 文件](https://ai.google.dev/) | Gemini API 完整說明 |
| [Make 官方教學](https://www.make.com/en/help) | 所有模組的使用說明 |
| [Discord Webhook 說明](https://support.discord.com/hc/en-us/articles/228383668) | Webhook 進階設定 |
| YouTube: `Make.com Gemini AI tutorial` | 視覺化教學參考 |

---

### 📝 下週預告

> 🔜 **Week 02：Google 表單魔法 — 從問卷到自動化文件工廠**
>
> - 條件跳題邏輯：表單根據填答者的選擇自動跳到不同題組
> - 資料驗證：用正規表達式阻擋不合格的填答
> - 測驗模式：自動批改、即時回饋、統計全班答題狀況
> - AutoCrat 批次模式：一鍵產出 100 份個人化 PDF
> - Google 試算表 10 個核心公式
>
> **課前任務：** 確認你的 Google 帳號能正常登入 Google 表單和試算表。

---

*雲端與數位內容管理 ｜ 國立金門大學 工業工程與管理學系 ｜ 2026 春季學期*
*Week 01 of 14*
