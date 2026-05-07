# SELA-handoff.md — Patient follow V7.0.0

> **產生時機：** V7.0.0「對齊 SELA Starter Kit V1.7.0」里程碑完成。
> 這是這個專案首次對齊 Kit 規範，也是 Kit 第一次接到「**已經做了 6 個月的成熟專案**」的反饋（不是新專案從 V0.1.0 起手）。

---

## 〇、專案速覽

- **專案名稱：** Patient follow（彰濱癌症中心個管病患追蹤系統）
- **專案類型：** React PWA（純前端，IndexedDB 儲存，無後端）
- **技術棧：** React 18 + TypeScript + Vite + TailwindCSS + Zustand + Dexie.js v4 + pptxgenjs / docx
- **規模：** 約 80 個檔案、~30k 行 TS/TSX；13 張 IndexedDB 表
- **使用 Kit 版本：** V1.7.0（首次接觸 — 專案先存在，Kit 後到）
- **完成版本：** V7.0.0（對齊里程碑）
- **完成日期：** 2026-05-07
- **Kit 之前的成熟度：** 已從 V3.x 演進到 V6.9.4，累積 50+ 個釋出版本

---

## 一、用 Kit 的整體感受

> 情境特殊：這專案不是用 Kit 起手做的，而是事後對齊。所以「感受」更接近「跨專案經驗的回照鏡」，不是「冷啟動體驗」。

### 預期外的順利

- **章法手冊（CLAUDE-MD-章法.md）句句切中：** 我們的 CLAUDE.md 從 V3 寫到 V6.9.4 才慢慢成熟，章法手冊上每一條（坑編號累積、業務對映表、版本歷程留近期、一句話總結放最末）都是我們自己摸索出來的同樣結論。讀手冊時的感覺是「對對對就是這樣」。
- **坑 #4（Schema 演進）和坑 #7（打包前驗證）已經把我們吃過的最大兩個悶虧寫進 Kit 了** — 我準備寫 handoff 時 grep 才發現，這兩條反而省下我打字的力氣。Kit 的「跨專案累積」機制有效。
- **logo CLAUDE.md §1 對照表的「React/Vue/Vite → svg/sela.svg + favicon/ 套組 → public/」直接給答案，不用想：** 從上傳到放對位置 5 分鐘搞定，包含 `index.html` 的標準片段（§4.1）也直接 copy。
- **雙版本交付規則（SPEC §12）就是寫我們：** 看到 Kit 把我們 V6.9.3 當血淋淋的範例引用，反而確認了這條規則的價值。

### 預期外的卡住

- **「改現有專案」流程 Kit 給的指引比較少：** Kit `CLAUDE.md` §2.2 對照表是「想做什麼類型的 app → 看哪個 reference」這種「開新專案」邏輯。**我這次是「我有個 30k 行的 React PWA，要對齊 Kit」** — 沒有 SOP。最後我自己列了一張清單分成 🔴 必做 / 🟡 建議 / 🟢 順便 / ❌ 不做，但 Kit 沒教我這套思維。下個「現有專案對齊 Kit」的 Claude 也會卡同一個地方。
- **配色衝突沒章節寫怎麼決：** Kit `CLAUDE.md` §6 說醫療系統用霧藍 `#5A7A8B`，但本專案 6 個月來一直用鋼藍 `#5B8FB9`（個管師習慣了）。`colors.md` §3 表格「醫療系統 → 北歐霧藍」沒提到「**已通過驗證的近似色該不該為對齊而改**」。**SELA 在本案立的原則是：除了首次生成外，對齊換色要先問**。這個決定點應該被 Kit 收錄成規則。
- **`templates/CLAUDE-template.md` 不適用既有 30k 行專案：** 是給 V0.1.0 起手用的，舊專案直接套會把累積的「踩過的坑 #1~#15」洗掉。我跳過了這份 template，繼續用既有 CLAUDE.md 結構。Kit 該明寫「既有專案不要套此 template」。

### 對 Kit 的整體評價

- ✓ 「跨專案坑庫」價值最高 — 看到 #4、#7、#10 已收錄就知道前面 11 個專案不是白踩的
- ✓ Logo 套組與品牌色鐵律乾淨明確，5 分鐘整合完
- ✓ 章法手冊與我自己摸索出的章法 95% 重疊，那 5% 差異是補強
- ✗ 「**現有專案對齊 Kit**」這條路徑沒 SOP — 比「開新專案」更該寫，因為大家手上都有舊專案

