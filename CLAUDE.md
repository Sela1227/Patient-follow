# CLAUDE.md — AI 開發交接文件
> 這個檔案專門給 Claude 讀。讀完即可繼續開發，不需要問問題。
> **每次打包都更新此檔**（版本歷史保持在 README.md）。

---

## 一、系統是什麼

**彰濱癌症中心個管系統**（Patient Follow-up System）

- 台灣彰濱秀傳醫院癌症中心使用
- 追蹤癌症病人、記錄治療事件、計算**國健署 115 年強制申報癌症品質指標（60 項）**
- 本地端 PWA（IndexedDB 儲存，無後端）
- 主要使用者：癌症個案管理師

---

## 二、技術棧

```
React 18 + TypeScript + Vite + TailwindCSS
Zustand（全域狀態）
Dexie.js（IndexedDB wrapper）
date-fns（日期計算）
lucide-react（圖標）
```

---

## 三、設計系統（必須遵守）

### 視覺基準
**PatientsPage** 是所有頁面的視覺參考標準。Nordic-minimalist 風格。

### CSS 變數
```css
var(--bg)             /* 頁面背景 #F7F9FB */
var(--bg-card)        /* 卡片背景 #FFFFFF */
var(--bg-hover)       /* hover 背景 */
var(--border)         /* 主要邊框 */
var(--border-light)   /* 次要邊框 */
var(--text)           /* 主要文字 */
var(--text-secondary) /* 次要文字 */
var(--text-hint)      /* 提示文字 */
var(--primary)        /* 主色 #5B8FB9 */
var(--primary-hover)  /* 主色 hover #4A7A9E */
var(--danger)         /* 危險色 #D97B7B */
var(--success)        /* 成功色 #6BAF8D */
```

### 顏色
```
Sidebar 背景：#37516b
主色（鋼藍）：#5B8FB9
成功綠：#6BAF8D / #5A9A78
警告黃：#E4B95A / #B8941F
危險紅：#D97B7B / #B56060（= var(--danger)）
```

### 元件模式
```tsx
// 卡片容器
<div className="rounded-2xl overflow-hidden"
  style={{ border:'1px solid var(--border)', background:'var(--bg-card)' }}>

// Tag chip 按鈕（選中/未選中）
style={{
  background: isActive ? '#5B8FB9' : 'rgba(91,143,185,0.10)',
  color: isActive ? '#fff' : '#5B8FB9',
}}

// 按鈕標準
btn-primary   → 主動作（儲存、確認）
btn-secondary → 次要動作（取消）
btn-danger    → 危險操作（刪除確認）
icon button   → w-8 h-8 rounded-xl flex items-center justify-center
```

### 禁忌
- ❌ `bg-white`、`text-slate-*`、`border-slate-*`、`text-gray-*`
- ❌ `rounded-lg` 用於 icon button（應為 `rounded-xl`）
- ❌ 裸 `<h2>`/`<h3>` 標籤（改用 `<p>` 加樣式）
- ✅ 所有顏色透過 CSS 變數或 inline `style={{ }}` 指定

---

## 四、當前版本（V6.2.9）

### 重要架構（V6.x）

| 版本 | 關鍵變更 |
|------|---------|
| V6.0 | 主背景色調整；RFA/TACE/HAIC 事件類型 |
| V6.1 | **帳號登入系統**：LoginPage / authService / 帳號管理 / 癌別過濾 |
| V6.2 | 流程模擬修正 + 指標計算修復 |
| V6.2.0 | ToolsPage 改 Tab 系統；移除重複資料統計 |
| V6.2.1 | 待補資料 + 缺漏清單整合；癌別過濾確認 |
| V6.2.2 | 指標缺漏可收合；Child-Pugh 5項評分；HCC/EC/EMC 重複欄位清除 |
| V6.2.3 | 死碼清除（8頁面 215K chars）；動態import→靜態；console清理；slate-*清零 |
| V6.2.4 | 按鈕風格全面統一；btn-danger新增；圖示按鈕 rounded-xl |
| V6.2.5 | 🔴 CANCER_CODE_MAP 中文 key 被誤清空（指標達成摘要全空）修復 |
| V6.2.6 | 🔴 事件代碼不匹配（chemo/rt/target/intervention vs chemotherapy…）修復 |
| V6.2.7 | 測試資料全面重設計（42位，覆蓋所有事件類型） |
| V6.2.8 | WorkCenter 病人點選修復；影像種類整合 EventFormModal |
| V6.2.9 | **PatientDetailPanel 移除（28K 重複代碼）；病人詳情統一入口** |

---

## 五、關鍵檔案路徑

