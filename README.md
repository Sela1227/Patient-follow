# 彰濱癌症中心 - 個管病患追蹤系統

基於 React + TypeScript + IndexedDB 的個管病患追蹤系統，支援國健署 13 癌診療核心測量指標。

**當前版本：V6.4.2**

## 重要警告

1. **資料存在瀏覽器**：清除瀏覽器資料會遺失所有病歷！
2. **iOS 7 天清除**：iOS Safari 會在 7 天未使用後自動清除網站資料
3. **必須定期備份**：將備份檔儲存到 iCloud / Google Drive / 電腦

---

## 版本歷史

### V6.4.2 (2026-04-04)
- **Q1: 切緣狀態簡化為兩個按鈕（R0 / R1+）**
  - 原本 4 個下拉選項（R0/R1/R2/Rx）→ 兩個 chip 按鈕
  - R0（切緣陰性）綠色，R1+（非陰性）紅色，視覺直觀
  - 顯示模式也同步更新為中文說明
- **Q2: 攝護腺癌指標（PC-1/2/3/5）修復**
  - 根本原因：isAdenocarcinoma() 只檢查 morphologyCode（舊欄位，現代病人不填）
  - 修復：同時檢查 cancerSpecificData.isAdenocarcinoma / histologyType / patient.histology
  - PC-1（DRE）、PC-2（PSA）、PC-5（根除術）現在可正確計算
- **Q2: 食道癌指標（EC-1/3/4）修復**
  - EC-1、EC-3：hasSurgery 已有 primarySurgeryCode fallback（本版確認已生效）
  - EC-4：isTanyN1_3M0 的 cN 比對改用 normalizeStage，避免 'N1'.includes('1') 誤判

### V6.4.1 (2026-04-04)
- **測試資料擴充（19 → 34 位病人）**：
  - **工作中心觸發病人（10 位，涵蓋所有 todo 類型）**：
    - WC01: 緊急 — 逾期事件（BC，化療追蹤逾期7天）
    - WC02: 緊急 — 今日回診（CRC，今天有 fu 事件）
    - WC03: 本週待辦 — 術後30天追蹤視窗（GC，術後第28天）
    - WC04: 本週待辦 — 術後90天追蹤視窗（OC，術後第88天）
    - WC05: 本週待辦 — 進行中化療（OVC，化療未完成）
    - WC06: 本週待辦 — 進行中放療（EC，放療未完成）
    - WC07: 本週待辦 — 60天未訪視（EMC，75天未訪視）
    - WC08: 需關注 — 失聯超過12個月（HCC，380天無聯繫）
    - WC09: 需關注 — 新收個案待建置（PAC，20天內無治療）
    - WC10: 本週待辦 — 本週有排程事件（CC，本週化療排程）
  - **補充指標覆蓋病人（5 位）**：
    - CRC03: CRC-1（malignant polyp T1 追加手術）
    - HCC03: HCC-6（BCLC 0/A/B 等待治療≤45天）
    - GC02: GC-5（病理I期內視鏡切除後1年胃鏡）
    - PAC02: PAC-8/9（Stage III 前導化療+全身性化療）
    - PC03: PC-5（局部侵犯根除性攝護腺切除術）

### V6.4.0 (2026-04-04)
- **Patient 欄位大清理（刪除 19 個確認無用欄位）**：
  idNumber, laterality, clinicalStageSystem, ypTNM, hasWashingCytology,
  rtModality, rtExecutionStatus, targetTherapyDate/Drug, immunotherapyDate/Drug,
  hormoneTherapyDrug, mdtNumber, hasMdtBeforeTreatment, lastFollowUpDate,
  lastImagingType, coreMetrics, pivkaII, dre
  （全部確認在整個 codebase 無任何引用，且已有對應的 cancerSpecificData 或事件記錄替代）
- **全新測試資料（19 位病人，13 癌別，60 指標完整覆蓋）**：
  BC01-03（BC-2/3/4）、CC01-02（CC-2/3/4）、CRC01-02（CRC-2/3）
  EC01（EC-2/4）、EMC01（EMC-1/2/8）、GC01（GC-1/4）
  HCC01-02（HCC-1/2/4/5）、LC01（LC-1）、OC01（OC-1/5）
  OVC01（OVC-1/2/3）、PAC01（PAC-1/3/4/5）、PC01-02（PC-2/3/4）、BLC01（BLC-1/2/3）
  每位病人有完整事件序列，用於驗證指標計算正確性
- **測試資料格式重寫**：新格式 ALL_TEST_PATIENTS（patient + events 配對）
  取代舊的 TEST_PATIENTS/TEST_EVENTS 分離格式，更清晰且型別安全

### V6.3.9 (2026-04-04)
- **全面重複 code 盤點與清理（~19,500 chars）**：
  - **PatientDetailModal.tsx（刪除 14,556 chars）**：完全沒有被任何檔案 import，純粹的 dead code
  - **indicatorHelpers.ts（新建共用 helper）**：從 3 個 indicator service 提出重複函式：
    - getEventsByType / getFirstEvent / daysBetween / stageMatches / normalizeStage
    - 每個函式原本在 nationalIndicatorService.ts / .2.ts / .3.ts 各定義一次（共 15 個函式定義 → 5 個）
  - **DeleteConfirmModal（提出為 components/DeleteConfirmModal.tsx）**：
    - PatientsPage 和 WorkPage 各自有完整的 DeleteConfirmModal 元件定義，現在統一共用
    - 支援 patientName 或 title 兩種 props 介面（兩頁的使用方式不同）
  - **TREATMENT_TYPES（統一從 autoFieldService export）**：
    - 原本在 autoFieldService / PatientDetailPage / WorkCenterPage 各自定義
    - 現在只在 autoFieldService 定義並 export，其他兩頁 import 使用
  - 同時清除各函式提出後的 orphaned code（孤立的函式 body、空 const 宣告等）

### V6.3.8 (2026-04-04)
- **🔴 修復：總覽頁與基本資料頁重複欄位問題**
  - 根本原因：OverviewTab 有內嵌 inline edit 表單（病歷號/性別/生日/狀態/個案分類），基本資料 tab 的 DiagnosisTab 也有相同欄位，兩者同時出現在頁面
  - 修復：OverviewTab 的「編輯」按鈕改為導向「基本資料」tab（navigateToTab 'basic'），不再在總覽頁內嵌 edit 表單
  - 移除 OverviewTab 的 6,000+ chars 重複 edit form code（isEditingBasic、basicForm、handleSaveBasic）
  - 總覽 tab 現在為純讀模式，全部編輯統一在 基本資料/分期/收案癌別 三個專屬 tab 進行

### V6.3.7 (2026-04-04)
- **指標缺漏（未達標）區塊 - 病人導覽改善**：
  - 依病人：病人 header row 右側新增直接 → 導覽圖示，不需先展開即可直接前往指標頁
  - 依病人：展開後的「前往指標頁」按鈕文字恢復（之前被清空剩 → 圖示）
  - 依指標：每個病人 row 改為全行可點擊（整行都是按鈕），點擊直接導覽至該病人指標頁
  - 兩種視圖都使用 navigateToPatient(id, 'indicators')

### V6.3.6 (2026-04-04)
- **修復：病人管理頁排序按鈕文字消失**
  - 原因：前版代碼清理時按鈕標籤被誤刪（中文字符被清空）
  - 修復「+ 新增排序」、「移除」、「重置」三個按鈕的文字
  - 截圖顯示兩個藍色小 pill 但沒有文字

### V6.3.5 (2026-04-04)
- **三條病人路徑模擬測試 + 修復**：
  - BC 乳癌（確診→手術→放療）：事件識別、首次治療日、BC-2 指標、auto 欄位計算均正常
  - HCC 肝癌 RFA（影像確診→無切片→RFA）：HCC-1/HCC-2 指標修復（RFA加入分母），首次治療日正確
  - PC 攝護腺癌 RT（切片→荷爾蒙→放療）：Gleason auto-compute 正常，但發現 PC-4 cN/cM 比對格式錯誤
- **🔴 修復：cN/cM 分期比對格式錯誤（影響 PC/EC/多指標）**
  - 病人存 cN='N0'、cM='M0'（帶前綴），但指標代碼比對 cN==='0'（無前綴）
  - 新增 normalizeStage() 函式，自動去除 N/M/T 前綴再比對
  - 修復 PC-4（局部侵犯放療）、EC-4（引導性化放療）等指標
- **PatientDetailPage TREATMENT_TYPES 補全新碼**：加入 rfa/tace/haic（之前漏掉）
- **nationalIndicatorService.ts 'immuno' 完全清除**

### V6.3.4 (2026-04-04)
- **🔴 修復：多癌別指標完全無法計算**
  - **HCC 肝癌**：HCC-2/HCC-4 的治療判斷（hadTx）只檢查手術+TACE，遺漏 RFA → RFA 病人落出分母，修復加入 rfa events 和 hasRFA
  - **免疫治療事件代碼錯誤**：getEventsByType(events, 'immuno') ← DB 代碼是 'immunotherapy'，影響 LC/BLC 免疫治療指標
  - **手術代碼依賴問題**（CRC/EC/LC/BLC）：指標計算依賴 primarySurgeryCode（舊版 ICD 手術代碼），現代系統用事件記錄，改為 fallback 到 surgeryEvents.length > 0
  - **PC 攝護腺癌**：PSA 現在從 cancerSpecificData.psa / psaValue / patient.psa 三處取值；gleasonScore 優先用 auto-computed 值

### V6.3.3 (2026-04-04)
- **Q1 🔴 修復：首次治療日永遠空白**
  - 根本原因：autoFieldService.ts 的 TREATMENT_TYPES 使用舊長碼（chemotherapy/radiation/targetTherapy/interventional）
  - DB 存的是短碼（chemo/rt/target/intervention），完全比對不到，所以首次治療日始終為空
  - 修復：TREATMENT_TYPES 加入雙碼支援，同時支援新短碼和舊長碼，以及 rfa/tace/haic
- **Q2 排序記憶：病人返回不再重置排序**
  - usePatientsPageStore 加入 sortKeys/setSortKeys 持久化
  - PatientsPage 的多重排序條件現在跨導覽保持不變
- **Q3 小選單改為按鈕點選**
  - 選項 ≤5 個的 select 欄位改為 radio（chip 按鈕）：
  - HCC BCLC 分期、Child-Pugh 5 項子評分、LC 組織型態、EC 腫瘤位置、PC Grade Group、EMC 術前影像類型
- **Q4 指標管理 Tab 重新命名（更精確）**
  - 待補資料 → 待補項目（含欄位缺失 + 指標缺漏）
  - 達成總覽 → 達成報告（各癌別×指標達成率表格）
  - 報表匯出 → 資料匯出