---

## 二、發現的「跨專案通用坑」（建議進 Kit）

### 提案前的兩個檢查（V1.7.0 規範）

#### 檢查 1：先 grep Kit

```bash
grep -in "Dexie\|schema 截斷" SELA-Starter-Kit/conventions/cross-project-pitfalls.md
# → 已是坑 #4，不重複提

grep -in "tsc.*build\|TS error" SELA-Starter-Kit/conventions/cross-project-pitfalls.md
# → 已是坑 #7，不重複提

grep -in "silent ignore\|未使用參數\|TS6133\|沒接住" SELA-Starter-Kit/conventions/cross-project-pitfalls.md
# → 結果為空，可提案
```

### 強烈建議加坑

#### 1. 函數加參數但內部沒接，silent ignore

- **症狀**：API 加了新參數，呼叫端正確傳值，但功能完全沒生效；UI 看起來沒效果但 console 沒任何錯誤
- **原因**：簽名加了 `themeId?: string`，但函數內部仍呼叫舊邏輯（如 `getThemeFromLocalStorage()`），傳入的參數形同被忽略
- **做法**：API 加參數要走完三步——「(1) 簽名接住 → (2) 函數內部用上 → (3) 呼叫端到底層的端到端驗證」。`tsc --noEmit` 的 TS6133（declared but never read）是早期信號，**寧可看到報錯也不要 silent ignore**。lint 規則啟用 `@typescript-eslint/no-unused-vars` warn 等級。
- **影響範圍**：所有 TypeScript 專案；React / Vue 重構期特別容易。也適用 Python（雖然 Python 沒同等的 unused param 檢查，但 ruff `ARG002` 可開）
- **證據**：本專案 CLAUDE.md 坑 #14（V6.9.4 修正紀錄）；HTML 投影片 V6.9.2 加 themeId 起，到 V6.9.4 才修正——**中間兩版主題切換完全無作用、沒人發現**
- **檢查 1 結果**：grep Kit「silent ignore / 未使用參數 / TS6133 / 沒接住」→ 全部無命中，**新坑**

### 可加但等更多證據確認

#### 2. lucide-react（或同類 tree-shaken icon library）import 缺項導致整頁白

- **症狀**：點某按鈕進入頁面 → 白屏；console: `Copy is not defined` 或 `X is not defined`
- **原因**：用某個 icon 卻忘了加進 `import { ... } from 'lucide-react'`；產品環境 tree-shake 後該 icon 沒進 bundle
- **做法**：使用 icon 前先 `grep -n "from 'lucide-react'"` 確認 import 區塊；多個 import block 要合併成一個（避免改了一個漏改另一個）
- **證據**：本專案 CLAUDE.md 坑 #12（V6.9.2 修正）
- **是否進 Kit**：偏 lucide 特定。建議**等第二個 React 專案踩到再考慮**，現在 N=1。

---

## 三、發現的「跨專案設計模式」

### 1. 「給同時拿到 Kit 的 Claude」開頭指引

- **本案發生情境**：V7.0.0 我在現有專案的 `CLAUDE.md` 最前面加了一個 `⚠️ 給同時拿到 SELA Starter Kit 的 Claude` 區塊，明寫「這是改現有專案、不是開新專案、以本檔為主 Kit 為輔、配色不要動、版號照 Kit §7」。原因是預期下次 SELA 開新對話時可能同時上傳 Kit + 本專案，新 Claude 會困惑該套哪個 SOP。
- **可推廣的原則**：**任何成熟專案的 CLAUDE.md 都該有「Kit 衝突仲裁」開頭區塊**——明寫「以本專案 CLAUDE.md 為主，Kit 為輔」、列出本專案哪些地方刻意不對齊 Kit 預設、以及為什麼。
- **代價 / 取捨**：每個專案 CLAUDE.md 多 5~10 行；但避免 Kit Claude 把成熟專案當新專案套模板。
- **建議寫入**：`CLAUDE-MD-章法.md` 加「章法十一：Kit 衝突仲裁開頭區塊（成熟專案專用）」，或加進 `templates/CLAUDE-template.md` 的「升版到 V1.0.0 後加入這段」備註

### 2. 「現有專案對齊 Kit」的清單分級法

