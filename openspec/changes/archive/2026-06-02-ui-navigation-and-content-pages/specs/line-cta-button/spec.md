## ADDED Requirements

### Requirement: LINE 官方帳號按鈕
系統 SHALL 在 header 右上角提供 LINE 官方帳號快速入口按鈕。

#### Scenario: 點擊 LINE 按鈕
- **WHEN** 使用者點擊 header 右上角的 LINE 按鈕
- **THEN** 以新分頁開啟 LINE Bot 加入連結，不離開平台

#### Scenario: LINE 按鈕視覺識別
- **WHEN** 使用者瀏覽任何頁面
- **THEN** LINE 按鈕以 LINE 品牌綠色（#06C755）顯示於 header 右上角，與其他按鈕有明確區別