### V6.3.2 (2026-04-04)
- **癌別欄位盤點與修復**：
  - BC 乳癌：ER/PR 數字欄位（erPercent/prPercent，%）確認保留，可填百分比數字
  - BC 乳癌：erStatus/prStatus 改為 auto 欄位，由 erPercent/prPercent 自動推算（>0%→陽性，0%→陰性）
  - PC 攝護腺癌：gleasonScore 改為 auto，由 gleasonPrimary + gleasonSecondary 自動計算（如 3+4=7）
  - PAC 胰臟癌：刪除重複的 caLevel 欄位（與 ca199 完全相同）
- **Auto 欄位即時更新**：edit mode 中修改 erPercent/gleasonPrimary 等欄位時，對應 auto 欄位即時重新計算
- **數字欄位顯示單位**：view mode 中數字欄位現在顯示單位（如 90 % / 12 ng/mL / 2.5 cm）
- **測試資料更新**：BC 病人改用 erPercent/prPercent 數值，PC 病人改用 gleasonPrimary/Secondary

### V6.3.1 (2026-04-04)
- **病人詳情頁改為 7 Tab 設計（原 5 Tab）**：
  - 原「診斷」tab（35K chars，7個section）拆為 3 個獨立 tab：
    - 基本資料：組織型態 + 診斷資訊（初診日/病理確診日/首次治療日/ICD碼）+ 備註
    - 分期：臨床分期（治療前）cTNM + 病理分期（術後）pTNM + 淋巴結/切緣
    - 收案/癌別：收案分類/完治率 + 癌別專屬欄位（ER/PR/HER2、EGFR 等）
  - 7 tab 順序：總覽 → 基本資料 → 分期 → 收案/癌別 → 事件 → 指標 → 追蹤
  - navigateToPatient() 新增 basic / staging / cancer 目標 tab
  - PatientTab type 更新並 export
- **指標缺漏展開按鈕統一樣式**：
  - 從右上角「▼ 展開」文字改為 ChevronDown/ChevronUp 圖示
  - 與其他 accordion 區塊（缺完治率、無治療記錄等）完全一致

### V6.3.0 (2026-04-04)
- **🔴 修復：curativeDenominator / curativeNumerator 欄位整合後遺失**
  - 根本原因：PatientDetailPanel（已刪除）的「基本指標」tab 有這兩個欄位，PatientDetailPage 沒有
  - 這兩個是指標計算的核心（分母=是否根治性治療，分子=完治狀態），遺失後所有完治率指標永遠計算錯誤
  - 修復：在 DiagnosisTab Column 3 新增「收案分類 / 完治率」Section：
    - 個案分類（Class 0/1/2/3）按鈕選擇器
    - 是否根治性治療（是/否）→ 納入分母
    - 完治狀態（已完治/未完治）→ 納入分子（僅根治性治療才顯示）
- **版本跳至 V6.3.0**：PatientDetailPanel 整合完成，病人詳情統一入口里程碑

### V6.2.9 (2026-04-04)
- **病人詳情頁統一入口（移除 PatientDetailPanel 重複代碼）**：
  - 刪除 IndicatorsPage 內嵌的 PatientDetailPanel（~28,000 chars 重複代碼）
  - 刪除 EventsTabPanel（~6,400 chars 重複代碼）
  - 所有病人點擊（IndicatorsPage / WorkCenterPage / PatientsPage）統一導航至 PatientDetailPage
  - NavigationStore 新增 initialPatientTab 欄位 + navigateToPatient() action
  - 從指標頁點擊病人 → 直接開啟「指標」tab
  - 從工作中心點擊病人 → 直接開啟「總覽」tab
  - 取代原本 setSelectedPatientId + setCurrentPage 兩步驟操作
- **CLAUDE.md 全面更新**（從 V5.93 補到 V6.2.9）：
  - 新增「導覽系統」章節（navigateToPatient 統一規範）
  - 新增「踩過的坑」章節（7 個避免重蹈的問題）
  - 更新所有事件代碼、設計系統、按鈕標準

### V6.2.8 (2026-04-04)
- **🔴 修復：工作中心病人無法點選**
  - 根本原因：點擊病人只呼叫 setSelectedPatientId，未呼叫 setCurrentPage('patients')
  - App.tsx 判斷：需要 currentPage === 'patients' && selectedPatientId 才顯示 PatientDetailPage
  - 修復：WorkCenterPage 點擊 handler 同時設定 patientId + currentPage
- **影像種類整合 EventFormModal**：
  - 新增事件時，若事件類型為「影像追蹤」，標題改為「影像種類」下拉選單
  - 選項來源：主檔維護 → 影像種類（CT/MRI/超音波等）
  - 選「其他」可手動輸入
  - 非影像事件維持原本的自由文字標題

### V6.2.7 (2026-04-04)
- **測試資料全面重設計（42 位病人，覆蓋所有功能測試）**：
  - 所有 18 種事件類型都有實例：surgery/chemo/rt/target/immunotherapy/hormone/intervention/rfa/tace/haic/brachytherapy/mdt/team_meeting/tx_plan_confirm/fu/imaging/lab/visit/new_dx/pathology/staging/recurrence
  - HCC 3 位：RFA（王志豪）、TACE×2+HAIC（陳志強）、新收無治療（吳美華）
  - HCC Child-Pugh 改用 5 項子分數（cpAscites/cpBilirubin/cpAlbumin/cpPT/cpEncephalopathy）
  - LC 4 位：EGFR標靶、ALK標靶+化療、PD-L1免疫治療、已結案
  - CC 1 位有完整 CCRT + 近接治療（brachytherapy × 3）
  - 工作中心四種觸發都有覆蓋：
    - 緊急（逾期未完成）：BC02化療追蹤、CRC02術後訪視
    - 本週待辦：LC01/GC01/CC01/PC01
    - 需關注（>90天未訪視）：BC03
    - 新收待建置（30天內無治療）：HCC03/EMC03/BLC03
    - 失聯（>1年）：HCC02

### V6.2.6 (2026-04-04)
- **🔴 修復：事件類型代碼與指標計算嚴重不匹配**
  - 根本原因：DB 使用短代碼（chemo/rt/target/intervention），autoFieldService 使用舊長代碼（chemotherapy/radiation/targetTherapy/interventional）
  - 影響：hasChemo / hasRT / hasTargetTherapy / hasPreopCCRT 等 auto 欄位永遠計算為「否」，指標計算全部錯誤
  - 修復：autoFieldService 4 個 helper function 改為同時接受新舊代碼（dual-code support 向後相容）
- **事件篩選頁標統一整合**：
  - PatientDetailPage 和 IndicatorsPage 的事件篩選選項統一（兩個頁面現在一致）
  - 新增：近接治療、抽血、回診 篩選選項
  - 色碼對照表（EV_COLORS/PT_EV_COLORS）統一並擴充：brachytherapy/lab/team_meeting/new_dx/staging 新增色碼

### V6.2.5 (2026-04-04)
- **🔴 修復：病人詳情「指標達成摘要」永遠顯示空白**
  - 根本原因：PatientDetailPage 的 CANCER_CODE_MAP 所有中文 key 在之前的 cleanup 被誤清空（變成空字串 ''）
  - 結果：癌別名稱無法對應到英文代碼 → getIndicatorsByCancer() 回傳空陣列 → 0 項達成
  - 修復：還原正確的 key（乳癌/子宮頸癌/大腸直腸癌/口腔癌/肝癌/肺癌/胃癌/食道癌/攝護腺癌/膀胱癌/卵巢癌/子宮內膜癌/胰臟癌）

### V6.2.4 (2026-04-04)
- **按鈕風格全面統一**：
  - 新增 btn-danger CSS class（危險操作用紅色，如確認刪除）
  - 新增 CSS vars：--primary / --primary-hover
  - 14 個頁面/元件的圖示按鈕 rounded-lg → rounded-xl
  - PatientDetailPage：取消/儲存/編輯 改為標準 btn-secondary / btn-primary
  - BatchOperationModal：取消 改為 btn-secondary
  - PDCAPage：action 按鈕改為 btn-secondary
  - ToolsPage：中斷按鈕 rounded-lg → rounded-xl
  - PatientFormModal：新增醫師虛線按鈕 rounded-xl
  - 全面清除 hover:bg-slate-200（ImportPage / EventFormModal / PatientDetailModal）→ hover:bg-[var(--bg-hover)]
  - PDCAPage：#B56060 hardcoded → var(--danger)

### V6.2.3 (2026-04-04)
- **死碼清除**：
  - 刪除 8 個未使用頁面（215,308 chars）：DashboardPage / EventsPage / IndicatorMonitorPage / IndicatorTrendPage / PlaceholderPages / ReminderPage / StatsPage / SummaryPage
  - 移除 QualityMonitorPage 主組件函式（9,989 chars），僅保留被 IndicatorsPage 使用的 sub-component exports
  - App.tsx 移除對已刪頁面的路由和 import
- **動態 import → 靜態 import**（修正 Vite bundle warning）：
  - ToolsPage: 移除 await import('../services/backupService') / import('../stores')
  - MasterDataPage: 移除 import('../db') 動態載入
  - WorkCenterPage: 移除 import('../stores').useAuthStore 動態載入
- **Console 清理**：移除 18 個 console.log/warn（保留 catch block 的 console.error）
- **樣式一致性**：批量修復 163 個殘留 slate-* Tailwind class → CSS vars
- Bundle JS: 1,293 kB（較之前 ~1,500 kB 縮減）

### V6.2.2 (2026-04-04)
- **指標缺漏區塊可收合**：待補資料 tab 底部的「指標缺漏（未達標）」區塊新增展開/收合切換，預設收合
- **HCC 重複欄位修正**：
  - hepatitisType 兩個合一（B/C肝 + 肝炎類型合併，選項整合為：B型肝炎/C型肝炎/B+C型/Non-B Non-C/無/其他）
- **EC（食道癌）重複欄位修正**：
  - 刪除 tumorSite（與 tumorLocation 重複）
  - 刪除 hasCcrtBeforeSurgery（與 hasNeoadjuvantCCRT auto 重複）
- **EMC（子宮內膜癌）重複欄位修正**：
  - 刪除重複的 p53 欄位（保留一個）
  - msi 改名為「MSI 狀態（基因）」以區分 IHC mmrStatus
- **HCC Child-Pugh 改為5項子分數評分器**：
  - childPugh / childPughScore → 改為 auto（由子分數自動計算）
  - 新增5個子分數選擇器：腹水 / 膽紅素 / 白蛋白 / PT延長 / 肝性腦病
  - 每項3個選項附分數（如「無(1分)」「輕度(2分)」「中重度(3分)」）
  - 選完後自動加總並分級（5-6分=A, 7-9分=B, 10-15分=C）
  - computeChildPugh() 函式加入 autoFieldService.ts

