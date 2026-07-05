---
layout: post
title:  "Docker Image 資安掃描工具-grype"
date:   2026-07-05 09:30:03 +0000
categories: 轉載
original: https://medium.com/@itkh/docker-image-%E8%B3%87%E5%AE%89%E6%8E%83%E6%8F%8F%E5%B7%A5%E5%85%B7-grype-190d80cedadf
---

在這個軟體開發環境都開始用Docker的時代，許多專案在開始前都會從Docker開始建立。同時DevSecOps的觀念越來越盛行的需求下，Docker的資安也成為了很重要的一環。本次就來介紹Docker Image 資安掃描工具「grype」。

這個軟體可以在各個平台上執行，是用 golang 開發的，並且它也是一個開源的軟體，安裝簡單，使用上也容易，掃描的報告也非常的清楚。

本次將介紹如何把 grype 安裝在mac。

因為 grype 有支援 homebrew 的安裝，所以可以直接使用。

    $brew install grype

安裝結果如下

接下來可以透過--version來確定是否安裝完成。

    $grype --version // grype 0.82.0

因為資安的相關資訊都是預先更新至本機grype的資料庫中，需要透過資料庫的資料做比對，因此會需要更新資料庫，更新的話可以用「db update」可以更新，並且也可以透過「db status」查看本機資料庫的狀態。

    $grype db update // 更新資料庫
    $grype db status // 查看資料庫狀態

指令執行結果如下

最後就可以準備檢查 docker image 了，而檢查指令也很簡單「grype {docker image name}」

    $grype {docker image name} // ex:grype golang:alpine3.16

執行結果如下

其中就會看到這個image裡面包了哪些東西，有什麼樣的安全問題。

其中

  * INSTALLED 目前安裝的版本
  * FIXED-IN 需要更新的版本
  * TYPE是包的檔案類型包含但不限於以下幾種：
    * deb (Debian軟體包)
    * binary (二進制文件)
    * apk (Alpine Linux軟體包)
    * go-module (Go语言模組)
  * VULNERABILITY 是哪一條資安通報
  * SEVERITY 是嚴重程度



在使用Docker hub上已經包好的image時，最好都要先進行資安掃描，確保其安全性，若是必要也是可以將這個檢查結合到CICD的一環，使其安全性可以更加的全面。另外這也可以成為SBOM(Software Bill of Materials)的資料來源之一。

今天就分享到這邊，謝謝大家。

註：

github [https://github.com/anchore/grype](https://github.com/anchore/grype)

homebrew [https://formulae.brew.sh/formula/grype](https://formulae.brew.sh/formula/grype)
