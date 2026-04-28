# 🗄️ Week 06：資料庫設計實戰 — Google Sheets 進階架構

**雲端與數位內容管理 ｜ 國立金門大學 工業工程與管理學系**

> **本週課程理念：** 你的試算表越做越大、越來越亂——
> 資料重複、更新一個地方忘了改另一個地方、用 VLOOKUP 找資料找到頭暈。
>
> 這不是你的錯，是「資料庫設計」沒做好。
> **今天我們學的不是新工具，而是讓既有工具發揮十倍效力的思維方式。**

---

## 🎯 本週學習目標

學完本週，你將能夠：
- ✅ 解釋「資料正規化」的概念，知道為什麼要把資料拆成多張表
- ✅ 設計「主表 + 子表」的多表關聯架構，消除資料重複
- ✅ 用 `VLOOKUP` 和 `QUERY` 跨表查詢，讓資料動態串聯
- ✅ 用 `TRIM`、`CLEAN`、`IFERROR` 等函數自動清洗骯髒資料
- ✅ 把 Week 02 的實習媒合試算表重構成正規化的多表架構

---

## 🏢 本週實作情境：【工管系廠商實習處理中心 — 資料重構計畫】

> 💼 **情境：** 系辦主任看了 Week 02 建立的實習媒合試算表，
> 提出了幾個問題：
>
> 1. 「台積電」在多筆申請裡重複出現，萬一電話號碼改了，要改幾個地方？
> 2. 同一家企業有三個學生申請，怎麼快速統計？
> 3. 為什麼有些欄位是空白的，有些又有多餘的空格？
>
> **今天的任務：用資料庫設計的思維，把混亂的試算表重構成乾淨的多表架構。**

---

## ⏰ 第 1 小時：資料正規化概念 — 為什麼資料要拆表？

> 🎯 **本小時目標：** 理解資料正規化的核心思維，
> 能夠判斷哪些資料應該獨立成一張表。

---

### 💡 觀念一：什麼是「資料異常」？

先看一個**反面教材**——一張沒有正規化的試算表：

| 申請ID | 學生姓名 | 學號 | 企業名稱 | 企業電話 | 企業地址 | 聯絡窗口 | 申請日期 |
|--------|---------|------|---------|---------|---------|---------|---------|
| A001 | 王大明 | 1112001 | 台積電 | 03-5636688 | 新竹市力行路3號 | 王經理 | 2026/03/01 |
| A002 | 李美麗 | 1112002 | 台積電 | 03-5636688 | 新竹市力行路3號 | 王經理 | 2026/03/02 |
| A003 | 陳志偉 | 1113001 | 台積電 | 03-5636688 | 新竹市力行路3號 | 王經理 | 2026/03/05 |
| A004 | 林小芳 | 1113002 | 聯電 | 03-5782258 | 新竹市新安路9號 | 李協理 | 2026/03/06 |

**這張表有三種資料異常：**
```
① 新增異常（Insertion Anomaly）
   → 如果要新增一家還沒有人申請的企業，放不進這張表
     （因為每列都必須有學生申請資料）

② 更新異常（Update Anomaly）
   → 台積電電話改了，必須同時修改 A001、A002、A003 三列
     → 萬一只改了兩列，資料就不一致了

③ 刪除異常（Deletion Anomaly）
   → 如果 A004 林小芳的申請被撤銷，聯電的所有資訊也跟著消失
```

---

### 💡 觀念二：什麼是正規化？

**正規化（Normalization）** 是把一張大表拆成多張小表，
讓每張表只負責「一種事情」，消除資料重複和異常。

**核心原則：** 一個事實只存在一個地方
```
❌ 錯誤做法（一張大表）：
學生資料 + 企業資料 + 申請記錄 → 全部塞在同一張表

✅ 正確做法（三張小表）：
學生資料表   → 只存學生的資料
企業資料表   → 只存企業的資料
申請記錄表   → 只存「誰申請了哪家企業」的關聯
```

---

### 💡 觀念三：主鍵與外鍵

**主鍵（Primary Key）：** 每張表中唯一識別一筆資料的欄位
```
學生資料表：學號（每個學生的學號都不一樣）
企業資料表：企業ID（E001、E002...）
申請記錄表：申請ID（A001、A002...）
```

**外鍵（Foreign Key）：** 用來連結兩張表的欄位
```
申請記錄表
    ├── 申請ID（主鍵）
    ├── 學號（外鍵 → 指向學生資料表的主鍵）
    └── 企業ID（外鍵 → 指向企業資料表的主鍵）
```

> [!NOTE]
> **用現實比喻：**
> 申請記錄表就像一張「婚姻登記」，
> 上面只記錄「某個學號的人申請了某個企業ID的公司」，
> 不需要把雙方的所有個人資料都複製一遍。
> 要看詳細資料，就去各自的資料表查。

---

### 📌 任務 1-1：分析 Week 02 的試算表，找出問題

打開 Week 02 建立的「Week02_實習媒合資料」試算表，
找出以下三類問題並記錄：

**找出重複的資料（應該獨立成表的）：**

| 重複資料 | 出現在哪些欄位 | 應該獨立成什麼表 |
|---------|-------------|----------------|
| 企業資料（名稱、產業別、聯絡人）| 企業名稱、... | 企業資料表 |
| 學生基本資料 | 姓名、學號、... | 學生資料表 |

