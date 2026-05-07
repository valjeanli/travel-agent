# 🧳 Travel Agent — 智能旅行規劃

由 **Hermes Agent Kanban** 自動化生成的旅行計劃儲存庫。每個旅行計劃由 4 個 AI 角色協作完成。

## 📁 目錄結構

```
/trips/                — 最終行程（分鐘級詳細計劃）
/resources/            — 研究報告（機票、住宿、美食、景點、交通、預算）
/docs/profiles.md      — 每個角色的 SOUL.md 與職責說明
/docs/workflow.md      — 完整工作流程管線圖
```

## 🗺️ 現有行程

| 目的地 | 日期 | 預算 | 狀態 |
|--------|------|------|------|
| 🇹🇼 **台灣 8天** 台北→台中→台南→高雄 | 5月31日–6月7日 | HK$10,000 ✅ | ✅ 已發佈 |

---

## 🤖 一鍵安裝 — 直接餵給你的 Hermes Agent

將以下整段 prompt 貼給你的 Hermes Agent，它會自動從這個 repo 讀取配置並建立好 4 個角色：

`````
Read the files from https://github.com/valjeanli/travel-agent and set up the 4 travel planning profiles exactly as defined in /docs/profiles.md:

1. Create 4 Hermes Agent profiles:
   - planner
   - researcher
   - budget-reviewer
   - publisher

2. For each profile, read the corresponding SOUL.md content from /docs/profiles.md and write it to the profile's SOUL.md file.

3. For each profile, create a config.yaml with:
   - model: deepseek-v4-flash
   - provider: opencode-go
   - base_url: https://opencode.ai/zen/go/v1
   - api_mode: chat_completions
   - reasoning_effort: medium

4. Ensure `hermes` is on PATH (symlink if needed at /usr/local/bin/hermes).

5. Verify with `hermes profile list` that all 4 profiles show the model configured.
`````

Agent 會自動完成：
1. 從 GitHub 讀取 `/docs/profiles.md` 取得每個角色的 SOUL.md
2. 執行 `hermes profile create <name>` 建立 4 個角色
3. 寫入 config.yaml（deepseek-v4-flash / opencode-go）
4. 寫入 SOUL.md
5. 設定 PATH 並驗證

---

## 🚀 開始新旅程

建立後，告訴你的 Agent：

```
Create a kanban task to plan an 8-day trip from Hong Kong to Taiwan,
departing 31 May, returning 7 Jun, with a budget of 10,000 HKD.
Assign it to the planner profile. All output in Traditional Chinese.
```

系統會自動執行完整管線：

| 步驟 | 角色 | 動作 |
|------|------|------|
| 1 | **Planner** | 分解行程為研究項目 |
| 2 | **Researcher (x5)** | 平行研究機票、住宿、美食、景點、交通 |
| 3 | **Budget Reviewer** | 檢查預算可行性、分配開支 |
| 4 | **Planner** | 組裝最終分鐘級行程 |
| 5 | **Publisher** | 推送到 GitHub |

---

## 🤖 角色一覽

| 角色 | 職責 |
|------|------|
| **Planner** (規劃員) | 分解行程、組裝最終分鐘級計劃 |
| **Researcher** (研究員) | 研究航班、住宿、美食、景點、交通 |
| **Budget Reviewer** (預算審查員) | 可行性檢查、預算分配、費用監控 |
| **Publisher** (發佈員) | 將完整計劃推送到 GitHub |

詳細角色定義與 SOUL.md 請見 [/docs/profiles.md](/docs/profiles.md)。
完整工作流程請見 [/docs/workflow.md](/docs/workflow.md)。

---

*由 Hermes Agent 自動生成 — 2026*
