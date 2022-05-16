---
date: 2022-05-16
layout: post
title: 用Gitlab Runner来打包并上传Harbor
description: Gitlab Runner with docker build command
categories:
- Blog
tags:
- Gitlab Docker

---

https://docs.gitlab.com/ee/ci/docker/using_docker_build.html


# docker-compose.yaml

```
version: '3.6'
services:
  gitlab-runner:
    image: 'gitlab/gitlab-runner:latest'
    restart: always
    volumes:
      - ./config:/etc/gitlab-runner
      - /data/gitlab-runner:/home/gitlab-runner
```

# ./config/config.toml

/root/.docker/config.json 里放docker hub的授权token

```
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "Dev1 Docker Runner"
  url = "https://gitlab.xxxx.com/"
  token = "xxxxxxxxxxxx"
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "docker:20.10.15"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/home/hugo/.docker/config.json:/root/.docker/config.json:ro","/var/run/docker.sock:/var/run/docker.sock", "/data/gitlab-runner/cache:/home/gitlab-runner/cache"]
    shm_size = 0
    cache_dir = "/home/gitlab-runner/cache"
```

# .gitlab-ci.yml  
```
workflow:
before_script:
  - docker info

build-job:
  stage: build
  script:
    - docker build -t test:latest .
    - docker push test:latest
```