---
layout: post
title: "gitlab仓库yml配置文件搭建"
tagline: "gitlab仓库yml配置文件搭建"
categories: gitlab
author: "kekxv"
---

之前写篇 `gitlab` 仓库搭建步骤教程，最近升级的时候发现命令实在是太长了，于是决定使用 `docker-compose.yml` 文件进行配置搭建。

[关于上一篇 gitlab 仓库搭建步骤教程可以点击这里](https://kekxv.github.io/gitlab/2020/07/06/gitlab%E4%BB%93%E5%BA%93%E6%90%AD%E5%BB%BA.html)

```yaml
version: "3"
services:
  gitlab:
    image: gitlab/gitlab-ce
    container_name: gitlab
    restart: always
    hostname: 'hostname' # 主机名，最好是实际的访问地址
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url '实际的访问地址'
        nginx['listen_https'] = false
        nginx['listen_port'] = 80
        gitlab_rails['gitlab_email_enabled'] = true
        gitlab_rails['gitlab_email_from'] = '邮箱账号'
        gitlab_rails['gitlab_email_display_name'] = '发送昵称'
        gitlab_rails['smtp_enable'] = true
        gitlab_rails['smtp_address'] = "smtp服务器地址"
        gitlab_rails['smtp_port'] = 465
        gitlab_rails['smtp_user_name'] = "邮箱账号"
        gitlab_rails['smtp_password'] = "邮箱密码"
        gitlab_rails['smtp_domain'] = "smtp 邮箱域名"
        gitlab_rails['smtp_authentication'] = "login"
        gitlab_rails['smtp_enable_starttls_auto'] = true
        gitlab_rails['smtp_tls'] = true
    ports:
      - '10080:80' # 映射80端口
      - '10443:443' # 映射443端口
    volumes:
      - '/docker_data/gitlab/config:/etc/gitlab' # 映射目录
      - '/docker_data/gitlab/logs:/var/log/gitlab' # 映射目录
      - '/docker_data/gitlab/data:/var/opt/gitlab' # 映射目录
    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "10"
```

1. `image` 为使用的镜像名称
1. `container_name` 为容器名
1. `restart` 是否自动重启
1. `hostname` 主机名，最好是实际的访问地址
1. `GITLAB_OMNIBUS_CONFIG` 的配置参数，为 gitlab.rb 参数
1. `ports` 端口映射列表
1. `volumes` 目录映射列表
1. `logging` 日志记录

更新gitlab版本

运行命令 `docker-compose pull` && `docker-compose up -d`

下载 [docker-compose.yml](https://kekxv.github.io/assets/docker.yml/gitlab.docker-compose.yml)

