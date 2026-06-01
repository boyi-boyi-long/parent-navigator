# AI Coding SOP
> 基於李董 AI Coding 課（0-3章）的標準開發流程

---

## 第〇階段：建立安全網（每個新專案必做）

### 步驟 1：建立專案資料夾
```bash
mkdir 專案名稱
cd 專案名稱
git init
```
**原因：** git init 讓 Git 開始監控這個資料夾，所有檔案的變化都會被追蹤。

---

### 步驟 2：建立 GitHub repo 並連結
```bash
echo "# 專案名稱" >> README.md
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/你的帳號/專案名稱.git
git push -u origin main
```
**原因：** commit = 電腦上存檔，push = 備份到雲端，兩個都要做才算真正安全。

---

### 步驟 3：建立三個 Branch
```bash
git checkout -b develop
git checkout -b staging
git checkout main
```

| Branch | 位置 | 用途 |
|--------|------|------|
| develop | 你的電腦 | Claude Code 在這裡開發，怎麼改都無所謂 |
| staging | 雲端測試機 | 模擬正式環境，抓只有雲端才出現的 Bug |
| main | 正式上線 | 用戶看到的，只透過 git push 更新 |

**原因：** 三層隔離，正式站不會因為開發中的爛程式碼炸掉。這也是你敢讓 Claude 全速跑的唯一理由。

---

### 步驟 4：啟動 Claude Code
```bash
claude
```
**原因：** 必須在專案資料夾內啟動，Claude Code 才知道要在哪裡工作。

---

### 步驟 5：讓 Claude 寫文件和規則
在 Claude Code 輸入：
```
在 /doc/README.md 寫清楚這個專案在做什麼。
之後所有回應和文件都要用繁體中文，把這個習慣寫進 CLAUDE.md
```
**原因：** CLAUDE.md 是 AI 的記憶外掛，每次重開都會讀取。沒有文件，換電腦或換模型就要從頭說明。

---

### 步驟 6：備份到 GitHub
在 Claude Code 輸入：
```
幫我 git add、commit、push，訊息寫「新增專案文件與開發規範」
```
**原因：** 把文件和規則備份到雲端，換電腦也不會消失。

---

## 第一階段：安裝 OpenSpec（每個新專案都要裝）

### 步驟 7：安裝 OpenSpec
在 Claude Code 輸入：
```
請安裝這個 OpenSpec：https://github.com/Fission-AI/OpenSpec
```
**原因：** OpenSpec 是裝在專案裡的，每個新專案都要重新裝。它讓你在動手前先寫好計畫，所有開發過程都有紀錄。

---

### 步驟 8：重新啟動 Claude Code
```bash
# 關掉 Claude Code，然後在 Git Bash 重新啟動
cd 專案名稱
claude
```
**原因：** 新安裝的指令需要重啟才能載入，就像裝完軟體要重開機。

---

## 第二階段：規格驅動開發（每次新功能都要做）

### 步驟 9：用 propose 寫規格
在 Claude Code 輸入：
```
/opsx:propose 描述你想做的功能
```
**原因：** 先寫規格再動手。Claude 會產出 proposal.md（為什麼做）、design.md（怎麼做）、tasks.md（任務清單），你確認方向後再開始，不會做到一半發現方向錯了。

---

### 步驟 10：確認任務清單
在 Claude Code 輸入：
```
幫我顯示 tasks.md 的內容
```
**原因：** 像老闆一樣掃一眼任務清單，確認方向沒有偏離。只需要確認，不需要自己動手。

---

### 步驟 11：睡前執行 apply
在 Claude Code 輸入：
```
/opsx:apply
```
**原因：** 讓 Claude Code 整晚自動執行所有任務。你去睡覺，早上看成果。這就是「李董不睡覺的秘訣」。

---

### 步驟 12：問進度
在 Claude Code 輸入：
```
現在進度如何？完成到哪裡，給我一個簡報
```
**原因：** 你是老闆，隨時可以要求進度報告。

---

### 步驟 13：歸檔
在 Claude Code 輸入：
```
/opsx:archive
```
**原因：** 把這次開發的所有決策紀錄歸檔。之後加新功能時 Claude 可以翻閱，不會忘記之前的設計邏輯。

---

## 第三階段：部署上線

### 步驟 14：部署到 Staging 測試
在 Claude Code 輸入：
```
我的 staging 虛擬機 IP 是：xxx.xxx.xxx.xxx
SSH 已開好
請幫我 deploy 到這台機器
使用 Docker 容器
完成後給我測試網址
```
**原因：** 先在雲端測試環境確認沒問題，才能上正式站。只有在真實伺服器才會出現的 Bug，在這裡抓到。

---

### 步驟 15：部署到 Production 正式站
在 Claude Code 輸入：
```
我的 production 虛擬機 IP 是：xxx.xxx.xxx.xxx
SSH 已開好
請幫我 deploy 到這台機器
使用 Docker 容器
完成後給我正式網址
```
**原因：** Staging 測試通過後才推上正式站，用戶永遠看到穩定的版本。

---

## 每次新功能的循環

```
/opsx:propose  描述新功能
       ↓
確認 tasks.md（像老闆掃一眼）
       ↓
/opsx:apply（睡前執行）
       ↓
早上確認成果
       ↓
/opsx:archive（歸檔）
       ↓
git push 到 staging 測試
       ↓
沒問題 → git push 到 main 上線
```

---

## 遇到 Bug 怎麼辦

直接把錯誤訊息貼給 Claude Code：
```
[把整段錯誤訊息複製貼上]
請幫我修復這個問題
```
**原因：** 不用解釋太多，Claude 看到錯誤訊息就知道怎麼修。

---

## 重要規則

- ❌ **絕對不要** 截圖或推送 API 金鑰到 GitHub
- ❌ **絕對不要** 在 Production 環境直接執行 Claude Code
- ✅ **每次改完** 都要 commit，說明這次改了什麼
- ✅ **每個新功能** 都先 propose 再 apply
- ✅ **每次完成** 都要 archive 留下紀錄

---

## 給 Claude 讀的指令

把這份文件放在專案的 `/doc/` 目錄，或直接在對話開頭說：

```
請閱讀這份 SOP 文件，之後的開發都按照這個流程進行
```

---

*Based on 李董 AI Coding 課 Chapters 0-3*