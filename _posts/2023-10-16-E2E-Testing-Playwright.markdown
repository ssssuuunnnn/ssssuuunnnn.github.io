---
layout: post
title:  "E2E Testing — Playwright"
date:   2023-10-16 06:13:55 +0000
categories: 轉載
original: https://medium.com/@itkh/e2e-testing-playwright-b514eda51d70
---

最近因為系統的流量越來越大，使用人數也是日益增加，前陣子團隊中的前端同仁也有想法將前端框架做升版，趁之前疫情時做程式重構，因為前端同仁就很快的進行，但是這時候就遇到了一個問題，沒有辦法確定重構程的前端程式的每個頁面是否正常，在以前的測試都是用手動測試，筆者覺得很沒效率，因此就趁這次機會導入自動化測試的做法，所以在最近就開始研究了幾個前端測試的服務(playwright, cypress)，本次就針對 playwright 做一個簡單的介紹，之後再介紹 cypress。

這套測試軟體服務操作起來也相當簡單，做到一個基本的測試，筆者大概花了一個下午就可以有將基本的頁面做簡測。

playwright 是用 node.js 所撰寫，因此起手示都是 npm 相關的操作。

    npm init playwright@latest

對，這樣就裝好了，就是這麼簡單，安裝完會產生一個資料夾，資料夾的內容如下。

  * playwright.config.ts => 在執行測試時的相關設定檔
  * package.json
  * package-lock.json
  * tests/ => 需要執行的腳本放置於此
    * example.spec.ts
  * tests-examples/ => 官方提供的範例檔
    * demo-todo-app.spec.ts



本次主要是操作 tests/example.spec.ts，需要測試的腳本也是寫在這邊。
相關的語法都在這裡 https://playwright.dev/docs/api/class-playwright

再來說明基本的幾個 cmd 常用指令

執行測試計劃，--headed為是否顯示執行視窗

    npx playwright test [--headed]

瀏覽測試報告

    npx playwright show-report

測試報告的內容會從瀏覽器呈現，大概會長這個樣子。

基本的操作暫時介紹到此。

這次使用的這套 Playwright，目前用起來算是一個方便的 e2e testing 的軟體，他可以同時自動進行多個瀏覽器的測試，各瀏覽器的測試結果也蠻清楚的，錯誤訊息也可以蠻快知道哪裡有問題，這套軟體也可以加入到CI/CD中，成為持續佈屬的一個環節。對於如果有需要進行 e2e testing 的人可以做為參考。

Playwright官網：https://playwright.dev/
