## ⏰ 3 ：AI 賦能雲端 — OCR 視覺辨識 × 收據自動核銷

> 🎯 **本小時目標：** 把 AI 的「眼睛」接入工作流。
> 讓 OCR.space 自動讀取上傳的收據圖片文字，回填試算表，
> 並推播人工覆核通知到 Discord——徹底取代人工核對的苦差事。

---

### 💡 觀念講解：OCR 是什麼？

**OCR（Optical Character Recognition，光學字元辨識）** 是把圖片裡的文字轉成電腦可以讀取的文字的技術。

| 輸入 | 輸出 |
|------|------|
| 收據圖片 | `柯文哲競選總部 110元` |
| 匯款截圖 | `王大明 轉帳 1500元` |
| 發票照片 | `統一編號 48776781` |

> [!NOTE]
> **OCR 辨識品質取決於圖片清晰度：**
> - ✅ 電子轉帳截圖、網銀畫面 → 辨識率接近 100%
> - ⚠️ 手寫字、印章、格線干擾 → 辨識率較低
>
> 這就是為什麼系統設計要加入「人工覆核」環節。
> **自動化負責苦力，人類負責最終決策。**

---

### 💡 本小時完整流程
```
[ Google 表單上傳收據圖片 ]
          │
          ▼
[ Google Sheets Watch Rows ]  ← Make 監控新報名
          │
          ▼
[ HTTP - Download a File ]    ← 下載收據圖片
          │
          ▼
[ HTTP - OCR.space ]          ← OCR 辨識圖片文字
          │
          ▼
[ JSON - Parse JSON ]         ← 解析辨識結果
          │
          ▼
[ Google Sheets Update Row ]  ← 回填 AI 判讀結果
          │
          ▼
[ Discord 通知 ]              ← 推播人工覆核警報
```

---

### 📌 任務 3-1：在表單加入收據上傳功能

1. 開啟 `金門聚落文化營 2026 — 線上報名表`
2. 新增題目，題型選「**檔案上傳**」，設定如下：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | 題目 | `請上傳匯款收據截圖（圖片格式）` |
    | 允許的檔案類型 | 勾選「**圖片**」 |
    | 最大檔案數量 | 1 |
    | 最大檔案大小 | 10 MB |

<img width="840" height="693" alt="image" src="https://github.com/user-attachments/assets/186ed59e-1da4-43ab-aa83-43a5aca3d50d" />

3. 建立測試資料：找一張收據或轉帳截圖，填寫表單並上傳，送出

<img width="1102" height="597" alt="image" src="https://github.com/user-attachments/assets/ee2278b5-c673-4a70-a5ef-49f743883e61" />

4. 確認試算表出現一個 Google Drive 連結，格式如下：
```
    https://drive.google.com/open?id=xxxxx
```

<img width="400" height="379" alt="image" src="https://github.com/user-attachments/assets/89da4031-76e4-4c42-966d-fdbb0d92cfae" />

---

### 📌 任務 3-2：設定 Google Drive 資料夾公開權限

> [!WARNING]
> Google 表單上傳的圖片預設是私人的，Make 無法直接存取。
> 需要先把資料夾設為公開，這個步驟只需要做一次。

