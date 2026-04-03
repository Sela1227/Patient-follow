# CLAUDE.md — AI 開發交接文件
> 這個檔案專門給 Claude 讀。讀完即可繼續開發，不需要問問題。
> 每次打包都更新此檔，版本歷史保持在 README.md。

---

## 一、系統是什麼

**彰濱癌症中心個管系統**（Patient Follow-up System）

- 台灣彰濱秀傳醫院癌症中心使用
- 追蹤癌症病人、記錄治療事件、計算 **國健署 115年強制申報癌症品質指標（60 項）**
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
**PatientsPage** 是所有頁面的視覺參考標準。

### CSS 變數（深淺色模式共用）
```css
var(--bg)          /* 頁面背景 */
var(--bg-card)     /* 卡片背景 */
var(--bg-hover)    /* hover 背景 */
var(--border)      /* 主要邊框 */
var(--border-light)/* 次要邊框 */
var(--text)        /* 主要文字 */
var(--text-secondary) /* 次要文字 */
var(--text-hint)   /* 提示文字 */
var(--danger)      /* 危險色 */
var(--success)     /* 成功色 */
```

### 顏色
```
主色：#5B8FB9（鋼藍）
Sidebar 背景：#4a8fa8（深天空藍）
Sidebar 選中文字：#ffffff
Sidebar 選中背景：rgba(255,255,255,0.20)
成功綠：#6BAF8D / #5A9A78
警告黃：#E4B95A / #B8941F
危險紅：#D97B7B / #B56060
```

### 元件模式
```tsx
// 卡片容器
<div className="rounded-2xl overflow-hidden"
  style={{ border: '1px solid var(--border)', background: 'var(--bg-card)' }}>

// 分隔線
style={{ borderBottom: '1px solid var(--border-light)' }}

// 左側 3px 強調邊框
style={{ borderLeft: '3px solid #5B8FB9' }}

// Chip 按鈕（選中/未選中）
style={{
  background: isActive ? '#5B8FB9' : 'var(--bg-hover)',
  color: isActive ? '#fff' : 'var(--text-secondary)',
}}
```

### 禁忌
- ❌ 不用 `bg-white`、`text-slate-800`、`border-slate-200` 等 hardcoded Tailwind 色彩
- ❌ 不用 `text-gray-*`，一律改用 CSS 變數
- ✅ 所有顏色透過 CSS 變數或 inline `style={{ }}` 指定

---

## 四、目前版本（V5.93）

### 最近重要版本
| 版本 | 關鍵變更 |
|------|---------|
| V5.83 | 測試資料重設計（45人，工作中心14種待辦全覆蓋） |
| V5.85 | 總覽頁 3×3 小格置中 + 主治醫師顯示 |
| V5.86 | 癌別欄位全面補完（41個缺口歸零）+ 頁標頭統一 |
| V5.87 | DiagnosisTab Column 3 可編輯癌別欄位 |
| V5.88 | 全頁面空字串掃描修復 |
| V5.89 | Sidebar #4a8fa8 + 影像指標自動計算 + 指標追蹤摘要 |
| V5.90 | 原始碼備份里程碑 |
| V5.91 | 43 個指標全部實作自動計算（原 28 個） |
| V5.92 | 品質監測頁 11 處中文修復 |
| V5.96 | Sidebar 顏色加深 + 乳攝/PET 自動抓取 + PDCAPage 中文補完 |

### 下次原始碼備份提醒：V5.95

---

## 五、關鍵檔案地圖

