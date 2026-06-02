# 部署日誌 — 育兒導航全攻略

紀錄日期：2026-06-02

---

## 專案基本資訊

| 項目 | 值 |
|------|-----|
| GitHub | https://github.com/boyi-boyi-long/parent-navigator |
| 前後端網址 | https://parent-navigator.onrender.com |
| LINE Bot ID | @303xbemb |
| LINE 加入連結 | https://line.me/R/ti/p/@303xbemb |
| 資料庫 | Aiven MySQL（babybaby-babybaby0901.a.aivencloud.com:28117） |
| 資料庫名稱 | parenting_navigator |

---

## Git Branch 結構

| Branch | 用途 |
|--------|------|
| `develop` | 開發用，Claude Code 在這裡工作 |
| `staging` | 雲端測試（目前未獨立部署） |
| `main` | 正式上線，Render 監聽此 branch 自動部署 |

---

## 完成的功能

- Flask 後端 + LINE Bot Webhook
- RAG 問答（ChromaDB + OpenAI gpt-4.1-nano）
- 網頁版前端（SPA，站內頁面路由）
- 導覽列 6 個分類頁面（醫療、補助、教育、FAQ、關於、首頁）
- 每日異動、熱門資訊、常用連結獨立頁面
- Chatbot Widget（position:fixed，所有頁面皆可用）
- LINE 官方按鈕（topbar，一鍵加好友）
- OpenSpec 規格驅動開發流程

---

## 遇到的問題與解法

### 問題 1：Docker pipe 編碼錯誤
**症狀：** 執行 SQL schema 時出現 `ERROR 1291: Column 'category' has duplicated value '????' in ENUM`

**原因：** PowerShell 透過 pipe 傳 SQL 給 Docker 時，中文字元編碼被截斷

**解法：** 改用 `-v` volume 掛載，讓 Docker 直接讀取主機上的 SQL 檔案
```bash
docker run --rm -v "本機路徑:/sql" mysql:8 mysql ... -e "source /sql/schema.sql"
```

---

### 問題 2：forum_schema.sql 資料庫名稱不符
**症狀：** `ERROR 1049: Unknown database 'parenting_nav'`

**原因：** `forum_schema.sql` 開頭有 `USE parenting_nav`，但 Aiven 上的資料庫是 `parenting_navigator`

**解法：** 執行前用 PowerShell 替換資料庫名稱，建立暫時修正版再執行，執行後刪除

---

### 問題 3：主要 schema 建在非預期的資料庫
**症狀：** 跑完 `parenting_navigator_schema.sql` 後，在 `defaultdb` 找不到表格

**原因：** SQL 檔案自己有 `CREATE DATABASE IF NOT EXISTS parenting_navigator` + `USE parenting_navigator`，表格建在 `parenting_navigator`，不是 Aiven 預設的 `defaultdb`

**解法：** 確認資料庫名稱後，改用 `parenting_navigator` 連線，`DB_NAME` 環境變數設為 `parenting_navigator`

---

### 問題 4：Render 找不到 Dockerfile
**症狀：** `failed to solve: failed to read dockerfile: open Dockerfile: no such file or directory`

**原因：** Render 的 Root Directory 沒有設定，預設從 repo 根目錄找 Dockerfile，但 Dockerfile 在 `Parent-Navigator-main/backend/`

**解法：** Render Dashboard → Settings → Root Directory 填入 `Parent-Navigator-main/backend`

---

### 問題 5：line-bot-sdk 版本 SyntaxError
**症狀：**
```
SyntaxError: invalid syntax
File "text_message_v2.py", line 23
from linebot.v3.messaging.models.dict[str,_substitution_object] import ...
```

**原因：** `line-bot-sdk==3.14.0` 的 `TextMessageV2` 檔案有 Python 語法錯誤，是該版本的 bug

**解法：** `requirements.txt` 改為 `line-bot-sdk>=3.17.0`

---

### 問題 6：LINE Webhook Verify 回傳 405
**症狀：** LINE Developers 按 Verify 時回傳 `405 Method Not Allowed`

**原因：** `/webhook` 路由只允許 POST，但 LINE Verify 會先發 GET 測試連線

**解法：** `app.py` 的 webhook route 改為同時接受 GET 和 POST
```python
@app.route("/webhook", methods=["GET", "POST"])
def webhook():
    if request.method == "GET":
        return "OK", 200
    # ... 原本的 POST 處理
```

---

### 問題 7：LINE Webhook URL 填錯
**症狀：** Verify 成功但收到 `404 Not Found`

**原因：** Webhook URL 填了 `https://parent-navigator.onrender.com`，沒有加 `/webhook`

**解法：** 改為 `https://parent-navigator.onrender.com/webhook`

---

## 環境設定備忘

**Render 環境變數（機密值請勿外洩）：**
```
FLASK_DEBUG=false
FLASK_PORT=5000
FLASK_SECRET_KEY=（自訂）
OPENAI_API_KEY=（OpenAI Dashboard 取得）
LINE_CHANNEL_ACCESS_TOKEN=（LINE Developers 取得）
LINE_CHANNEL_SECRET=（LINE Developers 取得）
DB_HOST=babybaby-babybaby0901.a.aivencloud.com
DB_PORT=28117
DB_NAME=parenting_navigator
DB_USER=avnadmin
DB_PASSWORD=（Aiven Dashboard 取得）
CHROMA_PERSIST_DIR=/app/chroma_db
WIKI_DIR=/app/wiki
SCHEDULER_TIMEZONE=Asia/Taipei
SCHEDULER_PUSH_HOUR=9
```

**OpenAI 模型：** `gpt-4.1-nano`（最便宜，知識截止 2025-08）

---

## 工具與版本

| 工具 | 用途 |
|------|------|
| Docker | 建立 Aiven 資料庫 schema（一次性使用，`--rm` 不留 container） |
| GitHub CLI (`gh`) | 建立 GitHub repo |
| OpenSpec | 規格驅動開發（`/opsx:propose` → `/opsx:apply` → `/opsx:archive`） |

---

## 未來待辦

- [ ] 將 wiki .md 知識庫載入 ChromaDB（執行 `wiki_loader.py`）
- [ ] staging branch 接 CI/CD 或獨立部署環境
- [ ] LINE Bot QR Code 展示頁（目前電腦版點 LINE 按鈕會顯示 QR Code，此為 LINE 設計，手機正常）
- [ ] 考慮改用 Digital Ocean Droplet + Docker Compose 取代 Render + Aiven
