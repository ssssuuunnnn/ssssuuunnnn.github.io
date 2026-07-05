---
layout: post
title:  "Vibe Coding 實驗室(二)，任務管理APP-結合 Cloud Firestore & Mobile Modify"
date:   2026-06-20 06:14:14 +0000
categories: 轉載
original: https://medium.com/@itkh/vibe-coding-%E5%AF%A6%E9%A9%97%E5%AE%A4-%E4%BA%8C-%E4%BB%BB%E5%8B%99%E7%AE%A1%E7%90%86app-%E7%B5%90%E5%90%88-cloud-firestore-mobile-modify-c97d74773558
---

![](https://cdn-images-1.medium.com/max/1024/0*OQjORDPFU9Lm8eyp)Photo by [Mae Mu](https://unsplash.com/@picoftasty?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)

在三天端午三天的連假，就來讓上個月做的任務管理APP增加一些進度，這次增加的功能中，也是學習到一些新東西(對我來說)，這次增加的項目。

目前結果：<https://ssssuuunnnn.github.io/matrix-todo/#matrix>

1.基礎設定  
— 建立 .github/copilot-instructions.md（記錄架構與開發慣例）  
— 加入 Google Analytics

2.Firebase 跨裝置同步  
— 新增 Google 登入（Firebase Auth）  
— Firestore 即時同步，支援跨裝置資料共享  
— 未登入維持 localStorage，首次登入自動遷移資料

3.手機 UI 優化

這次發現到原來gihub copilot有 **copilot-instructions.md** 的功能，其主要用途為讓開發者在專案中自訂 **AI 的系統提示詞（System Prompts）與行為準則** 。

簡單來說，它就像是給 Copilot 的「員工守則」，讓 AI 在幫你寫程式或回答問題時，能完全符合你或團隊的特定風格與規範。

### 核心功能與作用

當你在專案根目錄中加入這個檔案後，Copilot 在處理該專案的請求時會自動讀取它，並帶來以下改變：

  * **統一程式碼風格** ：你可以規定變數命名法（如：必須使用 camelCase）、縮排（2 格或 4 格空白）、是否使用大括號等。
  * **指定架構與設計模式** ：例如「在專案中一律使用 Repository Pattern」或「API 回傳格式必須符合特定 JSON 規範」。
  * **限制使用的套件或語法** ：你可以叫 Copilot 禁用某些舊版 API，或強制使用特定的第三方函式庫（例如：一律使用 Axios 而非 fetch）。
  * **調整回應語氣與語言** ：你可以要求 Copilot 一律用繁體中文回答，或者程式碼註解必須寫得非常詳細。



### 如何設定

設定非常簡單，只要在專案的根目錄（Root）或 .github/ 資料夾下，建立一個名為 **copilot-instructions.md** 的檔案即可。

### 內容

你可以根據專案需求自由修改，可以參考這次這個專案的內容：

<https://github.com/ssssuuunnnn/matrix-todo/blob/main/.github/copilot-instructions.md>

### 為什麼要用它？

  1. **減少重複溝通** ：不用每次開啟新對話都跟 Copilot 說「請用 TypeScript 寫」、「請用繁體中文回答」。
  2. **團隊程式碼一致性** ：當團隊所有人都 Pull 這個檔案後，每個人電腦上的 Copilot 都會遵循同一套規範，大幅降低 Code Review 的摩擦。
  3. **新人快速銜接** ：新進工程師就算不熟悉團隊的程式默契，Copilot 也會主動引導他們寫出符合團隊規範的程式碼。



**opilot-instructions.md** 在多人或直接用AI開發專案時非常有用，可以把相關規範設定好，接下來不管是在 Vibe Coding 或是手寫程式讓AI建議時，都會參照這個檔案，這樣一來在開發時，就不會變成多種寫法。最大的幫助我認為是在Vibe Coding時，AI可以變成真正懂專案規矩的虛擬隊友，這樣在概有專案的維護上可以更加的順暢。

同時也順手查了一下，github 是否有提供其他的 .md 可使用，下面也大概分享：

**CONTRIBUTING.md**

  * 當其他人準備在你的專案建立 Pull Request 或 Issue 時，GitHub 會在網頁上方**自動顯示一條連結** ，提醒對方「請先閱讀貢獻指南」。這用來規範如何開分支、Commit 怎麼寫、測試怎麼跑。



**README.md (special：個人檔案)**

  * **特殊功能** ：如果你建立一個**與你 GitHub 帳號同名** 的公開儲存庫（例如 github.com/your-username/your-username），並在裡面放 README.md，它就會轉化為你的**個人首頁大看板（Profile README）** ，支援動態統計圖表、WakaTime 狀態等多種小工具渲染。可以參考幾個厲害的開發者[anuraghazra](https://github.com/anuraghazra)、[thmsgbrt](https://github.com/thmsgbrt)



另外還有一些 .md 屬於比較用在開源軟體的部分，這邊就不再另外介紹了，例如：CODE_OF_CONDUCT.md、SECURITY.md、SUPPORT.md

總之，這次 AI 又給許多我不知道的東西，不管是 .md 或是 firebase的功能，對於未來運用到團隊上，讓人獲益良多。
