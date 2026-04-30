# CLAUDE.md — 彰濱癌症個管系統 開發交接文件
> 給 Claude 讀的。讀完直接動手，不需要問問題。
> 每次打包更新此檔。版本完整歷程在 README.md。

---

## 一、系統是什麼

**彰濱秀傳癌症中心個管追蹤系統**（Patient Follow-up System）

- 台灣彰濱秀傳醫院癌症個管師日常使用
- 核心功能：追蹤14種癌別病人治療進度、計算國健署115年強制申報60項品質指標、MDT多專科會議管理
- 純前端 PWA，IndexedDB 儲存，無後端，支援離線使用
- 使用者：郭美伶（KAO，帳號14002）、楊靜雯（YANG，帳號11750）及管理員

---

## 二、技術棧

```
React 18 + TypeScript + Vite
TailwindCSS 3（utility class only，不寫 scoped CSS）
Zustand（全域狀態）
Dexie.js v4（IndexedDB ORM）
date-fns v4（日期計算）
lucide-react（圖示）
pptxgenjs + docx（MDT 產出）
xlsx（Excel 匯出）
```

---

## 三、設計系統（違反會被打）

### 視覺基準
**PatientsPage.tsx** 是所有頁面的視覺參考標準。Nordic-minimalist 風格。

### CSS 變數（只用這些）
```css
var(--bg)             /* #F7F9FB  頁面底色 */
var(--bg-card)        /* #FFFFFF  卡片 */
var(--bg-hover)       /* #F0F4F8  hover/次要背景 */
var(--border)         /* #E8ECF1  標準邊框 */
var(--border-light)   /* #F0F4F8  輕分隔線 */
var(--text)           /* #2C3E50  主文字 */
var(--text-secondary) /* #7C8DB0  副文字 */
var(--text-hint)      /* #A8B5C8  提示文字 */
var(--primary)        /* #5B8FB9  主色鋼藍 */
var(--primary-hover)  /* #4A7A9E */
var(--success)        /* #6BAF8D  成功綠 */
var(--warning)        /* #E4B95A  警告黃 */
var(--danger)         /* #D97B7B  危險紅 */
```

### 重要顏色（inline style 用）
```
Sidebar 背景：#37516b
深版主色：#4A7A9E（hover 用）
成功深：#5A9A78
警告深：#B8941F
危險深：#B56060
```

### 元件標準寫法
```tsx
// 卡片容器（統一用這個）
<div className="rounded-2xl overflow-hidden"
  style={{ border:'1px solid var(--border)', background:'var(--bg-card)' }}>

// tab chip 按鈕（選中/未選中）
style={{
  background: isActive ? '#5B8FB9' : 'var(--bg-hover)',
  color: isActive ? '#fff' : 'var(--text-secondary)',
}}

// 彩色半透明 badge
style={{ background: 'rgba(91,143,185,0.15)', color: '#4A7A9E' }}
// 格式：rgba(R,G,B,0.15) 背景，深化同色當文字
```

### ❌ 絕對禁忌
- `bg-white` `text-gray-*` `border-slate-*`（全部改 CSS vars）
- 裸 `<h2>` `<h3>`（改 `<p>` + className）
- `rounded-lg` 用在 icon button（應為 `rounded-xl`）
- 顏色寫死在 className，應用 `style={{}}`

---

## 四、當前版本（V6.6.6）

| 版本 | 關鍵變更 |
|------|---------|
| V6.5.4 | Phase 1 MDT：LY第14癌別、多重癌tag、Event MDT flags、DB v6 |
| V6.5.5 | Phase 2 MDT：月份行事曆＋建立會議Modal |
| V6.5.6 | Phase 3 MDT：會議室（5個Section、病人自動名單、討論填寫、結論寫回） |
| V6.5.7 | Phase 4 MDT：PPTX＋DOCX＋LINE通知產出 |
| V6.5.8 | 測試資料重設計（MRN改T-開頭）、刪除功能真正有效 |
| V6.5.9 | 修復：seedDefaultData未載入TEST_MEETINGS |
| V6.6.0 | 修復：重置預載資料→成功後自動reload；deprecated meta tag |
| V6.6.1 | 🔴 修復：DB Schema v6缺少users表→密碼錯誤無法登入 |
| V6.6.2 | 修復：v5 schema截斷（只剩users一張表）；帳號改為14002/KAO、11750/YANG |
| V6.6.3 | 修復：自動重建用原生indexedDB.deleteDatabase+reload |
| V6.6.4 | 修復：transaction未列meetings/meetingCases→無限reload迴圈 |
| V6.6.5 | 帳號調整（無重大功能變更） |
| V6.6.6 | 修復：ensureDefaultAdmin改為per-account檢查，換版不需重置DB |