**找出應該是外鍵的欄位（連結兩張表的）：**
- 申請記錄只需要存「學號」和「企業ID」，其他資料去各自的表查

<img width="691" height="162" alt="image" src="https://github.com/user-attachments/assets/d30357db-2be4-4dca-85e1-d4350006edb4" />

---

### 📌 任務 1-2：設計三表架構

根據分析結果，設計以下三張表的欄位結構：

<img width="494" height="53" alt="image" src="https://github.com/user-attachments/assets/ab64483f-86de-4f1f-a2f2-449fc967376f" />

**表一：students（學生資料表）**

| 欄位名稱 | 資料類型 | 說明 |
|---------|---------|------|
| 學號 | Text | **主鍵**，格式：7位數字 |
| 姓名 | Text | |
| Email | Text | |
| 就讀年級 | Text | 二/三/四年級 |
| 指導教授 | Text | |
| 建立時間 | Date | 學生資料建立日期 |

**表二：companies（企業資料表）**

| 欄位名稱 | 資料類型 | 說明 |
|---------|---------|------|
| 企業ID | Text | **主鍵**，格式：E001、E002... |
| 企業名稱 | Text | |
| 產業別 | Text | |
| 縣市 | Text | |
| 聯絡窗口 | Text | |
| 聯絡Email | Text | |
| 月薪範圍 | Text | |
| 是否開放申請 | Text | 是/否 |

**表三：applications（申請記錄表）**

| 欄位名稱 | 資料類型 | 說明 |
|---------|---------|------|
| 申請ID | Text | **主鍵**，格式：A001、A002... |
| 學號 | Text | **外鍵** → 學生資料表 |
| 企業ID | Text | **外鍵** → 企業資料表 |
| 實習類型 | Text | 國內/海外/自行開發 |
| 開始日期 | Date | |
| 結束日期 | Date | |
| 審核狀態 | Text | 待審核/已核准/退件 |
| 申請時間 | Date | Timestamp |

> [!NOTE]
> **注意：申請記錄表不存姓名、不存企業名稱。**
> 只存學號和企業ID，要顯示名稱時用 VLOOKUP 去各自的表查。
> 這就是正規化的精髓——每個事實只存一次。

---

### 📋 工作表一：students（學生資料表）

#### 學號（A欄）— 主鍵，7位數字
 
| 項目 | 設定值 |
|------|--------|
| 範圍 | `students!A2:A1000` |
| 條件 | 自訂公式 |
| 無效資料 | 拒絕輸入 |
| 說明文字 | 請輸入7位數字，例如：1130001 |
 
```
=AND(ISNUMBER(VALUE(A2:A)), LEN(A2:A)=7, VALUE(A2:A)>=1000000, VALUE(A2:A)<=9999999)
```

<img width="938" height="637" alt="image" src="https://github.com/user-attachments/assets/f516a066-1f71-444e-96e7-937d9aaf6389" />

> ⚠️ **主鍵唯一性**：格式 → 條件式格式 → 自訂公式，背景設為紅色
> ```
> =COUNTIF($A$2:$A$1000,A2)>1
> ```

<img width="950" height="612" alt="image" src="https://github.com/user-attachments/assets/ee5562c6-53fb-4d3f-abab-74a190ce61a2" />

---
 
#### Email（C欄）
 
| 項目 | 設定值 |
|------|--------|
| 範圍 | `students!C2:C1000` |
| 條件 | 自訂公式 |
| 無效資料 | 拒絕輸入 |
 
```
=AND(ISNUMBER(FIND("@",C2:C)), ISNUMBER(FIND(".",C2:C)))
```

<img width="945" height="648" alt="image" src="https://github.com/user-attachments/assets/266dfaf5-0c81-4f1c-8ff9-e222e0b609cc" />

---
 
#### 就讀年級（D欄）
 
| 項目 | 設定值 |
|------|--------|
| 範圍 | `students!D2:D1000` |
| 條件 | 下拉式選單（從清單） |
| 無效資料 | 拒絕輸入 |
 
```
二年級,三年級,四年級
```

<img width="934" height="756" alt="image" src="https://github.com/user-attachments/assets/2277e2d4-5464-42f7-ab65-7b21aa735012" />

---
 
#### 建立時間（F欄）
 
| 項目 | 設定值 |
|------|--------|
| 範圍 | `students!F2:F1000` |
| 條件 | 日期 → 是有效的日期 |
| 無效資料 | 拒絕輸入 |

<img width="947" height="508" alt="image" src="https://github.com/user-attachments/assets/13146f65-f340-44b1-8cad-4c5e182c7cd5" />

---

## 📋 工作表二：companies（企業資料表）
 
### 企業ID（A欄）— 主鍵，格式 E001、E002...
 
| 項目 | 設定值 |
|------|--------|
| 範圍 | `companies!A2:A1000` |
| 條件 | 自訂公式 |
| 無效資料 | 拒絕輸入 |
| 說明文字 | 格式為 E 開頭加3位數字，例如：E001 |
 
```
=AND(LEFT(A2:A,1)="E", LEN(A2:A)=4, ISNUMBER(VALUE(MID(A2:A,2,3))))
```

<img width="946" height="618" alt="image" src="https://github.com/user-attachments/assets/30b1001a-ecff-4a08-9f67-f5a2a58edb76" />