### V6.2.1 (2026-04-04)
- **版號規則說明**：每次改動均更新版號（+0.01 小修、+0.1 新功能、+1.0.0 大重構）
- **指標管理：待補資料 + 缺漏清單整合**：
  - 移除「缺漏清單」獨立 tab chip
  - 指標缺漏清單（QMMissingTab）合併至「待補資料」tab 最底部，標題「指標缺漏（未達標）」
  - 「待補資料」chip 角標計數更新為：欄位缺失人數 + 指標缺漏人數
- **癌別過濾確認**：
  - PatientsPage / WorkCenterPage / IndicatorsPage：已正確依 allowedCodes 過濾 ✅
  - WorkCenterPage 統計卡（總病人數/追蹤中/本月新收/治療中）：基於 patientsWithEvents，資料載入時已過濾 ✅
  - WorkPage（工作及會議）/ PDCAPage（品質改善）：會議和品管專案為跨癌別資料，不需過濾，已確認不適用
- **系統工具頁 UI audit（6.2.0 同批）**：改 Tab 系統、移除重複資料統計、清零 slate-* class

### V6.2.0 (2026-04-04)

**V6.2.0 樣式/結構整理（UI audit）**：
  - **系統工具頁改版**：從滾動式改為 Tag 系統（備份/同步 / 資料匯出 / 資料庫 / 版本資訊）
  - **移除重複功能**：刪除 ToolsPage「資料統計」（與工作中心完全重複）
  - **全頁 h2/h3 → p 標籤**：PatientsPage/IndicatorsPage/PDCAPage/WorkPage/ToolsPage
  - **全頁 slate-* Tailwind class 清除**（共約 80+ 處）：
    - MasterDataPage: 35 處 → 0
    - PatientDetailPage: 27 處 → 0
    - ToolsPage: 8 處 → 0
    - WorkCenterPage: 7 處 → 0
    - PatientsPage: 4 處 → 0
    - WorkPage/IndicatorsPage: 各 3 處 → 0
  - 全部改用 CSS vars（--text / --text-secondary / --text-hint / --border / --bg-hover）
- **流程模擬修正（6項問題）**：
  1. 新增病人：癌別必填驗證，未選擇無法送出
  2. OverviewTab 組織型態：修復 require()（Vite 不支援），改為直接呼叫已匯入的 getHistologyOptions()
  3. OverviewTab 編輯：新增癌別選擇器，不需再繞回「編輯病人」modal
  4. 指標管理 PatientDetailPanel：新增「詳情」按鈕，可直接跳到 PatientDetailPage
  5. 缺漏清單點擊病人：自動切換到「核心測量」Tab（initialTab prop），省去手動切 Tab
  6. 工作中心：新增「新收個案待建置」類型，30天內新收且無任何治療記錄的追蹤中病人

**V6.2.0 補充修正（流程模擬 BC + HCC）**：
  - hasMammography：改為 type:auto，autoFieldService 自動偵測 imaging 事件標題含「乳房攝影/乳攝/mammograph」
  - mammographyDate：同步自動抓最近一次乳攝日期
  - hasSentinelBiopsy：從 type:text 改為 radio（是/否）
  - HCC 新增 hasHAIC（auto）、hasRT（auto）欄位
  - DiagnosisTab Column 3 讀取模式：合併 autoFieldService 計算值 + stored 值，依 metrics 順序顯示，auto 欄位標示⚡
  - InfoRow：新增 hint prop 顯示欄位來源標記
  - 移除殘留的 require()，改為正式 import computeAutoFields
  - 移除已過時的 CANCER_FIELD_LABELS 常數（6576 chars）
- **收案規則（切片確診原則）**：
  - 新增病人選擇癌別後顯示提示框：
    - 非肝癌：🟡「收案須以病理切片確診（請填寫病理確診日）」
    - 肝癌：🔵「肝癌可用影像診斷（CT/MRI/超音波）或病理切片作為收案依據」
  - 待補資料新增「缺病理確診日（非肝癌）」類別：自動排除肝癌，提醒其他癌別補填病理確診日

### V6.1.3 (2026-04-04)
- **主檔維護 / 系統工具 重新整理**：
  - 主檔維護移除「工具」Tab（刪除測試病人功能搬到系統工具）
  - 系統工具「資料庫管理」區整合刪除測試病人
- **共享資料夾同步功能（Chrome / Edge 限定）**：
  - 新增 syncService.ts：File System Access API 封裝
  - 系統工具新增「共享資料夾同步」區塊：
    - 連線/中斷共用磁碟資料夾
    - 立即同步：備份並寫入共享資料夾（含版本號/時間/使用者）
    - 比對版本：偵測本機與共享資料夾的版本差異
    - 衝突解決 Modal：顯示兩端版本資訊、建議較新版本、一鍵採用
  - 網路時間同步（timeapi.io → cloudflare → fallback 本機）
  - 不支援的瀏覽器顯示提示，自動降級為本機備份

### V6.1.2 (2026-04-04)
- **工作中心三格卡片改版**：隱藏展開列表，改為點擊統計卡才顯示對應病人清單
  - 緊急處理 / 本週待辦 / 需關注 三張卡，點擊後在下方展開病人列表
  - 卡片顯示選中狀態（加深底色 + 外框）、展開/收合提示
  - 病人列表依事件 type 自動對應圖示
  - 三個類別都是 0 時顯示「目前沒有待處理事項」
- **待補資料新增「無治療記錄」類別**：
  - 篩選條件：Class 1/2、追蹤中、無任何完成的治療事件
  - 涵蓋：手術/化療/放療/標靶/免疫/賀爾蒙/介入/RFA/TACE/HAIC/近接治療

### V6.1.1 (2026-04-04)
- **Sidebar 配色 #37516b**（深石板藍）：
  - 選中項目：rgba(255,255,255,0.15) 底色 + 白字
  - 未選中：透明底 + rgba(255,255,255,0.65) 淡白字
  - Hover：rgba(255,255,255,0.08)
  - 版本徽章、副標題、群組標籤、使用者區塊全部適配深色底
- **品質改善移至指標管理下方**：
  - 側欄群組：品質管理 → 指標管理 / 品質改善 → 資料管理 → 工作及會議
- **總覽頁「編輯」涵蓋所有可編輯欄位**：
  - 點「編輯」後 3×3 格子切換為表單模式
  - 可編輯：病歷號、性別（chip）、出生日期、個案分類（chip 0-3）、初診日、病理確診日、組織型態（chip 或文字）、追蹤狀態、死亡日期
  - 說明：⚡ 首次治療日與主治醫師由系統自動推算，無需手填
  - 存檔後同步更新所有欄位

### V6.1.0 (2026-04-04)
- **帳號登入系統**（新功能）：
  - 全新 LoginPage：帳號密碼登入，SHA-256 hash 存 IndexedDB，localStorage 持久登入
  - DB Schema v5 新增 users 表，首次啟動自動建立預設 admin（密碼：admin1234）
  - 角色：admin（全部可見）/ nurse（個管師，限定癌別）
  - 護理師可切換「瀏覽模式」（唯讀，可看全部癌別）
  - 瀏覽模式時頂部顯示黃色 banner 提示
- **主檔維護 → 帳號管理 Tab**：
  - 帳號列表：顯示角色/帳號/負責癌別/啟用狀態
  - 新增帳號：設定帳號名稱/顯示名稱/角色/密碼/負責癌別（多選 chip）
  - 編輯帳號：可改顯示名稱/角色/負責癌別/密碼（留空不更改）
  - 停用/啟用帳號（admin 帳號不可停用）
- **癌別過濾**：PatientsPage / WorkCenterPage / IndicatorsPage 依 allowedCancerCodes 過濾
- **Sidebar** 底部：顯示登入者名稱、角色圖示、登出按鈕、護理師瀏覽模式切換

### V6.0.2 (2026-04-03)
- **背景色修正**：主頁面背景恢復淺色（#F7F9FB），霧灰 #CBD5E1 改為 Sidebar 左導覽的背景色
  - Sidebar 選中項目：#5B8FB9（鋼藍）底色＋白色文字
  - Sidebar 未選中：透明底色＋深灰文字 #4A5568
  - Hover：rgba(91,143,185,0.12) 淡藍提示
  - 標題/副標題/分隔線改為深色調適配灰底
- **病人管理頁篩選列修正**：
  - 三列標籤從全部顯示「收案區間」→ 正確標示「收案區間」/「篩選條件」/「排序方式」
  - 年份選擇按鈕：預設顯示「年份」，選中後顯示「2026年」
  - 自訂日期按鈕：預設顯示「自訂」，選中後顯示日期範圍

### V6.0.1 (2026-04-03)
- **主背景色調整**：--bg #F7F9FB → #CBD5E1（霧灰），--bg-hover #F0F4F8 → #B8C4D4，--border #E8ECF1 → #94A3B8（深霧灰）
- **新增介入事件類型**：RFA（射頻消融）/ TACE（肝動脈栓塞化療）/ HAIC（肝動脈灌注化療）
  - 獨立 event code：rfa / tace / haic，sortOrder 71-73
  - autoFieldService 自動偵測：event.eventType === 'rfa/tace/haic' 或 title 含關鍵字
  - 指標服務自動計算：hasTACE / hasRFA 從 rfa/tace event 自動推算
  - HAIC 加入 HCC cancerSpecificData 欄位（hasHAIC，type auto）
  - 事件篩選 Tab 新增 RFA / TACE / HAIC 選項
  - ensureNewEventTypes 啟動時自動補種新事件類型（無需重置 DB）

### V6.0.0 (2026-04-03)
- **版本命名規則改為語義化三碼（major.minor.patch）**：
  - +V0.01：小 bug 修復
  - +V0.1：新增功能
  - +V1：大型重構
- **品質監測合併至指標管理頁（重大功能）**：
  - Sidebar「指標資料」→「指標管理」，「品質監測」項目移除
  - 頂層 tag chips 切換：待補資料 / 達成總覽 / 缺漏清單 / 趨勢預警 / 報表匯出
  - 「待補資料」chip 顯示待補件數紅色 badge，「缺漏清單」chip 顯示缺漏病人數
  - 切換到品質監測視圖時顯示日期篩選器，其他視圖顯示癌別篩選
  - quality_monitor 路由保留（重定向到 indicators）確保相容
  - QualityMonitorPage sub-components 全部 export，在 IndicatorsPage 直接 render

### V5.98 (2026-04-03)
- **全頁掃描修復（第四輪）**：
  - IndicatorsPage：4 個空白 label 補完（是否根治性治療、完治狀態、臨床分期標籤、組織型態）
  - WorkCenterPage：備份卡片標題「資料備份」（之前修過但未持久）
- **CLAUDE.md 更新**：移除原始碼備份提醒機制說明，確認每版自動打包

