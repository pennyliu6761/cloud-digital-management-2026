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
> **🏆 第 3 小時 Checkpoint 完成！**
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

### 🎯 第 3 小時學生練習題

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
https://docs.google.com/spreadsheets/d/你的試算表ID
```

截圖 Discord 收到的新格式通知。

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
