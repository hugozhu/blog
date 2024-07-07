---
date: 2024-07-10
layout: post
title: 使用acme docker来自动化3个月需要更新的免费域名证书
description: Use acme docker to renew ssls on apisix
categories:
- Blog
tags:
- apisix, docker, acme.sh, dnspod

---

{:toc}

# 问题和解决方案
免费域名证书需要三个月更新一次。
解决方案：apisix, docker, acme.sh, dnspod


# Apisix
https://apisix.org
Apisix是优秀的开源网关，更新证书不需要重启服务，可以作为所有服务的网关

# Acme Docker
https://github.com/hugozhu/acme_docker/blob/main/README.md
项目已配置好使用docker来更新证书并生成apisix的json

docker-compose.yaml
```
version: "3"
services:
  acme:
    image: neilpang/acme.sh
    container_name: acme
    #restart: always
    command: daemon
    env_file:
     - .env
    volumes:
      - ./cert:/acme.sh
```

# 重新申请免费证书
```
docker-compose up -d
docker exec acme acme.sh --set-default-ca --server letsencrypt
docker exec acme acme.sh --issue --dns dns_dp  -d hugozhu.site -d *.hugozhu.site -d *.go.hugozhu.site
docker-compose down

cat hugozhu.site.json
```

# apisix上更新证书
```
data=$( cat "hugozhu.site.json" )
curl 'http://127.0.0.1:9180/apisix/admin/ssls/1' \
        -H "X-API-KEY: $API_KEY" -X PUT -d "$data"`
```