> ⚠️ **主鍵唯一性**：條件式格式 → 自訂公式，背景設為紅色
> ```
> =COUNTIF($A$2:$A$1000,A2)>1
> ```

<img width="947" height="609" alt="image" src="https://github.com/user-attachments/assets/a4507ed5-da10-46f0-b704-554d9233c00b" />

---
 
### 聯絡Email（F欄）
 
| 項目 | 設定值 |
|------|--------|
| 範圍 | `companies!F2:F1000` |
| 條件 | 自訂公式 |
| 無效資料 | 顯示警告 |
 
```
=AND(ISNUMBER(FIND("@",F2:F)), ISNUMBER(FIND(".",F2:F)))
```

<img width="947" height="561" alt="image" src="https://github.com/user-attachments/assets/4fb5e906-a1e8-41c1-bc95-f05e964f1d4d" />

---
 
### 月薪範圍（G欄）
 
| 項目 | 設定值 |
|------|--------|
| 範圍 | `companies!G2:G1000` |
| 條件 | 下拉式選單（從清單） |
| 無效資料 | 顯示警告 |
 
```
25000以下,25000-30000,30000-35000,35000-40000,40000以上
```

<img width="942" height="802" alt="image" src="https://github.com/user-attachments/assets/eb373833-a988-4a5a-8649-0dc93c1dff8e" />

---
 
### 是否開放申請（H欄）

| 項目 | 設定值 |
|------|--------|
| 範圍 | `companies!H2:H1000` |
| 條件 | 下拉式選單（從清單） |
| 無效資料 | 拒絕輸入 |

```
是,否
```

<img width="948" height="703" alt="image" src="https://github.com/user-attachments/assets/9fd7e46f-84dc-4df9-b6a5-8f21c10cde67" />
 
---
 
## 📋 工作表三：applications（申請記錄表）
 
### 申請ID（A欄）— 主鍵，格式 A001、A002...
 
| 項目 | 設定值 |
|------|--------|
| 範圍 | `A2:A1000` |
| 條件 | 自訂公式 |
| 無效資料 | 拒絕輸入 |
| 說明文字 | 格式為 A 開頭加3位數字，例如：A001 |
 
```
=AND(LEFT(A2,1)="A", LEN(A2)=4, ISNUMBER(VALUE(MID(A2,2,3))))
```

<img width="947" height="611" alt="image" src="https://github.com/user-attachments/assets/de698d5c-9267-484a-98d1-9aa0c333e558" />

> ⚠️ **主鍵唯一性**：條件式格式 → 自訂公式，背景設為紅色
> ```
> =COUNTIF($A$2:$A$1000,A2)>1
> ```

<img width="952" height="607" alt="image" src="https://github.com/user-attachments/assets/5d684494-05c9-41c3-82b0-b6b00a650ad7" />

---
 
### 學號（B欄）— 外鍵 → students
 
| 項目 | 設定值 |
|------|--------|
| 範圍 | `B2:B1000` |
| 條件 | 自訂公式 |
| 無效資料 | 拒絕輸入 |
| 說明文字 | 學號必須已存在於學生資料表 |
 
```
=AND(ISNUMBER(VALUE(B2)), LEN(B2)=7, COUNTIF(students!$A$2:$A$1000,B2)>0)
```

<img width="953" height="620" alt="image" src="https://github.com/user-attachments/assets/e0ece4b8-b43d-4954-a45f-a7630a731abf" />

---
 
### 企業ID（C欄）— 外鍵 → companies
 
| 項目 | 設定值 |
|------|--------|
| 範圍 | `C2:C1000` |
| 條件 | 自訂公式 |
| 無效資料 | 拒絕輸入 |
| 說明文字 | 企業ID必須已存在於企業資料表 |
 
```
=AND(LEFT(C2,1)="E", LEN(C2)=4, COUNTIF(companies!$A$2:$A$1000,C2)>0)
```

<img width="946" height="615" alt="image" src="https://github.com/user-attachments/assets/fc9ce76e-8d52-48f1-9dba-00f50f0e5ba3" />

---
 
### 實習類型（D欄）
 
| 項目 | 設定值 |
|------|--------|
| 範圍 | `D2:D1000` |
| 條件 | 下拉式選單（從清單） |
| 無效資料 | 拒絕輸入 |
 
```
國內,海外,自行開發
```

<img width="942" height="543" alt="image" src="https://github.com/user-attachments/assets/6610b9d4-faa6-44d9-b157-6a25c485351c" />

---
 
### 開始日期（E欄）／ 結束日期（F欄）
 
| 項目 | 設定值 |
|------|--------|
| 範圍 | `E2:E1000` / `F2:F1000` |
| 條件 | 日期 → 是有效的日期 |
| 無效資料 | 拒絕輸入 |
 
> ⚠️ **進階**：確保結束日期在開始日期之後，F欄額外加驗證：
> ```
> =AND(ISNUMBER(F2), F2>E2)
> ```

<img width="943" height="505" alt="image" src="https://github.com/user-attachments/assets/4ae7fff4-94d2-4207-83b4-cb6270223b12" />
<img width="948" height="519" alt="image" src="https://github.com/user-attachments/assets/6b1b1cb2-5064-4f5c-89d9-6010fbd20200" />
<img width="948" height="620" alt="image" src="https://github.com/user-attachments/assets/a49cbdfa-3d1e-4d0f-b25f-e6f721d2b3c8" />

