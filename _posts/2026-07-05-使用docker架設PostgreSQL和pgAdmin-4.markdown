---
layout: post
title:  "使用docker架設PostgreSQL和pgAdmin 4"
date:   2026-07-05 10:40:00 +0000
categories: 轉載
original: https://medium.com/@itkh/%E4%BD%BF%E7%94%A8docker%E6%9E%B6%E8%A8%ADpostgresql%E5%92%8Cpgadmin-4-d82115b8d308
---

最近因為在研究新的服務有需要用到PostgreSQL，所以就來研究如何架設，因為是第一次使用，所以就先用了比較簡單的方式，之後再調整成docker-compose的方式做組合。

本次有個前題，是預設本機已經安裝好docker，並且對docker有基本的使用經驗。

首先，要先來安裝PostgreSQL，因為是用docker的方式，所以一如往常的先做 docker pull 。

    docker pull postgres

再來就是啟動 postgres 。

    docker run --name sqltutorial -e POSTGRES_PASSWORD=marviniscool -p 5432:5432 -d postgres

執行後在輸入「docker ps」，就會看到已經在運作中的 postgres，對就是這麼簡單。

接下來，是建立 pgAdmin 4。

第一，先把image pull下來。

    docker pull dpage/pgadmin4

一樣，執行docker run

    docker run --name pgadmin-container -p 5050:80 -e PGADMIN_DEFAULT_EMAIL=user@domain.com -e PGADMIN_DEFAULT_PASSWORD=catsarecool -d dpage/pgadmin4

依照我們在 docker run 寫的 port 為5050，就可以在瀏覽器上輸入「localhost:5050」進入到 pgAdmin 的介面。

帳號密碼輸入 user@domain.com 以及 catsarecool，就可以完成登入。

接下來我們要讓 pgAdmin 可以管理我們剛剛新增的 PostgreSQL，就是登入後點選 Add New Server。

其中General中的Name為必填，可以填自己好辨視的名稱。

接下來是Connetcion，其中Host可以從下面的指令查詢。

    docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' sqltutorial

再輸入Username：postgres，Password：marviniscool(這些都是在前先我們建立 PostgreSQL 資料庫時所設定的)

當你看到下面的畫面試就成功囉！

今天只是一個基礎的服務建立，之後如果有在進一步的研究(例如：組合成docker-compose、pgAdmin的操作、PostgreSQL的操作)，再來跟大家分享。