### V5.97 (2026-04-03)
- **指標資料頁待補追蹤強化**：
  - 新增「本季指標缺漏」分類：找出本季有活動、但必填癌別欄位有缺漏的病人
  - 新增「完治率待確認」分類：Class 1/2 有完治分母但尚未填完治分子的病人
  - 修正重複 title（缺個案分類 × 2、缺完治率 × 2 → 修正為缺臨床分期、缺組織型態）
  - subtitle 顯示本季倒數天數（如「N 項待補 · Q2 還有 45 天」）
  - 頁頭新增「品質監測」快速跳轉按鈕
- **Q2 合併方案確認**：品質監測（族群總覽/報表）與指標資料（個人填寫）定位不同，
  建議保持兩頁分開但強化連結；完整合併列為 V6.x 重構目標

### V5.96 (2026-04-03)
- **每次發版自動含 source zip，移除5版提醒機制**
- **zip 結構修正**：佈署版和原始碼版 zip 不再包含多餘的資料夾層，解壓縮即可直接使用
- **總覽頁首次治療自動推算**：從治療事件（手術/化療/放療等）自動找最早日期，顯示 ⚡ 標記，優先於手填值
- **指標資料頁「核心測量」Tab 強化**：
  - 空狀態文字：「此癌別尚無核心測量欄位」
  - 影響指標的欄位若為空，顯示紅色邊框提示
  - 欄位標籤旁顯示「→ BC-4, BC-2」標記，讓個管師知道填完哪個欄位可達標哪個指標
  - 填寫順暢度大幅提升：一眼看出缺什麼、填完影響什麼

### V5.95 (2026-04-03)
- **品質監測頁統計卡排版修正**：中間卡片「達成指標」從 `3/6達成指標` 改為分行清楚顯示數字與標籤（指標達標 / 總指標）
- **全頁面中文掃描修復（第三輪）**：
  - IndicatorsPage：癌別 select「全部癌別」、欄位 select「-- 選擇 --」、input placeholder「請輸入{欄位名稱}」
  - PatientsPage：病人數量文字「共 N 位病人」
  - PDCAPage：新增按鈕響應式文字「新增專案」
  - PatientDetailPage：刪除確認訊息「確定要刪除事件「{title}」嗎？」

### V5.94 (2026-04-03)
- **新增 CLAUDE.md**：AI 開發交接文件，任意新 Claude 對話讀完即可繼續開發
  - 設計系統（CSS 變數、顏色、元件模式、禁忌）
  - 關鍵檔案地圖、資料架構、唯一儲存原則
  - 43 個指標計算路由說明
  - 工作中心9種待辦觸發條件
  - 打包指令（複製即用）、檢查清單、常見問題
  - 測試資料設計、待辦清單

### V5.93 (2026-04-03)
- **邊欄顏色修正**：背景 #7aacc2 → #4a8fa8（加深），選中項目文字色 → #ffffff（純白，解決字看不見問題），hover 背景 → rgba(255,255,255,0.20)
- **乳房攝影指標自動抓取**：
  - DiagnosisTab 和 IndicatorsPage「核心測量」Tab：hasMammography / mammographyDate 欄位自動從影像事件偵測（標題含「乳房攝影」或「乳攝」）
  - hasPET / petDate 同樣自動從 PET 影像事件偵測（標題含「PET」或「正子」）
  - 已有手動值者保留，無手動值時自動回填
- **品質改善頁（PDCAPage）中文補完（15 處）**：
  - 篩選 chip：全部（N）/ 進行中（N）/ 已完成（N）
  - prompt 訊息：「請輸入 {階段} 內容（可多行）」
  - 日期顯示：開始：yyyy-mm-dd / 目標：yyyy-mm-dd
  - 表單 label：改善專案名稱＊ / 指標代碼＊ / 癌別代碼 / 指標名稱 / 現況基準值 / 目標達成率 / 開始日期 / 目標完成日期 / 問題描述與改善目標

### V5.92 (2026-04-03)
- **品質監測頁中文全面修復**（11 處）：
  - 癌別篩選器：選項「全部癌別」補回
  - 趨勢預警 Tab 統計卡標籤：嚴重（差≥20%）/ 警告（差10-20%）/ 注意（差<10%）
  - 警示卡副標題：目標 ≥N%・差距 N%・N 人未達標
  - 達成總覽癌別小卡：N/M 項達標
  - 指標詳細列副標題：N 人達標 / M 人適用，目標 ≥80%
  - 報表匯出 Tab 說明行：共 N 位病人
  - 匯出成功 toast：缺項清單已匯出（N 筆）/ 病人清單已匯出（N 位）

### V5.91 (2026-04-03)
- **指標自動計算全面補完**：43 個指標全部實作自動計算邏輯（原本僅 28 個）
  - HCC-1（極早期/早期治癒性療法率）、HCC-2（3個月影像追蹤）、HCC-4（1年≥3次影像）、HCC-5（R0切除率）、HCC-6（等待≤45天）
  - LC-1（IB-II期縱膈腔淋巴結≥3站）、LC-2（IIIA期縱膈腔淋巴結≥3站）
  - GC-1（R0切除率）、GC-3（術後30天死亡率）、GC-4（術後輔助化療率）、GC-5（1年胃鏡追蹤率）
  - EC-1（R0切除率）、EC-2（淋巴結≥15顆）、EC-3（術後30天死亡率）、EC-4（術前CCRT率）

### V5.90 (2026-04-03)
- **版本里程碑**：V5.90 穩定版，完成原始碼備份
- 健康掃描通過：所有頁面中文字元 ≥ 閾值、無空白 toast、Sidebar 顏色 #7aacc2、全 85 個 testData 癌別欄位有對應表單

### V5.89 (2026-04-03)
- **Sidebar 顏色**：#2E4057 → #7aacc2（明亮天空藍）
- **影像指標自動計算**：nationalIndicatorService3 啟用 imagingEvents，術前影像（確診前後90/30天內的影像事件）自動觸發 OVC/EMC 術前影像指標；HCC 術後3個月影像追蹤從影像事件自動推算
- **所有 toast 訊息中文化**：PatientDetailPage 6 個空字串 toast 全數修復（載入資料失敗/更新失敗/事件已刪除/儲存成功/儲存失敗：xxx）
- **指標資料頁強化**：
  - 快速追蹤摘要：個案分類/臨床分期/完治狀態 3 格小卡
  - 距上次訪視（>60天標紅）/距上次手術/待辦+逾期計數
  - TNM 展開顯示（cT/cN/cM → Stage）
  - InfoTab Section 標題補完：診斷資訊/病理分期（術後）/備註
  - 欄位 label 補完：確診日期/病理確診日/首次治療日/轉入醫院/組織型態

### V5.88 (2026-04-03)
- **全頁面空字串掃描修復**：系統性掃描 8 個主要頁面，修復所有空白按鈕/標籤
  - PDCAPage：ProjectFormModal（取消/儲存/建立）、CompleteModal（取消/確認完成）、modal 標題顯示專案名稱
  - PatientDetailPage：toggle complete toast（已標記完成/未完成）、找不到病人資料提示、返回病人列表按鈕
  - WorkCenterPage：備份卡片標題「資料備份」
  - IndicatorsPage：臨床期別標籤、病理期別標籤、個案分類警示文字（請先選擇個案分類）
  - MasterDataPage：取消/新增/儲存按鈕

### V5.87 (2026-04-03)
- **指標資料頁「基本指標/核心測量」Tab 編輯鈕文字還原**：取消/儲存/編輯按鈕文字空白修復；事件 Tab「新增事件」「化療排程」按鈕文字還原
- **資料唯一儲存確認**：DiagnosisTab、IndicatorsPage PatientDetailPanel、OverviewTab 均寫入同一筆 db.patients 記錄，無重複儲存；cancerSpecificData 合併邏輯改為 spread 合併（舊值 + 新值），避免遺漏
- **診斷頁 Column 3 癌別專屬欄位可編輯**：
  - 編輯模式：根據 CANCER_CORE_METRICS 定義自動生成輸入元件（chip 選擇 / select / 日期 / 文字）
  - 讀取模式：顯示現有值（InfoRow 格式）
  - DiagnosisTab form state 新增 cancerSpecificData，儲存時合併寫回
  - 不含 auto 自動計算欄位（type='auto'）

### V5.86 (2026-04-03)
- **各頁頭部風格統一**：
  - PatientsPage title → 病人管理；WorkCenterPage → 工作中心；IndicatorsPage → 指標資料
  - PatientsPage subtitle → 共 N 位病人；IndicatorsPage subtitle → N 項待補
  - PatientDetailPage 頂部 header 改用 CSS 變數（var(--bg-card)、var(--border)、var(--text)）
  - DashboardPage、StatsPage、SummaryPage、ReminderPage：h1/border/bg 改 CSS 變數
- **癌別專屬欄位全面補完（指標資料頁「核心測量」Tab 可編輯）**：
  - BC：新增 erStatus、prStatus、hasMammography、mammographyDate
  - LC：新增 pdl1Expression、hasEGFR、hasLobectomy、hasPET、petDate
  - OC：新增 hasBetelNut、hasSmoking、hasAlcohol
  - HCC：新增 hasCirrhosis、hasPortalVeinInvasion、hepatitisType
  - GC：新增 ebvStatus、hasPostOpChemo、hasNeoadjuvantChemo
  - EC：新增 tumorSite、hasCcrtBeforeSurgery
  - BLC：新增 isMuscleinvasive、carcinomaInSitu、hasIntravesical
  - OVC：新增 brca1、brca2、hrdStatus、hasDebulking
  - EMC：新增 mmrStatus、poleStatus、p53
  - CC：新增 hpvStatus、hpvType
  - PAC：新增 hasVascularInvasion、caLevel
  - CRC：新增 hasColonoscopy
- **testData key 名稱統一**：brafStatus→braf、krasStatus→kras、msiStatus→msi、emcType→endometrialType 等 10 組 key 更名，與 CANCER_SPECIFIC_FIELDS 對齊

### V5.85 (2026-04-03)
- **總覽頁資訊卡優化（置中小格）**：
  - 資訊改 3×3 小格置中顯示（值在上、標籤在下），視覺更緊湊
  - 新增主治醫師顯示（從 DB 載入，多位醫師以頓號分隔）
  - 欄位：病歷號／性別／年齡 ＋ 初診日／病理確診／個案分類 ＋ 組織型態／首次治療／主治醫師
  - 頂列：姓名 + 狀態徽章 + 癌別/Stage/Class + 編輯按鈕（同一行）
  - PatientDetailPage 新增 listPhysicians 載入，physicians 傳入 OverviewTab

### V5.84 (2026-04-03)
- **總覽頁重新設計（小卡佈局）**：
  - 頂部 4 格小卡：手術/化療/放療/其他治療計數，顏色醒目，一眼掌握治療歷程
  - 移除「新增事件」藍色大按鈕（可從「事件」Tab 新增）
  - 病人資訊卡：姓名列、關鍵日期 2×3 網格、底部最後事件/最後訪視
  - 逾期警示：有逾期事件時顯示橙紅警示條
  - 指標達成：進度條 + 達成比例 + 未達清單（最多2項）
  - 近期事件：最多 5 筆，緊湊列表
  - 近期待辦：有待辦時顯示

