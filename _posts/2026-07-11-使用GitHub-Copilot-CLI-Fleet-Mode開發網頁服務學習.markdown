---
layout: post
title:  "使用 GitHub Copilot CLI Fleet Mode 開發網頁服務學習"
date:   2026-07-11 09:08:03 +0000
categories: 轉載
original: https://medium.com/@itkh/%E4%BD%BF%E7%94%A8-github-copilot-cli-fleet-mode-%E9%96%8B%E7%99%BC%E7%B6%B2%E9%A0%81%E6%9C%8D%E5%8B%99%E5%AD%B8%E7%BF%92-2aa5da6565b3
description: "利用 GitHub Copilot CLI 的 Fleet Mode 多代理並行架構，拆解資料層、路由層與測試層，實作桃園市居家護理所開放資料的 Node.js Express API。"
image: https://miro.medium.com/v2/resize:fit:720/format:webp/1*vuSIVXtdfEyXNJPpNQJuXg.png
---

這週放颱風假前，跟一位新進的同仁在聊各自 AI 工具的使用，就有聊到 Subagent 的開發。因為之前有聽到相關的開發方式，但還沒有真正的實作到，趁颱風假時來研究，對於目前最常使用的 GitHub Copilot CLI 指的就是 **Fleet Mode** 支援多代理並行執行（Parallel Subagent Execution）。

主代理（Orchestrator）會將複雜的任務拆解，並分派給多個具備獨立上下文邊界與檔案權限的子代理（Subagent）同時執行。

本次以**桃園市居家護理所開放資料 (CSV)** 為例，說明如何設定並啟動多代理架構，實作一個具備資料解析、API 路由與自動化測試的 Node.js Express 網頁服務。

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*vuSIVXtdfEyXNJPpNQJuXg.png)

### 🛠️ 環境準備與工作區初始化

在使用子代理功能前，需確保 Copilot CLI 已更新至支援版本，且專案目錄已建立 Git 版控（子代理需依賴 Git 狀態進行工作區隔離）。

```bash
# 更新並啟動實驗模式
copilot /update
copilot --experimental

# 建立專案目錄並初始化
mkdir ty-nursing-api && cd ty-nursing-api
git init

# 將「1140529修-桃園市居家護理所V1.csv」放入此目錄中
```

### 📝 配置設定檔：`.github/copilot-cli.yml`

為了明確劃分各個子代理的職責，避免並行寫入時產生程式碼衝突，我們需在專案根目錄建立設定檔。你可以直接在 Copilot CLI 中輸入提示詞讓 AI 生成，或手動建立以下內容：

```yaml
version: "1.0"
fleet_modes:
  web-service-standard:
    description: "標準網頁服務開發架構，依職責劃分三個子代理"
    orchestrator:
      system_prompt: |
        你是資深架構師。請將任務拆解為資料層、路由層與測試層。
        必須確保子代理之間的檔案變更路徑不互相衝突。
    subagents:
      - id: "data-layer-agent"
        name: "Data Layer Expert"
        system_prompt: |
          專注於讀取與解析 CSV 檔案，並提供資料存取介面。
          限制：只能修改 `src/data/` 或 `src/models/` 目錄，嚴禁編寫 Express 路由。

      - id: "api-route-agent"
        name: "API & Route Expert"
        system_prompt: |
          負責實作 Express 路由、輸入驗證與統一的錯誤處理。
          限制：只能修改 `src/routes/` 或 `src/controllers/`。必須直接呼叫 Data Layer Agent 提供的介面。

      - id: "qa-test-agent"
        name: "QA & Testing Expert"
        system_prompt: |
          負責使用 Jest 或 Supertest 編寫 API 的整合測試。
          限制：只能修改 `tests/` 或 `*.test.js` 檔案。
```

### 🚀 執行任務與自動化驗證

設定檔就緒後，即可在 Copilot CLI 互動介面中套用該樣板並執行核心任務：

```bash
copilot --experimental /fleet --template web-service-standard "幫我讀取專案中的桃園居家護理所 CSV，並實作支援依『服務區域』篩選的 GET /api/nursing 與 GET /api/nursing/:id 接口"
```

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*qC9FwG0BVUuq8N8pPpBqDA.png)

執行期間的觀測與特點：

  * **即時儀表板**：終端機畫面上會同時出現多個子代理的進度條與任務日誌，展示各代理並行運作的狀態。
  * **上下文隔離**：各個 subagent 會嚴格遵循 YAML 中的路徑限制，`data-layer-agent` 產出的資料結構會直接對接給 `api-route-agent` 呼叫。

### 本機驗證

任務完成後，可透過 `/yolo` 指令直接在本地環境安裝相依套件並啟動服務：

```bash
/yolo npm install && node src/app.js
```

### 🎯 跨專案重用與擴充

這套在 `copilot-cli.yml` 定義的架構規範並不綁定特定的業務邏輯。若未來有不同的純前端專案或不同技術棧的需求，可依同樣邏輯進行微調。

例如**純前端專案**的 Subagents 劃分建議：

  * **UI Component Agent**：專職負責前端組件、樣式切版（CSS），不寫非同步請求。
  * **State & API Agent**：專職負責前端狀態管理與 API 串接、資料轉換（Data Mapping）。
  * **Frontend QA Agent**：專職負責編寫組件測試與互動邏輯驗證。

透過在設定檔中預先定義好各代理的職責與權限範圍，即可在不同專案中快速複製高效率的並行開發流程。
