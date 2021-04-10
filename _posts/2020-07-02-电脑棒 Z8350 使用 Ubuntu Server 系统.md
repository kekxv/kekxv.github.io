---
layout: post
title: "电脑棒 Z8350 使用 Ubuntu Server 系统"
tagline: "电脑棒 Z8350 使用 Ubuntu Server 系统"
categories: 随笔
author: "kekxv"
---

只是关于一个电脑棒的简单评测

因为贪玩，并且`树莓派zero`性能问题以及不支持`docker`，所以采购了一款差不多大小的单机板子，就是这款：

![z8350](/assets/imgs/old/upload/2019/12/201912212245415342818.jpg "z8350")

购买链接我就不放了，有兴趣可以自己搜一下。我购买的是4G+64G的版本（看起来真像小广告，啧啧啧）。
对了，这玩意有几个坑，记得看下后面的注意事项，万一踩坑了，希望能有帮助。

## 开始之前

请准备好以下东西，之后会用到：
1. HDMI 显示器
2. USB 以太网卡
3. USB 鼠标键盘
4. 可以随时上网搜索答案的PC或者手机
5. 闲置U盘

## 选择 Linux 发行版本
根据自己的爱好，选择一款喜欢的发行版本。
（要显示器的各位，可以考虑下 `kali linux`最新版，试试卧底模式，嘿嘿嘿）
我选择的是`ubuntu server`，简单点。
下载好之后，可以选择烧录到`U盘`作为启动盘，或者使用 `IODD` 之类的工具。
完成之后，设置BIOS从U盘启动，然后安装系统。
装完系统之后就可以愉快的玩耍了。
什么？不能？还有问题？那看看注意事项，看看能不能帮到你，或者给我评论留言。

对了，WiFi 操作可以参考一下 [How to connect to WiFi from the command line?](https://askubuntu.com/questions/461825/how-to-connect-to-wifi-from-the-command-line "How to connect to WiFi from the command line?")

## 注意事项

我手上这款有几个问题，不知道其他的是不是也会这样（并不只有一个套餐，所以emmmm，不是很确定）

### 开机必须接 HDMI

这个是一个很难受的坑，毕竟用我刷成了 `Ubuntu Server`，也就是不想用屏幕，但是这货没有 `HDMI`接入居然不开机！不开机！！！而且你完全不晓得为啥！！！这还是我测试多次之后发现的。这个的解决办法的话是采购一个`HDMI`欺骗器，嗯，没错，居然有这种玩意，据说是某些显卡不插屏幕的情况下不工作，所以诞生了这玩意，嗯，真是要好好感谢那些个显卡。

<img src="/assets/imgs/old/upload/2019/12/201912212256173180815.png" alt="hdmi欺骗器" width="300" />

这样就可以了吗？并没有！！！有没有发现这玩意是公头的，而电脑棒的也是公头的！！！并不能用好吧。不过幸好，万能的商家（不是同一个商家）有出售 `HDMI  母对母` 线。

<img src="/assets/imgs/old/upload/2019/12/201912212302221485693.png" alt="hdmi 母对母" width="300" />

把这三个接到一起，就可以无显示器开机了，这样就不需要买个HDMI显示器了，嘿嘿。（啥？你想要屏幕？那你买一个就行了呀）。

### BIOS 设置

重装系统嘛，肯定是要进`BIOS`得嘛，不然引导就不太好搞（当然也可以在Windows 下使用软件进行特殊方式的引导，不过比较麻烦，而且内存要求相对较高，喜欢折腾的可以试试），然后问题来了，如果手贱，将启动类型`Windows 8.1`改为了`Android`的话，基本上是开不了机的，设置似乎也设置不了，因为HDMI好像不输出了（为什么是基本上？为什么是好像？emmm，因为我没有针对测试过），在这种情况下，怎么办？一开始的时候我是真的不晓得咋整，都准备退货了（毕竟商家应该能解决），想想好像挺麻烦，所以想拆开看看，看看有没有电池啥的，毕竟有些BIOS掉电的话，还是可以恢复设置的。然后我就拆了（各位拆的时候请小心翼翼一点，划伤或者损坏的话，自己用起来也不太好看，退货的话，也损害了商家利益）。
emmmmmm
这货内部居然用了挡片之类的以及散热片粘死了！！！太伤心。
准备放弃的时候，嗯？这怎么还有一个按钮？这居然有一个不能从外面按的按钮！不用想，按它！
然后`BIOS`就被恢复设置了。嗯，没错，就是这样。
（这按钮可能还有其他功能，我没测试）

### 通电开机

需要在`BIOS`里面设置一下，否则的话，通电之后还需要按一下按钮，不太方便。

### WIFI 问题

最难受的就是 `WIFI` 问题了，重装之后，找不到`WIFI`设备，一度准备放弃，都开始准备买一个`mini WiFi`了，没错，就是那种很便宜的那种。
不过在我不懈的努力~~搜索~~之下，终于找到了解决方案：
原来内核是支持这款 `WIFI` 的（对了这款 `WiFi` 型号是 `AP6255`，支持`2.4G`以及`5G`，这要是不能用，多浪费，是吧），不过由于没有配置文件，所以无法正常加载：
```bash
dmesg | grep -i sdio
[    3.699477] mmc1: new ultra high speed SDR104 SDIO card at address 0001
[    9.911273] Bluetooth: Generic Bluetooth SDIO driver ver 0.1
[   10.107255] brcmfmac: brcmf_fw_map_chip_to_name: using brcm/brcmfmac43455-sdio.bin for chip 0x004345(17221) rev 0x000006
[   13.143419] brcmfmac: brcmf_sdio_htclk: HT Avail timeout (1000000): clkctl 0x50
[   14.160121] brcmfmac: brcmf_sdio_htclk: HT Avail timeout (1000000): clkctl 0x50
[   15.164314] brcmfmac: brcmf_sdio_htclk: HT Avail timeout (1000000): clkctl 0x50
```
并且会提示，无法加载对应的TXT配置。
这时候，我们可以将[ap6255](https://github.com/chewitt/ampak-firmware/tree/master/ap6255 "ap6255")里面的内容替换至`/lib/firmware/brcm/`，替换之前记得备份一下哦。替换之后，重启基本上会出现`WiFi`:
```bash
$ ifconfig -a
enx000ec6c95f38: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.237  netmask 255.255.255.0  broadcast 199.168.22.255
        inet6 fe80::20e:c6ff:fec9:5f38  prefixlen 64  scopeid 0x20<link>
        ether 00:0e:c6:c9:5f:38  txqueuelen 1000  (Ethernet)
        RX packets 69337  bytes 9711536 (9.7 MB)
        RX errors 0  dropped 2154  overruns 0  frame 0
        TX packets 32668  bytes 2778902 (2.7 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 2687  bytes 391630 (391.6 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2687  bytes 391630 (391.6 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.199  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::13bc:ecb4:3814:e35e  prefixlen 64  scopeid 0x20<link>
        ether c0:84:7d:a5:97:9a  txqueuelen 1000  (Ethernet)
        RX packets 166  bytes 19597 (19.5 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 65  bytes 7649 (7.6 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
记得用 `ifconfig -a`。




