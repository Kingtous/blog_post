---
layout: post
title: Manjaro Linux下搞定Boardcom WiFi/Bluetooth
subtitle: Manjaro Linux下搞定Boardcom WiFi/Bluetooth
author: Kingtous
date: 2020-07-23 16:01:14
header-img: img/about-bg.jpg
catalog: True
tags:
- Linux
---

Boardcom WiFi/蓝牙在Windows/Linux下的兼容性简直跟xxx一样。

Boardcom为Linux下的网卡提供了闭源驱动，但是实验发现在`Manjaro`下无法正常使用蓝牙（94352z网卡）...

多亏了广大网友，it works.

#### 1.[WiFi]安装闭源Boardcom STA驱动

使用`yay`一键搞定

```shell
yay boardcom-wl-dkms
```

注意：推荐安装`dkms`版，可以在内核升级时自动编译入内核。

安装完成后，重启生效，也可以使用rmmod和modprobe直接载入新模块。

```shell
rmmod b43 b43legacy ssb bcm43xx brcm80211 brcmfmac brcmsmac bcma wl
modprobe wl
```

但是此时，在我的环境下蓝牙还是不能使用，于是...

#### 2.[蓝牙]安装broadcom-bt-firmware

Github链接：[broadcom-bt-firmware](https://github.com/winterheart/broadcom-bt-firmware)

可以通过`yay broadcom-bt-firmware`一键安装（Manjaro真香）

如果遇到问题请查看Github上的README.md

一般重启即可～