### V5.83 (2026-04-03)
- **README 整理**：合併重複版本紀錄（從 1689 行壓縮至 596 行），移除 80 個版本各重複 2-8 次的冗餘條目，保留最完整版本
- **測試資料全面重設計（V5.83）**：從 130 人調整為 45 人，每人都有明確 demo 目的
  - **工作中心 14 種待辦全覆蓋**：逾期（BC03, LC02）、今日回診（CRC01）、術後30天（OC01, HCC01）、術後90天（CRC02, GC01, EC01）、本週事件（CRC03, PC01, BLC01, CC02, PAC03）、待訪視（LC03, OC02, OVC01, EMC01）、失聯（EC02, CC01）、進行中化療（BC02, HCC02, OVC02, EMC02, BLC02, PAC01）、進行中放療（LC01）
  - **本月新收（3人）**：HCC03, EC03, EMC03（initialDiagnosisDate 設為本月）
  - **狀態多樣**：追蹤中 41 人、已結案 2 人（CRC04, PC03）、已歿 1 人（BC04）
  - **個案分類 Class 0-3 各有代表**
  - **指標完治示範**：BC01, CRC01, CRC04, HCC03, OVC03, CC02, PC03 已完治
  - **PDCA 測試專案**：更新為 3 個完整案例（BC提升術前確診率、CRC縮短等待時間、LC基因檢測改善完成）

### V5.82 (2026-04-03)
- **工作中心文字修復**
  - 空狀態文字還原：「目前沒有待處理事項 🎉」
  - Description 字串還原：逾期 N 天 / 術後第 N 天 / 建立 N 天 / N 個月
- **工作中心計數為零根本原因說明**：todoItems 依賴事件日期的相對時間（術後30天需 25-35 天內手術、待訪視需 60 天無訪視等）。測試事件種入時日期已固定，若 DB 初始化超過數週，原有的 daysAgo/daysLater 已超出觸發視窗，計數正常顯示為零。
- **系統工具加入「更新測試事件」按鈕**：以今日為基準重新計算所有測試事件日期（刪除舊事件、重新種入），工作中心的術後追蹤、本週待辦、進行中化療/放療等區塊立即顯示正確資料。
### V5.81 (2026-04-02)
- **品質監測頁完整修復**
  - **根本原因**：CANCER_CODE_MAP 中 9 個癌別名稱被清空（大腸直腸癌/口腔癌/肝癌/胃癌/食道癌/膀胱癌/卵巢癌/胰臟癌/子宮頸癌 → 空字串），導致 getNationalIndicators('') 返回空陣列，缺漏清單完全無資料。已全部還原。
  - PageHeader：品質監測 / 共 N 位病人
  - DateFilterButton：收案區間
  - OverviewTab 統計卡：病人總數 / 達成指標 / 整體達成率；空狀態：尚無指標資料
  - MissingTab：groupBy chip → 依病人（N 人）/ 依指標（N 項）；空狀態：所有病人指標都已達成！
  - 匯出 toast：指標報表已匯出 / 指標明細已匯出 / 缺失欄位報告已匯出 / 病人清單已匯出 / 評鑑摘要已匯出
### V5.80 (2026-04-02)
- **病人管理頁文字全面修復**
  - STATUS_LABELS：inactive→非活躍、completed→已結案（空字串修復）
  - SORT_LABELS：病歷號/姓名/診斷日期/癌別/分期/主治醫師（空字串修復）
  - PatientCard：已歿/已結案狀態標籤、編輯/刪除選單文字
  - 頁首按鈕：批次操作、新增病人
  - 自訂日期選擇器：自訂起訖日 label
  - DeleteConfirmModal：標題/說明文字已於 V5.79 修復
### V5.79 (2026-04-02)
- **中文字串完整還原（完成）**：根據 README 改版歷程逐功能核對，修復剩餘缺漏
  - PatientsPage：全部醫師（select）、確定要刪除病人（DeleteConfirmModal 標題/說明/按鈕：取消/確認刪除）
  - PDCAPage：Do 執行、標記完成（完成按鈕）
  - QualityMonitorPage：嚴重（差≥20%）/警告（差10-20%）/注意（差<10%）趨勢預警分類
  - IndicatorsPage：否（非根治）、已完治（curative chips）、所有病人的指標都已達成（空狀態）
  - WorkCenterPage：術後30天追蹤/術後90天追蹤（TYPE_LABELS）
### V5.78 (2026-04-02)
- **中文字串大規模還原（持續）**：以 V5.75 bundle 為真相來源，系統性比對 V5.77 和 V5.75 的 2000+ 中文字串，逐一還原遺失的標籤
  - WorkCenterPage：統計卡、待辦分類、備份文字（195 CJK）
  - PatientsPage：搜尋欄、篩選標籤、狀態顯示（68 CJK）
  - PDCAPage：PDCA 階段標籤、表單欄位、完成按鈕（184 CJK）
  - QualityMonitorPage：報表標籤、匯出選項、預警分類（167 CJK）
  - IndicatorsPage：完治選項、指標分類標題、事件篩選（123 CJK）
  - PatientDetailPage：CANCER_FIELD_LABELS 212 個標籤完整還原（1168 CJK）
- PDCAProject interface：改為 optional fields，防止資料格式不符導致空白頁
### V5.77 (2026-04-02)
- **中文字串全面還原**：掃描所有 .tsx 原始檔，修復因 Python 腳本處理導致中文被清空的問題。涵蓋 WorkCenterPage（統計卡標籤、待辦分類、備份文字）、PatientsPage（篩選器、快速標籤）、PDCAPage（統計標籤、PDCA 階段說明）、QualityMonitorPage（分頁標籤、匯出選項）、IndicatorsPage（子頁籤、事件篩選）、PatientDetailPage（Tab 標籤、資訊卡欄位、治療摘要）
### V5.76 (2026-04-02)
- **PDCAPage 空白修正（根本原因）**：PDCAProject interface 中 indicatorId/indicatorName/cancerCode/description/createdAt/updatedAt 改為 optional，並加入 problem/goal/indicator/responsiblePerson/department 欄位；加入 phases 存取的 null-safety（?. 可選鏈）防止未定義資料造成整頁 crash
- **DiagnosisTab 完整重寫**：移除「基本資料」section（讀取和編輯模式一致，不再有僅編輯時才出現的區塊）；三欄：欄1=組織型態+診斷資訊、欄2=臨床分期+病理分期、欄3=癌別專屬+備註；中文標籤完整還原
- **基本資料改在總覽頁編輯**：總覽頁資訊卡「編輯」按鈕開啟 inline edit（姓名/病歷號/性別/生日/狀態/個案分類/死亡日期），儲存後刷新
### V5.75 (2026-04-02)
- **診斷頁欄列修正**：完整重寫三欄佈局，欄1=組織型態+診斷資訊，欄2=基本資料(編輯時)+臨床分期+病理分期，欄3=癌別專屬+備註，各欄用獨立 div.space-y-3 堆疊
- **工作中心統計卡置中**：2×2 大統計卡加 text-center，數字/標題/副標題全置中
- **總覽頁快速列移除**：移除「前往事件/診斷/指標」三個多餘按鈕，保留「新增事件」單一主按鈕
### V5.74 (2026-04-02)
- **PDCAPage 修正**：localStorage 空時自動補種測試 PDCA 專案，解決空白頁問題
- **總覽頁快速動作優化**：改為 4 格橫向工具列（新增事件/前往事件/前往診斷/前往指標），更緊湊
- **診斷頁欄位重排**：欄1=組織型態+診斷資訊，欄2=基本資料(編輯時)+臨床分期+病理分期，欄3=癌別專屬+備註
- **測試資料全面更新（V5.74）**：13癌×10人=130人，病歷號改為8碼數字格式（BC:00001001-00001010，LC:00002001-00002010...），資料更豐富：完整TNM、surgicalMargin、pathologyConfirmDate、firstTreatmentDate、cancerSpecificData 各癌別指標欄位
### V5.73 (2026-04-02)
- **病歷號 8 碼自動補零**：新增病人 Modal 和診斷頁編輯，僅接受數字，不足 8 碼離開/儲存時自動在前面補零（123 → 00000123）；輸入中即時預覽補零結果
- **診斷頁移除「基本資料」讀取區塊**：讀取時基本資料已在總覽頁顯示，診斷頁不再重複；編輯模式下基本資料欄位仍可在診斷頁欄2上方編輯
- **總覽頁加「編輯」按鈕**：資訊卡右上角有「編輯」按鈕，點擊導航到診斷頁進行編輯
### V5.72 (2026-04-02)
- **ZIP 打包修正**：改成壓縮目錄內容（cd 進目錄後 zip *），解壓縮後不再出現雙層同名資料夾
- **總覽頁加入基本資料**：病人資訊卡擴充為完整格式，包含病歷號、個案分類、初診日、病理確診日、首次治療日、組織型態，以 2×3 欄位網格呈現，底部加最後事件/最後訪視/治療事件統計
- **工作中心優化**：統計改 2×2 大卡（數字 4xl 更大）、移除兩欄佈局、備份列移到頁面最底部（橫向一列式）
### V5.71 (2026-04-02)
- **工作中心移除快速入口**：側邊欄已有導航，右欄只保留資料備份卡片
- **診斷頁三欄間距修正**：由 CSS grid 自動排列改為三個獨立 div（各自 space-y-3），Section 高度不影響其他欄的間距；欄位分配：欄1=基本資料+組織型態、欄2=診斷資訊+臨床分期、欄3=病理分期+備註(+癌別專屬)
### V5.70 (2026-04-02)
- **工作中心全面重設計**：
  - 統計改大卡（4欄），數字顯示 3xl，有副標題和顏色區分
  - 待辦數字卡（3欄）可點擊展開對應清單，紅/藍/黃三色
  - 待辦清單項目加大，顯示癌別標籤、圖示背景色
  - 右欄：快捷入口（病人/指標/品質）大按鈕含圖示+副標題，備份卡移入右欄
  - 桌面 2/3 + 1/3 兩欄主佈局（md:grid-cols-3，待辦佔2欄）
