<div align="center">
  <img src="src/assets/sela.svg" width="120" alt="SELA"/>
  <h1>彰濱癌症中心 - 個管病患追蹤系統</h1>
  <p>Patient follow System｜純前端 PWA，IndexedDB 儲存，無後端</p>
  <p><strong>V7.0.0</strong></p>
</div>

---

## 簡介

給彰濱秀傳癌症中心個管師（KAO 14002、YANG 11750）日常使用的個案追蹤系統。三大核心：

- **追蹤** 14 種癌別病人的治療進度與訪視週期
- **計算** 國健署 115 年強制申報 60 項品質指標（13 癌別）
- **管理** MDT 多專科會議：行事曆、會議室、PPTX/HTML 投影片產出

純前端、無後端、Dexie.js / IndexedDB 儲存，支援離線。

---

## 安裝

```bash
npm install
```

## 啟動

```bash
npm run dev      # 開發模式
npm run build    # 打包出 dist/
```

預設帳號：

| 帳號 | 密碼 | 角色 |
|------|------|------|
| `admin` | `admin1234` | 系統管理員（看全部癌別） |
| `14002` | `14002` | KAO（BC / LC / EC） |
| `11750` | `11750` | YANG（CRC / GC / LY / CC / OVC / EMC） |

## 使用

第一次進入點「重置預載資料」會帶入 140 位測試病人 + 3 場 MDT 測試會議；正式使用前用「資料庫 → 刪除所有測試病人」清掉。

詳細操作見 `使用說明書.md`（給個管師看，內容與 `USER_GUIDE.md` 相同）。

## 目錄結構

```
彰濱癌症中心/
├── src/
│   ├── App.tsx                         主入口
│   ├── version.ts                      版本號（升版必改）
│   ├── pages/                          各功能頁（PatientsPage / MDTPage / ToolsPage 等）
│   ├── components/                     共用元件
│   ├── services/                       業務邏輯（mdtOutputService、indicatorService 等）
│   ├── stores/                         Zustand 全域狀態
│   ├── db/                             Dexie schema 與測試資料
│   ├── types/                          型別定義（CORE_14_CANCERS、MDT_GROUP_CONFIG 等）
│   └── assets/sela.svg                 SELA 品牌標識
├── public/
│   ├── favicon/                        SELA logo 完整 favicon 套組
│   └── manifest.json                   PWA manifest
├── CLAUDE.md                           給接手 Claude 的工作上下文
├── DESIGN_SYSTEM.md                    Nordic-minimalist 設計系統
├── 使用說明書.md                       給個管師看的操作手冊（主版）
├── USER_GUIDE.md                       使用說明書的英文檔名複本（避中文檔名亂碼）
├── README.md                           本檔
├── README.OLD.md                       完整版本演進史（V3 → V6.9.4）
├── SELA-handoff.md                     給 SELA Starter Kit Claude 的反饋
└── .gitignore
```

## 技術棧

- React 18 + TypeScript + Vite
- TailwindCSS（utility-first）
- Zustand（全域狀態）
- Dexie.js v4（IndexedDB ORM）
- date-fns v4
- pptxgenjs / docx（MDT 投影片產出）
- xlsx（Excel 匯入匯出）

## 版本

**V7.0.0**（對齊 SELA Starter Kit V1.7.0 規範的里程碑版）

完整版本歷程見 [`README.OLD.md`](./README.OLD.md)（V3 → V6.9.4 的演進記錄）。
最新一版的新增內容直接看畫面右下角「版本資訊」或 `src/version.ts`。

---

> Made by **SELA**, with **Claude** · V7.0.0
