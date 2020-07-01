---
layout: post
title: "clangTools"
tagline: "clangTools聚合类"
categories: 随笔
author: "kekxv"
---

[clangTools](https://github.com/kekxv/clangTools "clangTools") 聚合类是为了将一些短小精湛的代码集合起来。


在进行`c/c++`开发的使用，由于自带的系统库支持的问题，或者api接口的问题，导致`c/c++`在不规范开发的时候，很容易出现冗余代码，或者各种查API手册的问题。

为了减少这些情况，所以讲一些常用的方法，并且第三方依赖较少的方法进行封装。

## 编译状态 build status

bazel : ![bazel-build CI](https://github.com/ClangTools/clangTools/workflows/bazel-build%20CI/badge.svg)

cmake : ![C/C++ CI](https://github.com/ClangTools/clangTools/workflows/C/C++%20CI/badge.svg)


## 支持功能

- [Base64](src/Base64),
- [ConfigTool](src/ConfigTool),
- [http](src/http),
- [JSON](src/JSON),
- [logger](src/logger),
- [memory_share](src/memory_share),
- [openssl](src/openssl),
- [Pipe](src/Pipe),
- [poll_tool](src/poll_tool),
- [Popen](src/Popen),
- [SHA1](src/SHA1),
- [socket](src/socket),
- [subprocess](src/subprocess),
- [thread_pool](src/thread_pool),
- [xml](src/xml)
- [i2c_tool](src/i2c_tool)
- [libusb_1_tool](src/libusb_1_tool)
- [opencv_tool](src/opencv_tool)
- [Hzk_tool](src/Hzk_tool)
- [plthook](src/plthook)([here](https://github.com/kubo/plthook))


## 使用要求

1. 需要支持`c++ 17`，主要是使用 `filesystem` 接口，可以自行去掉。
1. 需要`cmake`支持，版本：`>3.9`
1. 如果需要 `TLS` 支持，则需要`OpenSSL`。

## 功能

### JSON 以及 XML 解析

项目集成了`JSON` 以及 `XML` 的解析以及常用操作。`JSON`和`XML` 应该是很常用的传输格式了，所以将其加入到本工具仓库。

> 此处使用第三方： tinyxml2,CJsonObject.

### SHA1,base64

加入 `SHA1` 以及 `Base64`支持，由于 [kProxyCpp](https://github.com/kekxv/kProxyCpp "kProxyCpp") 项目需要，所以将其加入。主要用作：`WebSocket` 秘钥升级，以及图片的传输(指的是在不使用二进制传输的情况下)。

### logger 日志

在软件运行的时候，难免会遇到问题，这时候日志的作用就比较明显，但是`c/c++`系统库内并没有很好地日志库，这让人感到遗憾，所以在本项目里面加入了`logger`日志库，该日志库支持5种模式，分别为：debug，info，warning ，error ，fatal；分别对应调试信息，一般信息，警告信息，错误信息，强制输出信息。

同时支持文件输出。

### thread_pool 线程池

[thread_pool.md](https://github.com/kekxv/clangTools/blob/master/src/thread_pool.md "thread_pool.md")

#### 实现原理

“管理一个任务队列，一个线程队列，然后每次取一个任务分配给一个线程去做，循环往复。” 这个思路有神马问题？ 线程池一般要复用线程，所以如果是取一个 task 分配给某一个 thread，执行完之后再重新分配， 在语言层面基本都是不支持的：一般语言的 thread 都是执行一个固定的 task 函数， 执行完毕线程也就结束了(至少 c++ 是这样)。 so 要如何实现 task 和 thread 的分配呢？

让每一个 thread 都去执行调度函数：循环获取一个 task，然后执行之。

这样可以保证了 thread 函数的唯一性，而且复用线程执行 task 。

一个线程 pool，一个任务队列 queue ；

任务队列是典型的生产者-消费者模型，本模型至少需要两个工具：一个 mutex + 一个条件变量，或是一个 mutex + 一个信号量。mutex 实际上就是锁，保证任务的添加和移除(获取)的互斥性，一个条件变量是保证获取 task 的同步性：一个 empty 的队列，线程应该等待(阻塞)；

atomic 本身是原子类型，从名字上就懂：它们的操作 load()/store() 是原子操作，所以不需要再加 mutex。

### ssl_common OpenSSL 封装

2019年12月30日，终于把`OpenSSL`弄好了，为啥要说终于？因为资料少，加上 API 比较多，然后之前也没整过，然后导致载坑了3天，才搞完。然后将其封装为一个标记方便使用的工具类。

### socket 工具类

非阻塞读写TCP连接，做跨平台支持

结合 `ssl_common(OpenSSL)` 工具类，可以支持TLS加密。

可用于https server（包含WebSocket）

参考代码： [kProxyCpp](https://github.com/kekxv/kProxyCpp "kProxyCpp")