- **診斷頁改三欄**：md:grid-cols-3，六個 Section 在桌面三欄呈現
### V5.69 (2026-04-01)
- **診斷頁排版修正**：histologyType 不再重複存入 cancerSpecificData，癌別專屬欄位過濾掉重複的組織型態顯示
- **化療排程按鈕加回**：PatientEventsTab（病人管理事件）和 EventsTabPanel（指標資料事件）均加回「化療排程」按鈕
- **總覽頁全面優化**：病人資訊卡（姓名/狀態/癌別/分期/Class/關鍵日期）、治療摘要（手術/化療/放療/其他計數）、快速動作 4 鍵、指標達成進度條 + 未達清單、近期事件列表、近期待辦
- **指標資料缺漏提醒擴充**：從 5 項擴充到 10 項，新增：缺臨床分期、缺組織型態、缺病理確診日、缺初診日、缺癌別專屬資料；各類別只在有病人時才顯示
### V5.68 (2026-04-01)
- **病人管理「治療」Tab 改名「事件」**：完整重構為 PatientEventsTab，與指標資料頁事件 Tab 完全一致：年份分組、12種類型篩選（手術/化療/放療/標靶/免疫/賀爾蒙/介入/病理/影像/訪視/多專科）、圓形完成勾選、新增/編輯/刪除
- **工作中心統計修正**：本月新收改用 initialDiagnosisDate 判斷；治療中改為「狀態追蹤中且90天內有治療事件」
- **診斷 Tab 兩欄佈局**：所有 Section 改 grid-cols-2，Section 各自獨立卡片，視覺更清爽；頂部 header 統一控制編輯/儲存/取消
### V5.67 (2026-03-31)
- **死亡日期只在狀態「已歿」時顯示**（避免誤填）
- **指標資料改為先按編輯才能儲存**：基本指標 Tab 和核心測量 Tab 加入 isEditing 模式，未編輯時唯讀（pointer-events-none）並顯示提示
- **核心測量改兩欄網格**：每個指標獨立卡片，grid-cols-2，呈現更緊湊
- **事件 Tab 全功能化**：新增 EventsTabPanel，與病人管理治療 Tab 體驗一致：依年份分組、類型篩選、新增/編輯/刪除、完成勾選
### V5.66 (2026-03-31)
- **選項 ≤7 的 select 欄位全改為 chip 點選**（HER2 IHC、BCLC分期、Grade Group、殘存腫瘤、手術切緣等）
- **histologyType 統一**：核心測量的組織型態直接讀寫 patient.histology，不重複存入 cancerSpecificData
- **死亡日期欄位**：加入病人診斷 Tab 基本資料，SmartDateInput 輸入
- **死亡自動算術後/化療後30天死亡**：autoFieldService 改用 patient.deathDate 計算（事件備用），IndicatorsPage 自動帶入
### V5.65 (2026-03-31) ⭐ 指標頁優化 + 測試事件擴充
- **IndicatorsPage 桌面兩欄佈局**：基本指標 Tab 改 grid-cols-2，有效利用大螢幕
- **儲存按鈕移至 Header**：移除固定底部 bar，改在標題列右側（含分期資訊）
- **各 Tab 改 flex-1 overflow-y-auto**，撐滿剩餘高度
- **測試事件大幅擴充**：13 癌別涵蓋各種典型治療流程：
  - BC: 乳保手術 → AC×4 → T×4 → 放療 → 荷爾蒙；全乳切除 → Trastuzumab+chemo × 6 → 放療
  - LC: 肺葉切除 → 靶向；不可手術 IIIA → CCRT → Durvalumab 維持
  - CRC: 手術 → FOLFOX；直腸癌術前 CCRT → 手術 → 輔助化療
  - OC: 廣泛切除+頸廓清 → 放療；ENE+ → 術後 CCRT
  - HCC: RFA（早期/晚期）；TACE × 2；OVC: 完整分期手術 → 6 個療程
  - CC: 根除性子宮切除；CCRT + 近接治療 × 4
  - PAC: Whipple → Gemcitabine+Cap；EC: 術前 CCRT → Ivor-Lewis + Nivolumab
  - 每筆手術均附完整病理報告事件（含 TNM、R 狀態、淋巴結、分子標誌）
### V5.64 (2026-03-31)
- **DateFilterPanel 全面改用 fixed 定位**：面板不再受 overflow:auto 截斷，所有頁面的日期篩選彈出框都完整顯示
- **首次治療日改為自動推算**：從治療事件（手術/化療/放療/標靶/荷爾蒙/介入治療）中取最早完成的一筆，顯示「⚡ 自治療事件推算」；無事件時顯示「尚無治療事件」
- autoFieldService 新增 computeFirstTreatmentDate() 匯出函數
### V5.63 (2026-03-31)
- **修正醫師篩選**：patientService.listPatients 加入 patientPhysicians JOIN，physicianIds 正確填入；getPatientById 同步修正
- **autoFieldService**：從事件自動計算所有 type:auto 欄位（hasChemo/hasRT/hasCCRT/hasTACE/hasRFA/daysToSurgery 等 30+個）；IndicatorsPage 開啟病人時自動帶入，手動輸入值優先
- auto 欄位顯示改版：有值時綠底顯示計算結果 + 來源說明
### V5.62 (2026-03-31) ⭐ 測試資料全面改版
- **測試資料 13癌×10人=130人**，各癌別有正確對應科別醫師：
  - BC：一般外科（林建華/李岳聰）+ 血液腫瘤科
  - LC：胸腔內科（張竣期）+ 胸腔外科（李佳穎/黃榆涵）
  - CRC：一般外科（林建華/歐金俊）+ 血液腫瘤科
  - OC：口腔外科（張建明/王威群）
  - HCC：肝膽腸胃科（黃懷毅/陳忠宏/葉永祥）
  - GC：肝膽腸胃科 + 一般外科（吳鴻昇）
  - EC：胸腔外科（李佳穎/黃榆涵）
  - PC/BLC：泌尿科（楊哲學/吳其翔）
  - OVC/EMC/CC：婦產科（林坤沂/陳彥甫）
  - PAC：肝膽腸胃科 + 一般外科（歐金俊）
  - 每人有正確 TNM、期別、cancerSpecificData、組織型態、surgicalMargin
- **品質監測篩選面板修正**：DateFilterButton 移出 sticky PageHeader，消除截斷問題
- **指標資料圓圈改癌別 badge**：不再顯示姓名首字，改顯示癌別代碼（含色彩對應）
- **篩選醫師移除 Dr. 前綴**：節省空間，簡潔呈現
### V5.61 (2026-03-31)
- **癌別專屬欄位全面中文化**：所有 coreMetrics key 都有中文標籤，涵蓋 hasEGFR、egfrResult、hasPET、petDate、hasTargetTherapy、doi、ene、pni 等 100+ 個欄位
- **病理分期加切緣狀態**：R0/R1/R2/Rx 選單 + 切緣距離 (mm)，顯示時合併呈現「R0 (2mm)」
- **品質監測「前往填寫」修正**：改導向指標資料頁（IndicatorsPage），可直接填寫缺漏指標
- **資料一致性確認**：ER/PR/HER2/Ki67 等乳癌欄位統一存於 cancerSpecificData，無重複欄位
### V5.60 (2026-03-31) ⭐ TNM 修正 + 組織型態 + 欄位中文化
- **LC 肺癌 M 值修正（AJCC 9th）**：M1c → M1c1（單器官多處轉移）/ M1c2（多器官轉移）
- **組織型態選擇**（13 個癌別各有內建選項，點按式）：
  - BC：IDC/ILC/DCIS/黏液性癌等 8 項
  - LC：腺癌/鱗癌/SCLC/LCNEC 等 8 項
  - CRC/GC/EC/OC/HCC/PC/BLC/OVC/EMC/CC/PAC 各有對應選項
  - 整合到 DiagnosisTab 和 IndicatorsPage，兩處同步讀寫
- **全癌別專屬欄位中文化**：所有 13 癌別的 cancerSpecificData 欄位都有完整中文標籤
### V5.59 (2026-03-31) ⭐ TNM 完整校正 + 資料整合
- **TNM 全癌別校正（AJCC 9th/8th）**：
  - LC 肺癌：新增 T1d/T2c/N2a/N2b，IIID 期（AJCC 9th 新增）
  - CRC 大腸直腸癌：N 值精確化（N1a/b/c、N2a/b）
  - GC 胃癌：T1a/T1b、N3a/N3b 分級，期別表完整修正
  - EC 食道癌：T1a/T1b 分開，I/II/III/IVA/IVB 對應
  - OC 口腔癌：N2a/N2b/N2c 三型，IVA/IVB/IVC 明確
  - OVC 卵巢癌：FIGO 2023（T1c1/T1c2/T1c3、N1a/N1b、M1a/M1b）
  - EMC 子宮內膜癌：FIGO 2023 全新分期（含 IC、IIIC1/2）
  - CC 子宮頸癌：FIGO 2018（T1a1/T1a2、T1b1~T1b3、IIIC1）
  - BLC 膀胱癌：T2a/T2b/T3a/T3b/T4a/T4b 分開
  - PC 攝護腺癌：T 值修正（移除 T1/T2 整合值），M1a/b/c
  - PAC 胰臟癌：T1a/T1b/T1c 三亞型
- **SmartDateInput 全面套用**：EventFormModal/ChemoScheduleModal/PDCAPage 補齊
- **IndicatorsPage 分期整合**：臨床/病理分期改用 TNMInput，與診斷Tab共用同一資料庫
### V5.58 (2026-03-31)
- TNM 選值改為純點按，期別唯讀（只由 TNM 自動填入，有清除鍵）
- 各癌別 M 值依 AJCC 9th 校正：
  - BC/GC/EC/OC/HCC/PAC：只有 M0、M1
  - LC/CRC：M0、M1a、M1b、M1c
  - PC：M0、M1a（非區域淋巴結）、M1b（骨）、M1c（其他）
  - BLC：M0、M1a（非區域淋巴結）、M1b（遠端）
  - OVC：M0、M1（FIGO 系統）
### V5.57 (2026-03-31) ⭐ TNM 自動分期
- **TNMInput 元件**：T/N/M 各值以 AJCC 9th 癌別專屬 chip 按鈕選擇（不可手輸）
  - 每個癌別有各自有效的 T/N/M 值（BC/LC/CRC/GC/EC/OC/HCC/PC/BLC/OVC/PAC）
  - 三項選完後自動判斷期別，顯示 Stage + 說明
  - 「套用」按鈕一鍵填入臨床/病理期別
  - 期別仍可透過 StageSelector 手動調整
- **tnmStagingService.ts**：11 個癌別的 AJCC 9th TNM → 期別對應表
### V5.56 (2026-03-31)
- **SmartDateInput**：新元件，支援 YYYYMMDD 連續輸入自動格式化為 YYYY-MM-DD，不跳格
  - 套用：病理確診日、初診日、首次治療日、生日、事件日期、排程日期等全部日期欄位
- **StageSelector**：新元件，AJCC 9th 期別點按式選擇（0/I/IA/IIA/IIB/.../IV/IVA/IVB/IVC）
  - 自動根據癌別分期系統切換：一般癌症=AJCC、肝癌=BCLC、婦癌=FIGO
  - 臨床期別/病理期別改用點按選擇，不再接受自由輸入
