## ADDED Requirements

### Requirement: 導覽列切換站內頁面
系統 SHALL 在使用者點擊 header 導覽列連結時，切換顯示對應的內容區塊，不跳轉外部頁面。

#### Scenario: 點擊導覽列項目
- **WHEN** 使用者點擊「醫療資訊」、「補助資訊」、「教育資源」、「FAQ」或「關於平台」
- **THEN** 對應的 `<section>` 區塊顯示，其他 section 隱藏，當前連結標記為 active

#### Scenario: 點擊 Logo 回首頁
- **WHEN** 使用者點擊左上角 Logo
- **THEN** 首頁區塊顯示，導覽列 active 狀態清除

### Requirement: 分類卡片導向內容頁
系統 SHALL 在使用者點擊首頁分類卡片時，切換至對應分類的內容頁面，而非觸發 Chatbot。

#### Scenario: 點擊分類卡片
- **WHEN** 使用者點擊任一分類卡片（如「育兒補助申請」）
- **THEN** 切換至對應的補助資訊內容頁，Chatbot 不彈出

### Requirement: Chatbot 維持獨立浮動
系統 SHALL 確保 Chatbot widget 在所有頁面切換時持續顯示於右下角，與頁面路由無關。

#### Scenario: 頁面切換後 Chatbot 仍可用
- **WHEN** 使用者切換至任何內容頁面
- **THEN** Chatbot widget 仍顯示於右下角，可正常輸入與回覆