```
src/
├── pages/
│   ├── PatientsPage.tsx          視覺參考標準
│   ├── PatientDetailPage.tsx     病人詳情（Overview/診斷/事件/指標/追蹤 5 Tab）
│   ├── IndicatorsPage.tsx        指標資料頁（PatientDetailPanel）
│   ├── QualityMonitorPage.tsx    品質監測（達成總覽/缺漏清單/趨勢預警/報表匯出）
│   ├── WorkCenterPage.tsx        工作中心（儀表板 + 智慧提醒）
│   └── PDCAPage.tsx              品質改善（PDCA 循環）
│
├── components/
│   ├── Sidebar.tsx               左側欄（顏色 #4a8fa8）
│   ├── PageHeader.tsx            統一頁頭元件
│   ├── SmartDateInput.tsx        連續數字日期輸入（YYYYMMDD → 2026-03-05）
│   └── TNMInput.tsx              AJCC 9th TNM 點按式輸入 + 自動分期
│
├── services/
│   ├── nationalIndicatorService.ts   BC/CC/CRC/OC 指標計算
│   ├── nationalIndicatorService2.ts  PC/BLC/OVC/EMC/PAC 指標計算
│   ├── nationalIndicatorService3.ts  HCC/LC/GC/EC + unified router
│   └── masterDataService.ts          主檔查詢（癌別/醫師/科別等）
│
├── types/
│   ├── index.ts                  Patient/Event 型別 + CANCER_SPECIFIC_FIELDS（164 keys）
│   ├── nationalIndicators.ts     BC/CC/CRC/OC 指標定義
│   └── nationalIndicators2.ts    HCC/LC/GC/EC/PC/BLC/OVC/EMC/PAC 指標定義（共 43 個）
│
├── db/
│   ├── index.ts                  Dexie DB class + seedDefaultData() + refreshTestEvents()
│   └── testData.ts               45 位測試病人（每人明確 demo 目的）
│
└── version.ts                    APP_VERSION 常數
```

---

## 六、資料架構

### 資料庫（IndexedDB via Dexie）
```
db.patients         病人主檔（包含 cancerSpecificData）
db.events           治療事件
db.cancerTypes      13 個癌別
db.physicians       醫師主檔
db.departments      科別主檔
db.eventTypes       事件類型設定
db.patientPhysicians 病人-醫師 mapping
```

### 重要：唯一儲存位置原則
同一欄位只有一個儲存位置，不同頁面都寫入同一筆 `db.patients`：

| 欄位類型 | 儲存位置 |
|---------|---------|
| 基本資料（姓名/性別/生日） | OverviewTab 編輯後寫入 |
| 診斷/TNM/分期 | DiagnosisTab |
| 癌別專屬欄位 | DiagnosisTab Column 3 **或** IndicatorsPage 核心測量 |
| caseClass / curativeNumerator | IndicatorsPage PatientDetailPanel |
| cancerSpecificData | 上述任一頁面，用 `{ ...old, ...new }` merge 寫入 |

---

## 七、癌別專屬欄位（CANCER_SPECIFIC_FIELDS）

定義在 `src/types/index.ts` 的 `CANCER_SPECIFIC_FIELDS`，共 **164 個 key**，涵蓋 13 癌別。

新增欄位步驟：
1. 在 `types/index.ts` 的對應癌別加入 `CancerFieldDefinition`
2. DiagnosisTab 和 IndicatorsPage 會自動顯示（透過 CANCER_CORE_METRICS）

欄位類型：
```typescript
type: 'radio'    // chip 選擇（選項 ≤8 個用 chip，>8 用 select）
type: 'select'   // 下拉選擇
type: 'date'     // 日期（SmartDateInput）
type: 'text'     // 文字輸入
type: 'number'   // 數字輸入
type: 'auto'     // 自動計算（不顯示輸入框）
```

### 自動抓取欄位
| 欄位 | 來源 |
|-----|------|
| hasMammography / mammographyDate | 影像事件標題含「乳房攝影」/「乳攝」 |
| hasPET / petDate | 影像事件標題含「PET」/「正子」 |
| preOpImagingEvent（OVC/EMC） | 確診前後 90/30 天內的影像事件 |

---

## 八、國健署品質指標（43 個）

評估函數路由（`evaluateIndicatorUnified` in `nationalIndicatorService3.ts`）：
```
BC/CC/CRC → nationalIndicatorService.ts
OC        → nationalIndicatorService2.ts（部分）
HCC/LC/GC/EC/PC/BLC/OVC/EMC/PAC → nationalIndicatorService3.ts
```

所有 43 個指標均已實作自動計算邏輯（V5.91）。

---

## 九、工作中心智慧提醒規則

`WorkCenterPage.tsx` 的 `todoItems` useMemo：

| 類型 | 觸發條件 | 優先級 |
|------|---------|--------|
| overdue | 未完成事件 dueDate < 今天 | urgent 🔴 |
| today_fu | fu 事件 dueDate = 今天 | urgent 🔴 |
| post30 | 最後手術 25-35 天前 | high 🟡 |
| post90 | 最後手術 85-95 天前 | high 🟡 |
| week_event | 未完成事件 dueDate 在 7 天內 | high 🟡 |
| need_visit | 最後訪視 > 60 天 | high 🟡 |
| lost_contact | Class 1/2，最後事件 > 12 個月 | medium 🔵 |
| ongoing_chemo | 未完成 chemo 事件 | medium 🔵 |
| ongoing_rt | 未完成 rt 事件 | medium 🔵 |

