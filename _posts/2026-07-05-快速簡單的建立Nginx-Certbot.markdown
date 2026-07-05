---
layout: post
title:  "Easy to Build Nginx+Certbot"
date:   2026-07-05 10:15:00 +0000
categories: 轉載
original: https://medium.com/@itkh/%E5%BF%AB%E9%80%9F%E7%B0%A1%E5%96%AE%E7%9A%84%E5%BB%BA%E7%AB%8Bnginx-certbot-c289fe6f93bd
---

因為最近在建立新的系統，這次就來跟大家分享在本次建立系統時所發現對我來說的新玩意。

這次要跟大家分享的就是關於「docker-nginx-certbot」

先提供這個專案的github
[https://github.com/JonasAlfredsson/docker-nginx-certbot](https://github.com/JonasAlfredsson/docker-nginx-certbot)

Docker Hub
[https://hub.docker.com/r/jonasal/nginx-certbot](https://hub.docker.com/r/jonasal/nginx-certbot)

這個專案從名稱一看就知道是直接包成docker，馬上就知道啟動也是非常的簡單快速。

首先建在專案的目錄建立一檔案「docker-compose.yml」

    version: '3'

    services:
      nginx:
        image: jonasal/nginx-certbot:3
        restart: unless-stopped
        environment:
          - CERTBOT_EMAIL=your-email@example.com
        ports:
          - 80:80
          - 443:443
        volumes:
          - ./letsencrypt:/etc/letsencrypt
          - ./user_conf.d:/etc/nginx/user_conf.d

其中 your-email@example.com 需要改成你自己的email，這是需要在做網址認證的時候提供的。

另外需要說明的還有下面二個資料夾，

letsencypt：儲存相關SSL憑證的地方。

user_conf.d： 你的nginx config檔案放在這。

因此既然知道user_conf.d是放nginx config，那接下來就要開一個新檔案放你所想要認證的URL設定，設定當的內容如下

    server {
        # Listen to port 443 on both IPv4 and IPv6.
        listen 443 ssl default_server reuseport;
        listen [::]:443 ssl default_server reuseport;

        # Domain names this server should respond to.
        server_name {你的網址};

        # Load the certificate files.
        ssl_certificate         /etc/letsencrypt/live/{自己命名}/fullchain.pem;
        ssl_certificate_key     /etc/letsencrypt/live/{自己命名}/privkey.pem;
        ssl_trusted_certificate /etc/letsencrypt/live/{自己命名}/chain.pem;

        # Load the Diffie-Hellman parameter.
        ssl_dhparam /etc/letsencrypt/dhparams/dhparam.pem;

        return 200 'Let\'s Encrypt certificate successfully installed!';
        add_header Content-Type text/plain;
    }

接下來依照docker的做法，很簡單的只要執行下面指令。

    sudo docker compose up -d //-d 為背景執行

接下來大概要過個3~5分鐘就可以在網頁上輸入網址(使用https)，就會看到頁面顯示「Let's Encrypt certificate successfully installed!」

看到這個就表示成功了。

最後可以再到 letsencypt 的資料夾看 live 裡面會有相關的金鑰憑證。

這個image就是如此的方便，在github上也是有蠻多星星的。

由於 letsencrypt 所提供的憑證每次的效期為 90 天，「docker-nginx-certbot」會自動進行 renew的動作，所以也不用擔心過期。

這次就先分享到這邊，謝謝大家。

### 20240604 後記

如果有人需要手動 renew，這裡也提供指令可供使用。

    $ docker exec -it {docker name} /scripts/run_certbot.sh force
