# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**育兒導航小幫手 (Parent Navigator)** — A Taiwan parenting AI assistant delivered as a LINE Bot and web chat widget. It uses RAG (ChromaDB + OpenAI embeddings) to answer questions about government subsidies, vaccines, and childcare resources, scoped by the user's city.

All backend code lives in `Parent-Navigator-main/backend/`. The root-level `.md` files (e.g., `全國_育兒津貼.md`, `台北市_生育獎勵金.md`) are the wiki knowledge base source documents.

## Commands

All commands run from `Parent-Navigator-main/backend/`:

```bash
# Install dependencies
pip install -r requirements.txt

# Run development server (use_reloader=False is already set to prevent APScheduler double-start)
python app.py

# Run production server
gunicorn --bind 0.0.0.0:5000 --workers 2 --timeout 120 app:app

# Load/update wiki knowledge base into MySQL + ChromaDB
python wiki_loader.py                    # incremental (new files only)
python wiki_loader.py --rebuild          # full rebuild of all vectors
python wiki_loader.py --vectorize-only   # skip parsing, only push pending chunks to ChromaDB
python wiki_loader.py --wiki-dir ./wiki  # specify wiki directory

# Code formatting
black .

# Tests
pytest
```

**Environment setup:** Copy `.env.example` to `.env` and fill in values. Required keys: `LINE_CHANNEL_ACCESS_TOKEN`, `LINE_CHANNEL_SECRET`, `OPENAI_API_KEY`, `DB_HOST/DB_USER/DB_PASSWORD/DB_NAME`.

**Database init:** Run `parenting_navigator_schema.sql` (main tables) then `backend/forum_schema.sql` (forum tables) against MySQL. The `conversation_state` table is auto-created on first boot via `conv._ensure_state_table()`.

**Docker:**
```bash
docker build -t parenting-navigator .
docker run -p 5000:5000 --env-file .env parenting-navigator
```

## Architecture

### Request Flow (LINE Bot)

```
LINE user → POST /webhook → handler.handle()
  → conversation.handle_message()  [state machine — checked FIRST]
      → if IDLE and non-trigger keyword → returns None
  → rag_engine.generate_reply()    [RAG Q&A, only if state machine returns None]
  → flex_templates.*_flex()        [build Flex Message card]
  → LINE Reply Message API
```

### Conversation State Machine (`conversation.py`)

Six-step onboarding flow persisted in MySQL `conversation_state` table (not Redis):
```
IDLE → ASK_CITY → ASK_NICKNAME → ASK_BIRTHDATE → ASK_GENDER → ASK_FLAGS → DONE
```
Trigger words `開始設定 / 設定寶寶 / 新增寶寶` transition from IDLE. Any state can be cancelled with `取消`. Postback data uses `key=value` format (e.g., `setup_city=台北市`, `setup_gender=male`).

### RAG Pipeline (`rag_engine.py`, `wiki_loader.py`)

**Ingestion** (run manually via `wiki_loader.py`):
1. Parse `.md` files: extract YAML frontmatter (`tags`, `適用縣市`, `時序規則`) + body
2. Chunk body text at 500 chars with 50-char overlap
3. Write chunks to MySQL `rag_chunks` (with `is_indexed=0`)
4. Vectorize pending chunks via `rag_engine.add_chunks_to_chroma()` → ChromaDB
5. Parse timing rules → insert `milestones` rows

**Query** (called per user message):
1. `query_rag(question, city)` — ChromaDB cosine similarity search, filtered by `{$or: [{cities: $contains city}, {cities: $contains "全國"}]}`; results below `RAG_SCORE_THRESHOLD` (default 0.35) are dropped
2. `build_prompt()` — injects user context (city, children ages/flags) + top-K chunks into `config.SYSTEM_PROMPT`
3. `generate_reply()` — calls GPT-4o-mini at temperature 0.3

### Scheduled Jobs (`scheduler.py`)

- **Daily 09:00 Asia/Taipei**: `daily_push_job()` — queries MySQL view `v_today_push`, sends LINE Push Messages, marks `push_schedule` rows as `sent`/`failed`
- **Weekly Monday 02:00**: `crawl_all_targets()` — scrapes 5 source sites with MD5 change detection, updates wiki `.md` files

### Web Chat (`/chat` REST endpoint)

Used by `parenting-navigator-v5.html`. Accepts `{session_id, message, city}`, runs the same RAG pipeline as LINE but skips the state machine. Returns `{reply, sources, quick_replies}`.

### Key Config Values (`config.py`)

| Variable | Default | Notes |
|---|---|---|
| `RAG_SCORE_THRESHOLD` | 0.35 | Minimum cosine similarity to include a chunk |
| `RAG_TOP_K` | 5 | Number of chunks retrieved per query |
| `OPENAI_MODEL` | gpt-4o-mini | Can switch to gpt-4o |
| `WIKI_DIR` | `./wiki` | Deployment: `/app/wiki` via Docker/Render |
| `CHROMA_PERSIST_DIR` | `./chroma_db` | Must be a persistent disk on Render |

### Wiki File Format

Knowledge base files use YAML frontmatter:
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
Files named `全國_*.md` are tagged `cities: 全國` and returned for all city queries. Files named `台北市_*.md` are only returned when `city=台北市` is queried.

### Deployment (Render.com)

Defined in `backend/render.yaml`. ChromaDB requires a persistent disk (Render Starter plan, $7/month). The `WIKI_DIR` and `CHROMA_PERSIST_DIR` env vars must point to the mounted disk paths (`/app/wiki`, `/app/chroma_db`). Secrets (`LINE_CHANNEL_ACCESS_TOKEN`, `OPENAI_API_KEY`, DB credentials) are set in Render Dashboard, not in `render.yaml`.
