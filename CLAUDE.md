# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

所有回應與文件請使用**繁體中文**。

---

## 開發流程 SOP（每次新功能都要跑這個循環）

本專案採用規格驅動開發，基於 AI Coding SOP（見根目錄 `# AI Coding SOP.md`）。

### Branch 結構

| Branch | 用途 |
|--------|------|
| `develop` | Claude Code 在這裡開發，怎麼改都無所謂 |
| `staging` | 模擬正式環境，部署後測試只有雲端才出現的 Bug |
| `main` | 正式上線，只透過 merge 更新，**不直接在這裡開發** |

每次開始新功能前，確認你在 `develop`：
```bash
git checkout develop
```

### 新功能開發循環

```
1. /opsx:propose  描述新功能        ← 先寫規格，確認方向
2. 確認 tasks.md 任務清單           ← 像老闆掃一眼
3. /opsx:apply                      ← 讓 Claude 執行所有任務
4. 確認成果，git commit              ← 每次改完就 commit
5. /opsx:archive                    ← 歸檔，保留決策紀錄
6. merge develop → staging 測試     ← 雲端測試
7. 沒問題 → merge staging → main   ← 正式上線
```

### Git 規範

```bash
# 每次改完都要 commit（在 develop）
git add <修改的檔案>
git commit -m "簡短說明這次改了什麼"

# 測試通過後合併到 staging
git checkout staging
git merge develop

# staging 確認沒問題後上線
git checkout main
git merge staging
```

### 遇到 Bug

直接把錯誤訊息貼給 Claude Code，不需要額外解釋：
```
[整段錯誤訊息]
請幫我修復這個問題
```

---

## 專案說明

**育兒導航小幫手 (Parent Navigator)** — 台灣新手爸媽的 LINE Bot + 網頁版 AI 問答助手。透過 RAG（ChromaDB + OpenAI Embedding）回答政府補助、疫苗接種、托育資源等問題，並依使用者戶籍縣市過濾內容。

所有後端程式碼在 `Parent-Navigator-main/backend/`。根目錄的 `.md` 檔案（例如 `全國_育兒津貼.md`、`台北市_生育獎勵金.md`）是知識庫來源文件。

---

## 常用指令

所有指令在 `Parent-Navigator-main/backend/` 執行：

```bash
# 安裝套件
pip install -r requirements.txt

# 開發環境啟動（已設定 use_reloader=False，避免 APScheduler 重複啟動）
python app.py

# 正式環境啟動
gunicorn --bind 0.0.0.0:5000 --workers 2 --timeout 120 app:app

# 載入／更新 Wiki 知識庫到 MySQL + ChromaDB
python wiki_loader.py                    # 增量更新（只處理新檔案）
python wiki_loader.py --rebuild          # 強制重建所有向量
python wiki_loader.py --vectorize-only   # 只向量化，跳過 Markdown 解析
python wiki_loader.py --wiki-dir ./wiki  # 指定 wiki 資料夾路徑

# 程式碼格式化
black .

# 執行測試
pytest
```

**環境設定：** 複製 `.env.example` 為 `.env` 並填入值。必填：`LINE_CHANNEL_ACCESS_TOKEN`、`LINE_CHANNEL_SECRET`、`OPENAI_API_KEY`、`DB_HOST/DB_USER/DB_PASSWORD/DB_NAME`。

**資料庫初始化：** 先執行 `parenting_navigator_schema.sql`（主要表格），再執行 `backend/forum_schema.sql`（論壇表格）。`conversation_state` 表在首次啟動時由 `conv._ensure_state_table()` 自動建立。

**Docker：**
```bash
docker build -t parenting-navigator .
docker run -p 5000:5000 --env-file .env parenting-navigator
```

---

## 架構說明

### LINE Bot 請求流程

```
LINE 使用者 → POST /webhook → handler.handle()
  → conversation.handle_message()  [對話狀態機，優先處理]
      → 若 IDLE 且非觸發詞 → 回傳 None
  → rag_engine.generate_reply()    [RAG 問答，狀態機回 None 才執行]
  → flex_templates.*_flex()        [組合 Flex Message 卡片]
  → LINE Reply Message API
```