---
 
### 審核狀態（G欄）
 
| 項目 | 設定值 |
|------|--------|
| 範圍 | `G2:G1000` |
| 條件 | 下拉式選單（從清單） |
| 無效資料 | 拒絕輸入 |
 
```
待審核,已核准,退件
```

<img width="951" height="542" alt="image" src="https://github.com/user-attachments/assets/0c982424-0594-47b6-b854-b40c63e9cd0e" />

---
 
### 申請時間（H欄）
 
| 項目 | 設定值 |
|------|--------|
| 範圍 | `H2:H1000` |
| 條件 | 日期 → 是有效的日期 |
| 無效資料 | 拒絕輸入 |

<img width="953" height="514" alt="image" src="https://github.com/user-attachments/assets/94546ca6-23b5-4e84-9016-6bd8f48abbbe" />

---

> [!TIP]
> **🏆 第 1 小時 Checkpoint 完成！**
>
> - ✅ 能識別三種資料異常（新增/更新/刪除）
> - ✅ 理解正規化的核心原則：一個事實只存在一個地方
> - ✅ 設計出三表架構：學生表、企業表、申請記錄表

---

### 🎯 第 1 小時學生練習題

> [!NOTE]
> **⏱️ 練習時間：15 分鐘｜個人作業**

---

#### 【練習 1-A】資料異常診斷（基礎）

以下試算表有哪些資料異常？請各舉一個例子說明：

| 訂單ID | 客戶姓名 | 客戶電話 | 產品名稱 | 產品單價 | 數量 | 門市地址 |
|--------|---------|---------|---------|---------|------|---------|
| O001 | 陳大華 | 0912111111 | 蘋果 | 30 | 5 | 台北市信義路1號 |
| O002 | 陳大華 | 0912111111 | 香蕉 | 15 | 3 | 台北市信義路1號 |
| O003 | 李小花 | 0988222222 | 蘋果 | 30 | 2 | 高雄市中山路5號 |

請回答：
1. 這張表有哪三種資料異常？各舉一個具體例子
2. 應該拆成哪幾張表？畫出表格結構（欄位名稱就好）

---

#### 【練習 1-B】主鍵設計（進階）

以下四個欄位，哪些適合當主鍵？哪些不適合？說明理由：

1. 學生的姓名
2. 學生的學號
3. 訂單的建立時間
4. 企業的統一編號

---

#### 【練習 1-C】觀念問答（思考題）

請回答：

> ❓ 正規化會讓查詢變得更複雜（需要 VLOOKUP 跨表查詢），
> 但為什麼資料庫設計師還是選擇正規化？
> 請從「資料維護成本」和「查詢靈活性」兩個角度說明你的看法。

---

## ⏰ 第 2 小時：建立多表架構與跨表查詢

> 🎯 **本小時目標：** 實際建立三張工作表，
> 並用 VLOOKUP 和 QUERY 串聯三張表，讓資料動態更新。

---

### 📌 任務 2-1：建立三張工作表

新建一份 Google 試算表，命名：`Week06_實習資料庫`

在試算表底部建立三個工作表分頁：
- `students`（學生資料表）
- `companies`（企業資料表）
- `applications`（申請記錄表）

---

#### 學生資料表（students）

輸入以下欄位標題和 8 筆資料：

| 學號 | 姓名 | Email | 就讀年級 | 指導教授 | 建立時間 |
|------|------|-------|---------|---------|---------|
| 1112001 | 王大明 | wang@nqu.edu.tw | 三年級 | 林教授 | 2026/03/01 |
| 1112002 | 李美麗 | lee@nqu.edu.tw | 三年級 | 林教授 | 2026/03/02 |
| 1113001 | 陳志偉 | chen@nqu.edu.tw | 二年級 | 張教授 | 2026/03/05 |
| 1113002 | 林小芳 | lin@nqu.edu.tw | 二年級 | 張教授 | 2026/03/06 |
| 1114001 | 張建宏 | chang@nqu.edu.tw | 四年級 | 王教授 | 2026/03/08 |
| 1114002 | 吳雅婷 | wu@nqu.edu.tw | 四年級 | 王教授 | 2026/03/10 |
| 1112003 | 黃俊賢 | huang@nqu.edu.tw | 三年級 | 林教授 | 2026/03/12 |
| 1113003 | 劉心如 | liu@nqu.edu.tw | 二年級 | 張教授 | 2026/03/15 |

<img width="607" height="215" alt="image" src="https://github.com/user-attachments/assets/74ad6aac-e1e3-49c8-9c25-b4fe6d1cb637" />

---

#### 企業資料表（companies）

| 企業ID | 企業名稱 | 產業別 | 縣市 | 聯絡窗口 | 聯絡Email | 月薪範圍 | 是否開放 |
|--------|---------|--------|------|---------|---------|---------|---------|
| E001 | 台積電 | 半導體 | 新竹 | 王經理 | hr@tsmc.com | 35000-40000 | 是 |
| E002 | 日立製作所 | 電子製造 | — | 田中桑 | hr@hitachi.com | 40000以上 | 是 |
| E003 | 金門酒廠 | 食品製造 | 金門 | 陳主任 | hr@kmwine.com | 25000-30000 | 是 |
| E004 | 鴻海精密 | 電子製造 | 新北 | 廖副理 | hr@foxconn.com | 30000-35000 | 否 |
| E005 | 聯發科 | IC設計 | 新竹 | 張副總 | hr@mediatek.com | 40000以上 | 是 |

