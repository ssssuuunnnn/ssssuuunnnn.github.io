---
layout: post
title:  "比 Matt Pocock 更完整的一套：Addy Osmani 的 agent-skills 生產線"
date:   2026-08-02 19:14:00 +0800
description: "拆解 Chrome 團隊工程師 Addy Osmani 打造的 addyosmani/agent-skills，看它怎麼用 spec、plan、build、test、review、ship 六階段把工程紀律寫進 24 個 Skill，並與 Matt Pocock、Superpowers 兩套框架比較。"
image: /assets/images/2026-08-02-agent-skills-addy-osmani/agent-skills-lifecycle-flow.svg
---

上次介紹完 Matt Pocock 的 skills 之後，忍不住又去挖了一輪，這次挖到的是 Chrome 團隊工程師 Addy Osmani 弄的 addyosmani/agent-skills。

安裝方式（最近買了claude，所以就改用 claude code）

```
/plugin marketplace add addyosmani/agent-skills
/plugin install agent-skills@addy-agent-skills
```

不想整包安裝，也可以用作者提供的 skills CLI 單裝一個：

```
npx skills add addyosmani/agent-skills --skill interview-me
```

這個 CLI 厲害的地方是可以一鍵裝進 70 幾種 agent（Cursor、Codex、Gemini CLI、Windsurf 各個 CLI 都可以用）。

整個系統的核心邏輯：六個階段把工程紀律寫死

這套的骨架比 Matt Pocock 的三段式更長，拆成六個 slash command：

![agent-skills-lifecycle-flow](/assets/images/2026-08-02-agent-skills-addy-osmani/agent-skills-lifecycle-flow.svg)

/spec（定義要做什麼） ➔ /plan（拆解任務） ➔ /build（增量實作） ➔ /test（證明它是對的） ➔ /review（品質關卡） ➔ /ship（上線）

每個階段背後綁的不是單一 Skill，而是一整組——例如 /build 背後串了增量實作、TDD、context engineering、前端工程、API 設計等 8 個 Skill，/review 則會拉出 code review、簡化、資安、效能四個角度一起檢查。24 個 Skill 全部統一格式：Overview、When to Use、Process、Rationalizations、Red Flags、Verification 六個區塊，其中 Rationalizations 這欄最有意思，直接列出 AI 常用來偷懶跳步驟的藉口（例如「我晚點再補測試」），等於先把 AI 會鑽的偷懶的地方堵起來。(這個很酷XD)

**1. interview-me —— 意圖萃取，比 /grill-me 更早介入**

跟 Matt Pocock 的 /grill-me 概念類似，都是逐題拷問，但 interview-me 卡在更前面的位置：它假設你連自己要什麼都還沒講清楚。內部邏輯是每問一題就附上「我猜你的答案是...」，直到能在使用者開口前就預測出對方會怎麼回答，才算過關。作者舉的例子很到位——使用者說「幫我做一個實驗數據儀表板」，問兩題之後才發現，真正要的其實不是儀表板，是一份「清單」，兩者的範圍跟工作量完全不同。這一步的產出不是 Spec，而是一句「確認過的意圖陳述」，Spec 是下游的事。

**2. doubt-driven-development —— 決策做到一半就找人唱反調**

這是全部 24 個 Skill 裡我覺得設計最狠的一個，今年五月才新增。流程是 CLAIM（陳述決策）➔ EXTRACT（抽出最小可審查的產物）➔ DOUBT（找一個全新上下文的審查者，用對抗式提示去否證它）➔ RECONCILE（收斂結論）➔ STOP。

關鍵差異在於「in-flight」——它不是等 PR 寫完才審查，而是在決策做到一半、修正成本還很低的時候就殺出來唱反調。而且審查者只拿得到「產物 + 契約」，完全看不到原本那個 agent 的推理過程和對話脈絡，避免被同一套盲點帶著走。甚至可以選擇丟到 Codex 或 Gemini CLI 做跨模型審查，執行時還特別交代要走檔案 + stdin，避免產物裡的反引號或特殊字元被 shell 誤判執行——這種安全細節有考慮到。

**3. code-review-and-quality —— 訂定了 review 分析，程式變動量還控在 100 行內**

/review 背後這個 Skill 訂了具體的問的分級（Nit／Optional／FYI），還規定每次改動盡量控制在 100 行左右(以前常遇到工程師一次改了幾百行後才做一次commit，做 review 時就很痛苦)，方便審查也方便回溯。這種把 Google 內部工程文化直接寫進 Skill 流程的作法，是逐步驟寫進 Process 裡的具體規則。

跟之前介紹過的 Superpowers 和 Matt Pocock skills 擺在一起看，三套的定位其實蠻分明的。Matt Pocock 那套走輕量路線，專注在「需求對齊到寫程式」這一小段，適合個人專案快速上手；Superpowers 更像是一套通用技能框架，不特別綁死在軟體開發生命週期上。agent-skills 明顯是衝著團隊、正式產品去的——它把 VERIFY、REVIEW、SHIP 這幾個後段階段也寫進去，甚至連上線後的 observability 都算進來，範圍是三套裡最完整的。體感上最大的差異不是誰的 Prompt 寫得比較漂亮，而是 agent-skills 多了 doubt-driven-development 這種「決策做到一半就找人唱反調」的機制，把原本要等 Code Review 才會抓到的方向錯誤，提早攔在修正成本還低的階段。原來這是 google 工程師的開發方經驗，可以整理成 promp 直接使用，真的是賺到，也學習到了很多。

資料來源：addyosmani/agent-skills、Addy Osmani
