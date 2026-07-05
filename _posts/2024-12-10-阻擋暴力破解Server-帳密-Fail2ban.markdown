---
layout: post
title:  "阻擋暴力破解Server 帳密-Fail2ban"
date:   2024-12-10 09:39:34 +0000
categories: 轉載
original: https://medium.com/@itkh/%E9%98%BB%E6%93%8B%E6%9A%B4%E5%8A%9B%E7%A0%B4%E8%A7%A3server-%E5%B8%B3%E5%AF%86-fail2ban-8e2008426978
---

最近在針對作者本人維護的 Server ( Linux )做一些系統 Log 查詢的時候發現了有未知的網址在不斷的嘗試要透過 SSH 登入 Server，雖然沒有立即的問題，但是也是資安的議題，所以就花了一些時間來處理。

剛好同組的同仁有相關的經驗，所以就推薦了我使用 Fail2ban 這個 Linux 套件，來決解這種惡意的嘗試。

因此本次就來跟大家分享 Fail2ban 的基本心得跟操作

![](https://cdn-images-1.medium.com/max/800/0*OIeJvUjlfQYVGURd.jpeg)

首先，先來介紹安裝的方式，非常的簡單，大概只要三個步驟：

  1. 安裝


    
    
    $ apt-get install fail2ban

2\. 設定 config

一開始不會有 jail.local，需要另外新增
    
    
    $ vim /etc/fail2ban/jail.local

其中基本的內容如下
    
    
    [sshd]  
    enabled  = true  
    port     = ssh  
    filter   = sshd  
    logpath  = /var/log/auth.log // log要存到哪裡  
      
    findtime = 6000 // 多久的時間內  
    maxretry = 3 // 可以接受錯誤幾次  
    bantime  = 86400// 會被ban多久

3\. 重啟服務
    
    
    $ service fail2ban restart

就是這麼簡單。

在重啟後可以到 fail2ban.log 看到 fail2ban 的 log
    
    
    $ cat /var/log/fail2ban.log

log 內容如下圖

![](https://cdn-images-1.medium.com/max/1024/1*HQ0WjGebzRsB6cbR1fxDFA.png)

其中可以看到服務重新啟動的訊息

found：系統發現 sshd 的 IP

Ban：哪個 ip 被 ban 掉了

Restore Ban：原本被 Ban 的 IP 因為服務重啟而重新再 Ban

另外還會有 Unban：解 Ban

如果要看當下 Fial2ban 的狀態可以輸入以下
    
    
    $ fail2ban-client status  
    $ fail2ban-client status sshd // 如果要單看指定服務的狀態

會看到因為哪個服務而 Ban 的數量

![](https://cdn-images-1.medium.com/max/183/1*2Xsw_rJ_t03IR8zOs1-erA.png)

另外，Fail2ban 還有很多 action (類似 hook)可以使用，都放 /etc/fail2ban/action.d， 這些都可以自己去研究。

### 結語

因為剛好最近在整理 Server 的 Error Log，就發現 Server 不尋常的地方，所以立馬就進行了處理。

Fail2ban 不僅安裝簡單，在阻擋的對象也有各種方式(ex：sshd、openvpn、sendmail、bind…etc)，同時也有許多 action 可以使用，作者本身也還沒有都摸透，待之後有更深的了解再跟大家介紹。

今天就先到這邊，謝謝大家。
