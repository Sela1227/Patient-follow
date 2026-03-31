# 癌症個管系統 - 開發交班文件

> 最後更新：2026-03-31 V5.59
> 用途：新對話開始時上傳此文件，讓 Claude 快速了解專案狀態

---

## 一、專案概述

**系統**：彰濱癌症中心 - 個管病患追蹤系統
**技術**：React 18 + TypeScript + Vite + TailwindCSS + IndexedDB (Dexie.js) + Zustand
**工作目錄**：`/home/claude/case-manager-web/`

---

## 二、當前版本 V5.59

### 頁面架構（7頁）
```
日常工作
├─ WorkCenterPage      三層級待辦（預設全收合）
└─ PatientsPage        三行式篩選卡片 → PatientDetailPage（5 Tab）

品質管理
├─ QualityMonitorPage  4 Tab + DateFilterButton
└─ PDCAPage            PDCA + DateFilterButton

資料管理
├─ IndicatorsPage      待辦清單 + TNMInput 分期整合
└─ WorkPage

系統設定
├─ MasterDataPage
└─ ToolsPage → ImportPage
```

---

## 三、關鍵元件

### SmartDateInput（src/components/SmartDateInput.tsx）
連續數字輸入日期，不跳格：
```tsx
<SmartDateInput value={date} onChange={v => setDate(v)} className="input" />
// 輸入 20260305 → 自動格式化 2026-03-05
```
**套用範圍**：所有日期欄位（病理確診日、初診日、事件日期、排程日期等）

### TNMInput（src/components/TNMInput.tsx）
AJCC 9th 點按式 TNM + 自動期別：
```tsx
<TNMInput
  cancerCode="BC"  // 癌別代碼
  prefix="c"       // 'c'=臨床, 'p'=病理
  tValue={form.cT} nValue={form.cN} mValue={form.cM}
  onTChange={...}  onNChange={...}  onMChange={...}
  onStageChange={v => setStage(v)}  // 自動套用期別
/>
```
**整合頁面**：PatientDetailPage 診斷 Tab、IndicatorsPage 基本指標 Tab

### DateFilterButton（src/components/DateFilterPanel.tsx）
```tsx
import { DateFilterButton, calcDateRange, type DatePreset } from '../components/DateFilterPanel';
const dateRange = useMemo(() => calcDateRange(datePreset, start, end), [datePreset, start, end]);
// ⚠️ dateRange 必須用 useMemo，不能用 const
```
**套用頁面**：PatientsPage（三行卡片 inline）、QualityMonitorPage、PDCAPage

### StageSelector（src/components/StageSelector.tsx）
已整合進 TNMInput 流程，現在期別唯讀（只能由 TNMInput 套用，有清除鍵）

---

## 四、TNM 分期服務（src/services/tnmStagingService.ts）

**AJCC 9th** 更新的癌別：LC（+T1d/T2c/N2a/N2b/IIID）、CRC（N細分）、OVC（FIGO 2023）、EMC（FIGO 2023 全新）、CC（FIGO 2018）

**維持 AJCC 8th**：BC、GC、EC、OC、HCC、PC、BLC、PAC

**各癌別 M 值：**
| 癌別 | M 值 |
|------|------|
| BC/GC/EC/OC/HCC/PAC | M0、M1 |
| LC/CRC | M0、M1a、M1b、M1c |
| PC | M0、M1a、M1b、M1c |
| BLC | M0、M1a、M1b |
| OVC | M0、M1a、M1b |
| EMC/CC | M0、M1 |

---

## 五、設計系統

```css
Primary: #5B8FB9  Sidebar: #2E4057
--bg: #F7F9FB  --bg-card: #FFFFFF  --bg-hover: #F0F4F8
--border: #E8ECF1  --border-light: #F0F4F8
--text: #2C3E50  --text-secondary: #7C8DB0  --text-hint: #A8B5C8
--success: #6BAF8D  --warning: #E4B95A  --danger: #D97B7B
```

**列表慣用模式**：
```jsx
<div className="rounded-2xl overflow-hidden"
  style={{ border:'1px solid var(--border)', background:'var(--bg-card)' }}>
  <div style={{ borderBottom: isLast ? 'none' : '1px solid var(--border-light)' }}
    onMouseEnter={e => e.currentTarget.style.background = 'var(--bg-hover)'}
    onMouseLeave={e => e.currentTarget.style.background = 'transparent'}>
```

**Chip/Tab**：
```jsx
style={{ background: isActive ? '#5B8FB9' : 'rgba(91,143,185,0.10)', color: isActive ? '#fff' : '#5B8FB9' }}
```

**左側邊框強調**：
```jsx
// 危險: borderLeft: '3px solid #D97B7B'
// 成功: borderLeft: '3px solid #6BAF8D'
// 次要: borderLeft: '3px solid #A8B5C8'
background: 'var(--bg-hover)'
```

---

## 六、打包命令

```bash
cd /home/claude/case-manager-web
sed -i "s/APP_VERSION = '[0-9.]*'/APP_VERSION = '{版本}'/" src/version.ts
# 更新 README.md 版本號和歷史
rm -rf dist && npm run build
cp public/logo.jpg dist/ && cp public/manifest.json dist/ && cp README.md dist/
cd /tmp && rm -rf Patient-follow-V{版本} && cp -r /home/claude/case-manager-web/dist Patient-follow-V{版本}
zip -r /mnt/user-data/outputs/Patient-follow-V{版本}.zip Patient-follow-V{版本}
rm -rf Patient-follow-V{版本}
```

---

## 七、後續建議

### 優先級 1
- [ ] 病人詳情指標 Tab 加入「補填」功能（點未達指標 → 跳到對應欄位）
- [ ] PC 攝護腺癌期別整合 PSA + Grade Group 輸入

### 優先級 2
- [ ] 清理廢棄的 EventsPage.tsx
- [ ] Code splitting（目前 ~1.2MB）

### 優先級 3
- [ ] 化療排程甘特圖
- [ ] PDF 報告匯出

---

## 八、快速開始新對話

```
我要繼續開發癌症個管系統，目前版本 V5.59。
請先讀取 HANDOVER.md 了解專案狀態。
我想要 [描述需求]...
```