**注意**：日期從 DB 種入時就固定。測試資料過期後需到「系統工具」→「更新測試事件」重新以今日為基準種入。

---

## 十、開發流程

### 版本規則
- `+0.01`：bug fix / 文字修復 / 小調整
- `+0.1`：新功能
- 每版必須更新 `src/version.ts` 和 `README.md`

### Build + 打包指令（複製即用）
```bash
# 更新版本號
VERSION=5.XX
sed -i "s/APP_VERSION = '[0-9.]*'/APP_VERSION = '${VERSION}'/" src/version.ts

# Build
rm -rf dist && npm run build
cp public/logo.jpg dist/ && cp public/manifest.json dist/ && cp README.md dist/

# 佈署版（dist）
cd /tmp && rm -rf Patient-follow-V${VERSION} && mkdir Patient-follow-V${VERSION}
cp -r /home/claude/case-manager-web/dist/. Patient-follow-V${VERSION}/
zip -r /mnt/user-data/outputs/Patient-follow-V${VERSION}.zip Patient-follow-V${VERSION}
rm -rf Patient-follow-V${VERSION}

# 原始碼版（source）
cd /tmp && rm -rf Patient-follow-V${VERSION}-source && mkdir Patient-follow-V${VERSION}-source
cp -r /home/claude/case-manager-web/. Patient-follow-V${VERSION}-source/
rm -rf Patient-follow-V${VERSION}-source/node_modules Patient-follow-V${VERSION}-source/dist Patient-follow-V${VERSION}-source/.git
zip -r /mnt/user-data/outputs/Patient-follow-V${VERSION}-source.zip Patient-follow-V${VERSION}-source
rm -rf Patient-follow-V${VERSION}-source
```

### README.md 更新規則
每版在版本歷史最前面加入：
```markdown
### V5.XX (YYYY-MM-DD)
- 簡述改動
```

---

## 十一、必做檢查清單（打包前）

```bash
# 1. TypeScript 無錯誤
npm run build

# 2. 中文字元不為零（用 Python 快速掃）
python3 -c "
import re, os
for f in os.listdir('src/pages'):
    c = open(f'src/pages/{f}').read()
    n = len(re.findall(chr(0x4e00)+'-'+chr(0x9fff), c))
    # 實際用 [\u4e00-\u9fff]
    print(f'{f}: {n} CJK')
"

# 3. 無空白 toast
grep -r "toast\.\w\+('')" src/
```

---

## 十二、測試資料設計

`src/db/testData.ts`：45 位病人 × 13 癌別

工作中心觸發對照：
```
逾期：BC03, LC02
今日回診：CRC01
術後30天：OC01(d30), HCC01(d28)
術後90天：CRC02, GC01(d90), EC01(d88)
本週事件：CRC03, PC01, BLC01, CC02, PAC03
待訪視：LC03(80d), OC02(75d), OVC01(75d), EMC01(90d)
失聯：EC02, CC01（最後事件 400-430 天前）
進行中化療：BC02, HCC02, OVC02, EMC02, BLC02, PAC01
進行中放療：LC01
本月新收：HCC03, EC03, EMC03
```

---

## 十三、常見問題

| 問題 | 解法 |
|------|------|
| 工作中心全部顯示 0 | 系統工具 → 更新測試事件（重算日期） |
| TypeScript 編譯失敗 | 先 `npm run build` 確認錯誤，通常是 unused import 或 type mismatch |
| 新增癌別欄位後 form 不顯示 | 確認 `type` 不是 `'auto'`，DiagnosisTab 自動跳過 auto 欄位 |
| 指標計算結果不對 | 查 `evaluateIndicatorUnified` 路由，確認進入正確的 service |
| 打包後 zip 內資料夾名稱 | 用 `/tmp` staging 確保 zip 內資料夾名稱等於 zip 檔名 |

---

## 十四、待辦事項

### 優先級 1（功能缺口）
- [ ] 資料匯入工具（Excel → DB，含欄位 mapping UI）
- [ ] 指標 Tab：點未達指標 → 跳到對應填寫欄位

### 優先級 2（品質）
- [ ] 定期檢查所有頁面中文字元數
- [ ] 各頁面 PageHeader subtitle 是否有意義

### 原始碼備份
- 每次發版均自動包含 source zip，無需額外提醒

---

*最後更新：V5.96 / 2026-04-03*