---

## 五、業務對映表

### 5-1 癌別 → MDT → 個管師

| 癌別代碼 | MDT群組 | 負責個管師 | 排程（原則） |
|---------|--------|----------|------------|
| BC | 乳癌 | KAO（14002） | 每月第3個週四 07:30 |
| LC、EC | 胸腔癌 | KAO（14002） | 每月第3個週四 12:30 |
| OC | 頭頸癌 | 林伯儒 | 每月第3個週四 12:30 |
| LY | 頭頸癌（同場） | YANG（11750） | 每月第3個週四 12:30 |
| CRC、GC | 消化道癌 | YANG（11750） | 每月第1個週四 07:30 |
| HCC、PAC | 肝膽胰癌 | 林伯儒 | 每月第1個週四 07:30 |
| PC、BLC | 泌尿道癌 | 林伯儒 | 每月第2個週三 07:30 |
| CC、OVC、EMC | 婦癌 | YANG（11750） | 每月第2個週三 12:30 |

> **改這張表 = 改 types/index.ts 的 MDT_GROUP_CONFIG + CANCER_CODE_TO_MANAGER 兩處**

### 5-2 MDT Section → 資料來源

| Section | 自動來源 | 需手動填 |
|---------|---------|---------|
| 新個案 | 上次同癌別MDT後 initialDiagnosisDate 的病人 | 勾選誰要討論 |
| 前期追蹤 | 上次MDT有conclusion且nextAction未完成 | 本次進度更新 |
| 必要提報 | 系統偵測基準2.4八類（見§六） | 勾選類別 |
| 醫療小組 | 有「醫療小組」事件卡的病人 | 個管師回覆記錄 |
| 特殊議程 | 無（手動搜尋加入） | 全部手填 |

### 5-3 帳號對映

| 帳號 | 密碼 | 顯示名稱 | 負責癌別 |
|------|------|---------|---------|
| admin | admin1234 | 系統管理員 | 全部 |
| 14002 | 14002 | KAO | BC / LC / EC |
| 11750 | 11750 | YANG | CRC / GC / LY / CC / OVC / EMC |

---

## 六、必要提報自動偵測邏輯（基準2.4）

```
staging       → clinicalStage 為空 且 已有手術/化療/放療事件
second_primary → 同 mrn 有多筆不同 cancerTypeId 記錄
age85          → birthDate 計算年齡 ≥ 85 歲
neoadjuvant    → 有 event.isNeoadjuvant = true 的事件
positive_margin → caseClass=1 且 surgicalMargin='R1' 或 'R1+'
guideline      → event.requiresMDT=true（手動標記，系統無法自動判斷）
complex        → event.requiresMDT=true（同上）
comorbidity    → 暫無自動偵測（院內系統負責）
```

---

## 七、關鍵路徑（改功能查這裡，不要找 tree）

```
# 新增/修改癌別
types/index.ts          → CORE_14_CANCERS、MDT_GROUP_CONFIG、CANCER_CODE_TO_MANAGER
pages/PatientsPage.tsx  → CANCER_CODE_COLORS
pages/QualityMonitorPage.tsx → CANCER_CODE_MAP
db/index.ts             → CANCER_TYPE_MAP（seed 用）

# 修改 MDT 流程
pages/MDTPage.tsx        → 行事曆、建立會議 Modal
pages/MDTMeetingPage.tsx → 5個Section、討論填寫、完成會議
services/mdtOutputService.ts → PPTX/DOCX/LINE 產出

# 修改指標計算
services/nationalIndicatorService3.ts → 60項指標入口
services/autoFieldService.ts          → 事件欄位自動計算

# 修改病人表單
components/PatientFormModal.tsx → 新增病人
components/EventFormModal.tsx   → 事件記錄

# 修改 DB（必看坑 #4）
db/index.ts   → Schema 版本號 + 所有 table list
db/testData.ts → 測試病人（MRN T- 開頭）+ TEST_MEETINGS

# 修改帳號
services/authService.ts → ensureDefaultAdmin（per-account 新增）

# 修改打包流程
src/version.ts → APP_VERSION
```

---

## 八、DB Schema 規則（必讀，踩過三次坑）

當前版本：**v6**，共 14 張表：
`patients, events, cancerTypes, physicians, departments, imagingTypes, eventTypes, patientPhysicians, meetingTypes, workItems, meetingLocations, users, meetings, meetingCases`

**⚠️ Dexie 升版鐵律：新版 stores() 必須列出所有要保留的 table，未列出的 table 會被自動刪除。**