```
src/
├── pages/（11個）
│   ├── PatientsPage.tsx          視覺參考標準
│   ├── PatientDetailPage.tsx     病人詳情（5 Tab：總覽/診斷/事件/指標/追蹤）★唯一入口
│   ├── IndicatorsPage.tsx        指標管理（待補資料+缺漏整合，4個視圖）
│   ├── WorkCenterPage.tsx        工作中心
│   ├── PDCAPage.tsx              品質改善
│   ├── MasterDataPage.tsx        主檔維護（含帳號管理 Tab）
│   ├── ToolsPage.tsx             系統工具（4個 Tab）
│   ├── WorkPage.tsx              工作及會議
│   ├── ImportPage.tsx            資料匯入
│   ├── LoginPage.tsx             登入頁
│   └── QualityMonitorPage.tsx    （只含子組件 exports，無主頁）
│
├── components/
│   ├── EventFormModal.tsx        事件表單（影像→影像種類下拉）
│   ├── PatientFormModal.tsx      新增病人
│   ├── Sidebar.tsx               左側欄（#37516b）
│   └── ...
│
├── services/
│   ├── autoFieldService.ts       auto欄位計算（含 computeChildPugh）
│   ├── nationalIndicatorService3.ts  指標計算統一路由（60指標）
│   └── ...
│
├── db/
│   ├── index.ts                  Dexie DB Schema v5
│   └── testData.ts               42位測試病人（全事件類型覆蓋）
│
└── stores/index.ts               Zustand stores
```

---

## 六、導覽系統（重要）

```ts
// 統一使用 navigateToPatient() 導覽到病人詳情
const { navigateToPatient } = useNavigationStore();

// 用法
navigateToPatient(patientId, 'overview')    // 總覽 tab
navigateToPatient(patientId, 'basic')       // 基本資料 tab（人口學+診斷日期+ICD）
navigateToPatient(patientId, 'staging')     // 分期 tab（TNM）
navigateToPatient(patientId, 'cancer')      // 收案/癌別 tab（完治率+癌別欄位）
navigateToPatient(patientId, 'treatment')   // 事件 tab
navigateToPatient(patientId, 'indicators')  // 指標 tab
navigateToPatient(patientId, 'followup')    // 追蹤 tab

// ❌ 不要再直接用：
setSelectedPatientId(id);
setCurrentPage('patients');
// ✅ 改用 navigateToPatient(id, tab)
```

**App.tsx 渲染邏輯：**
```
currentPage === 'patients' && selectedPatientId → PatientDetailPage（初始 tab = initialPatientTab）
currentPage === 'patients' → PatientsPage
```

---

## 七、帳號系統

- 預設帳號：`admin` / `admin1234`（首次啟動自動建立）
- 角色：`admin`（全部）/ `nurse`（限定 allowedCancerCodes）
- 護理師可切換「瀏覽模式」（唯讀，看全部癌別）
- 癌別過濾套用：PatientsPage / WorkCenterPage / IndicatorsPage
- WorkPage / PDCAPage：跨癌別資料，不過濾

---

## 八、事件代碼（重要，雙碼向後相容）

DB 存的短碼 vs autoFieldService 舊長碼已修復為 dual-code：

| DB 代碼 | 說明 | autoFieldService |
|--------|------|-----------------|
| chemo | 化療 | chemo \|\| chemotherapy |
| rt | 放療 | rt \|\| radiation |
| target | 標靶 | target \|\| targetTherapy |
| intervention | 介入 | intervention \|\| interventional |
| surgery | 手術 | surgery |
| imaging | 影像 | imaging |
| rfa / tace / haic | 肝癌介入 | 完整代碼 |

---

## 九、踩過的坑（不要重蹈）

1. **CANCER_CODE_MAP 中文 key 被 cleanup regex 清空** → 指標達成摘要全顯示 0/0
2. **事件代碼長短不一致** → hasChemo/hasRT 等 auto 欄位永遠為「否」
3. **動態 import 混用靜態** → Vite bundle 警告；改為全靜態 import
4. **WorkCenter 點擊只設 patientId** → 忘記 setCurrentPage → 詳情頁不出現
5. **PatientDetailPanel 重複代碼** → 已移除，全部走 PatientDetailPage
6. **slate-* Tailwind class** → 深色模式失效，全部改 CSS vars
7. **h2/h3 裸標籤** → 語意錯誤，改用 `<p>` + style

---

## 十、打包規則

```bash
VERSION=6.x.x
cd /home/claude/case-manager-web && sed -i "s/APP_VERSION = '[0-9.]*'/APP_VERSION = '${VERSION}'/" src/version.ts
# 更新 README.md 版本記錄
rm -rf dist && npm run build
cp public/logo.jpg dist/ && cp public/manifest.json dist/ && cp README.md dist/ && cp CLAUDE.md dist/
cd dist && zip -r /mnt/user-data/outputs/Patient-follow-V${VERSION}.zip .
# Source zip（不含 node_modules/dist/.git）
cd /tmp && rm -rf src_tmp && mkdir src_tmp
cp -r /home/claude/case-manager-web/. src_tmp/
rm -rf src_tmp/node_modules src_tmp/dist src_tmp/.git
cd src_tmp && zip -r /mnt/user-data/outputs/Patient-follow-V${VERSION}-source.zip .
cd /tmp && rm -rf src_tmp
```

**版號規則：** +0.01 小修、+0.1 新功能、+1.0.0 大重構
