---
layout: post
title: "Raspberry Zero UPS"
tagline: "Raspberry Zero UPS"
categories: 随笔
author: "kekxv"
---

有一款小巧的开发板：`Raspberry Zero`，Broadcom BCM2835 SoC，它装有1 GHz ARM1176JZF-S单核CPU，Broadcom VideoCore IV @ 250 MHz GPU（仍支持高清）和512MB SDRAM；支持运行完整的`Linux (Arm系列)`系统，并且带有`WiFi`。

![树莓派zero](/assets/imgs/old/zb_users/upload/2020/03/202003172109357539136.jpg "树莓派zero")

当然，这只是本文的主角之一，另一个主角是对应的UPS(算是吧)：

![树莓派 UPS](/assets/imgs/old/zb_users/upload/2020/03/202003172113385662548.png "树莓派 UPS")

有人给树莓派做了一个电池🔋配件，能够让树莓派可以“脱离插座”，并且扩展了`USB 串口`功能，能够让`PC设备`通过`USB线`在供电的同时可以通过`串口`登录`树莓派zero`进行操作以及配置。

同时，该模块支持`I2C(IIC)`模式读取`电池电量`以及`电池温度`：

C++ 源码可以[点此](https://github.com/ClangTools/clangTools/blob/master/Example/test_i2c.cpp)查看。

Nodejs 源码可以[点此](https://github.com/kekxv/RaspberryPiZeroControl-Nodejs)查看。

C++ 源码

```c++
//
// Created by caesar kekxv on 2020/3/4.
//
// 树莓派电池模块
//     读取电池电压以及电池电量 🔋
//

#include <i2c_tool.h>
#include <logger.h>

int file_i2c;
int length;
unsigned char buffer[60] = {0};

int main(int argc, char *argv[]) {
    logger::instance()->init_default();

    i2c_tool i2CTool;
    int ret;
    double p, v;
    unsigned char dataV, dataP;

    i2CTool.setAddr(0x36); // 设备 I2C 地址为 0x36
    i2CTool.Open();
    std::vector<unsigned char> data;
    std::vector<unsigned char> wData;
    wData.push_back(0x02);
    ret = i2CTool.transfer(wData, &data, 1);
    if (ret <= 0) {
        logger::instance()->e(__FILENAME__, __LINE__, "Read 0x02 : %d", ret);
        return -1;
    }
    dataV = data[0];
    v = (double) (((dataV & 0xFF) << 8) + (dataV >> 8)) * 1.25 / 1000 / 16;
    data.clear();
    wData[0] = 0x04;
    ret = i2CTool.transfer(wData, &data, 1);
    if (ret <= 0) {
        logger::instance()->e(__FILENAME__, __LINE__, "Read 0x04 : %d", ret);
        return -2;
    }
    dataP = data[0];
    p = (double) ((int) ((dataP & 0xFF) << 8) + (int) (dataP >> 8)) / 256 - 5;

    logger::instance()->i(__FILENAME__, __LINE__, "电压 : %.02lfV; 电量 : %.02lf %%", v, p);
    return 0;
}
```

有兴趣的各位可以去了解一下
