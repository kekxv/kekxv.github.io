---
layout: post
title: "Guacamole 使用"
tagline: "Guacamole 使用"
categories: Guacamole
#image: /thumbnail-mobile.png
author: "kekxv"
#meta: "Springfield"
---

使用 docker-compose 使用和管理 Guacamole。

下载 [Guacamole.tar.gz](/assets/file/Guacamole.tar.gz) ：

```bash 
$ wget https://kekxv.github.io/assets/file/Guacamole.tar.gz
$ tar -zxvf Guacamole.tar.gz
$ cd Guacamole
$ # 可自行修改 docker-compose.yml 配置
$ docker-compose pull
$ # 启动
$ docker-compose up -d
```

访问 `Guacamole` ；`http[s]://域名:18080/guacamole/`;

![img.png](/assets/imgs/guacamole.login/img.png)

默认账号密码为 ： `guacadmin`/`guacadmin`。

使用 `Ctrl`+`Alt`+`Shift` (`Control`+`option`+`Shift` )可以呼出菜单，如果使用的`ssh`协议，能够在菜单内进行上传下载操作。

附 `docker-compose.yml` 配置：

```yaml
version: '3'
services:
  guacd:
    image: guacamole/guacd
    container_name: guacd
    restart: always
    networks:
      guacamole:
        aliases:
          - guacd

  mysql:
    image: mysql/mysql-server:5.7
    container_name: mysql
    restart: always
    # ports:
    # - 3306:3306
    volumes:
      - ./mysql/:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=mysql123
    networks:
      guacamole:
        aliases:
          - mysql

  guacamole:
    image: guacamole/guacamole
    container_name: guacamole
    restart: always
    depends_on:
      - guacd
      - mysql
    networks:
      guacamole:
        aliases:
          - guacamole
    environment:
      - MYSQL_HOSTNAME=mysql
      - MYSQL_PORT=3306
      - MYSQL_DATABASE=guacamole
      - MYSQL_USER=guacamole
      - MYSQL_PASSWORD=guacamole123
      - GUACD_HOSTNAME=guacd
      - GUACD_PORT=4822
    ports:
      - 18080:8080
    links:
      - guacd
      - mysql
networks:
  guacamole:
```

