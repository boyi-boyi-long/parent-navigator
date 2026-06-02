## 1. 前端路由基礎架構

- [x] 1.1 在 `index.html` 加入所有頁面的 `<section>` 容器（home, medical, subsidy, education, faq, about, news-daily, news-hot, quick-links）
- [x] 1.2 寫 `showPage(pageId)` JS 函式：隱藏所有 section，顯示目標 section，更新導覽列 active 狀態
- [x] 1.3 綁定 header 導覽列連結的 click 事件，呼叫 `showPage()`
- [x] 1.4 綁定 Logo click 事件回首頁

## 2. LINE CTA 按鈕

- [x] 2.1 在 header 右上角加入 LINE 官方帳號按鈕（綠色 #06C755，`target="_blank"`）
- [x] 2.2 填入實際 LINE Bot 加入連結（@303xbemb，已更新至 topbar）

## 3. 各內容頁面建立

- [x] 3.1 建立「醫療資訊」頁面：文章列表 + 摘要（mock 資料，5 筆）
- [x] 3.2 建立「補助資訊」頁面：補助項目列表（mock 資料，5 筆）
- [x] 3.3 建立「教育資源」頁面：資源列表（mock 資料，5 筆）
- [x] 3.4 建立「FAQ」頁面：accordion 手風琴，10 個常見問題
- [x] 3.5 建立「關於平台」頁面：平台介紹、資料來源說明
- [x] 3.6 建立「每日異動資訊」完整頁面：擴充首頁的 5 筆至 10 筆
- [x] 3.7 建立「熱門資訊」完整頁面：擴充首頁的 5 筆至 10 筆
- [x] 3.8 建立「常用連結」獨立頁面：政府官方資源，`target="_blank"`

## 4. 首頁分類卡片調整

- [x] 4.1 將分類卡片的 `data-query` 改為 `data-page`，對應各內容頁 ID
- [x] 4.2 分類卡片 click 事件改為呼叫 `showPage()` 而非 `sendMessage()`

## 5. Chatbot 解耦確認

- [x] 5.1 確認 Chatbot widget 的 `position: fixed` 不受 section 切換影響
- [x] 5.2 確認快速按鈕、輸入框在所有頁面切換後仍正常運作

## 6. 部署

- [x] 6.1 git add + commit
- [x] 6.2 merge develop → main + push（Render 自動部署）
- [x] 6.3 開啟 `https://parent-navigator.onrender.com` 確認各頁面切換正常
