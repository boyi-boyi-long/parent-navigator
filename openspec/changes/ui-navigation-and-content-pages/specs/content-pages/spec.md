## ADDED Requirements

### Requirement: 每日異動資訊獨立頁面
系統 SHALL 提供「每日異動資訊」獨立頁面，顯示結構化的新聞列表，可於站內閱讀。

#### Scenario: 點擊每日異動資訊
- **WHEN** 使用者點擊導覽列或首頁「每日異動資訊」連結
- **THEN** 切換至每日異動頁面，顯示完整新聞列表，不跳轉至 Chatbot

### Requirement: 熱門資訊獨立頁面
系統 SHALL 提供「熱門資訊」獨立頁面，顯示熱門文章列表。

#### Scenario: 點擊熱門資訊
- **WHEN** 使用者點擊「熱門資訊」連結或「查看全部」
- **THEN** 切換至熱門資訊頁面，顯示完整熱門文章列表

### Requirement: 分類內容頁面
系統 SHALL 為「醫療資訊」、「補助資訊」、「教育資源」各自提供站內內容頁面。

#### Scenario: 閱讀分類內容
- **WHEN** 使用者進入任一分類頁面
- **THEN** 顯示該分類的文章列表與摘要，不跳轉外部連結

### Requirement: FAQ 頁面
系統 SHALL 提供 FAQ 頁面，以手風琴（accordion）格式呈現常見問題與解答。

#### Scenario: 展開 FAQ 項目
- **WHEN** 使用者點擊 FAQ 問題
- **THEN** 展開顯示答案，其他問題可同時展開

### Requirement: 常用連結頁面
系統 SHALL 提供「常用連結」獨立頁面，列出政府官方資源連結（可開新分頁）。

#### Scenario: 點擊常用連結
- **WHEN** 使用者點擊任一常用連結
- **THEN** 以新分頁開啟外部連結，不離開平台
