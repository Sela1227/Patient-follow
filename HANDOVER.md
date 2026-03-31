# 癌症個管系統 - 開發交班文件

> 最後更新：2026-03-31 V5.47
> 用途：新對話開始時上傳此文件，讓 Claude 快速了解專案狀態

---

## 一、專案概述

### 系統名稱
彰濱癌症中心 - 個管病患追蹤系統

### 技術棧
- **前端**：React 18 + TypeScript + Vite
- **樣式**：TailwindCSS + CSS 變數設計系統（北歐簡約風）
- **資料庫**：IndexedDB (Dexie.js)
- **狀態管理**：Zustand
- **日期處理**：date-fns

### 核心功能
1. 病人管理（清單列、多重排序、收案日期篩選、癌別 badge）
2. 治療事件追蹤（手術、化療、放療等）
3. 國健署 115 年 60 項品質指標評估
4. 智慧待辦提醒（工作中心，預設收合）
5. 備份還原（JSON 格式）
6. Excel 資料匯入

---

## 二、目前版本狀態

### 當前版本：V5.47

### 頁面架構（7頁，待辦事項已移除）

```
日常工作
├─ 🏠 工作中心 (WorkCenterPage) - 三層級待辦（預設全收合）
└─ 👥 病人管理 (PatientsPage) → 詳情 (PatientDetailPage, 5 Tab)

品質管理
├─ 📈 品質監測 (QualityMonitorPage) - 4 Tab
└─ 🎯 品質改善 (PDCAPage)

資料管理
├─ 📋 指標資料 (IndicatorsPage)
└─ 💼 工作及會議 (WorkPage)

系統設定
├─ 🗄️ 主檔維護 (MasterDataPage)
└─ ⚙️ 系統工具 (ToolsPage)
     └─ 資料匯入 (ImportPage)
```

### 病人詳情頁 5 Tab
1. **總覽**：新增事件、逾期/待辦警示、真實指標達成率、最近事件
2. **診斷**：就地編輯所有欄位（點「編輯資料」切換），癌別專屬欄位顯示中文標籤
3. **治療**：依類型篩選、依年份分組、篩選後新增預設類型
4. **指標**：國健署指標即時評估（達成/未達/不適用）
5. **追蹤**：追蹤狀態、回診、訪視紀錄

### 病人管理頁重要功能
- 癌別代碼圓形 badge（13 種顏色）
- 多重排序（最多6條件疊加）：病歷號/姓名/診斷日期/癌別/期別/主治
- 收案日期篩選：本週/本月/本季/本年/指定年份/自訂區間
- 每列顯示：病歷號 · 癌別 · 期別 · Dr. 主治 · 初診日

### 品質監測 4 Tab（風格已對齊病人管理）
1. **達成總覽**：癌別 badge + 進度條、指標左側邊框
2. **缺漏清單**：首字縮寫頭像、膠囊型分組切換
3. **趨勢預警**：左側邊框預警卡
4. **報表匯出**：5種格式

---

## 三、關鍵檔案路徑

### 主要頁面
```
src/pages/
├─ WorkCenterPage.tsx
├─ PatientsPage.tsx         # 多重排序、日期篩選
├─ PatientDetailPage.tsx    # 5 Tab、診斷就地編輯
├─ QualityMonitorPage.tsx   # 4 Tab
├─ PDCAPage.tsx
├─ IndicatorsPage.tsx
├─ WorkPage.tsx
├─ MasterDataPage.tsx
├─ ToolsPage.tsx
└─ ImportPage.tsx
```

> ⚠️ EventsPage.tsx 已廢棄（檔案存在但已從路由移除）

### 共用元件
```
src/components/
├─ PageHeader.tsx        # 統一頁面標題列（所有頁面共用）
├─ EventFormModal.tsx    # 支援 defaultEventType prop
├─ PatientFormModal.tsx
├─ Sidebar.tsx           # 深藍灰 #1E2D3D
└─ BottomNav.tsx
```

