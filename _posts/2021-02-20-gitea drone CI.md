---
layout: post
title: "gitea drone CI"
tagline: "搭建 gitea 仓库以及 drone CI 工具"
categories: gitea
author: "kekxv"
---

搭建 `gitea` 仓库以及 `drone` `CI` 工具。

因为在用的服务器是低性能`4G`的服务器，勉强能跑起来`gitlab`，再加上 `gitlab runner` 跑的比较艰难，所以研究研究 `gitea`与`drone`方案；
嗯，坑还不少。

按照文档以及他人的方案进行部署，发现基本上都能跑起来，但是，也只是能跑起来，如何用起来，并没有如何说明，所以在此处做一个记录。

1. 注意`gitea`、`drone`与`drone runner`之间的网络互通问题。
2. 当部署在同一个机器上的时候，注意 `DRONE_GITEA_SERVER` 、 `DRONE_SERVER_HOST` 、 `DRONE_HOST` 、 `DRONE_RPC_HOST` 这几个参数，很容易就出现`gitea` 回调 `drone` 失败，
   `drone` 回调 `gitea` 检查授权失败。
3. 如果外部端口与内部端口不一致，`nginx` 部分请加上对应的端口监听。
4. 定义了`networks`或者`drone-runner`设置了`DRONE_RUNNER_NETWORKS`，需要设置`external: true`，并且执行`docker network create 网络名`，否则拉取数据将会出错。
5. 如果所有都搭建完成，`drone`也能识别项目，并且能够启动流水线，但永远都在 `Pending`，很有可能是 `drone-runner` 未启动成功，或者 `drone-runner` `platform` 对不上；
   举个例子：`Apple Silicon` 的 `docker` 仅支持`platform:os: linux,arch: arm64`！！！
6. 创建完仓库之后，需要在`drone`激活仓库，否则，`推送`等`流水线`操作**没有任何反应**。
7. `drone`只能绑定到用户上，不能全局绑定。

将两个配置配好之后，执行启动命令，即可。

一些其他方面的参考：https://rabbit52.com/2018/06/drone-start-guide/

### 完成图

![drone完成图](/assets/imgs/20210221-drone.png)


### 启动命令

```shell
# 增加一个 docker 网络 这里为：gitea_net
docker network create gitea_net
# 生成路径为 gitea ： /user/settings/applications
# 在用户设置内的应用里面增加设置
DRONE_GITEA_CLIENT_ID=GITEA里面生成的ID DRONE_GITEA_CLIENT_SECRET=GITEA里面生成的SECRET DRONE_RPC_SECRET=自定义的DRONE_RPC_SECRET docker-compose up
```

### gitea_drone.conf 配置

```yaml
# gitea
server {
    listen 80;
    listen 10080;
    server_name gitea.kekxv.com;

    resolver 127.0.0.11 ipv6=off;
    set $upstream_endpoint http://gitea:3000;
    client_max_body_size 500M;

    location / {
        proxy_set_header            Host $http_host;
        proxy_pass                  $upstream_endpoint;
    }
}
# drone
server {
    listen 80;
    listen 10080;
    server_name drone.kekxv.com;
    resolver 127.0.0.11 ipv6=off;
    set $upstream_endpoint http://drone-server;
    location / {
        proxy_set_header            Host $http_host;
        proxy_pass                  $upstream_endpoint;
    }
}
```

### docker-compose.yml 配置

```yaml
# https://docs.gitea.io/en-us/install-with-docker/
# https://docs.drone.io/installation/gitea/multi-machine/
# https://docs.drone.io/administration/agents/linux-amd64/
# https://github.com/drone/drone-yaml-v1/blob/master/samples/5_parallel.yml
version: "3.7"

services:
  reverseproxy:
    image: nginx:alpine
    container_name: reverseproxy
    restart: always
    ports:
      - 10080:80
    volumes:
      - ./nginx/conf.d/:/etc/nginx/conf.d/
    networks:
      gitea_net:
        aliases:
          - gitea.kekxv.com
          - drone.kekxv.com
    depends_on:
      - gitea
      - drone-server
  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    environment:
      - APP_NAME=Gitea
      - RUN_MODE=prod
      - ROOT_URL=http://gitea.kekxv.com/
      - SECRET_KEY=${GITEA_SECRET_KEY}
    restart: always
    networks:
      - gitea_net
    volumes:
      - ./gitea:/data
  drone-server:
    image: drone/drone:latest
    container_name: drone-server
    restart: always
    environment:
      - DRONE_DEBUG=true
      - DRONE_ADMIN=kekxv
      - DRONE_USER_CREATE=username:kekxv,admin:true
      - DRONE_SERVER_PORT=:80
      - DRONE_DATABASE_DATASOURCE=/var/lib/drone/drone.sqlite
      - DRONE_DATABASE_DRIVER=sqlite3
      - DRONE_GIT_ALWAYS_AUTH=false
      - DRONE_GITEA_SERVER=http://gitea.kekxv.com:10080
      - DRONE_GITEA_CLIENT_ID=${DRONE_GITEA_CLIENT_ID}
      - DRONE_GITEA_CLIENT_SECRET=${DRONE_GITEA_CLIENT_SECRET}
      - DRONE_RPC_SECRET=${DRONE_RPC_SECRET}
      - DRONE_SERVER_HOST=drone.kekxv.com:10080
      - DRONE_HOST=http://drone.kekxv.com:10080
      - DRONE_SERVER_PROTO=http
      - DRONE_TLS_AUTOCERT=false
      - DRONE_AGENTS_ENABLED=true
    volumes:
      - ./drone/data:/data
      - ./drone/drone:/var/lib/drone
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - gitea_net
  drone-runner:
    image: drone/drone-runner-docker:latest
    restart: always
    environment:
      - DRONE_RPC_HOST=drone.kekxv.com
      - DRONE_RPC_PROTO=http
      - DRONE_RPC_SECRET=${DRONE_RPC_SECRET}
      - DRONE_RUNNER_CAPACITY=2
      - DRONE_RUNNER_NAME=DRONE_RUNNER_NAME
      - DRONE_LOGS_TRACE=true
      - DRONE_LOGS_DEBUG=true
      - DRONE_RUNNER_NETWORKS=gitea_net
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - gitea_net
    deploy:
      mode: replicated
      replicas: 1
    depends_on:
      - reverseproxy
      - drone-server

networks:
  gitea_net:
    external: true
```