### 對話狀態機（`conversation.py`）

六步驟設定流程，狀態存 MySQL `conversation_state` 表（非 Redis，降低環境依賴）：
```
IDLE → ASK_CITY → ASK_NICKNAME → ASK_BIRTHDATE → ASK_GENDER → ASK_FLAGS → DONE
```
觸發詞：`開始設定 / 設定寶寶 / 新增寶寶`。任何狀態輸入 `取消` 可中斷。Postback data 格式：`key=value`（例如 `setup_city=台北市`、`setup_gender=male`）。

### RAG Pipeline（`rag_engine.py`、`wiki_loader.py`）

**知識庫建立**（手動執行 `wiki_loader.py`）：
1. 解析 `.md` 檔案：萃取 YAML Frontmatter（`tags`、`適用縣市`、`時序規則`）+ 正文
2. 正文切塊：500 字 / 塊，50 字重疊
3. 寫入 MySQL `rag_chunks`（`is_indexed=0`）
4. 呼叫 `rag_engine.add_chunks_to_chroma()` 向量化至 ChromaDB
5. 解析時序規則 → 插入 `milestones` 表

**查詢流程**（每次使用者問問題）：
1. `query_rag(question, city)` — ChromaDB cosine 相似度搜尋，套用縣市過濾 `{$or: [{cities: $contains city}, {cities: $contains "全國"}]}`；低於 `RAG_SCORE_THRESHOLD`（預設 0.35）的結果捨棄
2. `build_prompt()` — 將使用者資料（城市、小孩年齡 / 身分別）+ top-K chunk 注入 `config.SYSTEM_PROMPT`
3. `generate_reply()` — 呼叫 GPT-4o-mini，temperature=0.3

### 排程任務（`scheduler.py`）

- **每日 09:00 台北時間**：`daily_push_job()` — 查詢 MySQL view `v_today_push`，發送 LINE Push Message，更新 `push_schedule` 狀態為 `sent`／`failed`
- **每週一 02:00**：`crawl_all_targets()` — 爬取 5 個來源網站，用 MD5 change detection 更新 wiki `.md` 檔案

### 網頁版聊天（`/chat` REST endpoint）

由 `parenting-navigator-v5.html` 呼叫。接收 `{session_id, message, city}`，走相同的 RAG pipeline 但跳過對話狀態機。回傳 `{reply, sources, quick_replies}`。

### 重要設定值（`config.py`）

| 變數 | 預設值 | 說明 |
|---|---|---|
| `RAG_SCORE_THRESHOLD` | 0.35 | cosine 相似度最低門檻 |
| `RAG_TOP_K` | 5 | 每次查詢取回的 chunk 數量 |
| `OPENAI_MODEL` | gpt-4o-mini | 可改為 gpt-4o |
| `WIKI_DIR` | `./wiki` | 部署時為 `/app/wiki` |
| `CHROMA_PERSIST_DIR` | `./chroma_db` | Render 需掛載 persistent disk |

### Wiki 檔案格式

知識庫 `.md` 檔使用 YAML Frontmatter：
```yaml
---
tags: [台北市, 生育補助]
適用縣市: 台北市
時序規則:
  - 出生後60日內申請生育獎勵金
---
## 補助內容
...
```
`全國_*.md` 會在所有縣市查詢中被找到；`台北市_*.md` 只在 `city=台北市` 時出現。

### 部署（Render.com）

設定檔：`backend/render.yaml`。ChromaDB 需要 persistent disk（Render Starter 方案，$7/月）。`WIKI_DIR` 和 `CHROMA_PERSIST_DIR` 要指向掛載路徑（`/app/wiki`、`/app/chroma_db`）。機密值（`LINE_CHANNEL_ACCESS_TOKEN`、`OPENAI_API_KEY`、DB 帳密）在 Render Dashboard 設定，不寫進 `render.yaml`。
