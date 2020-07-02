---
layout: post
title: "github actions CI 自动编译测试"
tagline: "github actions CI 自动编译测试"
categories: CI
author: "kekxv"
---

最近在整一个复合类的项目（`cmake`项目），期望是跨平台，所以需要在各个平台上编译测试。

但是每次改完代码还要开个虚拟机编译测试，还挺麻烦的，于是试了下 github 的 actions 自动 CI 工具，还挺好的，目前已为项目自动编译 `Windows`、`Linux`、`MacOS` 平台。

cmake : ![C/C++ CI](https://github.com/ClangTools/clangTools/workflows/C/C++%20CI/badge.svg)

那么如何使用这个那么好的功能呢？

其实很简单：

1. 创建目录 `.github/workflows`
2. 创建一个 `yml` 后缀格式文件，例如 `ccpp.yml`。
3. 然后将下面内容输入到创建的`yml`格式文件里面：

```yml
name: C/C++ CI

on:
  push:
    branches: [ master,dev ]
  pull_request:
    branches: [ master,dev ]

jobs:
  ubuntu-build:

    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v1
      - name: configure
        run: mkdir build-ubuntu && cd build-ubuntu && cmake -DCMAKE_CXX_FLAGS="-Werror" ..
      - name: build
        run: cmake --build build-ubuntu

  win-build:

    runs-on: windows-latest

    steps:
      - uses: actions/checkout@v1
      - name: configure
        run: mkdir build-windows && cd build-windows && cmake ..
      - name: build
        run: cmake --build build-windows

  macOS-build:

    runs-on: macOS-latest

    steps:
      - uses: actions/checkout@v1
      - name: configure
        run: mkdir build-macOS && cd build-macOS && cmake -DCMAKE_CXX_FLAGS="-Werror" ..
      - name: build
        run: cmake --build build-macOS
```

1. `jobs`  表示工作任务。
2. `macOS-build:`、`win-build:`、`ubuntu-build:` 表示任务标签。
3. `runs-on:` 表示所运行的系统：`ubuntu-latest`,`windows-latest`,`macOS-latest`。
4. `- name:`当前项的别名
5. `run:`执行的命令

该文件的作用就是，分别在`ubuntu`，`Windows`以及`macOS`里面编译一边。

比之前自己手动下拉编译简单多了。

#### bazel

另外，还尝试给项目加上了 `bazel` 编译：

bazel : ![bazel-build CI](https://github.com/ClangTools/clangTools/workflows/bazel-build%20CI/badge.svg)

对应的文件可以为：

```yml
name: bazel-build CI

on:
  push:
    branches: [ master,dev ]
  pull_request:
    branches: [ master,dev ]

jobs:

  ubuntu-build:
    name: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1

      - name: run
        uses: ngalaiko/bazel-action/1.2.1@master
        with:
          args: build //...
```

对了，项目地址是：

https://github.com/ClangTools/clangTools

欢迎 `star` 、 `fork`。