升版範本：
```ts
this.version(7).stores({
  // 舊的照抄
  patients: '++id, mrn, name, cancerTypeId, status',
  events: '++id, patientId, eventType, eventDate, dueDate, status, isCompleted',
  cancerTypes: '++id, code, sortOrder, isActive',
  physicians: '++id, name, departmentId, isActive',
  departments: '++id, name, sortOrder, isActive',
  imagingTypes: '++id, code, sortOrder, isActive',
  eventTypes: '++id, code, sortOrder, isActive',
  patientPhysicians: '++id, patientId, physicianId, [patientId+physicianId]',
  meetingTypes: '++id, code, sortOrder, isActive',
  workItems: '++id, patientId, type, priority, dueDate',
  meetingLocations: '++id, name, sortOrder, isActive',
  users: '++id, username, role, isActive',
  meetings: '++id, mdtGroup, date, status',
  meetingCases: '++id, meetingId, patientId, sectionType',
  // 新增的 table 加這裡
  newTable: '++id, field1, field2',
});
```

**transaction 規則：** db.transaction('rw', [...]) 必須列出所有在 transaction 內存取的 table，漏掉就拋 objectStore not found（坑 #9）。

---

## 九、踩過的坑（編號永遠遞增，不重排）

**#1 CANCER_CODE_MAP 中文 key 被 cleanup regex 清空**
- 症狀：指標摘要頁所有癌別顯示 0/0
- 原因：清理工具誤刪 string literal 內的中文 key
- 做法：regex 清理前確認不會誤觸 string 內容

**#2 事件代碼長短不一致（chemo vs chemotherapy）**
- 症狀：hasChemo / hasRT 等自動欄位永遠為「否」
- 原因：DB 存短碼，autoFieldService 用長碼比對
- 做法：autoFieldService 用 dual-code：`e.eventType === 'chemo' || e.eventType === 'chemotherapy'`

**#3 WorkCenter 點擊只設 patientId 忘記切頁**
- 症狀：點工作中心項目，病人詳情頁沒出現
- 原因：只呼叫 setSelectedPatientId，沒有 setCurrentPage('patients')
- 做法：統一用 `navigateToPatient(id, tab)`，不要直接改 state

**#4 Dexie 升版 schema 截斷（最貴的坑）**
- 症狀：v6.5.4 把 v5 截斷成只有 users 一張表，DB 升級時 11 張表被刪除
- 原因：插入新 version 程式碼時意外截掉了 v5 的 stores() 結尾
- 做法：每次改 schema 後 grep 確認所有 version block 都完整

**#5 initDatabase 錯誤處理抓到 seedDefaultData 的錯誤誤判為 schema 損壞**
- 症狀：首頁無限 reload 迴圈
- 原因：seedDefaultData 的 transaction 未列 meetings/meetingCases → objectStore not found → 被誤判為 schema 損壞 → reload → 無限循環
- 做法：transaction table list 要完整；加 sessionStorage 防護（cm_db_recovery_v6）

**#6 Dexie db.delete() 後同一實例不能 reopen**
- 症狀：自動修復流程 db.delete() + db.open() 拋出第二個 NotFoundError
- 原因：Dexie 實例刪除後內部狀態不能重置
- 做法：改用原生 `indexedDB.deleteDatabase('CaseManagerDB')` + `window.location.reload()`

**#7 ensureDefaultAdmin 用 count > 0 跳過導致新帳號無法補建**
- 症狀：換版後舊帳號（nurse1/nurse2）存在，新帳號（14002/11750）不存在，登入密碼錯誤
- 原因：`if (count > 0) return` 只要表有任何帳號就跳過全部
- 做法：改為逐一 `where('username').equals(u.username).first()`，不存在才新增

**#8 測試資料 MRN 格式與刪除條件不一致**
- 症狀：「刪除所有測試病人」點了沒有任何病人被刪
- 原因：MRN 是 `00001001` 格式，但刪除靠 `mrn.startsWith('TEST')` 篩選
- 做法：測試資料 MRN 統一改 `T-` 開頭，刪除條件改 `mrn.startsWith('T-')`

**#9 seedDefaultData 的 transaction 漏列 table**
- 症狀：重置後啟動正常，但測試 MDT 會議沒有被建立；或拋 objectStore not found
- 原因：`db.transaction('rw', [...])` 沒有包含 `db.meetings` / `db.meetingCases` / `db.users`
- 做法：新增任何 table 存取，同步加進 transaction 的 table list

