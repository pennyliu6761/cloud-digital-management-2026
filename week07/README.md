# 🕷️ Week 07：網頁爬蟲不用寫 Code — ParseHub × Make 自動採集外部資料

**雲端與數位內容管理 ｜ 國立金門大學 工業工程與管理學系**

> **本週課程理念：** 到目前為止，我們處理的資料都是「自己產生的」——
> 表單填答、試算表手動輸入。
> 但真實的工管場景需要大量「外部資料」——競爭對手的職缺、
> 原物料行情、法規公告、市場調查。
>
> **今天我們讓系統自動去網路上蒐集資料，你只需要設定一次，
> 它就會每天自動幫你抓回來。**

---

## 🎯 本週學習目標

學完本週，你將能夠：
- ✅ 解釋網頁爬蟲的運作原理，知道合法爬蟲的邊界在哪裡
- ✅ 用 ParseHub 的點擊介面設計爬蟲，完全不需要寫程式碼
- ✅ 把爬下來的資料匯出為 JSON 或 CSV，再存入 Google Sheets
- ✅ 用 Make 定時觸發 ParseHub API，實現全自動資料採集
- ✅ 對爬回來的骯髒資料套用 Week 06 的清洗公式

---

## 🏢 本週實作情境：【工管系職缺情報站】

> 💼 **情境：** 工管系學生除了找實習，也需要了解畢業後的就業市場。
> 系辦希望每週自動蒐集人力銀行上「工業工程」相關的職缺資料，
> 讓學生知道：哪些技能最搶手？哪些縣市需求最大？平均薪資是多少？
>
> **今天的任務：**
> 1. 用 ParseHub 爬取公開職缺資料
> 2. 自動存入 Google Sheets（套用清洗公式）
> 3. 用 Make 設定每週自動執行
> 4. 為下週的 AI 分析做好資料準備

---

## ⏰ 第 1 小時：爬蟲概念 + ParseHub 基礎操作

> 🎯 **本小時目標：** 理解爬蟲的原理和法律邊界，
> 並完成第一個 ParseHub 爬蟲專案。

---

### 💡 觀念一：網頁爬蟲是什麼？

**網頁爬蟲（Web Scraper）** 是模擬「人用瀏覽器瀏覽網頁」的程式，
自動讀取網頁內容並把資料抽取出來。
```
你手動做的事：
  打開瀏覽器 → 連到 104 人力銀行 → 搜尋「工業工程」
  → 看到職缺列表 → 手動複製職缺名稱、薪資、地點
  → 貼到 Excel → 翻頁 → 重複 20 次

爬蟲自動做的事：
  設定好規則 → 一鍵執行 → 5 分鐘後取得 200 筆整理好的資料
```

**技術上怎麼運作：**
```
1. 爬蟲發送 HTTP GET 請求到目標網址
2. 伺服器回傳 HTML 原始碼
3. 爬蟲解析 HTML，找到你指定的元素
4. 把文字內容抽取出來，儲存為結構化資料（JSON / CSV）
5. 翻頁，重複以上步驟
```

---

### 💡 觀念二：爬蟲的法律邊界

> [!WARNING]
> **爬蟲不是無限制的！** 使用前必須確認以下事項：

| 條件 | 說明 |
|------|------|
| **robots.txt** | 每個網站根目錄都有這個檔案，規定哪些頁面允許爬蟲存取。例如：`104.com.tw/robots.txt` |
| **服務條款** | 許多網站明確禁止自動化爬取，違反可能構成違約 |
| **爬取頻率** | 爬得太快會讓對方伺服器過載，可能被視為 DDoS 攻擊 |
| **資料用途** | 個人學習研究 vs 商業用途，法律責任不同 |
| **登入資料** | 爬取需要登入才能看到的資料，風險更高 |

**本週使用的安全策略：**
- ✅ 只爬**公開可見**的資料（不需要登入）
- ✅ 控制爬取頻率（每週一次，不是每分鐘）
- ✅ 用於**學術研究和教學**目的
- ✅ 不轉售、不商業使用爬取的資料

