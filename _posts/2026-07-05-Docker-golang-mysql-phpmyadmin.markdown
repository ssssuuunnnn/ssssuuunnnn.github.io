---
layout: post
title:  "[Docker]golang+mysql+phpmyadmin"
date:   2026-07-05 09:45:03 +0000
categories: 轉載
original: https://medium.com/@itkh/docker-golang-mysql-phpmyadmin-99c3cc784344
---

最近接手一個以 golang 做為開發語言，並且使用 gin 框架開發，資料庫使用mysql 的專案。

因為這也是我第一次開發 golang，因為就先建立了一個基本的 Docker 開發環境，以下的 Dockerfile 及 docker-compose.yml都可以直接使用，放在這裡供大家參考。

Dockerfile

    FROM golang:latest

    WORKDIR /app

    COPY . .

    # RUN go mod init myapp
    RUN go get -u github.com/gin-gonic/gin
    RUN go get -u gorm.io/gorm
    RUN go get -u gorm.io/driver/mysql

    CMD ["go", "run", "main.go"]

docker-compose.yaml

    version: '3'

    services:
      app:
        build: .
        ports:
          - "8080:8080"
        depends_on:
          - db

      db:
        image: mysql:latest
        environment:
          MYSQL_ROOT_PASSWORD: rootpassword
          MYSQL_DATABASE: myapp
          MYSQL_USER: user
          MYSQL_PASSWORD: password
        volumes:
          - mysql_data:/var/lib/mysql

      phpmyadmin:
        image: phpmyadmin/phpmyadmin
        ports:
          - "8081:80"
        environment:
          PMA_HOST: db
          MYSQL_ROOT_PASSWORD: rootpassword

    volumes:
      mysql_data:

直接執行

    docker-compose up --build // 執行即可

目前這個 docker-compose.yaml 會自動建立資料庫及管理介面的基本環境，可以直接從 0 開始。

完整的程式放在 github [https://github.com/ssssuuunnnn/dolang-jin-docker](https://github.com/ssssuuunnnn/dolang-jin-docker)
