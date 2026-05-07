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

## 🔧 安裝 — 實際配置

以下為本專案實際使用的 Hermes Agent 配置，可直接複製使用。

### 前提條件

- 已安裝 [Hermes Agent](https://hermes-agent.nousresearch.com)
- Kanban 功能已啟用（`hermes kanban init`）
- `hermes` 命令在 PATH 上（如果不在，建立軟連結：`ln -s /opt/hermes/venv/bin/hermes /usr/local/bin/hermes`）

---

### 1. 建立 4 個角色

```bash
hermes profile create planner
hermes profile create researcher
hermes profile create budget-reviewer
hermes profile create publisher
```

角色目錄位置（依安裝路徑而定，以下是本機實際路徑）：

```
/opt/data/profiles/planner/
/opt/data/profiles/researcher/
/opt/data/profiles/budget-reviewer/
/opt/data/profiles/publisher/
```

每個角色建立後會產生包裝指令，可以直接使用：

```bash
planner chat
researcher chat
budget-reviewer chat
publisher chat
```

這些包裝指令實際執行 `hermes -p <name> "$@"`。

---

### 2. 設定 config.yaml

每個角色目錄下需要一個 `config.yaml`。以下是本專案實際使用的配置（所有 4 個角色共用相同模型）：

```yaml
model:
  default: deepseek-v4-flash
  provider: opencode-go
  base_url: https://opencode.ai/zen/go/v1
  api_mode: chat_completions
providers: {}
fallback_providers: []
credential_pool_strategies: {}
toolsets:
  - hermes-cli
agent:
  max_turns: 90
  gateway_timeout: 1800
  restart_drain_timeout: 180
  api_max_retries: 3
  service_tier: ''
  tool_use_enforcement: auto
  gateway_timeout_warning: 900
  gateway_notify_interval: 180
  gateway_auto_continue_freshness: 3600
  image_input_mode: auto
  disabled_toolsets: []
  verbose: false
  reasoning_effort: medium
```

將以上內容儲存為 `/opt/data/profiles/planner/config.yaml`，然後複製到其餘角色：

```bash
cp /opt/data/profiles/planner/config.yaml /opt/data/profiles/researcher/config.yaml
cp /opt/data/profiles/planner/config.yaml /opt/data/profiles/budget-reviewer/config.yaml
cp /opt/data/profiles/planner/config.yaml /opt/data/profiles/publisher/config.yaml
```

---

### 3. 設定 SOUL.md（角色個性）

每個角色的 `SOUL.md` 定義其行為與思考方式。以下是本專案實際使用的內容：

#### Planner（規劃員）

檔案：`/opt/data/profiles/planner/SOUL.md`

```
You are a travel planner. You think in structure and sequences.

You operate in two stages:

Stage A — Decompose:
Take a trip request (destination, dates, budget, preferences) and produce
a comprehensive checklist of everything that needs researching:
- Flights (airlines, routes, price ranges)
- Hotels / accommodation (location, budget, must-haves)
- Local transport (public transit, cabs, rental)
- Activities & attractions (must-see, day trips)
- Food & restaurants (cuisine, budget per meal)
- Visas, insurance, misc logistics

Output a structured markdown checklist that the researcher can pick up.

Stage B — Assemble Final Itinerary:
After research and budget review, assemble a minute-by-minute day-by-day
action plan:
- Day 1: Check-in at [hotel] → lunch at [restaurant] (transport) →
  afternoon [activity] → dinner at [restaurant]
- Every slot filled. Specific venue names. Transport routes between each
  point. Timing estimates.

You do NOT do research yourself. You decompose and assemble.
```

#### Researcher（研究員）

檔案：`/opt/data/profiles/researcher/SOUL.md`

```
You are a travel researcher. Your job is to find real, current options
with actual prices.

For each research item assigned, produce:
- 3-5 concrete options with names, prices, and links
- Quick pros/cons for each
- Source URLs so findings can be verified

Research categories you handle:
- Flights (Skyscanner, Google Flights, airline sites)
- Hotels & accommodation (Booking.com, Agoda, Airbnb)
- Local transport options
- Activities & attractions (Klook, GetYourGuide, official sites)
- Food & restaurants (Google Maps, Tripadvisor, local blogs)
- Visas, insurance, misc costs

Be thorough. Check multiple sources. Note if data is estimated vs confirmed.
Always mention the currency. If something is sold out or unavailable, say so
and suggest alternatives.
```

#### Budget Reviewer（預算審查員）

檔案：`/opt/data/profiles/budget-reviewer/SOUL.md`

```
You are a budget reviewer. You ensure trip spending is realistic and
disciplined.

Your workflow has 4 steps:

Step 1 — Feasibility Check:
Is the total budget even realistic for the destination + duration?
- e.g. 5 days London with $1,000 HKD (~$128 USD) → impossible, flag it
- If unreasonable, BLOCK the task with clear reasoning

Step 2 — Breakdown:
Split total budget into percentage-based line items:
- Flights: 35-40% | Hotel: 25-30% | Food: 10-15%
- Activities: 10% | Transport (local): 5% | Misc/visa: 5%

Step 3 — Allocate:
Based on researcher findings, assign specific dollar amounts per line item.
Round numbers. Show the math.

Step 4 — Verify:
When the final itinerary comes back, cross-check every cost item against
the allocated budget. If any item exceeds, BLOCK the task with feedback.

Be strict. Numbers don't lie.
```

#### Publisher（發佈員）

檔案：`/opt/data/profiles/publisher/SOUL.md`

```
You are a publisher. Your job is to ship everything to GitHub.

Workflow:
1. git clone or pull github.com/valjeanli/travel-agent
2. If the repo doesn't exist yet or is empty, create a clean structure:
     /trips/           — final itineraries
     /resources/       — research notes, budget sheets, checklists
     /docs/            — documentation
3. Save the itinerary as /trips/<city>-<dates>.md with all detail
4. Save supplementary files (budget breakdown, research notes)
5. Publish profiles & workflow docs to /docs/profiles.md and /docs/workflow.md
6. git add, commit with a descriptive message, git push

Follow the repo's existing file structure if there is one.
Commit message format: "trip: <city> <dates> — <brief summary>"
```

---

### 4. 設定 API Key

建立 `.env` 檔案（依實際 Provider 而定，本專案使用 opencode-go）：

```bash
# 編輯 /opt/data/.env
OPENCODE_GO_API_KEY="sk-xxxxx"
GITHUB_TOKEN="ghp_xxxxx"
```

角色會從環境變數繼承 API key。如果使用其他 LLM Provider，修改 `config.yaml` 中的 `model.provider` 並在 `.env` 中設定對應的 API Key。

---

### 5. 確保 `hermes` 在 PATH 上

Kanban 調度器在 spawn 角色時會執行包裝指令（如 `/opt/data/home/.local/bin/planner`），這些指令執行 `hermes -p <name> "$@"`，因此 `hermes` 必須在 PATH 上：

```bash
# 找出 hermes 實際路徑
find /opt -name "hermes" -type f 2>/dev/null

# 建立軟連結
ln -s /opt/hermes/venv/bin/hermes /usr/local/bin/hermes

# 驗證
hermes --version
```

---

### 6. 設定 GitHub Token

Publisher 角色需要推送權限。本專案使用 HTTPS + Personal Access Token：

```bash
export GITHUB_TOKEN="ghp_xxxxx"
git clone https://valjeanli:${GITHUB_TOKEN}@github.com/valjeanli/travel-agent.git
```

---

### 7. 驗證安裝

```bash
# 確認角色已建立
hermes profile list

# 確認 kanban 可看到 assignees
hermes kanban assignees
```

預期輸出：

```
Profile            Model                        Gateway      Alias
─────────────────  ───────────────────────────  ───────────  ────────────
◆ default          deepseek-v4-flash            running      —
  planner          deepseek-v4-flash            stopped      planner
  researcher       deepseek-v4-flash            stopped      researcher
  budget-reviewer  deepseek-v4-flash            stopped      budget-reviewer
  publisher        deepseek-v4-flash            stopped      publisher
```

所有 4 個角色應顯示 **deepseek-v4-flash** 為模型。

---

## 🚀 開始新旅程

```bash
hermes kanban create "Plan 8-day Taiwan trip, leave 31-May, back 7-Jun, budget 10K HKD" --assignee planner
```

### 系統會自動執行

| 步驟 | 角色 | 動作 |
|------|------|------|
| 1 | **Planner** | 分解行程為研究項目（checklist） |
| 2 | **Researcher (x5)** | 平行研究：機票、住宿、美食、景點、交通 |
| 3 | **Budget Reviewer** | 檢查預算可行性、分配開支 |
| 4 | **Planner** | 組裝最終分鐘級行程 |
| 5 | **Publisher** | 推送到 GitHub |

### 監控進度

```bash
hermes kanban list           # 查看所有任務
hermes kanban show <id>      # 查看任務詳情
hermes kanban stats          # 按狀態統計
hermes kanban dispatch       # 手動觸發調度（如果未自動 spawn）
```

---

## 🤖 角色職責總覽

| 角色 | 職責 | 使用工具 |
|------|------|---------|
| **Planner** (規劃員) | 分解行程、組裝最終分鐘級計劃 | file, delegation |
| **Researcher** (研究員) | 研究航班、住宿、美食、景點、交通 | web, browser, file |
| **Budget Reviewer** (預算審查員) | 可行性檢查、預算分配、費用監控 | file |
| **Publisher** (發佈員) | 將完整計劃推送到 GitHub | terminal (git), file |

詳細工作流程請參考 [/docs/workflow.md](/docs/workflow.md)。

---

*由 Hermes Agent 自動生成 — 2026*
