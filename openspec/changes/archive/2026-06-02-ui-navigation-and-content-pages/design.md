## Context

目前 `backend/templates/index.html` 是單一 HTML 檔案，所有導覽連結均為 `href="#"`，點擊無任何頁面切換效果。分類卡片綁定 `data-query` 觸發 Chatbot，造成功能混用。後端為 Flask，僅提供一個 `GET /` 路由回傳此 HTML。

## Goals / Non-Goals

**Goals:**
- 在單一 HTML 檔案內實作前端路由（SPA），不新增後端路由
- 每個導覽分類有獨立的內容區塊，切換時平滑顯示
- Chatbot widget 浮動於所有頁面，完全解耦
- LINE 按鈕置於 header 右上角

**Non-Goals:**
- 不實作真實資料庫查詢，內容頁面使用結構化靜態資料
- 不實作會員登入、論壇功能
- 不更動後端 Flask 路由

## Decisions

**決策 1：純前端 SPA，不用 History API**

選擇以 CSS `display` 切換 + JavaScript 管理「當前頁面」狀態，而非 `window.history.pushState`。

理由：部署在 Render 免費方案，無法設定 catch-all redirect，用 hash route 或純 JS 切換最穩定，不會有 F5 重整 404 的問題。

**決策 2：內容頁面寫在同一 HTML，以 `<section>` 區隔**

每個頁面對應一個 `<section id="page-xxx">` 區塊，預設 `display:none`，切換時只顯示目標區塊。

理由：不需要額外的 HTTP 請求，載入快，且符合目前單檔部署的架構。

**決策 3：LINE 按鈕使用 `target="_blank"` 開新分頁**

使用者點擊後跳到 LINE Bot 加入頁，不離開當前平台。

## Risks / Trade-offs

- [SEO 較差] 所有內容在同一 URL → 可接受，平台以功能為主不以 SEO 為優先
- [內容為靜態假資料] 各分類頁面的文章清單為 mock data → 日後可對接後端 API
- [首次載入略大] 所有頁面 HTML 一次載入 → 可接受，內容量不大

## Migration Plan

1. 修改 `backend/templates/index.html`：加入各內容 section + 前端路由 JS
2. 本地測試確認切換正常、Chatbot 不受影響
3. Git commit → push main → Render 自動部署
4. 回滾：`git revert` 上一個 commit 即可
