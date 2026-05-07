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
- Kanban 功能已啟用（`hermes kanban init`）
- `hermes` 命令在 PATH 上（`which hermes`）

---

### 1. 建立 4 個角色（Profiles）

每個角色就是一個 Hermes Agent Profile，擁有自己的模型配置、個性設定（SOUL.md）和工作目錄。

```bash
# 一次建立全部 4 個角色
hermes profile create planner
hermes profile create researcher
hermes profile create budget-reviewer
hermes profile create publisher
```

建立後可用以下指令確認：

```bash
hermes profile list
```

預期輸出：

```
Profile            Model     Gateway      Alias
─────────────────  ────────  ───────────  ────────────
◆ default          ...       running      —
  planner          —         stopped      planner
  researcher       —         stopped      researcher
  budget-reviewer  —         stopped      budget-reviewer
  publisher        —         stopped      publisher
```

每個角色建立後會產生：
- `~/.hermes/profiles/<name>/SOUL.md` — 角色個性設定
- `~/.hermes/profiles/<name>/config.yaml` — 模型配置（建立後需手動加入）
- `~/.hermes/profiles/<name>/workspace/` — 工作目錄
- 系統指令別名（如 `planner chat`、`researcher chat`）

---

### 2. 設定模型配置（config.yaml）

每個角色需要一個 `config.yaml` 指定使用的 LLM。以下使用 deepseek-v4-flash（經 opencode-go 提供商）作為範例。

為每個角色建立相同的 `config.yaml`：

```bash
# 以 planner 為例，其餘 3 個角色做法相同
cat > ~/.hermes/profiles/planner/config.yaml << 'EOF'
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
  api_max_retries: 3
  reasoning_effort: medium
EOF

# 複製到其餘 3 個角色
cp ~/.hermes/profiles/planner/config.yaml ~/.hermes/profiles/researcher/config.yaml
cp ~/.hermes/profiles/planner/config.yaml ~/.hermes/profiles/budget-reviewer/config.yaml
cp ~/.hermes/profiles/planner/config.yaml ~/.hermes/profiles/publisher/config.yaml
```

> 💡 如果使用其他模型（如 Claude、GPT），請修改 `model.default` 和 `model.provider`。

---

### 3. 設定角色個性（SOUL.md）

每個角色的 `SOUL.md` 定義其行為模式。以下是各角色的核心指令，完整版請參考 `/docs/profiles.md`。

**Planner（規劃員）— `~/.hermes/profiles/planner/SOUL.md`：**

```markdown
You are a travel planner. You operate in two stages:
Stage A: Decompose a trip request into a research checklist.
Stage B: After research, assemble a minute-by-minute day-by-day itinerary.
You do NOT do research yourself.
Output format: Traditional Chinese markdown.
```

**Researcher（研究員）— `~/.hermes/profiles/researcher/SOUL.md`：**

```markdown
You are a travel researcher. Find real options with prices for flights,
hotels, food, attractions, and transport. Produce 3-5 options per item
with pros/cons and source URLs. Use web search and browser tools.
Output in Traditional Chinese.
```

**Budget Reviewer（預算審查員）— `~/.hermes/profiles/budget-reviewer/SOUL.md`：**

```markdown
You are a budget reviewer. Check if the budget is realistic for the
destination. Break down into line items. Allocate specific amounts.
Verify final plan stays within budget. If over budget, BLOCK the task.
Output in Traditional Chinese.
```

**Publisher（發佈員）— `~/.hermes/profiles/publisher/SOUL.md`：**

```markdown
You are a publisher. Clone the GitHub repo, save all files (itinerary,
research, budget, docs), and commit+push. Output in Traditional Chinese.
```

---

### 4. 設定 API Key

角色會從環境變數或 `.env` 檔案繼承 API key。建立或更新 `.env`：

```bash
# 編輯 ~/.hermes/.env 或直接設定環境變數
export OPENCODE_GO_API_KEY="sk-xxxxx"
export GITHUB_TOKEN="ghp_xxxxx"
```

或使用設定精靈（互動式）：

```bash
planner setup    # 會問 model、API key、terminal 等
```

---

### 5. 確保 `hermes` 在 PATH 上

Kanban 調度器需要 `hermes` 命令在 PATH 上才能 spawn 角色：

```bash
# 找出 hermes 位置
which hermes || find /opt -name "hermes" -type f 2>/dev/null

# 如果不在 PATH 上，建立 symlink
ln -s /opt/hermes/venv/bin/hermes /usr/local/bin/hermes
```

驗證：

```bash
hermes --version
```

---

### 6. 設定 GitHub 推送權限（Publisher 專用）

Publisher 角色需要能推送到 GitHub。有兩種方式：

**方式 A — 使用 GITHUB_TOKEN（建議）：**

```bash
export GITHUB_TOKEN="ghp_xxxxx"
git clone https://valjeanli:${GITHUB_TOKEN}@github.com/valjeanli/travel-agent.git
```

**方式 B — 使用 SSH Key：**

```bash
ssh-keygen -t ed25519 -C "your-email"
# 將公鑰加到 GitHub Settings → SSH and GPG keys
```

---

### 7. 確認所有角色就緒

```bash
hermes profile list
```

所有 4 個角色應顯示配置了模型（不再是 `—`），且 `hermes kanban assignees` 應列出全部 4 個：

```bash
hermes kanban assignees
```

輸出：

```
planner          0 tasks
researcher       0 tasks
budget-reviewer  0 tasks
publisher        0 tasks
```

---

## 🚀 開始新旅程

### 基本指令

```bash
hermes kanban create "Plan [N]-day [city] trip, [dates], [budget]" --assignee planner
```

### 實例

```bash
hermes kanban create "Plan 5-day Tokyo trip, Apr 2026, 150k JPY" --assignee planner
```

### 系統會自動執行

| 步驟 | 角色 | 動作 |
|------|------|------|
| 1 | **Planner** | 分解行程為研究項目（checklist） |
| 2 | **Researcher** | 平行研究機票、住宿、美食、景點、交通 |
| 3 | **Budget Reviewer** | 檢查預算可行性、分配開支 |
| 4 | **Planner** | 組裝最終分鐘級行程 |
| 5 | **Budget Reviewer** | 最終預算確認 |
| 6 | **Publisher** | 推送到 GitHub |

### 監控進度

```bash
hermes kanban list           # 查看所有任務
hermes kanban show <id>      # 查看任務詳情
hermes kanban stats          # 按狀態統計
hermes kanban dispatch       # 手動觸發調度
```

---

## 🤖 工作流程

詳細流程請參考 📄 文件：

- [/docs/workflow.md](/docs/workflow.md) — 完整管線圖與任務相依鏈
- [/docs/profiles.md](/docs/profiles.md) — 4 個角色的完整 SOUL.md 與 config.yaml 範例

| 角色 | 職責 | 使用工具 |
|------|------|---------|
| **Planner** (規劃員) | 分解行程、組裝最終分鐘級計劃 | file, delegation |
| **Researcher** (研究員) | 研究航班、住宿、美食、景點、交通 | web, browser, file |
| **Budget Reviewer** (預算審查員) | 可行性檢查、預算分配、費用監控 | file |
| **Publisher** (發佈員) | 將完整計劃推送到 GitHub | terminal (git), file |

---

*由 Hermes Agent 自動生成 — 2026*
