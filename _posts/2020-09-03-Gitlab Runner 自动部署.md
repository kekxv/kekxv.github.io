---
layout: post
title: "Gitlab Runner 自动部署"
tagline: "Gitlab Runner 自动部署"
categories: Gitlab Runner
author: "kekxv"
---

`Vue` 之类的开发，都需要编译，然后生成编译后的文件，然后打包发布，往往需要好几个步骤，那么我们是否可以让它自动进行，不需要我们人为参与呢？

当然是可以的，借助 `CI/CD` 工具，我们可以实现自动编译，自动部署。免去人为的干预。

这里使用 `gitlab` 仓库与 `gitlab runner`，搭建方式可以参考 [gitlab仓库搭建](https://kekxv.github.io/gitlab/2020/07/06/gitlab%E4%BB%93%E5%BA%93%E6%90%AD%E5%BB%BA.html)

我们需要将`gitlab` 仓库与 `gitlab runner`配好，然后在项目的设置里面：**设置=>CI/CD=>变量** 里添加变量，例如这里需要的 `ssh 私钥`、`用户名和服务器地址`、`服务器路径`:

分别配置 ：

1. SSH_PRIVATE_KEY (ssh 私钥)
1. SERVER_USER_HOST (用户名和服务器地址 例如：user@localhost)
1. SERVER_MASTER_PATH (服务器路径 例如：/var/www/html/vue)

**注意：千万千万不要写错变量名**

然后为 项目添加 `CI/CD` 配置:

`.gitlab-ci.yml`

```yml
image: node:10-alpine

stages:
  - build
  - deploy

cache:
  key: ${CI_COMMIT_REF_NAME}
  paths:
    - node_modules/

job-build:
  stage: build
  only:
    - master
    - dev

  script:
    - npm set registry https://registry.npm.taobao.org
    - npm install --progress=false
    - npm run build
  artifacts:
    expire_in: 1 week
    paths:
      - dist
image: node:10-alpine

stages:
  - build

cache:
  key: ${CI_COMMIT_REF_NAME}
  paths:
    - node_modules/

job-deploy:
  stage: deploy
  only:
    - master

  script:
    - echo "http://mirrors.aliyun.com/alpine/v3.9/main/" > /etc/apk/repositories
    - apk add --no-cache rsync openssh
    - mkdir -p ~/.ssh
    - echo "$SSH_PRIVATE_KEY" >> ~/.ssh/id_dsa
    - chmod 600 ~/.ssh/id_dsa
    - echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config
    - # rsync -rav --delete dist/ "$SERVER_USER_HOST:$SERVER_MASTER_PATH" # 把dist/下的所有文件拷贝到$SERVER_MASTER_PATH的路径下，会把原来存在的都删除注意别写错路径
    - rsync -rav dist/ "$SERVER_USER_HOST:$SERVER_MASTER_PATH"   # 将删除参数去掉，默认不删除

```

接下来就可以推送，然后在 gitlab 里面查看进度。