<img width="614" height="152" alt="image" src="https://github.com/user-attachments/assets/c0b0ba8f-d1e3-4fb3-987d-71d2ede6e5d1" />

---

#### 申請記錄表（applications）

| 申請ID | 學號 | 企業ID | 實習類型 | 開始日期 | 結束日期 | 審核狀態 | 申請時間 |
|--------|------|--------|---------|---------|---------|---------|---------|
| A001 | 1112001 | E001 | 國內 | 2026/07/01 | 2026/08/31 | 已核准 | 2026/03/01 |
| A002 | 1112002 | E002 | 海外 | 2026/07/15 | 2026/09/15 | 已核准 | 2026/03/02 |
| A003 | 1113001 | E001 | 國內 | 2026/07/01 | 2026/07/31 | 待審核 | 2026/03/05 |
| A004 | 1113002 | E003 | 自行開發 | 2026/08/01 | 2026/09/30 | 已核准 | 2026/03/06 |
| A005 | 1114001 | E004 | 國內 | 2026/07/01 | 2026/08/31 | 退件 | 2026/03/08 |
| A006 | 1114002 | E002 | 海外 | 2026/06/15 | 2026/08/15 | 已核准 | 2026/03/10 |
| A007 | 1112003 | E001 | 國內 | 2026/07/01 | 2026/09/30 | 待審核 | 2026/03/12 |
| A008 | 1113003 | E003 | 自行開發 | 2026/07/01 | 2026/07/31 | 已核准 | 2026/03/15 |

<img width="576" height="216" alt="image" src="https://github.com/user-attachments/assets/cfca718c-ece4-4da3-bb3a-1bb464410c96" />

---

### 📌 任務 2-2：用 VLOOKUP 跨表查詢

新建第四個工作表分頁，命名 `view_applications`（綜合查詢視圖）。

這張表的目的是把三張表的資料合併顯示，方便閱讀，
但**原始資料不會重複存放**，全部透過公式動態抓取。

**欄位設計：**

| A | B | C | D | E | F | G | H |
|---|---|---|---|---|---|---|---|
| 申請ID | 學號 | 學生姓名 | 企業ID | 企業名稱 | 產業別 | 審核狀態 | 實習天數 |

**填入公式：**

A2 — 直接參照申請記錄表：
```
=applications!A2
```

B2 — 直接參照：
```
=applications!B2
```

C2 — **VLOOKUP 查學生姓名**：
```
=IFERROR(VLOOKUP(B2, students!A:B, 2, FALSE), "查無此學號")
```

D2 — 直接參照：
```
=applications!C2
```

E2 — **VLOOKUP 查企業名稱**：
```
=IFERROR(VLOOKUP(D2, companies!A:B, 2, FALSE), "查無此企業")
```

F2 — **VLOOKUP 查產業別**（注意回傳第 3 欄）：
```
=IFERROR(VLOOKUP(D2, companies!A:C, 3, FALSE), "—")
```

G2 — 直接參照審核狀態：
```
=applications!H2
```

H2 — **計算實習天數**：
```
=IFERROR(DATEDIF(applications!E2, applications!F2, "D"), "日期未填")
```

把所有公式往下拖曳到第 9 列（8 筆資料）。

<img width="489" height="216" alt="image" src="https://github.com/user-attachments/assets/83595068-baa7-4dfe-8e42-8c20690ce952" />

---

### 📌 任務 2-3：用 QUERY 建立動態篩選視圖

新建第五個工作表 `view_approved`，
用 QUERY 函數自動篩選出所有「已核准」的申請，並整合資料：
```
=QUERY(view_applications!A:H,
  "SELECT A, B, C, E, F, H
   WHERE G = '已核准'
   ORDER BY H DESC",
  1)
```

**公式解析：**
- `SELECT A, B, C, E, F, H`：只顯示申請ID、學號、姓名、企業名稱、產業別、實習天數
- `WHERE G = '已核准'`：只要審核狀態是「已核准」的
- `ORDER BY H DESC`：依實習天數由多到少排序
- `1`：第一列是標題列

<img width="373" height="160" alt="image" src="https://github.com/user-attachments/assets/d787e719-181e-4d93-a613-652fa0e89336" />


> [!NOTE]
> **為什麼要用「視圖（View）」工作表？**
>
> 真實的資料庫（MySQL、PostgreSQL）有「View（視圖）」的概念：
> 不實際儲存資料，而是用查詢動態產生結果。
>
> 我們在 Google Sheets 用 QUERY 函數模擬同樣的效果：
> - `applications`、`students`、`companies` = 原始資料表（只改這裡）
> - `view_applications`、`view_approved` = 視圖（自動更新）
>
> 只要原始資料改了，所有視圖都會自動更新，不需要手動同步。

> [!TIP]
> **🏆 第 2 小時 Checkpoint 完成！**
>
> - ✅ 建立三張正規化的原始資料表
> - ✅ 用 VLOOKUP 跨表串聯學生、企業、申請資料
> - ✅ 用 QUERY 建立動態篩選視圖，原始資料更新時自動同步

---

### 🎯 第 2 小時學生練習題

> [!NOTE]
> **⏱️ 練習時間：20 分鐘｜個人作業**