### 核心服務
```
src/services/
├─ nationalIndicatorService.ts   # Part 1 (CC/BC/CRC)
├─ nationalIndicatorService2.ts  # Part 2 (OC/HCC/LC/GC/EC)
├─ nationalIndicatorService3.ts  # Part 3 + 統一入口 evaluateIndicatorUnified()
├─ reportService.ts
├─ patientService.ts             # updatePatient 接受完整 Partial<Patient>
├─ eventService.ts
├─ backupService.ts
└─ masterDataService.ts          # 含 listPhysicians()
```

---

## 四、設計系統（src/index.css）

```css
/* 主色 */
Primary: #5B8FB9（鋼藍）

/* CSS 變數 */
--bg: #F7F9FB          /* 頁面背景 */
--bg-card: #FFFFFF
--bg-hover: #F0F4F8
--bg-input: #F0F4F8
--border: #E8ECF1
--border-light: #F0F4F8
--text: #2C3E50
--text-secondary: #7C8DB0
--text-hint: #A8B5C8
--success: #6BAF8D
--warning: #E4B95A
--danger: #D97B7B
--radius-lg: 16px  --radius-md: 10px  --radius-sm: 6px
```

### 列表慣用模式
```jsx
// 外層
<div className="rounded-2xl overflow-hidden"
  style={{ border: '1px solid var(--border)', background: 'var(--bg-card)' }}>
// 每列
<div style={{ borderBottom: isLast ? 'none' : '1px solid var(--border-light)' }}
  onMouseEnter={e => e.currentTarget.style.background = 'var(--bg-hover)'}
  onMouseLeave={e => e.currentTarget.style.background = 'transparent'}>
```

### 強調（左側邊框）
```jsx
// 逾期/嚴重: borderLeft: '3px solid #D97B7B'
// 達成/成功: borderLeft: '3px solid #6BAF8D'
// 一般/次要: borderLeft: '3px solid #A8B5C8'
// background: 'var(--bg-hover)'
```

---

## 五、版本號規則

- **+0.1**：新功能或大更動
- **+0.01**：修復 bug

### 每次發版必做
1. 更新 `src/version.ts`：APP_VERSION + VERSION_HISTORY
2. 更新 `README.md`：版本號 + 版本歷史
3. 打包：
```bash
sed -i "s/APP_VERSION = '[0-9.]*'/APP_VERSION = '{版本}'/" src/version.ts
rm -rf dist && npm run build
cp public/logo.jpg dist/ && cp public/manifest.json dist/ && cp README.md dist/
zip -r /mnt/user-data/outputs/Patient-follow-V{版本}.zip dist/*
```

---

## 六、技術細節備忘

### 指標評估
- 統一入口：`evaluateIndicatorUnified(indicator, patient)` in nationalIndicatorService3.ts
- 參數順序：先 indicator，後 patient（PatientWithEvents 含 events 陣列）
- threshold 預設 80

### 癌別代碼對應
```typescript
const CANCER_CODE_MAP = {
  '乳癌': 'BC', '子宮頸癌': 'CC', '大腸直腸癌': 'CRC',
  '口腔癌': 'OC', '肝癌': 'HCC', '肺癌': 'LC', '胃癌': 'GC',
  '食道癌': 'EC', '攝護腺癌': 'PC', '膀胱癌': 'BLC',
  '卵巢癌': 'OVC', '子宮內膜癌': 'EMC', '胰臟癌': 'PAC',
};
```

---

## 七、後續開發建議

### 優先級 1
- [ ] 品質監測篩選列 UI 改成按鈕風格（非裸 input）
- [ ] 病人詳情指標 Tab 加入「補填」功能
- [ ] 缺漏清單「前往填寫」後自動捲動到對應指標

### 優先級 2
- [ ] 清理廢棄的 EventsPage.tsx
- [ ] Code splitting 減少載入大小（目前 ~1.2MB）

### 優先級 3
- [ ] 化療排程甘特圖
- [ ] PDF 報告匯出
- [ ] 多使用者後端支援

---

## 八、快速開始新對話

```
我要繼續開發癌症個管系統，目前版本 V5.47。
請先讀取 HANDOVER.md 了解專案狀態。
我想要 [描述需求]...
```

---

*此文件隨版本更新，建議每次大版本後重新匯出*