**#10 TEST_MEETINGS 沒有在 seedDefaultData 裡被呼叫**
- 症狀：預載資料後，多專科會議頁面空白（沒有測試會議）
- 原因：loadTestData() 有寫 MDT 邏輯，但 seedDefaultData 沒呼叫它；改寫後直接在 seedDefaultData 內建立 meetings
- 做法：MDT 會議直接在 seedDefaultData 的 transaction 內建立，不要另一個函數

---

## 十、導覽系統

```ts
// 統一用這個，不要直接改 state
const { navigateToPatient } = useNavigationStore();
navigateToPatient(patientId, 'overview');   // 總覽
navigateToPatient(patientId, 'basic');      // 基本資料
navigateToPatient(patientId, 'staging');    // 分期
navigateToPatient(patientId, 'cancer');     // 收案/癌別
navigateToPatient(patientId, 'treatment'); // 事件
navigateToPatient(patientId, 'indicators');// 指標
navigateToPatient(patientId, 'followup');  // 追蹤

// App.tsx 渲染邏輯
currentPage === 'patients' && selectedPatientId → PatientDetailPage
currentPage === 'patients'                      → PatientsPage
```

---

## 十一、打包指令（複製貼上執行）

```bash
VERSION=6.x.x
cd /home/claude/case-manager-web

# 更新版本號
sed -i "s/APP_VERSION = '[0-9.]*'/APP_VERSION = '${VERSION}'/" src/version.ts

# 更新 README.md 版本記錄（手動）

# 更新說明書版本號（自動，不需手動）
TODAY=$(date +%Y-%m-%d)
sed -i "s/> 版本 V[0-9.]*/> 版本 V${VERSION}/" USER_GUIDE.md
sed -i "s/| 更新日期 [0-9-]*/| 更新日期 ${TODAY}/" USER_GUIDE.md

# Build
rm -rf dist && npm run build
cp public/logo.jpg dist/ && cp public/manifest.json dist/
cp README.md dist/ && cp CLAUDE.md dist/
cp USER_GUIDE.md dist/  # 每版必備，使用者快速上手文件
cp 使用說明書.md dist/   # ⚠️ 每版必須附，說明書不存在就不能打包

# 佈署版
cd dist && zip -r /mnt/user-data/outputs/Patient-follow-V${VERSION}.zip .

# Source 版
cd /tmp && rm -rf src_tmp && mkdir src_tmp
cp -r /home/claude/case-manager-web/. src_tmp/
rm -rf src_tmp/node_modules src_tmp/dist src_tmp/.git
cd src_tmp && zip -r /mnt/user-data/outputs/Patient-follow-V${VERSION}-source.zip .
cd /tmp && rm -rf src_tmp
```

**版號規則：** +0.0.1 bug fix；+0.1 新功能；+1.0 架構大改

---

## 十一-b、打包前必做（功能說明書）

每次打包前先更新 `使用說明書.md`（放在專案根目錄）：
- 新功能 → 加到對應章節
- 移除功能 → 刪除對應說明
- 打包時一起複製進 dist/

說明書格式：讓個管師（非工程師）看得懂，不寫程式術語，只寫操作步驟。

---

## 十二、煙霧測試（每次升版必跑）

> ⚠️ **每版打包前必須更新 `USER_GUIDE.md`**，說明本版新功能操作方式，打包時一併放進 dist/。

```bash
cd /home/claude/case-manager-web

# TypeScript 編譯檢查
npm run build 2>&1 | grep "error TS"
# 期望：空白（無任何 error TS）

# 確認版本號正確
grep "APP_VERSION" src/version.ts
```

瀏覽器手動確認（換版後）：
1. 重新整理頁面 → 不出現 console error
2. 能用 `admin` / `admin1234` 登入
3. 多專科會議頁有 3 場測試會議（重置後才有）

---

## 十三、下版候選工作

按優先序：

1. **PPTX 版面細化** — 實際拿去開會才知道格式夠不夠用；YANG 和 KAO 有機會試用時可以同步調整
2. 指標摘要頁「待補資料」件數點擊後直接跳到對應病人
3. 資料匯入工具（Legacy Excel → IndexedDB，含欄位對映 UI）
4. MDT 統計報表（每月各癌別討論件數、必要提報類別分布）
5. 工作中心：放療超過56天提示（rt_prolonged）整合 MDT Section 3

---

## 十四、一句話總結

V6.6.6 把這半年來最惡心的三個連環 DB bug（schema 截斷→users 被刪→無限 reload→帳號補建邏輯錯誤）全部解決，MDT 四個 Phase 也完整落地。現在系統可以穩定交給 KAO 和 YANG 試用，下版優先做 PPTX 版面微調，再來才是資料匯入工具。
