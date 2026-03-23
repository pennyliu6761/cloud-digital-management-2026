# 🚀 20260318：雲端大腦建置、API 串接與 AI 視覺自動化

> [!NOTE]
> **課程理念：** 今天你們不是在學「工具怎麼用」，而是在學「如何讓工具自己工作」。  
> 一個能自動運轉的數位系統，比一百個只會用工具的人更有競爭力。

## 🎯 本週學習目標

學完本週，你將能夠：
- ✅ 建立符合企業規格的雲端資料夾架構與權限控管
- ✅ 讓 Google 表單在有人填寫後，自動寄出個人化 PDF 憑證
- ✅ 用手機 App 執行現場報到，數據即時顯示在視覺化儀表板

## 🏢 本週實作情境：【金門聚落文化體驗營】

> [!NOTE]
> 💼 **你的角色：** 你是一位剛接到任務的專案經理。老闆要你在本週末前，為一場「百人規模的金門聚落文化體驗營」建立一套完整的數位報名與核銷系統，**預算為零、不能請工程師、不能寫任何程式碼。**

> [!TIP]
> 📌 **系統需求清單（今日任務）：**
> 1. 報名表單 → 自動寄出個人化 PDF 入場憑證
> 2. 現場工作人員用手機掃描點名，後台即時看到報到率

---

## ⏰ 第 1 小時：雲端大腦 × 自動報到閉環
> [!NOTE]
> 🎯 **本小時目標：** 從零開始建立「單一真理來源 (Single Source of Truth, SSOT)」，讓所有資料流都從同一個地方出發、回流。

---

### 📌 任務 1-1：建立「防洩漏」雲端專案大腦

**工具：** Google Drive（免費）

#### 步驟說明

1. **建立主資料夾**
   登入 Google Drive，點擊左上角「**+ 新增**」→「**新資料夾**」，命名為：

    ```
    2026_金門聚落文化營_專案大腦
    ```

2. **建立三層子資料夾**（這是企業標準的 **3-Stage 工作流**）

    在主資料夾內建立以下三個子資料夾：

    | 資料夾名稱 | 用途說明 |
    |-----------|---------|
    | `01_參考素材` | 蒐集到的靈感、範例、原始資料 |
    | `02_執行草稿` | 工作中的文件，可能頻繁修改 |
    | `03_最終定稿` | 對外發送或存檔的版本，不隨意更動 |
    
    <img width="528" height="393" alt="image" src="https://github.com/user-attachments/assets/46641a7e-88c4-4ff4-9497-bf9e59f4a45f" />


3. **建立第一份工作文件（命名有學問！）**

   進入 `02_執行草稿`，新增一份 Google 文件，命名為：
   ```
   20260317_聚落營隊_對外報價單_v1
   ```
   
   > [!TIP]
   > 💡 **命名公式解析：**
   > `YYYYMMDD` + `_` + `專案名稱` + `_` + `文件用途` + `_` + `版本號`
   > 這個命名規則在任何公司都通用，讓你三年後也能秒找到檔案！
   
   <img width="703" height="291" alt="image" src="https://github.com/user-attachments/assets/95cfc277-f762-4e76-8183-f20c41651970" />


4. **設定機密防洩漏權限**

    - 點擊文件右上角「**共用**」
    <img width="1816" height="310" alt="image" src="https://github.com/user-attachments/assets/b4e8145d-c774-470d-8bf2-2a01e5637b6a" />

    - 在「新增使用者」欄位輸入同學信箱，角色設定為「**加註者**」（可以留言但無法編輯，更無法下載）
    <img width="498" height="408" alt="image" src="https://github.com/user-attachments/assets/d5f349c5-a670-46b5-b7e8-3086618dad57" />

    - 點擊右上角「**⚙️ 設定**」→ **取消勾選** 以下兩個選項：
        - `檢視者和加註者可以看見下載、列印和複製選項`
        - `編輯者可以變更權限並新增使用者`
    
        <img width="485" height="373" alt="image" src="https://github.com/user-attachments/assets/a1c42c0f-53a8-4108-9381-2e03627907ad" />
    > [!WARNING]
    > 🔒 **為什麼這很重要？**
    > 企業常見資料外洩場景：員工把「可下載」的投標文件分享給外部廠商，對方一鍵下載後直接轉賣競爭對手。這個設定就是防火牆。

