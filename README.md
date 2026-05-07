# 🧳 Travel Agent — 智能旅行規劃

由 **Hermes Agent Kanban** 自動化生成的旅行計劃儲存庫。每個旅行計劃由 4 個 AI 角色協作完成：規劃員、研究員、預算審查員、發佈員。

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

---

## 🔧 安裝到 Hermes Agent

### 前提條件

- 已安裝 [Hermes Agent](https://hermes-agent.nousresearch.com)
- Kanban 功能已啟用

### 1. 建立 4 個角色

```bash
hermes profile create planner
hermes profile create researcher
hermes profile create budget-reviewer
hermes profile create publisher
```

### 2. 設定模型

每個角色預設使用 deepseek-v4-flash (opencode-go 提供商)：

```bash
# 為每個角色執行設定精靈
planner setup
researcher setup
budget-reviewer setup
publisher setup
```

或直接複製 config.yaml（參考 `/docs/profiles.md` 中的配置範例）。

### 3. 設定 SOUL.md（可選）

每個角色的 `SOUL.md` 定義其行為與個性：

| 角色 | SOUL.md 位置 |
|------|-------------|
| planner | `~/.hermes/profiles/planner/SOUL.md` |
| researcher | `~/.hermes/profiles/researcher/SOUL.md` |
| budget-reviewer | `~/.hermes/profiles/budget-reviewer/SOUL.md` |
| publisher | `~/.hermes/profiles/publisher/SOUL.md` |

各角色的完整 SOUL.md 內容請參考 `/docs/profiles.md`。

### 4. 確保 `hermes` 在 PATH 上

Kanban 調度器需要 `hermes` 命令在 PATH 上：

```bash
ln -s /path/to/hermes /usr/local/bin/hermes
```

### 5. 確認角色已就緒

```bash
hermes profile list
```

應顯示 4 個角色（planner、researcher、budget-reviewer、publisher）均已配置模型。

---

## 🚀 開始新旅程

```bash
hermes kanban create "Plan [N]-day [city] trip, [dates], [budget]" --assignee planner
```

### 實例

```bash
hermes kanban create "Plan 5-day Tokyo trip, Apr 2026, 150k JPY" --assignee planner
```

系統會自動：

1. **Planner** → 分解行程為研究項目
2. **Researcher** → 平行研究機票、住宿、美食、景點、交通
3. **Budget Reviewer** → 檢查預算可行性、分配開支
4. **Planner** → 組裝最終分鐘級行程
5. **Budget Reviewer** → 最終預算確認
6. **Publisher** → 推送到 GitHub

### 監控進度

```bash
hermes kanban list           # 查看所有任務
hermes kanban show <id>      # 查看任務詳情
hermes kanban stats          # 按狀態統計
```

---

## 🤖 工作流程

詳細流程請參考：

- [/docs/workflow.md](/docs/workflow.md) — 完整管線圖與相依鏈
- [/docs/profiles.md](/docs/profiles.md) — 4 個角色的 SOUL.md 與配置

| 角色 | 職責 |
|------|------|
| **Planner** (規劃員) | 分解行程、組裝最終分鐘級計劃 |
| **Researcher** (研究員) | 研究航班、住宿、美食、景點、交通 |
| **Budget Reviewer** (預算審查員) | 可行性檢查、預算分配、費用監控 |
| **Publisher** (發佈員) | 將完整計劃推送到 GitHub |

---

*由 Hermes Agent 自動生成 — 2026*
