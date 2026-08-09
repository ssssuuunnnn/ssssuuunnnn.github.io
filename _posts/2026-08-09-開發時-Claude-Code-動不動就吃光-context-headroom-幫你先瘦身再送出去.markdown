---
layout: post
title:  "開發時 Claude Code 動不動就吃光 context，headroom 幫你先瘦身再送出去"
date:   2026-08-09 21:58:00 +0800
description: "拆解 headroomlabs-ai/headroom 這套壓縮工具，看它怎麼在資料送進 LLM 之前先做壓縮，並整理 wrap 模式的安裝、設定與驗證方式。"
image: /assets/images/2026-08-09-headroom/headroom-compression-pipeline.png
---

最近使用 AI CLI 這類 coding agent 一天到晚讀檔案、跑指令、印一大坨 log，這些東西全部塞進 context，沒多久就把視窗吃滿，逼你開新對話重新對齊上下文，常常額度就用完了。因為種種token使用上的問題，就問了一下也在使用AI開發的同事，就推薦了我 headroomlabs-ai/headroom，所以就來研究一下。headroom 定位很直白：這些東西在送進 LLM 之前先被壓縮過，答案不變，token 少一大截，coding agent 場景大概能省 15–20%。

核心流程：資料要先過三關才能見到 LLM

![headroom-compression-pipeline](/assets/images/2026-08-09-headroom/headroom-compression-pipeline.png)

ContentRouter 先判斷這坨資料是 JSON、程式碼還是散文，分派給對應的壓縮器；CacheAligner 負責盯著會打斷 provider KV cache 的易變內容，但從不改寫 prompt 本身；CCR 則是把原始資料留在本機快取，LLM 真的需要完整版時再自己呼叫 headroom_retrieve 拿回來。這段是骨架，純粹拿來配合開發用，怎麼裝、開哪些功能才是重點。

1. 裝法選 pip/uv，npm 版沒有 CLI

```
uv tool install --python 3.13 "headroom-ai[all]"   # 推薦，獨立虛擬環境
pip install "headroom-ai[all]"                       # 也會裝好 headroom CLI
```

npm 版 headroom-ai 只是給你自己寫程式呼叫的 library，import { compress } from 'headroom-ai'，沒有 headroom 這個指令——wrap、doctor 這些操作全部靠 CLI，走 npm 那條線是接不上的。

2. 只用 wrap，proxy 跟 library 模式先跳過

Proxy 模式是給多語言、多服務混用設計的；Library 模式要自己寫程式碼串接。純粹想開發時省 token，wrap 是最貼近的層級——一行指令包住 Claude Code，一行指令解除：

```
headroom wrap claude
headroom unwrap claude
```

wrap 執行時會順手裝 Serena（語意化程式碼導覽），而且是裝在 user scope（Claude Code 對應到 ~/.claude.json），意思是這個專案 unwrap 之後，其他專案還是留著這個外掛，不想要就加 --code-memory none 關掉。

3. 保留預設的 cache 模式，別手動切 token 模式

headroom 預設走 --mode cache，只壓縮每輪新進來的內容，前面已經送出的 prompt prefix 保持位元組不變。這對日常開發特別重要——你在同一個對話裡反覆叫 Claude Code 改同一支檔案，prefix cache 有沒有命中，直接影響回應速度跟費用。除非你很在意極限壓縮率，否則不用碰 --mode token，它會犧牲 cache 穩定性換取更多節省。

4. HEADROOM_OUTPUT_SHAPER、headroom learn 先關著

這兩個功能都會主動介入，前期不建議開：

HEADROOM_OUTPUT_SHAPER 會調整模型的 thinking 力度，判斷「這輪只是讀完檔案的收尾動作」就自動調低——如果你在寫的是需要深度推理的複雜邏輯，有機率被誤判成例行步驟
headroom learn 會去翻你過去的失敗 session，自動把修正寫進 CLAUDE.md，還沒摸熟工具行為前先開，容易讓專案設定檔被你不熟悉的規則悄悄改動

用個一兩週，確定壓縮沒讓 Claude Code 判斷跑掉，再考慮要不要開。

5. 上手第一步，跑三個指令確認接上

```
headroom wrap claude
headroom doctor       # 健康檢查，確認壓縮路徑真的有在跑
headroom dashboard     # 即時看省了多少 token
```

headroom doctor 這步不能跳，之前有人反應包了 agent 之後感覺沒效果，一查其實是壓縮路徑沒接上。

6. 壓縮會不會失真？準確率怎麼保證

裝完之後心裡難免會冒出一個問題：壓縮這麼狠，Claude Code 會不會因此看漏關鍵資訊、判斷跟著跑掉？headroom 在這點上做得比想像中誠實，機制拆開來看有幾層：

CCR 架構本身就不是刪除，是暫存：壓縮後的原始內容一律先存進本機 SQLite（雜湊索引），LLM 拿到的是壓縮版加一個 headroom_retrieve 工具，真的需要細節時可以自己呼叫拿回完整資料，不是壓完就沒了。
有 --lossless 模式可以當對照組：不放心的話，可以整個關掉有損壓縮，走無損模式直接原封轉發，拿來跟正常壓縮模式比對回應差異，比空口猜要準。
官方跑過標準 benchmark，指令是公開的：GSM8K、TruthfulQA、SQuAD v2、BFCL 四項壓縮前後的準確率對照，數字沒有掉——這幾行測試指令你自己也能重跑：

```
git clone https://github.com/chopratejas/headroom.git
cd headroom
pip install -e ".[evals,html]"
pytest tests/test_evals/ -v -s
```

Error Handling 是 fail-open 設計：壓縮流程本身出錯時，預設行為是讓原始內容直接通過，不會硬壓出一個壞掉的版本讓 LLM 誤判，等於留了一道安全閥。

真要拿它配合日常開發，最簡單的驗證方式是挑一支專案裡典型的檔案（常改的 API 程式碼、常見的 log 格式），先跑一次 headroom wrap claude --lossless 讓 Claude Code 處理一個任務，再跑一次正常壓縮模式處理同樣的任務，比對兩次的回應品質有沒有差異——這個對照只需要花十分鐘，比空猜安心很多。

跟之前介紹過的 Superpowers、Matt Pocock skills、agent-skills 擺在一起看，其實完全不是同一條賽道。那三套管的是「AI agent 該按照什麼流程做事」，headroom 管的是「不管你用什麼流程，資料進出的路上先幫你瘦身」——兩者根本不衝突，甚至該疊著用：用 agent-skills 定義好 /spec ➔ /plan ➔ /build 的開發流程，同時掛上 headroom 去壓縮每個階段跑出來的工具輸出跟 log，省下來的 token 直接轉成更長的 context window 可以用。真要說差異，headroom 動的是基礎設施層，headroom wrap 完就自動生效，不用改任何一行程式碼；agent-skills 那些動的是流程層，靠 Prompt 說服 AI 按規矩來——一個省的是帳單上的數字，一個省的是你盯著它做事的力氣，棒棒。

資料來源：headroomlabs-ai/headroom