**實戰**
1. 擁有者建立內容
<img width="1903" height="480" alt="image" src="https://github.com/user-attachments/assets/d71acfc2-e6da-40da-9a0d-dfa8a81bb150" />

2. 加註者編輯內容
<img width="1911" height="675" alt="image" src="https://github.com/user-attachments/assets/69e71005-cf21-4310-8aa2-0d5032207a17" />

3. 擁有者接受修訂
<img width="1897" height="660" alt="image" src="https://github.com/user-attachments/assets/dd268cab-f5e1-4e66-ab53-0d42945230dd" />

<img width="631" height="388" alt="image" src="https://github.com/user-attachments/assets/01ff1b2e-10ae-4a52-9f0d-069f25883972" />

4. 擁有者還原版本(災難復原)
<img width="621" height="670" alt="image" src="https://github.com/user-attachments/assets/544b0075-0cc9-43c3-b0e2-d43aeb40fe76" />

<img width="1906" height="629" alt="image" src="https://github.com/user-attachments/assets/32573e26-867b-40c2-91cf-c10da0055bc1" />

<img width="611" height="212" alt="image" src="https://github.com/user-attachments/assets/4819770e-9c96-49b2-9b66-237447f3129a" />

---

### 📌 任務 1-2：AutoCrat — 自動寄出個人化 PDF 報到憑證

**工具：** Google 表單 + Google 文件 + AutoCrat 外掛（均免費）

**🎯 目標效果：** 有人填完表單送出 → 系統自動在 5 分鐘內寄出一份有他名字的 PDF 憑證到信箱。

---

#### 步驟一：建立報名表單

