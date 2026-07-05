---
layout: post
title:  "Introduction API gateway- KrakenD"
date:   2023-10-16 03:59:52 +0000
categories: 轉載
original: https://medium.com/@itkh/api-gateway-krakend-d1ded092a0e3
---

近年來，組識對於 API 的需求也是日益增加，並且因為公司內部本身就有許多對外或對內的系統在營運，同時也有許多外部的合作夥伴。而因為筆者所任職的公司，非常適合採用微服務(Micro Service)的系統架構模式，而筆者所屬的單位則也是公司產品線中的其中之一(同時擁有backend及frontend)，所以就在最近開始推動系統相關的結構調整。

由於目前在舊有系統上都已經採用API的串接模式來與內外部合作單位進行資料上的傳遞。而目前可開始調整API gateway 做為第一步。

本篇先行介紹「KrakenD」做為說明：

使用KrakenD時可使用docker進行安裝，並且主要是需要編輯的檔案為 krakend.json

    docker pull devopsfaith/krakend
    docker run -p 8080:8080 -v $PWD:/etc/krakend/ devopsfaith/krakend

第二步，編輯 krakend.json 即可

    {
        "version": 3,
        "name": "My lovely gateway",
        "port": 8080,
        "cache_ttl": "3600s",
        "timeout": "3s",
        "extra_config": {
          "telemetry/logging": {
            "level":  "DEBUG",
            "prefix": "[KRAKEND]",
            "syslog": false,
            "stdout": true
          },
          "telemetry/metrics": {
            "collection_time": "60s",
            "proxy_disabled": false,
            "router_disabled": false,
            "backend_disabled": false,
            "endpoint_disabled": false,
            "listen_address": ":8090"
          },
          "security/cors": {
            "allow_origins": [ "http://192.168.99.100:3000", "http://localhost:3000" ],
            "allow_methods": [ "POST", "GET" ],
            "allow_headers": [ "Origin", "Authorization", "Content-Type" ],
            "expose_headers": [ "Content-Length" ],
            "max_age": "12h"
          }
        },
        "endpoints": [
            {
                "endpoint": "/supu",
                "method": "GET",
                "input_headers": ["Authorization", "Content-Type"],
                "backend": [
                    {
                        "host": [
                            "http://127.0.0.1:8000"
                        ],
                        "url_pattern": "/__debug/supu",
                        "extra_config": {
                            "qos/ratelimit/proxy": {
                                "max_rate": 2,
                                "capacity": 2
                            },
                            "qos/circuit-breaker": {
                                "interval": 60,
                                "timeout": 10,
                                "max_errors": 1
                            }
                        }
                    }
                ]
            },
            {
                "endpoint": "/private/{user}",
                "backend": [
                    {
                        "host": [ "http://api.github.com" ],
                        "url_pattern": "/",
                        "allow": [
                            "authorizations_url",
                            "code_search_url"
                        ]
                    }
                ],
                "extra_config": {
                    "auth/validator": {
                        "alg": "RS256",
                        "audience": ["http://api.example.com"],
                        "roles_key": "http://api.example.com/custom/roles",
                        "roles": [ "user", "admin" ],
                        "jwk_url": "https://albert-test.auth0.com/.well-known/jwks.json"
                    }
                }
            },
            {
                "endpoint": "/show/{id}",
                "backend": [
                    {
                        "host": [
                            "http://showrss.info/"
                        ],
                        "url_pattern": "/user/schedule/{id}.rss",
                        "encoding": "rss",
                        "group": "schedule",
                        "allow": ["items", "title"],
                        "extra_config": {
                            "qos/ratelimit/proxy": {
                                "max_rate": 1,
                                "capacity": 1
                            },
                            "qos/circuit-breaker": {
                                "interval": 60,
                                "timeout": 10,
                                "max_errors": 1
                            }
                        }
                    },
                    {
                        "host": [
                            "http://showrss.info/"
                        ],
                        "url_pattern": "/user/{id}.rss",
                        "encoding": "rss",
                        "group": "available",
                        "allow": ["items", "title"],
                        "extra_config": {
                            "qos/ratelimit/proxy": {
                                "max_rate": 2,
                                "capacity": 2
                            },
                            "qos/circuit-breaker": {
                                "interval": 60,
                                "timeout": 10,
                                "max_errors": 1
                            }
                        }
                    }
                ],
                "extra_config": {
                    "qos/ratelimit/router": {
                        "max_rate": 50,
                        "client_max_rate": 5,
                        "strategy": "ip"
                    }
                }
            }
        ]
    }

krakend.json 中我覺得最需要注意的內容是 extra_config、endpoints，相關說明可以參考 [https://www.krakend.io/docs/overview/](https://www.krakend.io/docs/overview/)

可以使用以下網址確認服務是否正常執行中

    http://localhost:8088/__health

若回傳以下表示正常運作中

    {"agents":{},"now":"2023-10-12 07:43:18.460924514 +0000 UTC m=+6.115917113","status":"ok"}

另外要編輯 krakend.json 可以使用下面網址進行產生，不需要全部使用手刻。

https://designer.krakend.io/#!/

目前找的這套 KrakenD 在建立已經變的相當易上手，基本把環境執行起來可能不用10分鐘，在 krakend.json 的編輯及理解也不難，市面上也有其他 API gateway的服務(另外也有研究了gravitee.io，再跟大家分享)，若大家所在的系統服務若是越漸複雜，也可以考慮採用API gateway做為解決方案。

因為 API gateway 在整體系統架構中負責 API 的管控，因此 API 來源的驗證可以再結合額外的驗證系統(OAuth 2等)做資訊傳遞上安全的把關，而目前所看到的 API gateway也基本上都有提供基本內建或是外掛的服務。

未來只要 API gateway 的架構建立，只要是 frontend 有增加的需求時，只需要在佈屬新的 docker container並且設定新的 krakend.json 即可。反之，若frontend 消滅時則只需要刪除該 container 即可，在維護上的成本可以降低。若 frontend 是組織內部的單位時，更可以採用 BFF (Backend for Frontend)的方式讓 frontend 自行佈屬所需要的 API gateway，如此一來在維護成本及開發效率上會有很大的幫助。

註：在此backend及frontend非工程師職能，為在資訊系統環境下的服務角色。

KarkenD官方網站：https://www.krakend.io/