---

#### 【練習 2-A】新增 VLOOKUP 欄位（基礎）

在 `view_applications` 工作表再新增兩個欄位：

1. **指導教授**：從 `students` 表查詢（提示：VLOOKUP 回傳第 5 欄）

```
=VLOOKUP(B2,students!A:E,5)
```

2. **企業聯絡Email**：從 `companies` 表查詢（提示：VLOOKUP 回傳第 6 欄）

```
=VLOOKUP(D2, companies!A:F, 6, FALSE)
```

<img width="719" height="218" alt="image" src="https://github.com/user-attachments/assets/94804d71-a685-4d67-8d07-28c0a22b1c32" />

---

#### 【練習 2-B】QUERY 進階篩選（進階）

在新的工作表 `view_analysis` 建立以下三個 QUERY 查詢，
每個查詢各占一個區域（中間空幾列隔開）：

**查詢一：各實習類型的申請人數統計**
```
=QUERY(applications!A:H,
  "SELECT D, COUNT(A)
   WHERE D IS NOT NULL
   GROUP BY D
   LABEL COUNT(A) '申請人數'",1)
```

<img width="533" height="155" alt="image" src="https://github.com/user-attachments/assets/9e7e4def-2c2c-44ec-8200-ced731a8403b" />

**查詢二：各企業的申請人數（用企業ID查，再用VLOOKUP換成企業名稱）**
想想看：QUERY 的 GROUP BY 結果要怎麼再套一層 VLOOKUP？
```
=QUERY(applications!A:H,
  "SELECT C, COUNT(A)
   WHERE C IS NOT NULL
   GROUP BY C
   LABEL COUNT(A) '申請人數'",1)
```
企業名稱資料欄位公式，後續複製公式即可
```
=VLOOKUP(D3, companies!A:B, 2, 0)
```

**查詢三：只顯示「三年級」學生的申請記錄**
（提示：需要先在 view_applications 加入「就讀年級」欄位）
```
=QUERY(view_applications!A:I,
  "SELECT A, B, C, E, F, H
   WHERE I = '"&L1&"'
   ORDER BY H
   DESC", 1)
```

<img width="595" height="215" alt="image" src="https://github.com/user-attachments/assets/f85d94eb-e082-4677-baee-b1201bd80702" />

<img width="500" height="135" alt="image" src="https://github.com/user-attachments/assets/7a9056a3-8a9d-49dd-a21f-993c0a8f51c4" />
<img width="505" height="187" alt="image" src="https://github.com/user-attachments/assets/4dbc67ee-ac0c-473c-b3e5-30a94ca27384" />

---

#### 【練習 2-C】觀念問答（思考題）

請回答，貼到課堂共用文件：

> ❓ 當台積電（E001）的聯絡窗口從「王經理」換成「李副總」時，
> 在正規化的三表架構裡，你只需要改幾個地方？
> 如果是沒有正規化的一張大表（有 3 筆台積電的申請記錄），
> 你需要改幾個地方？
>
> 這個差異說明了正規化的什麼優勢？

---

## ⏰ 第 3 小時：資料清洗 — 讓骯髒資料變乾淨

> 🎯 **本小時目標：** 學會用公式自動偵測和修正骯髒資料，
> 讓進入資料庫的資料品質有保障。

---

### 💡 觀念：什麼是「骯髒資料」？

現實中的資料幾乎不可能完美，常見的骯髒資料類型：

| 問題類型 | 範例 | 影響 |
|---------|------|------|
| 多餘的空格 | `" 台積電 "` | VLOOKUP 找不到 |
| 大小寫不一致 | `"tsmc"` vs `"TSMC"` | COUNTIF 計算錯誤 |
| 格式不統一 | `"2026-3-1"` vs `"2026/03/01"` | 日期計算出錯 |
| 隱藏字元 | 從其他系統複製貼上的奇怪字元 | 公式全部失效 |
| 空白儲存格 | 必填欄位是空的 | 統計結果不準確 |
| 重複資料 | 同一筆申請被輸入兩次 | 計算人數多算 |

---

### 📌 任務 3-1：建立「骯髒資料」測試集

新建工作表 `dirty_data`，輸入以下故意做壞的資料：

| 學號 | 姓名 | Email | 企業名稱 | 月薪 | 申請日期 |
|------|------|-------|---------|------|---------|
| `1112001 ` | `王大明` | `WANG@NQU.EDU.TW` | ` 台積電 ` | `32,000` | `2026-3-1` |
| `1112002` | `  李美麗  ` | `lee@nqu.edu.tw` | `台積電` | `45000` | `2026/03/02` |
| `abc` | `陳志偉` | `chen@nqu` | `聯電` | `-5000` | `2026/03/05` |
| `1113002` |  | `lin@nqu.edu.tw` | `金門酒廠 ` | `25000` |  |
| `1112001 ` | `王大明` | `wang@nqu.edu.tw` | `台積電` | `32000` | `2026/03/01` |

> 這張表刻意包含：多餘空格、大小寫不一致、格式錯誤、
> 負數月薪、空白欄位、學號重複（第 1 和第 5 列）。

---

### 📌 任務 3-2：十個資料清洗公式

新建工作表 `cleaned_data`，對每個欄位套用清洗公式：

---

#### 公式 1：`TRIM` — 去除多餘空格
```
=TRIM(dirty_data!A2)
```