1. 前往 [Google 表單](https://forms.google.com) 建立新表單
2. 標題設定：`金門聚落文化營 2026 — 線上報名表`
3. 新增以下必填題目：

    | 題號 | 題目 | 題型 |
    |------|------|------|
    | Q1 | 姓名 | 簡答 |
    | Q2 | Email 信箱 | 簡答 |
    | Q3 | 參加場次 | 單選（A場：週六上午 / B場：週六下午 / C場：週日全天） |
    | Q4 | 聯絡電話 | 簡答 |
    | Q5 | 是否為 VIP 貴賓？ | 單選（是 / 否） |

    <img width="640" height="886" alt="image" src="https://github.com/user-attachments/assets/1223f07c-5993-4b5d-bea5-2de99dd59f43" />

4. 點擊右上角「📤 傳送」→「🔗 連結」→ 縮短 URL 並複製備用
    <img width="539" height="482" alt="image" src="https://github.com/user-attachments/assets/44fc5726-cf3d-48bd-bf0c-93e16cd7b36f" />

5. 填寫問卷
    <img width="645" height="891" alt="image" src="https://github.com/user-attachments/assets/2c254cfa-1334-40df-b92c-70a09f33956f" />

6. 點擊右上角「**⋮**」→「**回覆**」→ 點擊綠色試算表圖示「**建立試算表**」→ 命名為 `金門聚落文化營_報名總表`
    <img width="788" height="842" alt="image" src="https://github.com/user-attachments/assets/53180fef-734b-45aa-9b47-5d5e1b9e1f0f" />

    <img width="550" height="257" alt="image" src="https://github.com/user-attachments/assets/04185617-cbaa-4d96-b7b3-8b97995096ab" />

    <img width="993" height="288" alt="image" src="https://github.com/user-attachments/assets/40403bb2-136e-4757-94ca-624a74b0df7d" />

---

#### 步驟二：設計憑證模板（含動態標籤）

1. 新建一份 Google 文件，命名為 `【模板】報到憑證`
2. 輸入以下內文（`<<標籤>>` 是 AutoCrat 的特殊語法，等下會自動替換成真實資料）：

<img width="1064" height="717" alt="image" src="https://github.com/user-attachments/assets/061a2111-92fd-4d33-b735-a5dc1bff42d8" />

---

#### 步驟三：安裝並設定 AutoCrat

1. 開啟 `金門聚落文化營_報名總表` 試算表
2. 點擊上方選單「**擴充功能**」→「**外掛程式**」→「**取得外掛程式**」
    <img width="1003" height="321" alt="image" src="https://github.com/user-attachments/assets/9daa5417-e97b-4a67-be58-6ada28dc0aa4" />

3. 搜尋 `AutoCrat`，點擊安裝（需要授權 Google 帳號，點擊允許）
    <img width="692" height="567" alt="image" src="https://github.com/user-attachments/assets/54a9acbf-7dc7-4551-948b-ac4d9921c63b" />

    <img width="457" height="430" alt="image" src="https://github.com/user-attachments/assets/4e295e87-e94e-490e-8e5e-b28000ef5ebf" />

4. 安裝後：「**擴充功能**」→「**AutoCrat**」→「**Open**」
    <img width="1002" height="434" alt="image" src="https://github.com/user-attachments/assets/c79e7f0e-b7ea-41c0-993e-d42ad0f63bed" />


5. 點擊「**+ NEW JOB**」，開始設定：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | Job name | `寄發報到憑證` |
    | Template | 選擇剛剛建立的 `【模板】報到憑證` |
    
    <img width="713" height="458" alt="image" src="https://github.com/user-attachments/assets/ef6ea0f1-ebc3-4c5a-a97e-6a9bb1f400a6" />

    <img width="730" height="470" alt="image" src="https://github.com/user-attachments/assets/1e1fe46b-318e-4f85-bf80-78e62b460df4" />

    <img width="586" height="377" alt="image" src="https://github.com/user-attachments/assets/4b4b2f23-9d73-40dc-948c-f4665b5f9d09" />

    <img width="719" height="262" alt="image" src="https://github.com/user-attachments/assets/ced9eb32-9e64-4c25-863c-22aef9710bdd" />


6. **標籤對應（最關鍵步驟！）**

    系統會自動偵測模板中的 `<<標籤>>`，你需要將每個標籤對應到試算表的欄位：

    | 模板標籤 | 對應試算表欄位 |
    |---------|--------------|
    | `<<姓名>>` | 姓名 |
    | `<<Email信箱>>` | Email 信箱 |
    | `<<參加場次>>` | 參加場次 |
    | `<<Timestamp>>` | Timestamp |

    <img width="717" height="459" alt="image" src="https://github.com/user-attachments/assets/45a512f3-4085-4103-9191-6b49c057914e" />


    | 模板標籤 | 對應試算表欄位 |
    |---------|--------------|
    | Output name | `<<姓名>>_報到憑證_<<Timestamp>>` |
    | Output type | **PDF** |

    <img width="741" height="487" alt="image" src="https://github.com/user-attachments/assets/300d44df-4402-4b4f-9d04-a9471c1879a1" />

    
    | 模板標籤 | 對應試算表欄位 |
    |---------|--------------|
    | Output folder | 選擇 Drive 的 `03_最終定稿` 資料夾 |
    
    <img width="732" height="479" alt="image" src="https://github.com/user-attachments/assets/46608f03-926d-4275-941c-1022734eacb4" />

    <img width="731" height="477" alt="image" src="https://github.com/user-attachments/assets/2f2a949e-3512-47cb-a995-eefce7d39043" />

    <img width="737" height="478" alt="image" src="https://github.com/user-attachments/assets/646546da-d00d-4d04-afcb-06ac15cc4b5d" />



7. **設定收件人與觸發器**
   - **Send emails?** → 選擇 `Yes`
   - **To:** 點擊對應欄位，選擇 `<<Email信箱>>`
   - **Subject:** `【金門聚落文化營】您的報到憑證已送出 — <<姓名>>`
   - **Trigger:** 點擊「**Add trigger**」→ 選擇 `Form trigger`（表單觸發）→ 選擇你的報名表單 → 儲存

    <img width="729" height="471" alt="image" src="https://github.com/user-attachments/assets/3449eaaa-2a6f-4bab-a5c7-5b13a7507e15" />

    <img width="735" height="480" alt="image" src="https://github.com/user-attachments/assets/a42f5ae0-5db4-481a-86f1-b3acc6a402ce" />

    <img width="739" height="474" alt="image" src="https://github.com/user-attachments/assets/6ea062fa-58ab-4dec-b4af-d1a571b41ce2" />

    <img width="732" height="474" alt="image" src="https://github.com/user-attachments/assets/38e3b4f9-0d98-41f8-8da7-6b01a0c65a93" />


---

#### 步驟四：壓力測試

1. 用另一個瀏覽器或手機，**實際填寫並送出** 報名表單
2. 等待約 **3～5 分鐘**
3. 打開填寫的信箱，你應該會收到一封附有 PDF 的自動回信！

<img width="631" height="80" alt="image" src="https://github.com/user-attachments/assets/b66ab4f4-5d04-432a-9ccd-414aa0b8ec59" />

<img width="383" height="477" alt="image" src="https://github.com/user-attachments/assets/e45f163a-e63f-40f8-9e7f-ebd032130332" />


> [!NOTE]
> **🏆 Checkpoint 1-2 完成！**
> 你剛剛做到了什麼？你用零程式碼，實現了一個完整的「事件驅動自動化」——有人填表（事件發生）→ 系統自動處理（AutoCrat 執行）→ 個人化輸出（PDF 寄出）。這是現代企業工作流的基礎模式。

---

### 📌 任務 1-3：AppSheet + Looker Studio — 手機報到 App 與即時戰情室

**工具：** AppSheet（免費版）、Looker Studio（完全免費）

---

#### 步驟一：準備試算表（加入報到狀態欄）

1. 開啟 `金門聚落文化營_報名總表`
2. 在最後一欄新增以下欄位（手動輸入欄位名稱，不需要填任何資料）：

    | 新增欄位名稱 | 說明 |
    |-----------|------|
    | `報到狀態` | 預設空白，報到後填入「已報到」 |
    | `報到時間` | AI 自動記錄時間戳記 |

    <img width="1864" height="293" alt="image" src="https://github.com/user-attachments/assets/c169d234-7e2d-4caa-8b33-1831d628e2a6" />


---

#### 步驟二：建立 AppSheet 手機報到 App

1. 在試算表上方選單：「**擴充功能**」→「**AppSheet**」→「**建立應用程式**」
    <img width="1001" height="301" alt="image" src="https://github.com/user-attachments/assets/1198ec45-78bc-43f1-81b1-7e21fa306a79" />


3. AppSheet 會在新分頁開啟，並自動讀取試算表欄位

4. **建立「一鍵報到」Action 按鈕（最關鍵！）**
   - 左側選單點擊「**Behavior**」→「**Actions**」→「**+ New Action**」

    <img width="310" height="153" alt="image" src="https://github.com/user-attachments/assets/f70c97c0-f1ec-46f7-a568-6edaada09fd4" />

    <img width="681" height="341" alt="image" src="https://github.com/user-attachments/assets/381462b4-c836-484f-9e70-619f6f27d1ef" />

    
   - 設定如下：

    | 設定項目 | 填入內容 |
    |---------|---------|
    | Action name | `確認報到` |
    | For a record of this table | 報名總表 |
    | Do this | **Set the values of some columns in this row** |
    | Column: 報到狀態 | 輸入 `"已報到"` |
    | Column: 報到時間 | 輸入 `NOW()` |

   - 點擊「**Display**」→ Appearance: 選擇圖示 ✅ → Highlight color: 綠色
    
    <img width="1903" height="938" alt="image" src="https://github.com/user-attachments/assets/3f6223b3-a7e0-4ab8-be19-35e3e194b87c" />


5. **設定 App 顯示欄位**（只顯示工作人員需要的資訊）
   - 左側選單點擊「**Data**」→「**Columns**」
   - 隱藏不必要欄位（點擊眼睛圖示）：只顯示 `姓名`、`參加場次`、`報到狀態`
   
   <img width="1887" height="932" alt="image" src="https://github.com/user-attachments/assets/a57572d8-bcf5-405c-97e7-edb3eea93e0b" />

   <img width="322" height="573" alt="image" src="https://github.com/user-attachments/assets/4f1ec78e-6da9-488c-b01a-296cc7ea94e1" />


4. **如果沒有上述畫面(非首次執行)，請透過以下方式建立Action**
    <img width="522" height="242" alt="image" src="https://github.com/user-attachments/assets/a015de7a-a482-4a80-a472-e806d1950782" />

    <img width="535" height="234" alt="image" src="https://github.com/user-attachments/assets/93a976d3-3a38-4c25-9f43-40cc6c18e756" />

    <img width="523" height="556" alt="image" src="https://github.com/user-attachments/assets/6079d09f-6a1a-4d2b-90a5-7c6c069e8c13" />

    找到您雲端硬碟中的試算表(路徑不一定相同)
    <img width="943" height="584" alt="image" src="https://github.com/user-attachments/assets/7b4dca40-5b93-4b86-b987-402ead65bf98" />

    <img width="1902" height="695" alt="image" src="https://github.com/user-attachments/assets/e3a281d5-924a-44d3-b703-2c6dd509c696" />

    <img width="684" height="348" alt="image" src="https://github.com/user-attachments/assets/70e88fe0-1b21-4c3f-b0de-5016fa9da510" />

    <img width="1900" height="653" alt="image" src="https://github.com/user-attachments/assets/3adb7861-8924-4176-bc06-844bb333792c" />

    左側(Behavior)-->上方(Columns)隱藏不想顯示的欄位
    <img width="1900" height="843" alt="image" src="https://github.com/user-attachments/assets/605f5e95-b23b-492a-a546-35b98b830597" />


5. **儲存並測試**
   - 點擊右上角「**SAVE**」→ Open App in Browser
    <img width="1903" height="687" alt="image" src="https://github.com/user-attachments/assets/56451201-4235-4116-80a6-6043cc2d6501" />

   - (可嘗試下載 AppSheet App → 用同一個 Google 帳號登入)
   - 找到一筆資料，點擊「確認報到」按鈕
   <img width="1912" height="300" alt="image" src="https://github.com/user-attachments/assets/52cd6e4e-24ef-49b2-8a5a-bfae7179928a" />

   <img width="644" height="293" alt="image" src="https://github.com/user-attachments/assets/cbbea03b-20d5-4fa0-89ac-bc8278cc1fa9" />

   - 回到電腦的 Google 試算表，該列應即時更新為「已報到」！
    <img width="1873" height="153" alt="image" src="https://github.com/user-attachments/assets/4758f427-58e9-4c5b-846f-641f07fc1a7b" />


---

#### 步驟三：Looker Studio 建立即時戰情儀表板

1. 前往 [Looker Studio](https://lookerstudio.google.com/) → 點擊「**建立**」→「**報表**」
<img width="1691" height="912" alt="image" src="https://github.com/user-attachments/assets/baa32aa4-ffce-4803-b4e1-bc308133f9b4" />

2. 選擇資料來源：「**Google 試算表**」→ 選擇 `金門聚落文化營_報名總表`
3. 點擊「**新增至報表**」
<img width="1899" height="681" alt="image" src="https://github.com/user-attachments/assets/04d0d3b7-5425-4b36-a5ec-05382c8700b2" />

<img width="1908" height="604" alt="image" src="https://github.com/user-attachments/assets/39293d14-b6fe-44d5-8d40-39a65e0bd1d9" />


4. **建立統計卡片（總報名人數）**
   - 右側「**新增圖表**」→ 選擇「**計分卡 (Scorecard)**」
   - 維度：無；指標：`Count(姓名)` → 卡片標題改為「📋 總報名人數」

<img width="337" height="923" alt="image" src="https://github.com/user-attachments/assets/36a248e0-4bf2-45be-9c0e-c755aa80d913" />

<img width="399" height="711" alt="image" src="https://github.com/user-attachments/assets/be25e2e4-4fbd-4705-b45c-521aec172229" />


5. **建立報到狀態圓餅圖**
   - 「**新增圖表**」→「**圓形圖 (Pie Chart)**」
   - 維度：`報到狀態`；指標：`Count(姓名)`
   - 標題：「🟢 報到狀態分布」

6. **建立場次報名長條圖**
   - 「**新增圖表**」→「**長條圖 (Bar Chart)**」
   - 維度：`參加場次`；指標：`Count(姓名)`
   - 標題：「📅 各場次報名人數」

<img width="1897" height="923" alt="image" src="https://github.com/user-attachments/assets/ced86a39-0528-4041-bce0-b89c7fdc6cf4" />


7. 點擊右上角「**分享**」→ 複製連結，貼到 LINE 群組，讓所有工作人員都能用手機即時追蹤

<img width="1540" height="974" alt="image" src="https://github.com/user-attachments/assets/4b22e463-a9ca-413c-a151-efea679ab904" />

> [!WARNING]
> **🏆 第 1 小時 Checkpoint 完成！**
> 恭喜！你剛剛以零成本、零程式碼建立了：
> - ✅ 符合企業規格的雲端資料架構
> - ✅ 自動寄發 PDF 憑證的報名系統
> - ✅ 手機操作的現場報到 App
> - ✅ 即時顯示戰況的視覺化儀表板
> 在外面，這套系統至少要請工程師開發 **兩週、花費三萬元**。你用了不到 60 分鐘。

---

### 🎯 練習題
⏱️ 練習時間：15 分鐘｜個人作業完成下列三題，截圖存檔(word/文件)。

---

**【練習 1-A】雲端資料夾架構設計（基礎）**
情境：你是工工系系學會活動長，需要規劃本學期「系學會活動專案大腦」。請在你的 Google Drive 建立以下結構，並截圖整個資料夾展開的樣子：
```
2026_工工系學會_專案大腦/
├── 01_參考素材/
│   └── （建立一份空白 Google 文件，命名：範本_活動企劃格式參考）
├── 02_執行草稿/
│   └── （建立一份空白 Google 文件，命名：20260317_迎新晚會_企劃書_v1）
└── 03_最終定稿/
    └── （空資料夾，尚無定稿）
```
完成後，回答以下問題（在截圖旁邊用文字說明）：
> [!TIP]
> ❓ 如果同一份企劃書改了三次，你會怎麼命名 v2、v3？如果要「對外發送」定稿版，應該放在哪個子資料夾？

---

**【練習 1-B】AutoCrat 模板設計挑戰（進階）**
在課堂範例的「入場報到憑證」基礎上，修改模板，加入以下兩個新標籤：
- `<<聯絡電話>>`：顯示報名者的聯絡電話
- `<<場次說明>>`：根據場次顯示集合地點
> [!TIP]
> 💡 提示：`<<場次說明>>` 是一個「新欄位」，你需要先去試算表手動新增這一欄並填入對應的地點說明文字（例如 A場：金城鎮中央公園 / B場：金湖鎮瓊林聚落），再回到 AutoCrat 做標籤對應。

重新執行 AutoCrat（送出一筆新表單），確認 PDF 中出現了新欄位，截圖 PDF 的畫面。

---

【練習 1-C】Looker Studio 儀表板擴充（挑戰）
在課堂建立的戰情儀表板上，再新增一個圖表：
- 圖表類型：表格（Table）
- 維度：`姓名`、`參加場次`、`報到狀態`
- 排序：依「參加場次」字母排序
- 圖表標題：`📋 完整報名名單`
完成後，點擊右上角「取得報表連結」模式，確認在手機上能正常瀏覽，截圖手機畫面。

---