- **本案發生情境**：對齊 V1.7.0 時我把所有要做的事分四級——🔴 必做（鐵律違反，如缺 .gitignore、zip 命名格式錯）/ 🟡 建議（章法不對齊但可商量，如 README 結構）/ 🟢 順便（既然在改）/ ❌ 不做（明確不對齊但有理由保留現狀）。
- **可推廣的原則**：**對齊外部規範時用「鐵律 / 建議 / 順便 / 不做」四級判斷**，不是「全做」也不是「逐項問」。明寫「不做」的理由比明寫「做了什麼」更重要——避免下次 Claude 又想對齊回去。
- **代價 / 取捨**：規劃階段多 10 分鐘；但避免「為對齊而對齊」毀壞既有設計。
- **建議寫入**：新增 `conventions/migration-to-kit.md`（暫名），給「現有專案對齊 Kit」用。或併進 `start-project-decisions.md` 末段。

---

## 四、Kit 該瘦身或調整的地方

### Kit 規範修改建議

#### 1. `CLAUDE.md` §2 加「2.6 改現有大型專案的對齊路徑」

- **現狀**：§2 只寫「開新專案 SOP」；§2.2 對照表全是「新建專案的技術棧 → reference」邏輯。
- **建議改成**：加一節「2.6 改現有大型專案」，寫明——
  - 不要套 `templates/CLAUDE-template.md`（會洗掉累積的坑）
  - 列出「鐵律 / 建議 / 順便 / 不做」四級檢查清單（範例見上節設計模式 #2）
  - 在專案 CLAUDE.md 開頭加 Kit 衝突仲裁區塊（範例見上節設計模式 #1）
- **理由**：現實上多數人會「先做專案，後接觸 Kit」。沒這節 Kit Claude 會誤套「開新專案 SOP」破壞既有結構。

#### 2. `colors.md` §3 補「對齊換色要問用戶」明文規則

- **現狀**：表格「醫療系統 → 北歐霧藍 #5A7A8B」沒講「我已經用 #5B8FB9 一年了該不該改」。
- **建議改成**：在 §3 表格下加一條明文規則：
  > **首次生成專案時用 Kit 預設色。但既有專案對齊 Kit 時，主色與派生色的更動必須先問用戶——即使只是 5% 飽和度差距、即使 Kit 預設與現狀理論上更標準。已被使用者驗證的色票就是該專案的事實標準，不為「對齊」而改。**
- **理由**：本案 #5B8FB9 vs #5A7A8B 視覺幾乎不可分，但 KAO/YANG 已用 6 個月，動了徒增風險、無實質好處。SELA 本案立的明文原則：「**配色除了首次生成外，對齊換色要先問**」。

#### 3. `deployment/SPEC.md` §12.4 部署檔命名約定要鬆綁

- **現狀**：Kit 預設「`-source.zip` 給 Git Pusher 用（觸發 Actions build），`-dist.zip` 救命用」。
- **本案實際**：Sela 在本專案要求「**部署檔不要加 dist 後綴**」（`Patient follow V7.0.0.zip` 直接是部署版），`-source` 留給救命備份。理由：本專案直接把 dist 推 GitHub Pages，不依賴 GitHub Actions auto-build。
- **建議改成**：§12.4 加一段「**命名後綴可調整**」——
  > Kit 預設 `-source` / `-dist` 後綴反映「Git Pusher 觸發 Actions build」的工作流程。但若專案直接用 build 後產物部署（不靠 Actions），可調整為「無後綴 = 部署版、`-source` = 備份版」。在專案 CLAUDE.md 明寫該專案實際採用哪種約定即可。
- **理由**：Kit 不該強制單一命名約定，要尊重各專案實際的 build/deploy 流程。本案是第一個發現此差異的專案。

### Kit 結構性建議

> 不是改某條規範，是調整 Kit 整體運作方式。

- **加入 React PWA reference**（Kit V1.7.0 README 已標暫緩）：本專案可以當 reference 候選——React 18 + Vite + Dexie + Zustand 是目前最有實戰驗證的純前端 PWA 組合。但要注意 reference 是「**架構模式**」，本案的具體業務（13 癌、MDT、國健署 60 指標）必須在反入 Kit 時剝離。
- **「現有專案對齊 Kit」的 reference**：本案的 V6.9.4 → V7.0.0 過程（含此 handoff）可以當第一個範例。

---

