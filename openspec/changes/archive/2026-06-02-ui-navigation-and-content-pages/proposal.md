## Why

目前平台導覽列連結無實際目標頁面，點擊分類後直接觸發 Chatbot，導致資訊閱讀與 AI 問答功能混淆。使用者無法在站內瀏覽分類內容，且缺少直接連結至 LINE 官方帳號的入口。

## What Changes

- 導覽列「醫療資訊、補助資訊、教育資訊、FAQ、關於平台」各自對應站內內容頁面（單頁 SPA 切換，不跳外部連結）
- 右上角加入 LINE 官方帳號按鈕，點擊開新分頁導向 LINE Bot 加入連結
- 「每日異動資訊」、「熱門資訊」、「常用連結」各自獨立為站內瀏覽頁面
- Chatbot widget 維持浮動於所有頁面右下角，作為獨立互動入口，不與內容頁面混淆
- 首頁保持現有佈局，分類卡片改為導向對應內容頁而非觸發 Chatbot

## Capabilities

### New Capabilities
- `in-page-navigation`: 單頁應用路由，點擊導覽列與分類卡片在站內切換頁面內容
- `content-pages`: 醫療、補助、教育、FAQ、關於、每日異動、熱門資訊、常用連結等各分類的靜態內容頁面
- `line-cta-button`: 右上角 LINE 官方帳號快速入口按鈕

### Modified Capabilities
- 無現有 spec 需異動

## Impact

- `backend/templates/index.html`：加入前端路由邏輯與各頁面內容區塊
- 分類卡片 `data-query` 屬性改為 `data-page` 導向對應頁面
- Chatbot widget 與頁面路由完全解耦，維持獨立運作
