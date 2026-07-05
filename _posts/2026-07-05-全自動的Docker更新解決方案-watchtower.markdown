---
layout: post
title:  "全自動的Docker更新解決方案-watchtower"
date:   2026-07-05 09:08:03 +0000
categories: 轉載
original: https://medium.com/@itkh/%E5%85%A8%E8%87%AA%E5%8B%95%E7%9A%84docker%E6%9B%B4%E6%96%B0%E8%A7%A3%E6%B1%BA%E6%96%B9%E6%A1%88-29727fe60d65
---

最近接手了一個專案，所以就趁著這個機會學習他人的經驗來擴展自己的知識。也剛好在這幾天也在處理該Server上的Docker設定，就看到Server上使用這個服務，發現很方便也很實用，就來跟大家分享。

這次分享的是 watchtower，這也是一個開源的專案，也是蠻多人使用，我看了一下這個專案也發展了好一陣子。

這個軟體他本來是直接包成Docker，並且使用Go語言進行開發。這個軟體主要是會定時的掃描Server上目前在執行的Docker有沒有新的版本，如果有新的就會主動進行更新，真的是非常的方便，由其是使用Docker開發的專案，在更新上也是非常的便利。

在軟體安裝及執行上也是非常的簡單，只需要幾個步驟就可以完成。

第一，起手示就是「docker pull」

    docker pull containrrr/watchtower

接下來執行

    docker run --detach \
     --name watchtower \
     --volume /var/run/docker.sock:/var/run/docker.sock \
     containrrr/watchtower

這樣就可以完成了，只要執行成功，就可以從「docker logs」就可以看到掃描的狀況，如下圖所示。

像是我們看到如果watchtower有發現新的images時，就會自動更新，並且會把舊的image進行刪除，如此一來也不會因為一直抓新版本而佔用過多的空間。

再來我們說明一下可使用的參數：

  * --cleanup：更新新image後刪除舊image
  * -i 3600：更新頻率
  * -s "0 0 3 * * *"：指定更新時間(寫法同crontab)
  * --run-once：只更新一次(手動更新)



參數最後可填寫指定更新的的container名稱

以上是最基本的參數設定，有需要還可以到官網的文件

### 結語

上面的範例是使用了一般公開的image，而自己在開發時所會打包的docker可以透過CICD先上傳到私有的docker registry(Amazon Elastic Container Registry之類的)，這樣一來watchtower也可以自動更新私有的image。

另外如果image有更新或取得新image有遇到問題時也可以透過notify的方式通知到Email、Slack、Teams或Gotify。

在這個docker化的世界中，能有自個docker自動更新的服務也真是方便，因為最近在開發go，規模也不需要到使用k8s，就會需要用到這樣的服務，某種程度上也算是自動化的CICD的一環。

今天就分享到這邊，謝謝大家。