1. 開啟 [Google Drive](https://drive.google.com)
2. 找到表單自動建立的上傳資料夾，名稱類似：
```
    金門聚落文化營 2026 — 線上報名表（檔案回覆）
```
3. 右鍵 →「**共用**」
4. 「**一般存取權**」從「**受限**」改成「**知道連結的人**」
5. 右側角色確認是「**檢視者**」
6. 點擊「**完成**」

<img width="493" height="429" alt="image" src="https://github.com/user-attachments/assets/d22d13ee-8a8d-47c3-bb08-5915f035810c" />

---

### 📌 任務 3-3：申請 OCR.space 免費 API Key

**工具：** [OCR.space](https://ocr.space/ocrapi/freekey)（完全免費，不需要信用卡）

1. 前往 [ocr.space/ocrapi/freekey](https://ocr.space/ocrapi/freekey)
2. 填入你的 **Email**（其他欄位可以不填）
3. 點擊送出
4. 去信箱收第一封驗證信 → 點擊確認連結
5. 再收第二封信 → 裡面有你的 **API Key**，格式如下：
```
    K8xxxxxxxxxxxxxxxxx
```

<img width="799" height="430" alt="image" src="https://github.com/user-attachments/assets/1c782b1f-c6be-4968-8630-2a367c0918cf" />

---

### 📌 任務 3-4：Make 串接完整 OCR 流程

> [!NOTE]
> 建議新建一個劇本專門處理「收據辨識核銷流程」，
> 與第 2 小時的「VIP 警報流程」分開管理。

---

#### 節點一：Google Sheets Watch Rows

設定同第 2 小時，**不需要過濾器**，所有新報名都要處理。

| 設定項目 | 填入內容 |
|---------|---------|
| Spreadsheet | `金門聚落文化營_報名總表` |
| Sheet | `表單回覆 1` |
| Limit | `2` |

<img width="649" height="857" alt="image" src="https://github.com/user-attachments/assets/fa759969-e22f-4a01-b66a-525c7aa21ea1" />
<img width="366" height="556" alt="image" src="https://github.com/user-attachments/assets/b827412b-ca53-46c6-9347-e40ce8b4e252" />

---

#### 節點二：HTTP - Download a File（下載收據圖片）

1. 新增「**HTTP**」→「**Get a File**」
2. 設定如下：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | URL | 見下方 |
    | Method | `GET` |

3. **URL 欄位**輸入以下公式（把 Google Drive 連結轉換成可直接下載的格式）：
```
    https://drive.google.com/uc?export=download&id={{substring(1.`請上傳匯款收據截圖（圖片格式）`; 33)}}
```

    > [!NOTE]
    > **公式說明：**
    > - `https://drive.google.com/open?id=` 這段文字共 33 個字元
    > - `substring(...; 33)` 從第 33 個字元開始取到結尾，只留下純 ID
    > - 組裝成 `uc?export=download&id=純ID` 格式，才能直接下載圖片

<img width="1051" height="400" alt="image" src="https://github.com/user-attachments/assets/728b54eb-c9cd-4aa8-bb5f-e208358a5609" />

---

#### 節點三：HTTP - Make a Request（OCR.space 辨識）

1. 新增「**HTTP**」→「**Make a Request**」
2. 設定如下：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | URL | `https://api.ocr.space/parse/image` |
    | Method | `POST` |
    | Authentication type | `No authentication` |
    | Body content type | `multipart/form-data` |

<img width="658" height="694" alt="image" src="https://github.com/user-attachments/assets/d680696f-437c-42c3-9392-416d05e4496e" />

3. **Body content** 新增三個 Field：

    **Field 1：API Key**

    | Name | Field type | Value |
    |------|-----------|-------|
    | `apikey` | Text | 你的 OCR.space API Key |

    **Field 2：圖片檔案**

    | Name | Field type | File | File name |
    |------|-----------|------|-----------|
    | `file` | File | HTTP Download a File 的 `Data` 變數 | `receipt.jpg` |

    **Field 3：語言設定**

    | Name | Field type | Value |
    |------|-----------|-------|
    | `language` | Text | `cht` |

4. **Parse response** → 選 `Yes`

<img width="395" height="713" alt="image" src="https://github.com/user-attachments/assets/66d626e8-4967-4d1c-bdb4-90cb7e46f68b" />
<img width="382" height="594" alt="image" src="https://github.com/user-attachments/assets/1c0c1d30-4baf-4d89-a915-1798ccfb88c9" />

---

#### 節點四：JSON - Parse JSON（解析辨識結果）

1. 新增「**JSON**」→「**Parse JSON**」
2. **JSON string** 欄位：帶入 OCR.space 節點傳來的 `Data` 變數

<img width="688" height="354" alt="image" src="https://github.com/user-attachments/assets/7e0a9314-42a1-45db-ad0e-e1b5e6f9d272" />

---

#### 節點五：Google Sheets Update Row（回填辨識結果）

1. 先回到 Google 試算表，手動新增一欄：`AI判讀結果`

<img width="554" height="147" alt="image" src="https://github.com/user-attachments/assets/e7312dea-cb2e-44aa-b0a1-d997e8526a66" />

2. 新增「**Google Sheets**」→「**Update a Row**」
3. 設定如下：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | Spreadsheet | `金門聚落文化營_報名總表` |
    | Sheet | `表單回覆 1` |
    | Row Number | 節點一 Watch Rows 傳來的 `Row number` 變數 |
    | AI判讀結果（欄位） | Parse JSON 傳來的 `ParsedResults[].ParsedText` 變數 |

<img width="946" height="926" alt="image" src="https://github.com/user-attachments/assets/c7853542-96d3-498c-9f83-3f9c3c71b958" />
<img width="1782" height="357" alt="image" src="https://github.com/user-attachments/assets/b6071f83-3e5a-4595-8a28-b0524d784084" />
<img width="537" height="336" alt="image" src="https://github.com/user-attachments/assets/fd8a957d-9abd-4c83-9f69-35b97debf122" />

---

#### 節點六：Discord 人工覆核通知

1. 新增「**Discord**」→「**Send a Message by Webhook Bot**」
2. Webhook URL 填入你的 Discord Webhook URL
3. **Message 欄位**輸入：
```
    💰 **【收據辨識通知】**

    系統已自動辨識新報名者的匯款收據，請人工覆核。

    ▸ **報名者：** {{姓名}}
    ▸ **場次：** {{參加場次}}

    **OCR 辨識結果：**
    {{ParsedText}}

    ⚠️ 辨識結果僅供參考，請開啟試算表確認金額是否正確。
    **黃金原則：自動化負責苦力，人類負責最終決策。**
```

<img width="385" height="543" alt="image" src="https://github.com/user-attachments/assets/ce26d65c-a711-420e-9267-1fe36d825e66" />
<img width="618" height="773" alt="image" src="https://github.com/user-attachments/assets/8193518b-92a1-4040-bfdd-80e00015d87c" />
<img width="1766" height="334" alt="image" src="https://github.com/user-attachments/assets/ea1cbc5b-91aa-418c-9879-3cd464529307" />
<img width="350" height="369" alt="image" src="https://github.com/user-attachments/assets/6995a186-bff6-48ec-a782-0af4adccc32a" />
<img width="594" height="567" alt="image" src="https://github.com/user-attachments/assets/01eedabf-0822-4c73-a066-f7e62d666e8d" />
<img width="1775" height="293" alt="image" src="https://github.com/user-attachments/assets/d323edf8-ae65-48c2-a95e-e94d63f546c6" />
<img width="921" height="843" alt="image" src="https://github.com/user-attachments/assets/06b67185-4930-43ca-be10-efc4e553d8ba" />

---

### 🎯 終極壓力測試！

1. 點擊 Make 左下角「**Run once**」
2. 填寫一筆新表單，上傳一張**電子轉帳截圖**（建議用網銀截圖，辨識率較高）
3. 觀察資料依序流經每個節點：
```
    📋 Google Sheets 抓到新報名資料
         ↓
    📥 HTTP 下載收據圖片
         ↓
    🔍 OCR.space 辨識圖片文字（約 1～3 秒）
         ↓
    📊 JSON 解析完成
         ↓
    📝 試算表 AI判讀結果 欄位自動填入
         ↓
    🔔 Discord 收到人工覆核通知
```

> [!WARNING]
> **🚨 OCR 辨識品質說明**
>
> OCR 辨識品質受圖片品質影響很大：
>
> | 圖片類型 | 預期辨識品質 |
> |---------|------------|
> | 網路銀行轉帳截圖 | ⭐⭐⭐⭐⭐ 幾乎完美 |
> | 電子發票截圖 | ⭐⭐⭐⭐ 很好 |
> | 清晰的印刷收據 | ⭐⭐⭐ 尚可 |
> | 手寫、有印章的收據 | ⭐⭐ 較差 |
>
> **這就是 Discord 人工覆核通知存在的原因。**
> 系統自動辨識節省了人工打字的時間，
> 但最終確認金額的責任還是在人。

> [!TIP]
> **🏆 3  Checkpoint 完成！**
>
> 你串接完成的系統：
> - ✅ 表單上傳圖片 → OCR 自動辨識文字
> - ✅ 辨識結果自動回填試算表
> - ✅ Discord 推播人工覆核通知
> - ✅ 人工保有最終確認權
>
> 這套系統如果對外發包，報價至少 **NT$ 80,000**。
> 你用零成本、零程式碼完成了。

---

### 🎯 3 學生練習題

> [!NOTE]
> **⏱️ 練習時間：20 分鐘｜個人 + 分組討論**

---

#### 【練習 3-A】Prompt 改版：換一張更清楚的收據（個人，基礎）

找一張**網路銀行轉帳截圖**或**超商繳費截圖**重新上傳，
觀察 OCR 辨識結果是否比手寫收據更準確。

截圖比較兩張收據的辨識結果，說明圖片品質對 OCR 的影響。

---

#### 【練習 3-B】擴充 Discord 通知內容（個人，進階）

目前 Discord 通知只顯示 OCR 全文，請修改 Message，
在通知最後加入一個試算表的直接連結，讓覆核人員點一下就能開啟：
```
📊 點此開啟試算表覆核：
https://drive.google.com/file/d/1EuxDcNWbN96EUv9fPMI6rcVZLIvvVM6m/view
```

<img width="535" height="718" alt="image" src="https://github.com/user-attachments/assets/3a54070f-5583-41c0-a7f6-970ef7c86302" />

---

#### 【練習 3-C】期末延伸構想（分組，2～3 人，挑戰）

你們小組要提出「第 14 週期末專題」的初步構想：

1. **選定一個真實情境**（工工系的痛點、金門大學的行政流程、或金門縣的在地問題）
2. **說明使用哪些工具**（至少用到這三週學過的 3 種以上工具）
3. **畫出資料流程圖**（從使用者第一個動作到系統最後輸出，可手繪拍照）

**下週課程最後 10 分鐘，每組 2 分鐘說明構想，老師給初步回饋。**

> [!NOTE]
> **範例構想（僅供參考）**
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
                  [ HTTP 下載圖片 ]
                           ↓
                  [ OCR.space 辨識 ]
                           ↓
              [ 自動回填 + Discord 覆核通知 ]
```

### 💰 價值換算

| 功能項目 | 外包報價估算 | 你的成本 |
|---------|-----------|---------|
| 自動化報名 + PDF 憑證系統 | NT$ 30,000 | 免費 |
| 手機報到 App + 儀表板 | NT$ 70,000 | 免費 |
| Discord 自動警報系統 | NT$ 15,000 | 免費 |
| OCR 收據辨識核銷 | NT$ 50,000 | 免費 |
| **合計** | **NT$ 165,000+** | **NT$ 0** |

### 🤔 課後思考問題

1. **系統瓶頸：** 如果同時有 500 人在同一分鐘送出表單，這套系統哪個環節最可能出問題？如何改善？

2. **OCR 的限制：** 今天的實作中，手寫收據辨識品質很差。除了換清楚的圖片，你還能想到哪些方式改善這個問題？

3. **延伸應用：** 除了收據核銷，「圖片上傳 → OCR 辨識 → 自動入庫」這個流程還可以用在哪些場景？


## 💡 進階補充：用 Google Apps Script 取代 Make

> [!NOTE]
> 這是給對程式有興趣的同學的延伸閱讀。
> 不需要完全看懂，理解「為什麼要這樣做」比「怎麼寫」更重要。

---

### 為什麼要取代 Make？

| | Make 免費版 | Google Apps Script |
|---|---|---|
| 每月操作次數 | 1,000 次 | **無限制** |
| 費用 | 超過要付費 | **完全免費** |
| 執行速度 | 受方案限制 | 毫秒級 |
| 與 Google 整合 | 需要授權設定 | **原生整合，不需要授權** |
| 學習門檻 | 低（拖拉介面） | 中（需要寫簡單程式碼） |

**結論：** 活動規模大、報名人數多時，Apps Script 比 Make 更穩定、更省錢。

---

### Google Apps Script 是什麼？

Google Apps Script 是 Google 提供的**免費雲端程式環境**，語法類似 JavaScript，
可以直接操控所有 Google 服務（試算表、表單、Gmail、Drive 等），不需要架設任何伺服器。
```
Make 的做法：
Google 表單 → Make（第三方平台）→ Discord

Apps Script 的做法：
Google 表單 → Apps Script（住在 Google 裡面）→ Discord
```

---

### 📌 任務：建立 Discord 多頻道自動分流通知系統

#### 步驟一：在 Discord 為每個頻道建立 Webhook

為以下每個頻道各建立一個 Webhook：

| 頻道 | 用途 |
|------|------|
| `#vip-報名警報` | VIP 貴賓報名時通知 |
| `#a場次報到` | A 場次新報名通知 |
| `#b場次報到` | B 場次新報名通知 |
| `#c場次報到` | C 場次新報名通知 |

**每個頻道的操作步驟：**

1. 對頻道點擊「**⚙️ 編輯頻道**」
2. 左側選單「**整合**」→「**Webhook**」
3. 點擊「**新 Webhook**」

   > [!WARNING]
   > 必須點「新 Webhook」自己建立，才能複製 URL。
   > 由 Make 或其他平台建立的 Webhook，Discord 不允許複製網址。

4. 名稱填入對應頻道名稱（例如：`a場次報到機器人`）
5. 確認頻道是正確的
6. 點擊剛建立的 Webhook 展開 →「**複製 Webhook 網址**」
7. 貼到記事本備用

<img width="296" height="203" alt="image" src="https://github.com/user-attachments/assets/e5dbfc74-fd4a-414c-834b-22984ae24caa" />
<img width="1032" height="639" alt="image" src="https://github.com/user-attachments/assets/7fd2ca2f-4f4b-4346-9f2d-72d20eaf8362" />


---

#### 步驟二：開啟 Google Apps Script

1. 開啟 `金門聚落文化營_報名總表` 試算表
2. 上方選單「**擴充功能**」→「**Apps Script**」
3. 會開啟一個程式碼編輯器

<img width="743" height="294" alt="image" src="https://github.com/user-attachments/assets/47de2f3a-affc-4646-91c4-2c6920e2e052" />

---

#### 步驟三：貼入程式碼

清空編輯器原有內容，貼入以下完整程式碼：
```javascript
// ====== 設定區（只需要修改這裡）======
const WEBHOOK_VIP     = "https://discord.com/api/webhooks/vip頻道URL";
const WEBHOOK_SCENE_A = "https://discord.com/api/webhooks/a場次頻道URL";
const WEBHOOK_SCENE_B = "https://discord.com/api/webhooks/b場次頻道URL";
const WEBHOOK_SCENE_C = "https://discord.com/api/webhooks/c場次頻道URL";
// =====================================

// 發送訊息到指定頻道的共用函式
function sendToDiscord(webhookUrl, message) {
  UrlFetchApp.fetch(webhookUrl, {
    method: "post",
    contentType: "application/json",
    payload: JSON.stringify({ content: message })
  });
}

// 表單送出時自動執行
function onFormSubmit(e) {
  const values    = e.values;
  const timestamp = values[0];  // Timestamp
  const name      = values[1];  // 姓名
  const email     = values[2];  // Email信箱
  const scene     = values[3];  // 參加場次
  const phone     = values[4];  // 聯絡電話
  const isVip     = values[5];  // 是否為VIP貴賓

  // VIP 警報 → #vip-報名警報 頻道
  if (isVip === "是") {
    sendToDiscord(WEBHOOK_VIP,
      `🚨 **【VIP 報名警報】** 🚨\n\n` +
      `有貴賓剛剛完成報名，請相關人員注意！\n\n` +
      `▸ **姓名：** ${name}\n` +
      `▸ **場次：** ${scene}\n` +
      `▸ **信箱：** ${email}\n` +
      `▸ **電話：** ${phone}\n` +
      `▸ **時間：** ${timestamp}\n\n` +
      `請於 30 分鐘內完成接待確認。`
    );
  }

  // 場次分流 → 各場次頻道
  const sceneMessage =
    `📋 **【新報名通知】**\n\n` +
    `▸ **姓名：** ${name}\n` +
    `▸ **場次：** ${scene}\n` +
    `▸ **時間：** ${timestamp}`;

  if (scene === "A場：週六上午") {
    sendToDiscord(WEBHOOK_SCENE_A, sceneMessage);
  } else if (scene === "B場：週六下午") {
    sendToDiscord(WEBHOOK_SCENE_B, sceneMessage);
  } else if (scene === "C場：週日全天") {
    sendToDiscord(WEBHOOK_SCENE_C, sceneMessage);
  }
}

// ====== 測試函式（測試完成後可以保留備用）======
function testOnFormSubmit() {
  const fakeEvent = {
    values: [
      "2026/03/24 14:00:00",  // Timestamp
      "王小明",                // 姓名
      "wang@test.com",         // Email信箱
      "C場：週日全天",          // 參加場次
      "0912345678",            // 聯絡電話
      "是"                     // 是否為VIP貴賓
    ]
  };
  onFormSubmit(fakeEvent);
}
```

> [!WARNING]
> **欄位順序很重要！**
> `values[0]`、`values[1]`... 對應的是試算表從左到右的欄位順序。
> 如果你的表單欄位順序不同，請對應調整數字。
> 可以開啟試算表，從左到右數欄位順序來確認。

---

#### 步驟四：填入真實的 Webhook URL

把設定區的四個 `"...URL"` 換成你在步驟一複製的真實網址：
```javascript
const WEBHOOK_VIP     = "https://discord.com/api/webhooks/123456/abcdef...";
const WEBHOOK_SCENE_A = "https://discord.com/api/webhooks/234567/bcdefg...";
const WEBHOOK_SCENE_B = "https://discord.com/api/webhooks/345678/cdefgh...";
const WEBHOOK_SCENE_C = "https://discord.com/api/webhooks/456789/defghi...";
```

點擊上方「**💾 儲存**」（或 `Ctrl+S`）

<img width="1019" height="827" alt="image" src="https://github.com/user-attachments/assets/05daf7af-39d5-4ddf-87c3-af1662c7df13" />
<img width="802" height="420" alt="image" src="https://github.com/user-attachments/assets/11fcef63-9188-471f-ad07-5fd28928d9fe" />

---

#### 步驟五：先用測試函式驗證

在編輯器上方的函式選單，**切換成 `testOnFormSubmit`**，點擊「**▶ 執行**」。

**預期結果：**
- ✅ `#vip-報名警報` 收到 VIP 警報（因為測試資料 isVip = 是）
- ✅ `#c場次報到` 收到新報名通知（因為測試資料場次 = C場）
- ❌ `#a場次報到` 和 `#b場次報到` 不應該收到

<img width="311" height="235" alt="image" src="https://github.com/user-attachments/assets/dd1b5449-cddd-441b-9802-dda8fb012a97" />
<img width="318" height="156" alt="image" src="https://github.com/user-attachments/assets/0ed022bb-cd85-49f1-96d3-0d9867a221cb" />

---

#### 步驟六：設定觸發條件（讓系統自動執行）

> [!NOTE]
> 這步驟完成後，GAS 就會在每次表單送出時**自動執行**，
> 不需要你在場，也不需要手動點執行。

1. 左側選單點擊「**⏰ 觸發條件**」（時鐘圖示）
2. 右下角「**+ 新增觸發條件**」
3. 設定如下：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | 執行的函式 | `onFormSubmit` |
    | 執行部署 | `Head` |
    | 活動來源 | `試算表` |
    | 活動類型 | `提交表單時` |
    | 失敗通知設定 | `每天通知` |

4. 點擊「**儲存**」
5. 跳出 Google 授權視窗 → 選擇你的帳號 → 點擊「**允許**」

<img width="279" height="319" alt="image" src="https://github.com/user-attachments/assets/acafd05a-5d5f-4920-9126-ac13bc0565c1" />
<img width="1311" height="811" alt="image" src="https://github.com/user-attachments/assets/669ea511-7d0f-4761-8cd1-8859df98ed61" />

---

#### 步驟七：真實表單最終測試

1. 開啟報名表單連結
2. 填寫一筆真實資料：
    - 姓名：任意
    - 場次：「**A場：週六上午**」
    - VIP：「**否**」
3. 送出表單

**預期結果：**
- ✅ `#a場次報到` 收到通知
- ❌ `#vip-報名警報` 不收到（因為不是 VIP）

<img width="320" height="153" alt="image" src="https://github.com/user-attachments/assets/09758096-bfd7-4552-8815-18ae9a1b1932" />

---

### 程式碼邏輯對照表

| GAS 程式碼 | 對應的 Make 功能 |
|-----------|---------------|
| `const values = e.values` | Google Sheets Watch Rows 取得資料 |
| `if (isVip === "是")` | Make Filter 過濾器 |
| `sendToDiscord(WEBHOOK_VIP, ...)` | Discord Send Message 節點 |
| `if / else if` 場次判斷 | Make Router 多路分流 |
| 觸發條件「提交表單時」 | Make Scenario 的 Form Trigger |

> [!TIP]
> **你會發現：**
> Make 的每個「節點」，背後都對應著幾行程式碼。
> 學會 Make 之後再來看程式碼，會發現其實沒有那麼難——
> 因為你已經理解了背後的邏輯。
>
> **No-Code 工具是學習程式邏輯最好的跳板。**

---

### 為什麼要取代 Make？

| | Make 免費版 | Google Apps Script |
|---|---|---|
| 每月操作次數 | 1,000 次 | **無限制** |
| 費用 | 超過要付費 | **完全免費** |
| 執行速度 | 受方案限制 | 毫秒級 |
| 與 Google 整合 | 需要授權設定 | **原生整合，不需要授權** |
| 學習門檻 | 低（拖拉介面） | 中（需要寫簡單程式碼） |

**結論：** 活動規模大、報名人數多時，Apps Script 比 Make 更穩定、更省錢。


---

---

## 💡 進階補充 Part 2：GAS 整合 OCR.space 收據辨識

> [!NOTE]
> 本節在 Part 1 的基礎上，加入 OCR 收據辨識功能。
> OCR 觀念、Drive 資料夾公開設定、OCR.space API Key 申請方式
> 請參考本章「第 3 小時：Make 串接 OCR 流程」的說明，這裡不再重複。
>
> 完成後整個系統**完全不需要 Make**，一支 GAS 程式搞定所有事。

---

### 與 Part 1 的差異

| | Part 1（基礎版） | Part 2（完整版） |
|---|---|---|
| VIP 警報 | ✅ | ✅ |
| 場次分流 | ✅ | ✅ |
| OCR 收據辨識 | ❌ | ✅ |
| 自動判斷 PDF / 圖片 | ❌ | ✅ |
| 回填試算表 AI判讀結果 | ❌ | ✅ |
| Discord #ai辨識 覆核通知 | ❌ | ✅ |

---

### 完整流程
```
表單送出
    │
    └→ GAS 自動執行
          │
          ├→ VIP 警報 → #vip-報名警報
          ├→ 場次分流 → #a/#b/#c 場次報到
          └→ OCR 辨識收據
                │
                ├→ 自動判斷檔案類型（PDF / PNG / JPG）
                ├→ 回填試算表 AI判讀結果欄位
                └→ 覆核通知 → #ai辨識
```

---

### 📌 步驟一：為 #ai辨識 頻道建立 Webhook

在 Part 1 已建立的四個 Webhook 基礎上，再為 `#ai辨識` 頻道新增一個：

1. 對 `#ai辨識` 頻道點擊「**⚙️ 編輯頻道**」
2. 「**整合**」→「**Webhook**」→「**新 Webhook**」
3. 名稱填：`OCR辨識機器人`
4. 展開後「**複製 Webhook 網址**」備用

> [!WARNING]
> 必須點「**新 Webhook**」自己建立，才能複製 URL。
> 由 Make 或其他平台建立的 Webhook，Discord 不允許複製網址。

<!-- 📸 截圖：#ai辨識 頻道 Webhook 設定完成，可複製網址 -->

---

### 📌 步驟二：確認試算表有 AI判讀結果 欄位

第 3 小時 Make 版已在試算表新增了 `AI判讀結果` 欄位，
請確認該欄位是**第幾欄**（A=1, B=2...），填入程式碼的 `OCR_COLUMN` 設定。

<!-- 📸 截圖：試算表 AI判讀結果 欄位，確認是第幾欄 -->

---

### 📌 步驟三：用完整版程式碼取代 Part 1

開啟 Apps Script，**清空全部內容**，貼入以下完整版程式碼：
```javascript
// ====== 設定區（只需要修改這裡）======
const WEBHOOK_VIP     = "https://discord.com/api/webhooks/vip頻道URL";
const WEBHOOK_AI_OCR  = "https://discord.com/api/webhooks/ai辨識頻道URL";
const WEBHOOK_SCENE_A = "https://discord.com/api/webhooks/a場次頻道URL";
const WEBHOOK_SCENE_B = "https://discord.com/api/webhooks/b場次頻道URL";
const WEBHOOK_SCENE_C = "https://discord.com/api/webhooks/c場次頻道URL";
const OCR_API_KEY     = "你的OCR.space API Key";
const SPREADSHEET_ID  = "試算表網址中 /d/ 和 /edit 之間的那串字";
const SHEET_NAME      = "表單回覆 1";
const OCR_COLUMN      = 10;  // AI判讀結果欄位是第幾欄（A=1, B=2...）
// =====================================

// 發送訊息到 Discord 指定頻道
function sendToDiscord(webhookUrl, message) {
  UrlFetchApp.fetch(webhookUrl, {
    method: "post",
    contentType: "application/json",
    payload: JSON.stringify({ content: message })
  });
}

// 下載檔案並自動判斷類型，呼叫 OCR.space 辨識
function runOcr(driveUrl) {

  // 取出純 ID
  const fileId = driveUrl.replace("https://drive.google.com/open?id=", "");
  const downloadUrl = "https://drive.google.com/uc?export=download&id=" + fileId;

  // 下載檔案
  let fileResponse = UrlFetchApp.fetch(downloadUrl, {
    followRedirects: true,
    muteHttpExceptions: true
  });

  // 檢查是否跳出確認頁面（大檔案才會有）
  let content = fileResponse.getContentText();
  if (content.includes("confirm=")) {
    const confirmMatch = content.match(/confirm=([^&"]+)/);
    if (confirmMatch) {
      const confirmUrl = downloadUrl + "&confirm=" + confirmMatch[1];
      fileResponse = UrlFetchApp.fetch(confirmUrl, {
        followRedirects: true,
        muteHttpExceptions: true
      });
    }
  }

  // 自動判斷檔案類型
  const contentType = fileResponse.getHeaders()["Content-Type"] || "";
  let fileName;

  if (contentType.includes("pdf")) {
    fileName = "receipt.pdf";
  } else if (contentType.includes("png")) {
    fileName = "receipt.png";
  } else if (contentType.includes("jpeg") || contentType.includes("jpg")) {
    fileName = "receipt.jpg";
  } else {
    // 無法從 Content-Type 判斷時，從檔案內容開頭猜測
    const firstBytes = fileResponse.getContentText().substring(0, 5);
    if (firstBytes.startsWith("%PDF")) {
      fileName = "receipt.pdf";
    } else {
      fileName = "receipt.jpg";  // 預設當成 jpg 處理
    }
  }

  Logger.log("檔案類型：" + contentType + "，使用檔名：" + fileName);

  // 轉成對應的 Blob
  const fileBlob = fileResponse.getBlob().setName(fileName);

  // 呼叫 OCR.space API
  const ocrResponse = UrlFetchApp.fetch("https://api.ocr.space/parse/image", {
    method: "post",
    payload: {
      apikey: OCR_API_KEY,
      file: fileBlob,
      language: "cht",
      scale: true,
      OCREngine: "2"
    },
    muteHttpExceptions: true
  });

  // 解析回傳結果
  const result = JSON.parse(ocrResponse.getContentText());

  if (result.IsErroredOnProcessing) {
    return "OCR 辨識失敗：" + result.ErrorMessage;
  }

  return result.ParsedResults[0].ParsedText;
}

// 把 OCR 結果回填到試算表
function updateSheet(rowNumber, ocrText) {
  const ss = SpreadsheetApp.openById(SPREADSHEET_ID);
  const sheet = ss.getSheetByName(SHEET_NAME);
  sheet.getRange(rowNumber, OCR_COLUMN).setValue(ocrText);
}

// 表單送出時自動執行（主函式）
function onFormSubmit(e) {
  const values      = e.values;
  const rowNumber   = e.range.getRow();   // 取得這筆資料在試算表的列號
  const timestamp   = values[0];          // Timestamp
  const name        = values[1];          // 姓名
  const email       = values[2];          // Email信箱
  const scene       = values[3];          // 參加場次
  const phone       = values[4];          // 聯絡電話
  const isVip       = values[5];          // 是否為VIP貴賓
  const receiptUrl  = values[6];          // 請上傳匯款收據截圖

  // ── 1. VIP 警報 ──────────────────────────
  if (isVip === "是") {
    sendToDiscord(WEBHOOK_VIP,
      `🚨 **【VIP 報名警報】** 🚨\n\n` +
      `有貴賓剛剛完成報名，請相關人員注意！\n\n` +
      `▸ **姓名：** ${name}\n` +
      `▸ **場次：** ${scene}\n` +
      `▸ **信箱：** ${email}\n` +
      `▸ **電話：** ${phone}\n` +
      `▸ **時間：** ${timestamp}\n\n` +
      `請於 30 分鐘內完成接待確認。`
    );
  }

  // ── 2. 場次分流通知 ───────────────────────
  const sceneMessage =
    `📋 **【新報名通知】**\n\n` +
    `▸ **姓名：** ${name}\n` +
    `▸ **場次：** ${scene}\n` +
    `▸ **時間：** ${timestamp}`;

  if (scene === "A場：週六上午") {
    sendToDiscord(WEBHOOK_SCENE_A, sceneMessage);
  } else if (scene === "B場：週六下午") {
    sendToDiscord(WEBHOOK_SCENE_B, sceneMessage);
  } else if (scene === "C場：週日全天") {
    sendToDiscord(WEBHOOK_SCENE_C, sceneMessage);
  }

  // ── 3. OCR 收據辨識 ───────────────────────
  // 只有上傳了收據才執行
  if (receiptUrl && receiptUrl.includes("drive.google.com")) {

    const ocrText = runOcr(receiptUrl);
    updateSheet(rowNumber, ocrText);

    sendToDiscord(WEBHOOK_AI_OCR,
      `💰 **【收據辨識通知】**\n\n` +
      `系統已自動辨識新報名者的匯款收據，請人工覆核。\n\n` +
      `▸ **報名者：** ${name}\n` +
      `▸ **場次：** ${scene}\n\n` +
      `**OCR 辨識結果：**\n` +
      `${ocrText}\n\n` +
      `⚠️ 辨識結果僅供參考，請開啟試算表確認金額是否正確。\n` +
      `**黃金原則：自動化負責苦力，人類負責最終決策。**`
    );
  }
}

// ====== 測試函式 ======
function testOnFormSubmit() {
  const fakeEvent = {
    range: { getRow: () => 2 },
    values: [
      "2026/03/24 14:00:00",
      "王小明",
      "wang@test.com",
      "C場：週日全天",
      "0912345678",
      "是",
      "https://drive.google.com/open?id=你的測試收據圖片ID"
    ]
  };
  onFormSubmit(fakeEvent);
}

// ====== 除錯函式（OCR 失敗時使用）======
function debugDownload() {
  const driveUrl = "https://drive.google.com/open?id=你的測試收據圖片ID";
  const fileId = driveUrl.replace("https://drive.google.com/open?id=", "");
  const downloadUrl = "https://drive.google.com/uc?export=download&id=" + fileId;

  const response = UrlFetchApp.fetch(downloadUrl, {
    followRedirects: true,
    muteHttpExceptions: true
  });

  Logger.log("Status code: " + response.getResponseCode());
  Logger.log("Content type: " + response.getHeaders()["Content-Type"]);
  Logger.log("Content length: " + response.getContent().length);
  Logger.log("First 500 chars: " + response.getContentText().substring(0, 500));
}
```

> [!WARNING]
> **兩個地方需要特別確認：**
>
> 1. `values[6]` 對應的是收據上傳欄位，
>    如果你的表單欄位順序不同，請對應調整數字。
>
> 2. `SPREADSHEET_ID` 是試算表網址中 `/d/` 和 `/edit` 之間的那串字：
>    ```
>    https://docs.google.com/spreadsheets/d/【這裡就是ID】/edit#gid=874013079
>    ```
>    注意：`gid=` 後面的數字是工作表編號，不是試算表 ID。

<!-- 📸 截圖：Apps Script 完整版程式碼畫面 -->

---

### 📌 步驟四：測試

#### 先用測試函式驗證

1. 把 `testOnFormSubmit` 裡的圖片 ID 換成試算表裡真實的收據圖片 ID
2. 函式選單切換到 `testOnFormSubmit` → 點「**▶ 執行**」
3. 確認以下全部正確：

    | 檢查項目 | 預期結果 |
    |---------|---------|
    | `#vip-報名警報` | ✅ 收到 VIP 警報 |
    | `#c場次報到` | ✅ 收到場次通知 |
    | `#ai辨識` | ✅ 收到 OCR 覆核通知 |
    | 試算表 `AI判讀結果` 欄 | ✅ 自動填入辨識文字 |

<!-- 📸 截圖：Discord 四個頻道同時收到通知的畫面 -->
<!-- 📸 截圖：試算表 AI判讀結果欄位自動填入的畫面 -->

---

#### 若 OCR 失敗，先執行除錯函式

1. 把 `debugDownload` 裡的圖片 ID 換成真實 ID
2. 函式選單切換到 `debugDownload` → 點「**▶ 執行**」
3. 點擊下方「**執行記錄**」，確認輸出：

    | Content type 顯示 | 代表意思 |
    |------------------|---------|
    | `application/pdf` | 檔案是 PDF，程式會自動處理 ✅ |
    | `image/jpeg` | 檔案是 JPG 圖片 ✅ |
    | `text/html` | 下載失敗，檢查 Drive 資料夾是否已設為公開 ❌ |

<!-- 📸 截圖：執行記錄顯示檔案類型判斷結果 -->

---

#### 觸發條件不需要重新設定

Part 1 已設定好「提交表單時執行 `onFormSubmit`」，
完整版主函式名稱相同，**不需要重新設定觸發條件**。

---

### 為什麼 GAS 能自動判斷 PDF 或圖片？

> [!NOTE]
> Make 版本需要手動在 Field 2 填入 `receipt.jpg` 作為檔案名稱。
> GAS 版本則會自動偵測，不需要人工判斷：
>
> ```
> 下載檔案
>     │
>     ├→ Content-Type 包含 "pdf"          → receipt.pdf
>     ├→ Content-Type 包含 "png"          → receipt.png
>     ├→ Content-Type 包含 "jpeg"/"jpg"   → receipt.jpg
>     └→ 以上都不符合，看檔案開頭：
>           ├→ 開頭是 "%PDF"              → receipt.pdf
>           └→ 其他                       → receipt.jpg（預設）
> ```
>
> Google Drive 有時會把圖片轉成 PDF 格式儲存，
> 自動判斷讓系統不管使用者上傳什麼格式都能正確處理。

---

### 完整程式碼對照表：GAS vs Make

| GAS 程式碼 | 對應的 Make 功能 |
|-----------|----------------|
| `const values = e.values` | Google Sheets Watch Rows 取得資料 |
| `if (isVip === "是")` | Make Filter 過濾器 |
| `sendToDiscord(WEBHOOK_VIP, ...)` | Discord Send Message 節點 |
| `if / else if` 場次判斷 | Make Router 多路分流 |
| `runOcr(receiptUrl)` | HTTP Download + OCR.space 節點 |
| `updateSheet(rowNumber, ocrText)` | Google Sheets Update Row 節點 |
| 觸發條件「提交表單時」 | Make Scenario 的 Form Trigger |

---

### Make vs Apps Script 最終選擇建議

| 情境 | 建議工具 |
|------|---------|
| 報名人數 < 200 人 | Make 免費版就夠 |
| 報名人數 > 200 人 | Apps Script |
| 需要串接非 Google 的服務 | Make（支援 1,000+ 種服務）|
| 所有流程都在 Google 生態系內 | Apps Script（更快更穩）|
| 完全不想寫任何程式碼 | Make |
| 想學一點程式基礎 | Apps Script |

> [!TIP]
> **學習路徑建議：**
> ```
> Make（理解自動化邏輯，No-Code 入門）
>     ↓
> GAS Part 1（VIP 警報 + 場次分流，看懂程式碼和 Make 的對應關係）
>     ↓
> GAS Part 2（加入 OCR，學會呼叫外部 API 與處理檔案）
>     ↓
> 期末專題：自選情境，自己設計整套系統
> ```

### 📚 課後自學資源

| 資源 | 用途 |
|------|------|
| [OCR.space API 文件](https://ocr.space/ocrapi) | 所有 API 參數說明 |
| [Make 官方教學](https://www.make.com/en/help) | 所有模組的使用說明 |
| [Discord Webhook 說明](https://support.discord.com/hc/en-us/articles/228383668) | Webhook 進階設定 |

---

### 📝 下週預告

> 🔜 **Week 02：Google 表單魔法 — 從問卷到自動化文件工廠**
>
> - 條件跳題邏輯：表單根據填答者選擇自動跳到不同題組
> - 資料驗證：用正規表達式阻擋不合格的填答
> - 測驗模式：自動批改、即時回饋、統計全班答題狀況
> - AutoCrat 批次模式：一鍵產出 100 份個人化 PDF
> - Google 試算表 10 個核心公式
>
> **課前任務：** 確認你的 Google 帳號能正常登入 Google 表單和試算表。

---

*雲端與數位內容管理 ｜ 國立金門大學 工業工程與管理學系 ｜ 2026 春季學期*
*Week 01 of 14*