### V5.55 (2026-03-31) ⭐ 病人管理篩選介面全面改版
- 三行式篩選卡片（同截圖風格）：
  - Row 1 收案區間：全部/本週/本月/本季/本年/指定年（popup）/指定區間（popup）
  - Row 2 篩選：狀態 select / 癌別 select / 醫師 select / 搜尋輸入
  - Row 3 排序：排序欄位 select + 方向按鈕（最多2組），重置按鈕
- 新增醫師篩選條件
- 統計列移至快速篩選按鈕上方
### V5.54 (2026-03-31)
- 修正日期篩選不生效：dateRange 改為 useMemo，sortedPatients 依賴正確追蹤（病人管理/品質監測/品質改善）
- DateFilterButton 按鈕大小與狀態/癌別按鈕統一（移除多餘 text-sm）
- 品質監測/品質改善確認 DateFilterButton 正常運作
### V5.53 (2026-03-31)
- Sidebar：移除收展功能，固定寬度 w-56；Logo 放大（48px 圓角矩形）
- Sidebar 標題區改三行：系統名稱（大字）、版本 badge + 副標題（獨立一行）
- DateFilterPanel：修正 z-index（backdrop z-40、面板 z-50），確保面板正常彈出
### V5.52 (2026-03-31)
- Sidebar 版本號移到軟體名稱下方（副標題同行顯示）
- 抽出可重用 DateFilterPanel 元件（快捷/指定年/自訂區間）
- 病人管理改用 DateFilterPanel（功能不變，架構統一）
- 品質監測：日期篩選改為快捷按鈕面板（取代裸 date input），預設本年
- 品質改善：新增開始日期篩選快捷按鈕
### V5.51 (2026-03-31)
- Sidebar 顏色調淡（#1E2D3D → #2E4057 霧藍）
- 病人基本資料加入出生日期欄位，旁邊即時顯示年齡
- 化療連續排程 Modal 修正圓角（改 overflow:hidden + 內層滾動），同步套用設計系統
### V5.50 (2026-03-31) ⭐ 工作會議/主檔維護/系統工具風格統一
- 三頁主容器改 var(--bg)，.card 改設計系統白卡
- Tab bar 改膠囊型 chip（同病人管理）
- WorkItemCard：勾選框改綠色設計系統、type badge 改灰底、選單改設計系統
- 日曆格子：今日改鋼藍邊框+淡藍底
- MasterDataPage：七個 Tab 改膠囊型、列表文字/停用標籤/自動 badge 全用 CSS 變數
- ToolsPage：備份/匯出/匯入/資料庫管理卡片套設計系統、重置確認 Modal 更新
- 打包 zip 資料夾名稱改為 Patient-follow-V{version}（不再是 dist）
### V5.49 (2026-03-31) ⭐ 指標資料頁風格統一
- 待辦分類改清單列式卡片（統一外框），分類圖示改左側邊框語意色
- 病人列改首字縮寫 badge（同病人管理）
- PatientDetailPanel：Header/Tab 改膠囊型、Section helper 灰底標題
- 個案分類/完治狀態按鈕改設計系統色（統一鋼藍）
- 所有輸入框改 .input class，核心測量欄位 radio/select 同步更新
### V5.48 (2026-03-31) ⭐ 品質改善頁風格統一
- 統計格改單卡橫向分欄（CSS 變數）
- 篩選改膠囊型 chip（同病人管理）
- 專案列表改清單列式（同病人管理），展開後 PDCA 四格改左側邊框
- 進度環顏色更新（達成=綠，進行=藍）
- 所有按鈕、Modal 套用設計系統
### V5.47 (2026-03-31) ⭐ 品質監測頁風格統一
- 統計格、癌別列、指標列全面套用設計系統（CSS 變數）
- 癌別列改用代碼 badge（達成=綠、未達=紅），含右側進度條
- 展開後指標列改為白底+左側邊框（非彩色底）
- 缺漏清單：頭像改首字縮寫、膠囊型分組切換按鈕
- 趨勢預警：統計格改白底、預警卡改左側邊框樣式
### V5.46 (2026-03-31)
- **病人管理：收案日期篩選**（本週/本月/本季/本年/指定年/自訂區間）
- **工作中心：待辦預設收合**，版面更清爽
### V5.45 (2026-03-31) ⭐ 病人管理列優化
- **癌別代碼圓形 badge**（BC/CC/CRC... 各有獨立顏色）取代姓氏首字
- **多重排序**（最多6條件疊加）：病歷號/姓名/診斷日期/癌別/期別/主治醫師
- 每列顯示更多資訊：期別、主治醫師、初診日期
### V5.44 (2026-03-30)
- 病人詳情總覽 Tab 移除重複的「治療紀錄」「查看指標」按鈕
- 癌別專屬欄位顯示中文標籤（hasSentinelBiopsy → 哨兵淋巴結切片 等）
- 治療 Tab 篩選後新增事件自動預設該類型
### V5.43 (2026-03-30)
- 病人詳情總覽 Tab 移除重複的資訊卡和事件統計格
- 確診日移入 Header subtitle
- Header 副標題格式：`BC001 · 乳癌 · IIA期 · 確診 2025/01`
### V5.42 (2026-03-30)
- **移除「待辦事項」頁面**，功能已整合在工作中心
- 側邊欄與手機底部導航同步精簡
- 工作中心快捷入口由4格改為3格
### V5.41 (2026-03-30) ⭐ 病人管理頁優化
- **病人列表改為清單列（List Row）樣式**，大幅減少視覺疲勞
- 快速篩選統一藍色系，有數量才顯示
- 病人頭像改用首字縮寫，簡潔一致
- 修正 EventFormModal 圓角問題（`overflow-hidden` 結構修正）
### V5.40 (2026-03-30) ⭐ 北歐設計系統
- **主色改為鋼藍 #5B8FB9**（原為橘色）
- 頁面背景 `#F7F9FB`、卡片圓角 16px、陰影極淡
- 側邊欄改為深藍灰 `#1E2D3D`，active 項目用藍色半透明
- 病人詳情 Tab Bar 改為膠囊型（pill）
- 按鈕、輸入框、badge 全面對齊設計規範
### V5.39 (2026-03-30)
- 北歐設計色彩收斂：工作中心、事件 badge 統一灰色系
- 逾期/待辦警示改為左側細邊框樣式（非彩色底）
- 品質監測 Tab Bar 移出 PageHeader，底線顯示正常
### V5.38 (2026-03-30)
- **全站 9 頁套用統一 PageHeader 元件**
- 每頁一致的 sticky 標題列、圖示框、副標題區
### V5.37 (2026-03-30)
- **診斷 Tab 支援就地編輯（Inline Edit）**
- 覆蓋基本資料、診斷日期、cTNM/pTNM 分期、淋巴結數
- 修正 PatientFormModal 圓角問題
### V5.36 (2026-03-30)
- 工作中心移除多餘橘色標頭
- 病人詳情總覽 Tab 使用真實指標資料
- 加入快速操作按鈕及逾期/待辦警示區塊
### V5.35 (2026-03-30)
- **病人詳情指標 Tab 整合國健署指標評估**
- 即時顯示病人的指標達成狀態
- 分組顯示：過程面 vs 結果面
- 顯示達成率進度條
### V5.34 (2026-03-30)
- 工作中心整合版（儀表板 + 智慧提醒）
- 三層級待辦：緊急 / 本週 / 需關注
### V5.30 (2026-03-30)
- **導航結構整合**
- 側邊欄分組：日常工作、品質管理、資料管理、系統設定
- 手機版「更多」選單重新設計
- 手機底部導航加入「品質」快捷入口
### V5.29 (2026-03-30) ⭐ SNQ進階功能
- **品質改善專案 (PDCA)**
  - PDCA四階段追蹤
  - 連結指標、記錄改善成效
  - 專案進度視覺化
- **成果摘要頁**
  - 一頁式年度成果展示
  - 達成率圓環圖、可匯出報告
### V5.28 (2026-03-30) ⭐ 評鑑準備
- **指標趨勢預警功能**
- 📈 趨勢圖表：月度達成率走勢
- ⚠️ 指標預警：自動檢測未達標指標
- 🔴 三級預警：嚴重/警告/注意
- 💡 改善建議：針對不同嚴重度
- 📊 統計卡片：達成率、達標數、預警數
### V5.27 (2026-03-30) ⭐ 評鑑準備
- **評鑑報表匯出功能**
- 📊 指標達成率報表（CSV）
- 📋 指標明細報表（含病人清單）
- ⚠️ 缺失欄位報告
- 👥 病人清單匯出
- 📈 評鑑摘要報告
### V5.26 (2026-03-30)
- **快速篩選按鈕**
- 一鍵篩選常用條件：
  - 📞 待訪視（超過60天未訪視）
  - 🔴 逾期（有逾期未處理事件）
  - ⚠️ 失聯（12個月無追蹤紀錄）
  - 🟣 術後30天（25-35天）
  - 🟤 術後90天（85-95天）
  - 💉 治療中（化療/放療進行中）
- 即時顯示各類別病人數量
- 支援與其他篩選條件組合
- 「清除篩選」一鍵重置
### V5.25 (2026-03-30)
- **批次操作功能**
- 支援多選病人同時處理
- 4 種批次操作：
  - 🏥 批次標記訪視
  - 📞 批次電話追蹤
  - 📅 批次新增事件
  - 📋 批次更新狀態
- 支援關鍵字搜尋、狀態篩選、癌別篩選
- 全選/取消全選功能
### V5.24 (2026-03-30) ⭐ 重大更新
- **全新「智慧提醒」功能**
- 7 種提醒類型：
  - 🏥 術後追蹤（30/90/180/365天）
  - 💉 化療週期提醒
  - ☢️ 放療後追蹤
  - 📅 回診提醒/逾期警告
  - 📞 訪視提醒
  - ⚠️ 失聯警告
  - 🖼️ 影像追蹤
- 4 級優先順序：緊急(紅)、高(橙)、中(黃)、低(綠)
- 可自訂提醒參數，設定自動儲存
- 點擊提醒直接跳轉病人詳情
- 手機底部導航新增「提醒」入口
### V5.23 (2026-03-30) ⭐ 重大更新
- **全新「每日待辦中心」儀表板**
- 9 種智慧待辦分類：
  - 🔴 逾期未處理
  - 🟠 失聯待追蹤（12個月無事件）
  - 🔵 今日預約回診
  - 🟣 術後 30 天追蹤
  - 🟤 術後 90 天追蹤
  - 🟡 超過 60 天未訪視
  - 🟢 本週待完成
  - 🩷 進行中化療
  - 🩵 進行中放療