> [!NOTE]
> **本週使用的目標網站：**
> 我們爬取**政府公開資料平台**或**免費求職網站的公開頁面**，
> 這類資料明確對外公開，風險最低。
> 具體網站請依老師課堂指定為準。

---

### 💡 觀念三：ParseHub 的運作方式

ParseHub 是一個**視覺化爬蟲工具**，讓你用點擊的方式告訴它「我要這個資料」：
```
傳統爬蟲（寫程式）：
  <div class="job-title">工業工程師</div>
  soup.find('div', class_='job-title').text  ← 要懂 HTML 和 Python

ParseHub（點擊介面）：
  直接用滑鼠點擊「工業工程師」這幾個字
  → ParseHub 自動識別選取規則
```

**ParseHub 的執行模式：**

| 模式 | 說明 | 本週使用 |
|------|------|---------|
| 桌面版執行 | 在你的電腦上跑，需要開著電腦 | 第 1-2 小時 |
| 雲端執行 | 在 ParseHub 的伺服器上跑 | 免費版限制較多 |
| API 觸發 | 用 Make 呼叫 API 啟動執行 | 第 3 小時 |

---

### 📌 任務 1-1：安裝 ParseHub 並建立第一個專案

> [!NOTE]
> **課前任務確認：** 你應該已經下載並安裝好 ParseHub 桌面版。
> 如果還沒有，請前往 [parsehub.com](https://www.parsehub.com/) 下載。

1. 開啟 ParseHub 桌面版
2. 點擊「**New Project**」
3. 在 URL 欄位輸入目標網址（老師課堂指定）
4. 點擊「**Start Project on this URL**」
5. ParseHub 會開啟內建瀏覽器，載入目標網頁

<!-- 📸 截圖：ParseHub 桌面版初始畫面 -->
<!-- 📸 截圖：New Project 設定畫面，輸入目標網址 -->

---

### 📌 任務 1-2：選取第一個資料元素

**目標：** 選取職缺名稱

1. 在 ParseHub 內建瀏覽器中，找到第一個職缺的**職缺名稱**
2. 用滑鼠**點擊**職缺名稱
3. ParseHub 會用綠色高亮顯示你選取的元素，並自動命名為 `selection1`

<!-- 📸 截圖：ParseHub 選取第一個職缺名稱，顯示綠色高亮 -->

---

### 📌 任務 1-3：讓 ParseHub 自動選取同類元素

點擊第一個職缺名稱後，ParseHub 通常會問你：
「要選取這一個，還是所有類似的？」

1. 選擇「**Select all**」（選取所有類似的元素）
2. 確認頁面上所有職缺名稱都被高亮顯示

> [!NOTE]
> **ParseHub 怎麼知道「類似的元素」？**
> 它分析你點擊的元素的 HTML 結構（CSS class、標籤層級），
> 自動找到所有符合相同規則的元素。
> 這就是為什麼你只需要點一次，它能抓整頁的資料。

<!-- 📸 截圖：所有職缺名稱都被綠色高亮顯示 -->

---

### 📌 任務 1-4：新增更多欄位

重複以上步驟，依序選取：

| 欄位名稱 | 點擊位置 |
|---------|---------|
| `job_title` | 職缺名稱 |
| `company_name` | 公司名稱 |
| `location` | 工作地點 |
| `salary` | 薪資範圍 |
| `date_posted` | 刊登日期 |

每個欄位選取後，在左側面板重新命名（清楚的英文名稱，方便後續處理）。

<!-- 📸 截圖：ParseHub 左側面板顯示五個欄位都設定完成 -->

---

### 📌 任務 1-5：設定翻頁（Pagination）

只爬一頁資料量太少，需要設定翻頁：

1. 在左側面板點擊「**Add command → Click**」
2. 在網頁上點擊「下一頁」或「第 2 頁」按鈕
3. 設定：
    - **Click type**：「Click Next Page」
    - **Max pages**：`3`（免費版建議不超過 5 頁）

<!-- 📸 截圖：翻頁設定完成，顯示 Click 命令 -->

---

### 📌 任務 1-6：執行並下載資料

1. 點擊左上角「**Get Data**」→「**Run**」
2. 等待執行完成（通常 1-3 分鐘）
3. 執行完成後點擊「**Download**」→ 選擇「**CSV**」格式
4. 打開下載的 CSV 檔案，確認資料正確

<!-- 📸 截圖：ParseHub 執行完成，顯示抓取筆數 -->
<!-- 📸 截圖：下載的 CSV 檔案，在 Excel 或 Google Sheets 開啟 -->

> [!TIP]
> **🏆 第 1 小時 Checkpoint 完成！**
>
> - ✅ 理解爬蟲原理和法律邊界
> - ✅ 完成第一個 ParseHub 專案，成功抓取職缺資料
> - ✅ 下載 CSV 並確認資料結構正確

---

### 🎯 第 1 小時學生練習題

> [!NOTE]
> **⏱️ 練習時間：15 分鐘｜個人作業**

---

#### 【練習 1-A】robots.txt 檢查（基礎）

查看以下三個網站的 robots.txt，判斷是否允許爬蟲：

1. `https://data.gov.tw/robots.txt`（政府資料開放平台）
2. `https://www.104.com.tw/robots.txt`（104 人力銀行）
3. `https://www.ptt.cc/robots.txt`（PTT）

填寫下表並貼到課堂共用文件：

| 網站 | 是否允許爬蟲 | 有哪些限制 |
|------|------------|----------|
| 政府資料開放平台 | | |
| 104 人力銀行 | | |
| PTT | | |

---

#### 【練習 1-B】新增欄位（進階）

在你完成的 ParseHub 專案裡，再新增一個欄位：

- 欄位名稱：`job_url`
- 內容：每個職缺的詳細頁面連結（通常要右鍵 → Copy link）

> [!TIP]
> 在 ParseHub 中選取連結：點擊元素後，
> 在左側面板的 **Attribute** 下拉選單選擇 `href`，
> 就能抓取連結網址而不是文字。

截圖 ParseHub 設定完成，並確認 CSV 下載後有 `job_url` 欄位。

---

#### 【練習 1-C】觀念問答（思考題）

請回答，貼到課堂共用文件：

> ❓ 如果你是 104 人力銀行的工程師，
> 你會怎麼設計系統來防止爬蟲大量抓取你的資料？
> 請至少提出三種防爬蟲的技術手段，並說明爬蟲方有什麼對應的方法。

---

## ⏰ 第 2 小時：資料清洗與匯入 Google Sheets

> 🎯 **本小時目標：** 把爬下來的骯髒資料清洗乾淨，
> 建立結構化的職缺資料庫，為下週的 AI 分析做準備。

---

### 💡 觀念：爬蟲資料為什麼特別骯髒？

爬下來的資料通常有以下問題：
```
薪資欄位：「月薪 35,000～50,000元」→ 需要拆成數字
地點欄位：「台北市中山區」→ 有時混雜郵遞區號
日期欄位：「3天前」、「剛剛」→ 相對時間，無法排序
空白欄位：部分職缺沒有填薪資
HTML 殘留：「工業工程師&amp;」→ HTML 實體字元
```

---

### 📌 任務 2-1：把 CSV 匯入 Google Sheets

1. 開啟新的 Google Sheets
2. 命名：`Week07_職缺情報站`
3. 建立工作表分頁：`raw_data`（原始資料）和 `cleaned_data`（清洗後）
4. 在 `raw_data` 工作表：「**檔案**」→「**匯入**」→ 上傳 ParseHub 下載的 CSV
5. 設定：
    - Import location：`Replace current sheet`
    - Separator type：`Comma`

<!-- 📸 截圖：CSV 成功匯入 raw_data 工作表 -->

---

### 📌 任務 2-2：分析原始資料的問題

在匯入後，逐欄檢查資料品質，記錄發現的問題：

| 欄位 | 發現的問題 | 清洗方法 |
|------|---------|---------|
| job_title | 有些有多餘空格 | TRIM + CLEAN |
| company_name | 部分包含括號說明 | REGEXREPLACE |
| location | 地點有時包含完整地址，有時只有縣市 | LEFT 或 REGEXEXTRACT |
| salary | 「月薪 X 萬～Y 萬」格式不統一 | REGEXEXTRACT 取數字 |
| date_posted | 「N天前」格式，無法直接排序 | 轉換為實際日期 |

---

### 📌 任務 2-3：在 `cleaned_data` 工作表套用清洗公式

切換到 `cleaned_data` 工作表，對每個欄位套用清洗公式：

---

**欄位 1：職缺名稱（標準清洗）**
```
=TRIM(CLEAN(raw_data!A2))
```

---

**欄位 2：公司名稱（去除括號說明）**

部分公司名稱格式為「台積電（TSMC）」，只取括號前的名稱：
```
=TRIM(IFERROR(LEFT(raw_data!B2, FIND("（", raw_data!B2)-1), TRIM(raw_data!B2)))
```

**公式解析：**
- `FIND("（", B2)` — 找到全形左括號的位置
- `LEFT(..., 位置-1)` — 取括號前的文字
- `IFERROR(..., TRIM(B2))` — 如果沒有括號，就直接 TRIM 原始文字

---

**欄位 3：縣市（只取前三個字）**

地點格式可能是「台北市中山區信義路」，只需要縣市：
```
=IFERROR(
  IF(MID(raw_data!C2,3,1)="市", LEFT(raw_data!C2,3),
  IF(MID(raw_data!C2,3,1)="縣", LEFT(raw_data!C2,3),
  LEFT(raw_data!C2,3))),
  raw_data!C2
)
```

> [!TIP]
> 更簡潔的方式是用 `REGEXEXTRACT`：
> ```
> =IFERROR(REGEXEXTRACT(raw_data!C2, "^.{2}[市縣]"), LEFT(raw_data!C2, 3))
> ```
> 這個正規表達式的意思是：從開頭取 2 個字，後面接「市」或「縣」。

---

**欄位 4：薪資下限（從薪資範圍字串抽取數字）**

薪資欄位格式通常是「35,000 ~ 50,000」或「3.5萬～5萬」，
先處理「元」格式：
```
=IFERROR(
  VALUE(REGEXREPLACE(
    REGEXEXTRACT(raw_data!D2, "(\d[\d,]+)"),
    ",", ""
  )),
  ""
)
```

**公式解析：**
- `REGEXEXTRACT(..., "(\d[\d,]+)")` — 抽取第一組數字（含逗號）
- `REGEXREPLACE(..., ",", "")` — 去掉逗號
- `VALUE(...)` — 轉成數字
- `IFERROR(..., "")` — 無法解析時留空

---

**欄位 5：刊登日期（把相對時間轉換為估算日期）**

「3天前」→ 今天減3天：
```
=IFERROR(
  IF(REGEXMATCH(raw_data!E2, "天前"),
    TODAY() - VALUE(REGEXEXTRACT(raw_data!E2, "(\d+)天前")),
  IF(REGEXMATCH(raw_data!E2, "剛剛"),
    TODAY(),
  IF(REGEXMATCH(raw_data!E2, "週前"),
    TODAY() - VALUE(REGEXEXTRACT(raw_data!E2, "(\d+)")) * 7,
  DATEVALUE(raw_data!E2)))),
  raw_data!E2
)
```

設定儲存格格式為日期（`YYYY/MM/DD`）。

---

**欄位 6：資料品質標記**

為每筆資料自動評分，方便篩選：
```
=IF(
  AND(
    NOT(ISBLANK(cleaned_data!A2)),
    NOT(ISBLANK(cleaned_data!B2)),
    NOT(ISBLANK(cleaned_data!D2))
  ),
  "✅ 完整",
  "⚠️ 不完整"
)
```

<!-- 📸 截圖：cleaned_data 工作表，顯示清洗前後的對照（至少兩欄並排比較）-->

---

### 📌 任務 2-4：建立職缺資料庫的分析視圖

新建工作表 `analysis`，用 QUERY 建立統計分析：

**各縣市職缺數量：**
```
=QUERY(cleaned_data!A:F,
  "SELECT C, COUNT(A)
   WHERE F = '✅ 完整'
   GROUP BY C
   ORDER BY COUNT(A) DESC
   LABEL C '縣市', COUNT(A) '職缺數'",
  1)
```

**薪資下限高於 40000 的職缺：**
```
=QUERY(cleaned_data!A:F,
  "SELECT A, B, C, D
   WHERE E >= 40000
   AND F = '✅ 完整'
   ORDER BY E DESC",
  1)
```

**本週（7天內）刊登的職缺：**
```
=QUERY(cleaned_data!A:F,
  "SELECT A, B, C, D, E
   WHERE E >= date '"&TEXT(TODAY()-7,"yyyy-mm-dd")&"'
   AND F = '✅ 完整'
   ORDER BY E DESC",
  1)
```

<!-- 📸 截圖：analysis 工作表，顯示三個 QUERY 分析結果 -->

> [!TIP]
> **🏆 第 2 小時 Checkpoint 完成！**
>
> - ✅ 把 CSV 匯入 Google Sheets 的 raw_data 工作表
> - ✅ 用六個清洗公式處理職缺資料的各種骯髒問題
> - ✅ 建立 analysis 視圖，可以即時查詢各縣市職缺數和薪資分布

---

### 🎯 第 2 小時學生練習題

> [!NOTE]
> **⏱️ 練習時間：20 分鐘｜個人作業**

---

#### 【練習 2-A】新增分析視圖（基礎）

在 `analysis` 工作表新增一個查詢：

**關鍵字篩選**：找出職缺名稱包含「工程師」的所有職缺：
```
=QUERY(cleaned_data!A:F,
  "SELECT A, B, C, D
   WHERE A CONTAINS '工程師'
   AND F = '✅ 完整'",
  1)
```

截圖查詢結果，並說明結果筆數是否符合你的預期。

---

#### 【練習 2-B】薪資分析（進階）

在 `analysis` 工作表計算以下統計指標：
```
平均薪資下限：  =AVERAGEIF(cleaned_data!F:F, "✅ 完整", cleaned_data!E:E)
最高薪資下限：  =MAXIFS(cleaned_data!E:E, cleaned_data!F:F, "✅ 完整")
最低薪資下限：  =MINIFS(cleaned_data!E:E, cleaned_data!F:F, "✅ 完整")
薪資中位數：    =MEDIAN(FILTER(cleaned_data!E:E, cleaned_data!F:F="✅ 完整"))
```

截圖四個統計指標的結果，並用一句話說明這些數字對工管系學生的意義。

---

#### 【練習 2-C】觀念問答（思考題）

請回答：

> ❓ 本週的爬蟲資料清洗和 Week 06 的資料清洗有什麼不同？
> 外部資料（爬蟲）和內部資料（表單填答）各自的主要骯髒資料類型是什麼？
> 在系統設計上，你要怎麼因應這兩種不同的資料來源？

---

## ⏰ 第 3 小時：Make 定時觸發 ParseHub — 全自動資料採集

> 🎯 **本小時目標：** 用 Make 呼叫 ParseHub API，
> 設定每週自動執行一次爬蟲，資料自動更新到 Google Sheets。

---

### 💡 觀念：ParseHub 的 API 模式

ParseHub 除了桌面版手動執行，也提供 API 讓外部程式觸發：
```
手動模式（第 1-2 小時）：
  你 → 點擊 Run → ParseHub 執行 → 你下載 CSV

API 自動模式（第 3 小時）：
  Make 定時觸發 → ParseHub API 開始執行 → 執行完成
  → Make 呼叫另一個 API 取得結果 → 自動寫入 Google Sheets
```

**ParseHub API 的兩個關鍵端點：**

| 端點 | 方法 | 用途 |
|------|------|------|
| `/run` | POST | 觸發執行一個專案 |
| `/runs/{run_token}/results` | GET | 取得執行結果的 JSON 資料 |

---

### 📌 任務 3-1：取得 ParseHub API Key 和 Project Token

1. 登入 [ParseHub](https://www.parsehub.com/) 網頁版
2. 點擊右上角帳號 → 「**API**」
3. 複製 **API Key**（格式類似：`txxxxxxxxxxxxxx`）
4. 點擊你在第 1 小時建立的專案
5. 在專案設定頁面找到 **Project Token**（格式類似：`txxxxxxxxx`）
6. 把兩個值貼到記事本備用

<!-- 📸 截圖：ParseHub 網頁版的 API Key 和 Project Token 位置 -->

---

### 📌 任務 3-2：在 Hoppscotch 測試 ParseHub API

在搬到 Make 之前，先用 Hoppscotch 驗證 API 設定正確。

**步驟一：觸發執行**

1. 方法：**POST**
2. URL：`https://www.parsehub.com/api/v2/projects/你的ProjectToken/run`
3. Body 選 `application/x-www-form-urlencoded`
4. 新增以下 Body 欄位：

    | Key | Value |
    |-----|-------|
    | `api_key` | 你的 API Key |

5. 點擊「**Send**」
6. Response 會回傳一個 `run_token`，記下來：
```json
    {
      "run_token": "tGXXXXXXXXXX"
    }
```

<!-- 📸 截圖：Hoppscotch 觸發 ParseHub 執行，回傳 run_token -->

**步驟二：等待執行完成並取得結果**

執行需要 1-3 分鐘，等待後：

1. 方法：**GET**
2. URL：
```
    https://www.parsehub.com/api/v2/runs/你的run_token/results?api_key=你的API_Key&format=json
```
3. 點擊「**Send**」
4. 確認回傳的 JSON 包含爬取的資料

<!-- 📸 截圖：Hoppscotch 取得 ParseHub 執行結果的 JSON -->

---

### 📌 任務 3-3：在 Make 建立全自動劇本

**劇本架構：**
```
[Schedule 每週一 9:00]
    ↓
[HTTP POST → 觸發 ParseHub 執行]
    ↓
[Tools → Sleep 3分鐘（等待執行完成）]
    ↓
[HTTP GET → 取得執行結果 JSON]
    ↓
[JSON → Parse JSON]
    ↓
[Iterator → 逐筆處理每個職缺]
    ↓
[Google Sheets → Add a Row（寫入 raw_data）]
    ↓
[Discord → 推播完成通知]
```

---

#### 節點一：Schedule 觸發器

1. 新建劇本
2. 觸發器選「**Schedule**」
3. 設定：
    - Run scenario：`Every week`
    - Day of week：`Monday`
    - Time：`09:00`

<!-- 📸 截圖：Schedule 觸發器設定畫面 -->

---

#### 節點二：HTTP POST 觸發 ParseHub

1. 新增「**HTTP → Make a request**」
2. 設定：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | URL | `https://www.parsehub.com/api/v2/projects/你的ProjectToken/run` |
    | Method | `POST` |
    | Body type | `application/x-www-form-urlencoded` |

3. Body 新增欄位：

    | Key | Value |
    |-----|-------|
    | `api_key` | 你的 ParseHub API Key |

4. **Parse response** → `Yes`

---

#### 節點三：Sleep（等待執行完成）

ParseHub 執行需要時間，Make 需要暫停等待：

1. 新增「**Flow Control → Sleep**」
2. **Delay**：`180`（秒，即 3 分鐘）

> [!NOTE]
> **為什麼要 Sleep？**
> 如果 Make 馬上去拿結果，ParseHub 可能還在執行中，
> 回傳的資料是空的。
> 3 分鐘通常足夠，如果你的爬蟲很複雜可以調整到 5 分鐘。

---

#### 節點四：HTTP GET 取得執行結果

1. 新增「**HTTP → Make a request**」
2. 設定：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | URL | `https://www.parsehub.com/api/v2/runs/{{2.run_token}}/results` |
    | Method | `GET` |

3. Query String 新增：

    | Key | Value |
    |-----|-------|
    | `api_key` | 你的 ParseHub API Key |
    | `format` | `json` |

4. **Parse response** → `Yes`

> `{{2.run_token}}` 是第二個節點（HTTP POST）回傳的 run_token 變數。

---

#### 節點五：Iterator 逐筆處理

ParseHub 回傳的是一個 JSON 陣列，需要用 Iterator 拆開：

1. 新增「**Flow Control → Iterator**」
2. **Array**：選擇 HTTP GET 回傳的資料陣列（通常是 `data` 或 `results` 欄位）

---

#### 節點六：Google Sheets 寫入

1. 新增「**Google Sheets → Add a Row**」
2. 連結到 `Week07_職缺情報站` 的 `raw_data` 工作表
3. 欄位對應：

    | 試算表欄位 | Iterator 變數 |
    |---------|-------------|
    | job_title | `{{value.job_title}}` |
    | company_name | `{{value.company_name}}` |
    | location | `{{value.location}}` |
    | salary | `{{value.salary}}` |
    | date_posted | `{{value.date_posted}}` |
    | 爬取時間 | `{{now}}` |

---

#### 節點七：Discord 完成通知

在 Iterator 之外（整個迴圈結束後）新增 Discord 通知：

1. 新增「**Discord → Send a Message by Webhook Bot**」
2. 訊息：
```
    ✅ **【職缺情報站】本週資料更新完成**

    ▸ **執行時間：** {{formatDate(now; "YYYY/MM/DD HH:mm")}}
    ▸ **資料來源：** ParseHub 自動採集

    請前往試算表查看最新職缺資訊。
```

<!-- 📸 截圖：Make 完整劇本畫布（七個節點的完整流程）-->

---

### 📌 任務 3-4：手動測試完整流程

不等到下週一，先手動執行一次確認流程正確：

1. 點擊 Make 左下角「**Run once**」
2. 觀察每個節點依序執行
3. 確認 Google Sheets `raw_data` 工作表有新增資料
4. 確認 Discord 收到完成通知

<!-- 📸 截圖：Make 執行完成，顯示每個節點的處理筆數 -->
<!-- 📸 截圖：Google Sheets raw_data 工作表新增了職缺資料 -->

> [!TIP]
> **🏆 第 3 小時 Checkpoint 完成！**
>
> - ✅ 取得 ParseHub 的 API Key 和 Project Token
> - ✅ 在 Hoppscotch 測試 ParseHub API 兩個端點
> - ✅ 在 Make 建立七節點的全自動採集劇本
> - ✅ 設定每週一自動執行，資料自動更新到 Google Sheets

---

### 🎯 第 3 小時學生練習題

> [!NOTE]
> **⏱️ 練習時間：20 分鐘｜個人 + 分組**

---

#### 【練習 3-A】加入錯誤處理（個人，基礎）

回想 Week 03 學的錯誤處理，為以下兩個節點加入錯誤路線：

1. **HTTP GET 取得結果** 節點：如果 ParseHub 還在執行中（回傳資料為空），
   推播 Discord 警告：「ParseHub 執行超時，請手動確認」
2. **Google Sheets Add a Row** 節點：如果寫入失敗，
   推播 Discord 錯誤通知並記錄到錯誤記錄試算表

截圖加入錯誤處理後的完整劇本畫布。

---

#### 【練習 3-B】整合挑戰（分組，2～3 人）

**設計一套「工管系就業市場週報系統」：**

1. **ParseHub** 每週自動採集職缺資料
2. **Google Sheets** 的 `raw_data` 自動更新，`cleaned_data` 自動清洗
3. **Looker Studio** 建立即時儀表板（職缺數趨勢、薪資分布、熱門縣市）
4. **Make** 每週一發送 Discord 週報通知，包含：
    - 本週新增職缺數
    - 本週平均薪資下限
    - 職缺最多的前三個縣市

繳交：Make 劇本截圖 + Looker Studio 儀表板截圖 + Discord 週報截圖

---

#### 【練習 3-C】延伸思考（挑戰）

> ❓ 目前的架構每週執行一次，`raw_data` 會不斷累積新資料。
> 三個月後試算表可能有幾千筆資料，其中有大量重複的職缺（
> 同一個職缺可能出現在多週的資料中）。
>
> 請設計一套「去重機制」，確保同一個職缺只被記錄一次。
> 你的設計方案需要說明：
> 1. 用什麼欄位判斷「是不是同一個職缺」
> 2. 在 Make 的哪個節點加入判斷
> 3. 具體使用哪個 Make 模組或 Google Sheets 公式實現

---

## 🏆 本週總結

### 📊 本週技能地圖
```
【爬蟲概念】
    ├── 運作原理：HTTP 請求 → HTML 解析 → 結構化資料
    ├── 法律邊界：robots.txt、服務條款、使用目的
    └── ParseHub 定位：No-Code 視覺化爬蟲工具

        ↓ 設定好爬蟲

【資料清洗（外部資料版）】
    ├── 公司名稱：去除括號說明
    ├── 地點：REGEXEXTRACT 只取縣市
    ├── 薪資：REGEXEXTRACT 抽取數字
    ├── 日期：相對時間轉絕對日期
    └── 資料品質標記：自動評估完整性

        ↓ 資料乾淨後

【全自動化採集】
    ├── ParseHub API：/run 觸發 + /results 取得
    ├── Make 七節點劇本：Schedule → HTTP × 2 → Sleep → Iterator → Sheets → Discord
    └── 每週定時自動執行，無需人工介入
```

### 🔄 本週和前後週的連結
```
Week 05（API 深潛）：學了如何閱讀 API 文件、使用 Hoppscotch
    ↓ 本週直接應用
    → 用同樣的 SOP 閱讀 ParseHub API 文件並測試

Week 06（資料庫設計）：學了資料清洗的十個公式
    ↓ 本週深化
    → 爬蟲資料的骯髒問題更複雜，需要 REGEXEXTRACT 等進階公式

Week 08（下週）：AI 情緒分析
    ↓ 本週是前置準備
    → 把職缺的「工作描述」文字欄位送給 Gemini 分析
      需要的技能關鍵字，自動分類職缺等級
```

### 🤔 課後思考問題

1. **道德邊界：** ParseHub 免費版每個專案限制爬取 200 頁。
   如果你用付費版大量爬取某個求職網站的所有職缺，
   存起來做商業分析，這在法律和道德上各有什麼疑慮？

2. **資料新鮮度：** 你的職缺資料每週更新一次，
   但有些職缺可能在週一爬下來後，週三就已經下架了。
   這種「資料新鮮度」問題在工管的哪些應用場景中會造成嚴重後果？
   你有什麼設計方案可以減輕這個問題？

3. **爬蟲 vs API：** 104 人力銀行有提供官方的合作 API 給企業使用。
   假設你可以選擇，你會優先用官方 API 還是爬蟲？
   兩種方式各自的優缺點是什麼？

### 📚 本週自學資源

| 資源 | 用途 |
|------|------|
| [ParseHub 官方教學](https://www.parsehub.com/quickstart) | ParseHub 完整操作說明 |
| [ParseHub API 文件](https://www.parsehub.com/docs/ref/api/v2/) | API 端點完整說明 |
| [REGEXEXTRACT 說明](https://support.google.com/docs/answer/3098244) | 正規表達式抽取資料 |
| [Public APIs（資料來源）](https://github.com/public-apis/public-apis) | 合法的公開 API 替代爬蟲 |

---

### 📝 下週預告

> 🔜 **Week 08：AI 文字分析 — Gemini × Make 情緒辨識與自動分類**
>
> - Gemini API 的 Prompt Engineering：怎麼問才能得到結構化回答
> - 用 AI 分析職缺描述，自動分類「技能要求等級」
> - Make Router 根據 AI 分類結果分流處理
> - 建立「AI 輔助職缺篩選系統」，讓學生找到最適合的職缺
>
> **課前任務：**
> 確認你的 Google AI Studio 帳號可以正常登入，
> 並取得一個 Gemini API Key（免費，前往 [aistudio.google.com](https://aistudio.google.com/)）。

---

*雲端與數位內容管理 ｜ 國立金門大學 工業工程與管理學系 ｜ 2026 春季學期*
*Week 07 of 14*