## 五、留在這個專案、**不要回流 Kit** 的東西

> 這節避免 Kit Claude 把彰濱業務邏輯誤收進 Kit。

- ❌ **CORE_14_CANCERS**（13~14 癌別主檔）— 純醫療業務
- ❌ **MDT_GROUP_CONFIG**（MDT 群組與排程）— 彰濱秀傳特定
- ❌ **CANCER_CODE_TO_MANAGER**（KAO / YANG / 林伯儒分工）— 此醫院特定
- ❌ **國健署 115 年 60 項指標定義** — 台灣醫療業務
- ❌ **PPTX/HTML 投影片版面細節** — 業務模板，跟 Kit 無關
- ❌ **Nordic-minimalist 設計系統 DESIGN_SYSTEM.md** — 本專案視覺風格（Kit `colors.md` 已有「霧藍主題」，足夠）
- ❌ **業務帳號 14002 / 11750** — 真實人員代碼
- ⚠️ **「ensureDefaultAdmin per-account 檢查」原則** — 這條本案的坑 #7 偏 Dexie + 帳號 seed 特定，**不要進 Kit**；但「跨版本資料 seed 要 per-record 檢查不要 count > 0 跳過」的原則也許可以。等第二個專案踩到再說。

---

## 六、Kit Claude 的建議行動清單

### 建議升 Kit 版本

**V1.8.0**（b+1，新增章節）

理由：本案發現「現有專案對齊 Kit」是 Kit 沒覆蓋的路徑，要新增 §2.6 + 設計模式兩條，屬於結構性新增不只是補坑；不是 bug fix（不算 c+1），也沒大改既有規範（不算 a+1）。

### 必做

- [ ] **新增坑 #39「函數加參數但內部沒接 silent ignore」**（grep 確認無重複）— 收進 G 章工具鏈與依賴版本，或新開「TypeScript 通病」子章
- [ ] **`CLAUDE.md` §2 新增 §2.6「改現有大型專案的對齊路徑」**：列「鐵律/建議/順便/不做」四級檢查清單，引用本案為範例
- [ ] **`CLAUDE-MD-章法.md` 加章法十一「Kit 衝突仲裁開頭區塊」**（成熟專案專用）
- [ ] **`colors.md` §3 補「已通過驗證的近似色不為對齊而改」注解**
- [ ] **更新 Kit 自身 README 的版本紀錄**：V1.8.0 標明「第一份從成熟專案（非新專案）來的 handoff，學到既有專案路徑」

### 暫緩

- [ ] **lucide-react import 漏項坑** — 等第二個 React 專案踩到再決定（現在 N=1）
- [ ] **React PWA reference** — Kit 已標暫緩，本案可當候選但別急；等下個 React PWA 專案有對照組再做
- [ ] **「per-record 檢查不要 count > 0 跳過」原則** — 等第二個 seed 類專案驗證

### 不做

- [ ] **把 13 癌別 / MDT 群組 / 國健署指標收進 Kit** — 業務邏輯，不通用
- [ ] **改 Kit 預設醫療系統色為 #5B8FB9** — 本案是個案近似色，Kit 應持續用 #5A7A8B 標準
- [ ] **把 Nordic-minimalist DESIGN_SYSTEM.md 收進 Kit** — `colors.md` 已涵蓋足夠

---

## 七、給 Kit Claude 的最後備註

這份 handoff 有個特殊性：**這專案不是用 Kit 做的，是事後對齊**。Kit V1.6.0 的 handoff 機制原本設計是「V1.0.0 完成時產出」，但本案早已過 V6.x，第一次 handoff 反而是在「V7.0.0 對齊里程碑」。這代表 handoff 機制其實也適用「**任何重大版本里程碑**」，不限於 V1.0.0。可考慮把 template 的「產生時機」加一條「**首次對齊 Kit 時，無論版本號為何**」。

另外，本案是 Kit 第一次直接被引用為案例（SPEC §12「從 Patient Follow-up V6.9.3 的慘痛教訓而來」）。被引用很榮幸，也代表 Kit 機制「以慘痛換規範」確實在跨專案累積經驗——下個踩同樣坑的專案可以省一輪。

下次本專案達到實質里程碑（例如 KAO/YANG 正式上線、或加入 LY 第 14 癌別後的穩定版）會再產出 handoff。

---

> Made by **SELA**, with **Claude** · V7.0.0