**效果：**
- `" 1112001 "` → `"1112001"`
- `"  李美麗  "` → `"李美麗"`

---

#### 公式 2：`CLEAN` — 去除隱藏字元
```
=CLEAN(dirty_data!B2)
```

**效果：** 移除從其他系統（PDF、Excel）複製貼上時帶進來的不可見字元

---

#### 公式 3：`TRIM` + `CLEAN` 組合（標準清洗）
```
=TRIM(CLEAN(dirty_data!B2))
```

> [!TIP]
> 這個組合是清洗文字欄位的**標準公式**，幾乎適用於所有情況。
> 先 `CLEAN` 去掉隱藏字元，再 `TRIM` 去掉多餘空格。

---

#### 公式 4：`LOWER` / `UPPER` / `PROPER` — 統一大小寫
```
=LOWER(dirty_data!C2)      → wang@nqu.edu.tw（全小寫）
=UPPER(dirty_data!C2)      → WANG@NQU.EDU.TW（全大寫）
=PROPER(dirty_data!B2)     → 王大明（首字大寫，適合英文名）
```

Email 統一用 `LOWER`：
```
=LOWER(TRIM(CLEAN(dirty_data!C2)))
```

---

#### 公式 5：`SUBSTITUTE` — 替換字元

去除月薪欄位裡的逗號，先讓它變成文字在替換","成為空字串""，最後再轉成純數字：
```
=VALUE(SUBSTITUTE(TEXT(E2,"0"), ",", ""))
```

**效果：** `"32,000"` → `32000`（數字型態）

---

#### 公式 6：`IFERROR` + `VALUE` — 安全轉換數字

如果轉換失敗（例如月薪是文字），顯示 0 而不是錯誤：
```
=IFERROR(VALUE(REGEXREPLACE(dirty_data!E2, "[,]", "")), 0)
```

---

#### 公式 7：`IF` + `ISNUMBER` — 驗證數字格式

學號必須是純數字，用公式驗證：
```
=IF(ISNUMBER(VALUE(TRIM(dirty_data!A2))), "✅ 格式正確", "❌ 格式錯誤")
```

---

#### 公式 8：`IF` + `ISBLANK` — 偵測空白欄位
```
=IF(ISBLANK(dirty_data!B2), "⚠️ 姓名未填", TRIM(dirty_data!B2))
```

---

#### 公式 9：`DATEVALUE` — 統一日期格式

把不同格式的日期統一轉換：
```
=IFERROR(DATEVALUE(dirty_data!F2), "❌ 日期格式錯誤")
```

再格式化顯示：
```
=TEXT(IFERROR(DATEVALUE(dirty_data!F2), ""), "YYYY/MM/DD")
```

**效果：**
- `"2026-3-1"` → `"2026/03/01"`
- `"2026/03/02"` → `"2026/03/02"`（不變，已正確）

---

#### 公式 10：`COUNTIF` — 偵測重複資料

在新欄位「是否重複」，偵測學號是否出現超過一次：
```
=IF(COUNTIF(dirty_data!$A$2:$A$6, TRIM(A2))>1, "⚠️ 重複", "✅ 唯一")
```

<img width="444" height="160" alt="image" src="https://github.com/user-attachments/assets/dcd897c3-e385-4fb8-975b-e50d0ec51f66" />
<img width="615" height="153" alt="image" src="https://github.com/user-attachments/assets/0478cd3f-5f28-465f-88b8-8690e5ed9b0d" />

---

### 📌 任務 3-3：建立資料品質儀表板

新建工作表 `quality_dashboard`，用公式自動計算資料品質指標：
| 指標 | 數值 |
|---|---|
| 總筆數 | ```=COUNTA(dirty_data!A2:A6)``` |
| 有空白姓名的筆數 | ```=COUNTIF(dirty_data!B2:B6, "")``` |
| 學號格式錯誤 | ```=COUNTIF(dirty_data!A2:A6, "abc")``` |
| 重複學號筆數 | ```=SUMPRODUCT((COUNTIF(dirty_data!A2:A6, dirty_data!A2:A6)>1)*1)``` | 
| Email 包含 @ 的筆數 | ```=COUNTIF(dirty_data!C2:C6, "*@*")``` |
| 資料完整率 | ```=TEXT((COUNTA(dirty_data!B2:B6)/COUNTA(dirty_data!A2:A6)), "0%")``` |

<img width="244" height="157" alt="image" src="https://github.com/user-attachments/assets/3617b177-a755-4a02-b16d-1c1d9f5facd0" />

> [!TIP]
> **🏆 第 3 小時 Checkpoint 完成！**
>
> - ✅ 識別六種常見的骯髒資料類型
> - ✅ 掌握十個資料清洗公式（TRIM、CLEAN、LOWER、REGEXREPLACE 等）
> - ✅ 建立資料品質儀表板，自動監控資料狀況

---

### 🎯 第 3 小時學生練習題

> [!NOTE]
> **⏱️ 練習時間：20 分鐘｜個人 + 分組**

---

#### 【練習 3-A】全欄清洗（個人，基礎）

對 `dirty_data` 工作表的**所有欄位**套用完整清洗公式：

