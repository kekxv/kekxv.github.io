---
layout: post
title: "gitlab pages jekyll 自动部署"
categories: gitlab
author: "kekxv"
---

本篇简单的介绍一下如何使用`gitlab`搭建`jekyll`静态`blog`。感兴趣的可以看看。

这里使用的是当前 blog 模板，各位可以做个参考。

#### 使用到的工具

- `ruby`
- `gcc`、`g++`、`make`(部分包需要本地编译)
- `bundler` 
- `jekyll` 

为了编译的速度以及节省时间，我使用了 `node:10-alpine`的`docker`镜像。

在开始的时候，我们需要将一些必需文件进行安装。

为了安装速度，我们可以选择使用国内源，~~比如淘宝源码~~ 改用**清华大学源**，**靠谱**：

`Ruby`修改国内源方式为在`Gemfile`文件头部增加清华源地址：

```shell script
# source 'https://rubygems.org'
source 'https://mirrors.tuna.tsinghua.edu.cn/rubygems/'
```

`apk` 包管理同样使用**清华源**修改源方式为：

```shell script
sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories
```

(我们可以将该命令放置在`CI`文件里)

#### `.gitlab-ci.yml`

CI 文件：

```yaml
image: node:10-alpine # use nodejs v10 LTS
cache:
  key: ${CI_COMMIT_REF_NAME}
  paths:
    - node_modules/

before_script:
  - sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories
  - apk add ruby ruby-dev openssl-dev gcc g++ make zlib-dev git
  - gem install bundler

pages:
  script:
    - bundle install
    - bundle exec jekyll build
  artifacts:
    paths:
      - _site
  only:
    - master
```

为了成功编译出静态`blog`，我们先在`before_script`进行必要的一些工具的安装。

在 `pages` 项目里，`artifacts.paths`值`_site`为当前项目生成的静态blog目录，部分模板生成目录为`public`。请根据实际情况进行修改。


#### 参考链接：

- [将 Hexo 部署到 GitLab Pages](https://hexo.io/zh-cn/docs/gitlab-pages.html)
- [清华镜像源地址:https://mirrors.tuna.tsinghua.edu.cn](https://mirrors.tuna.tsinghua.edu.cn)