- 快速統計卡片：追蹤中、逾期、失聯、待訪視人數一目瞭然
- 可展開/收合的分類卡片，點擊病人直接跳轉
- 新增統計頁籤：癌別分布、追蹤狀態分析、術後追蹤提醒
### V5.22 (2026-03-30)
- 新增肝膽腸胃科醫師：黃懷毅、陳忠宏、葉永祥、顏聖烈
- 新增婦產科醫師：陳彥甫
- 血液腫瘤科已有：陳彥谷
### V5.21 (2026-03-29)
- 指標監測頁面整合國健署 60 項指標
- 新增指標版本切換：國健署115年(60項) / 舊版核心指標
- 整體統計支援兩種指標版本顯示
- 個案明細支援兩種指標版本評估
- 國健署指標顯示：資料來源(癌登/自填)、可自動計算標記
### V5.20 (2026-03-29)
- 建立國健署 60 項指標自動計算服務
- 完整實作 13 癌別指標評估邏輯
- 支援分子/分母條件判斷、缺失欄位提示
- 整合輔助函數：組織型態判斷、期別比對、日期計算
- 提供統一評估函數 evaluateIndicatorUnified()
- 提供彙總統計函數 aggregateIndicatorResults()
### V5.19 (2026-03-29)
- Patient 結構擴充：新增 30+ 通用欄位支援國健署指標
- 新增淋巴結手術欄位：哨兵淋巴結、骨盆腔淋巴結、縱膈腔淋巴站數、腋下淋巴結
- 新增介入治療欄位：TACE 日期、RFA 類型/日期
- 新增追蹤檢查欄位：追蹤影像次數、追蹤內視鏡日期、術前影像日期/類型
- 新增分子標記欄位：MMR/p53/POLE 檢測狀態
- 新增 13 個癌別專屬資料介面（BCSpecificData, LCSpecificData 等）
### V5.18 (2026-03-29)
- 建立國健署 115 年 60 項強制申報指標主檔
- 涵蓋 13 癌別：CC(5), BC(3), CRC(3), OC(6), HCC(5), LC(2), GC(4), EC(4), PC(5), BLC(3), OVC(3), EMC(8), PAC(9)
- 指標定義包含：分子/分母條件、資料來源、癌登欄位說明、證據等級、選取理由
- 標註可自動計算指標(49項)與需手動填報指標(11項)
- 提供輔助函數：getIndicatorsByCancer(), getIndicatorById(), getIndicatorSummary()
### V5.17 (2026-03-24)
- 指標管理頁面重新設計：改為待辦清單模式
- 五大待辦分類：缺個案分類、缺完治率、超過30天無紀錄、需訪視(>90天)、失聯超過1年
- 點展開顯示病人清單，點病人直接填寫指標
- 已歿病人自動排除待辦清單
### V5.16 (2026-03-24)
- 修正預設測試病人癌別對應錯誤（BC/CC順序問題）
### V5.15 (2026-03-24)
- 全頁面篩選狀態保留：病人管理、指標管理、指標監測
- 切換頁籤時保留搜尋關鍵字、癌別篩選、狀態篩選等設定
### V5.14 (2026-03-24)
- 指標監測頁面篩選狀態保留（切換頁籤不重置）
- 空白範本更新：包含全部13癌別專屬欄位
- 預設測試病人加入個案分類(Class 0-3)
- 52位測試病人：Class 1(32人), Class 2(15人), Class 3(4人), Class 0(1人)
### V5.13 (2026-03-23)
- 預設測試資料更新：52位病人+71筆事件
- 每癌別4人，涵蓋各種指標情境
- 包含達標、未達標、缺資料、死亡等案例
- 可直接DEMO指標監測頁面各項計算
### V5.12 (2026-03-23)
- 完整品質指標整合（核心+品質合併為統一指標系統）
- 乳癌新增：乳房X光攝影、術前組織學確診（共5項）
- 肺癌新增：組織病理診斷、NSCLC-NOS、IIIA期肺葉切除（共5項）
- 肝癌新增：BCLC分期註記、BCLC 0/A接受TACE、AFP追蹤（共8項）
- 攝護腺癌新增：T3期放療合併荷爾蒙（共6項）
- 指標監測頁面：支援所有新增品質指標計算
### V5.11 (2026-03-23)
- 指標監測：加入大腸直腸癌品質指標（CRC-1~6）
  - Malignant Polyp高風險12週內手術
  - 全大腸檢查比率
  - 結腸癌III期術後6週內化療
  - 直腸癌CCRT後18週內手術
- 指標監測：加入胃癌品質指標（GC-1~3）
  - R0切除比率、術後30天死亡率、II-III期術後輔助化療
### V5.10 (2026-03-23)
- 新增「指標監測」頁面
- 整體統計：依癌別顯示指標達成率、分子/分母、閾值
- 個案明細：選擇病人查看各指標符合狀態
- 時間區間篩選（依診斷日期）
- 支援負向指標判斷（如死亡率）
- 自動計算部分指標（如術後天數、是否有治療）
### V5.09 (2026-03-23)
- 建立完整核心測量指標定義（13癌共55項指標）
- 建立品質指標定義（團隊層級共20項指標）
- 指標包含：名稱、分子/分母定義、閾值、相關欄位、負向指標標記
- 新增 getCoreIndicators(), getQualityIndicatorsByTeam() 函數
### V5.08 (2026-03-22)
- 移除所有 emoji 符號
- 病人列表改為 grid 佈局，一行顯示 4-5 張卡片
- 病人卡片改為垂直排列：頭像 → 姓名 → 病歷號 → 狀態+癌別
### V5.07 (2026-03-22)
- 病人資料頁新增「指標」按鈕（在新增事件旁邊）
- 病人頭像男藍女粉配色恢復
- 病人小卡縮小，一頁顯示更多
- 移除所有 emoji 符號
- Modal 圓角對稱修正
### V5.06 (2026-03-22)
- **病人編輯 → 填寫指標**：直接跳至指標頁面
- **指標欄位優化**：
  - radio 類型：是/否用按鈕選擇
  - auto 類型：灰色不可填，從事件自動判斷
  - 資料不足時顯示「資料不足」
- **自動判斷項目**：
  - 治療狀態：化療/放療/CCRT/標靶/免疫/荷爾蒙/TACE
  - 術前/術後治療判斷
  - 天數計算：就醫到手術、手術到化療、放療總天數
  - 乳癌分子亞型（根據 ER/PR/HER2/Ki67）
### V5.05 (2026-03-22)
- **備份功能優化**：支援選擇儲存位置（桌面瀏覽器）
- **Google Calendar 匯入**：工作及會議頁面支援匯入 CSV
- **乳癌欄位優化**：
  - ER/PR 直接輸入百分比（0%=陰性）
  - 分子亞型由 ER/PR/HER2/Ki67 自動判斷
- 測試病人增至 39 人（每癌別 3 人）
- 主檔維護新增「工具」：一鍵刪除測試病人
- auto 欄位類型：灰色不可填，從事件自動判斷
### V5.0 (2026-03-21) 資料結構大改版
- **配合國健署 13 癌診療核心測量指標重新設計**
- **13 癌癌別主檔**
  - 子宮頸癌、乳癌、大腸直腸癌、口腔癌、肝癌、肺癌
  - 胃癌、食道癌、攝護腺癌、膀胱癌、卵巢癌、子宮內膜癌、胰臟癌
- **病人資料大幅擴充**
  - 關鍵日期：最初診斷、組織確診、首次治療、死亡日期
  - 臨床/病理分期：完整 TNM、期別、分期系統（AJCC8/FIGO/BCLC）
  - 手術詳細：手術代碼、切緣狀態/距離、淋巴結檢查數/陽性數、殘餘腫瘤
  - 放療詳細：開始/結束日期、方式、總劑量、分次、近接治療
  - 化療詳細：開始日期、處方、同步化放療
  - 全身性治療：標靶/免疫/荷爾蒙治療日期與藥物
  - MDT：討論日期、編號、治療前是否經 MDT
- **癌別專屬欄位（cancerSpecificData）**
  - 各癌別獨立的指標欄位，取代舊版 coreMetrics
  - 標記關聯到哪些管考指標
- **事件記錄擴充**
  - 手術/放療/化療/影像/病理專屬欄位
  - 新增：近接治療、影像追蹤、抽血追蹤、病理報告、分期完成
- 癌別主檔新增：分期系統、是否為 13 癌標記
### V4.7 (2026-03-21)
- 匯入工具可下載空白範本（依癌別）
- 匯入時驗證格式（必填、日期格式、數值）
- 核心測量指標欄位（依癌別不同）
- 指標管理新增「核心指標」頁籤
### V4.6 (2026-03-20)
- 指標管理頁面
- 快速篩選：未填分類、未填完治、待完治、失聯、需訪視
### V4.5 (2026-03-20)
- 統計報表頁面
- 核心指標：失聯率、留治率、完治率、訪視率
### V4.4 (2026-03-20)
- Excel 資料匯入工具
- 新增事件類型：標靶、免疫、賀爾蒙、介入治療、醫療小組

---

## 支援的 13 癌別

| 代碼 | 癌別 | 分期系統 |
|------|------|----------|
| CC | 子宮頸癌 | FIGO |
| BC | 乳癌 | AJCC8 |
| CRC | 大腸直腸癌 | AJCC8 |
| OC | 口腔癌 | AJCC8 |
| HCC | 肝癌 | BCLC |
| LC | 肺癌 | AJCC8 |
| GC | 胃癌 | AJCC8 |
| EC | 食道癌 | AJCC8 |
| PC | 攝護腺癌 | AJCC8 |
| BLC | 膀胱癌 | AJCC8 |
| OVC | 卵巢癌 | FIGO |
| EMC | 子宮內膜癌 | FIGO |
| PAC | 胰臟癌 | AJCC8 |

---

## 使用方式

### 方式一：使用 npm（推薦）

```bash
# 1. 解壓縮 ZIP 檔到任意位置

# 2. 開啟終端機，進入專案資料夾
cd /path/to/case-manager-web

# 3. 安裝依賴（只需第一次或更新後執行）
npm install

# 4. 啟動開發伺服器
npm run dev

# 5. 瀏覽器會自動開啟，或手動訪問 http://localhost:5173
```

### 方式二：使用 Python 簡易伺服器

```bash
# 進入 dist 資料夾
cd dist && python3 -m http.server 8080

# 開啟瀏覽器訪問 http://localhost:8080
```

---

## 手機測試

1. 確保手機和電腦在同一個 WiFi 網路
2. 在電腦執行 `npm run dev -- --host`
3. 在手機瀏覽器輸入終端機顯示的區網網址

---

## 專案結構

```
case-manager-web/
├── dist/                    # 建置輸出（可直接部署）
├── src/
│   ├── components/         # 共用元件
│   ├── pages/              # 頁面
│   ├── services/           # 服務層
│   ├── types/              # 型別定義（V5.0 大改版）
│   └── version.ts          # 版本資訊
└── README.md
```

---

## 📄 授權

© 2026 SELA - 彰濱癌症中心
