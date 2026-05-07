# 🧳 Travel Agent — 智能旅行規劃

由 **Hermes Kanban Agent** 自動化生成的旅行計劃儲存庫。每個旅行計劃由 4 個 AI 角色協作完成：規劃員、研究員、預算審查員、發佈員。

## 📁 目錄結構

```
/trips/          — 最終行程（分鐘級詳細計劃）
/resources/      — 研究報告（機票、住宿、美食、景點、交通、預算）
/docs/           — 角色定義與工作流程文檔
```

## 🗺️ 現有行程

| 目的地 | 日期 | 預算 | 狀態 |
|--------|------|------|------|
| 🇹🇼 **台灣 8天** 台北→台中→台南→高雄 | 5月31日–6月7日 | HK$10,000 ✅ | ✅ 已發佈 |

## 🤖 工作流程

這個儲存庫由 4 個 AI 角色協作完成：

| 角色 | 職責 |
|------|------|
| **Planner** (規劃員) | 分解行程、組裝最終分鐘級計劃 |
| **Researcher** (研究員) | 研究航班、住宿、美食、景點、交通 |
| **Budget Reviewer** (預算審查員) | 可行性檢查、預算分配、費用監控 |
| **Publisher** (發佈員) | 將完整計劃推送到 GitHub |

詳細流程請參考 [/docs/workflow.md](/docs/workflow.md) 和 [/docs/profiles.md](/docs/profiles.md)。

## 🚀 如何開始

```
hermes kanban create "Plan [N]-day [city] trip, [dates], [budget]" --assignee planner
```

---

*由 Hermes Agent 自動生成 — 2026*