1. 學號：`TRIM` + 驗證是否為 7 位數字
2. 姓名：`TRIM` + `CLEAN` + `ISBLANK` 偵測空值
3. Email：`LOWER` + `TRIM` + `CLEAN` + 驗證包含 `@`
4. 企業名稱：`TRIM` + `CLEAN`
5. 月薪：`REGEXREPLACE` 去逗號 + `VALUE` 轉數字 + 驗證大於 0
6. 申請日期：`DATEVALUE` + `TEXT` 統一格式

把清洗結果放在 `cleaned_data` 工作表，截圖清洗前後對照。

---

#### 【練習 3-B】整合重構（分組，2～3 人）

把 Week 02 建立的原始「實習媒合試算表」（一張大表）
重構成本週學習的三表正規化架構：

1. 把原始資料拆成 `students`、`companies`、`applications` 三張工作表
2. 在拆的過程中，對每個欄位套用適當的清洗公式
3. 建立 `view_applications` 跨表查詢視圖
4. 建立 `quality_dashboard` 資料品質儀表板

繳交：Google Sheets 連結
展示：說明你們在清洗過程中發現了哪些原始資料的問題。

---

#### 【練習 3-C】延伸思考（挑戰）

> ❓ 本週學的是在 Google Sheets 裡用公式做資料清洗。
> 但如果資料量很大（例如 10 萬筆），這個方法還適用嗎？
>
> 請調查以下兩個工具，說明它們如何解決大規模資料清洗問題：
> 1. **Google Sheets + GAS（Apps Script）**：可以用程式批次處理
> 2. **Python Pandas**：資料科學常用的資料處理工具
>
> 你認為工管系的畢業生，在職場上最可能碰到的是哪種規模的資料問題？

---

## 🏆 本週總結

### 📊 本週技能地圖
```
【資料正規化概念】
    ├── 三種資料異常：新增/更新/刪除異常
    ├── 正規化原則：一個事實只存一次
    └── 主鍵 / 外鍵 / 多表關聯設計

        ↓ 概念 → 實作

【多表架構建立】
    ├── students / companies / applications 三表設計
    ├── VLOOKUP 跨表串聯（姓名、企業名稱、產業別）
    └── QUERY 動態篩選視圖（已核准名單、統計分析）

        ↓ 資料進來前

【資料清洗】
    ├── TRIM / CLEAN：去空格和隱藏字元
    ├── LOWER / UPPER：統一大小寫
    ├── REGEXREPLACE + VALUE：格式轉換
    ├── ISBLANK / ISNUMBER：格式驗證
    ├── DATEVALUE + TEXT：日期統一
    └── COUNTIF：重複偵測
```

### 🔄 本週和前後週的連結
```
Week 02：建立了一張混亂的大表（實習媒合資料）
    ↓ Week 06 重構
    → 三表正規化架構，資料不再重複

Week 05：學了 JSON 格式
    ↓ 和本週的正規化概念互相印證
    → JSON 的巢狀物件也是一種「關聯」的表達方式

Week 07（下週）：ParseHub 爬蟲抓回來的資料
    ↓ 直接套用本週的清洗公式
    → 外部資料通常很骯髒，清洗是必要步驟

Week 09：Looker Studio 進階視覺化
    ↓ 乾淨的多表資料是視覺化的基礎
    → 資料不乾淨，圖表就不可信
```

### 🤔 課後思考問題

1. **反正規化（Denormalization）：** 在某些情況下，資料庫設計師會刻意把表合併（反正規化），犧牲維護便利性來換取查詢速度。你能想到一個工管系的情境，在這個情境下「不正規化」反而是更好的選擇嗎？

2. **Google Sheets 的限制：** Google Sheets 最多支援 1,000 萬個儲存格。如果一家企業有 100 萬筆客戶資料，Google Sheets 還適合嗎？應該改用什麼工具？

3. **資料清洗的成本：** 業界估計，資料科學家 60-80% 的時間花在資料清洗，而不是建模和分析。你認為從「系統設計」的角度，有哪些方法可以在資料進入系統之前就確保品質，減少事後清洗的需要？（提示：回想 Week 02 的資料驗證功能）

### 📚 本週自學資源

| 資源 | 用途 |
|------|------|
| [Google Sheets QUERY 完整語法](https://support.google.com/docs/answer/3093343) | QUERY 函數進階用法 |
| [REGEXREPLACE 正規表達式](https://support.google.com/docs/answer/3098245) | 更複雜的字串替換 |
| [資料庫正規化入門（英文）](https://www.geeksforgeeks.org/normal-forms-in-dbms/) | 正規化的學術定義（1NF/2NF/3NF）|

---

### 📝 下週預告

> 🔜 **Week 07：網頁爬蟲不用寫 Code — ParseHub × Make 自動採集外部資料**
>
> - 什麼是網頁爬蟲？合法的爬蟲怎麼做？
> - ParseHub 免費版：點擊介面設定爬蟲，不需要寫任何程式碼
> - 把爬下來的資料自動存入 Google Sheets（套用本週的清洗公式）
> - 用 Make 定時觸發 ParseHub，實現全自動資料採集
>
> **課前任務：**
> 1. 下載並安裝 [ParseHub 桌面版](https://www.parsehub.com/quickstart)（Windows / Mac 均可）
> 2. 想一個你想爬取資料的網站（例如：人力銀行的工管相關職缺、金門縣政府的公告）

---

*雲端與數位內容管理 ｜ 國立金門大學 工業工程與管理學系 ｜ 2026 春季學期*
*Week 06 of 14*
