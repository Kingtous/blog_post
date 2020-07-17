---
layout: post
title: 在EdgeBoard Z3（Xilinx开发板）中集成RTL8811CU驱动的历程
subtitle: 未完...
author: Kingtous
date: 2020-07-15 21:32:48
header-img: img/post-bg-unix-linux.jpg
catalog: True
tags:
- Linux
---

**目前进度：已可以在平台上支持网卡的识别、扫描。WiFi连接还有循环连接的问题，还没找到解决方法，等灵感吧**

PS：这个板子基于内核`4.14.0-xilinx-v2018.3`构建，该有的网络工具一样没有hhh，比较“干净”（阉割）的计算卡Linux系统。

### 安装RTL8811CU/RTL8822CU驱动

**注1**：5.4.1版本驱动安装后用`dmesg`打印后会发现有崩溃现象，改为5.8.1版本问题解决。5.8.1版本地址：[地址](https://github.com/brektrou/rtl8821CU/tree/5.8.1)

由于没有对应的平台预设，我们需要更改以下内容再进行`make`和`make install`

**注2**：开发版有`dkms`的话尽量不要编译，`Edgeboard Lite`中没有`dkms`，只能编译

- 修改ARM_S3C6K4（任意找一个即可）的`PLATFORM`参数如下
    - ![ARM64](http://img.kingtous.cn/img/20200715223255.png)
    - 填写的`PLATFORM`config一定要为`y`，否则无效
- 注释EXTRA_FLAGS=-mabi
    - 使用`sed`指令完成：`sudo sed -i 's/-msoft-float//' /lib/modules/$(uname -r)/build/arch/arm/Makefile`
- 关闭HW_TX_MODE（否则编译报错）
    - **这点一定得注意，这个错误会在编译快完成时出现，若出现问题则又需要全部重新编译**
    - ![关闭CONFIG_MP_VHT_HW_TX_MODE](http://img.kingtous.cn/img/20200715222938.png)
- 编译完成后`make install`
- 打开`dmesg`若出现`registered`字样则表示加载成功

**FAQ:**

- 问：`dmesg`加载成功了，但是`ifconfig`中没看到`wlanx`字样
- 答：输入`rfkill list`查看WLAN是否被`soft block/hard block`了，若被`soft block`则可以尝试使用`rkill unblock wifi`解除软封锁，若被`hard block`，则可能是硬件故障...

### 安装网卡工具

为了尝试各种工具是否可用，尝试过`iwconfig`、`iw`、`wpa_supplicant`，但其中只有`wpa_supplicant`支持WPA加密的WiFi网络。

#### 编译安装wpa_supplicant WPA2网卡工具

`wpa_supplicant`下载地址：[w1.fi官网](http://w1.fi/wpa_supplicant/)

```bash
make
make install
```

正常编译完后会在原目录生成可执行文件`wpa_cli`、`wpa_supplicant`等可执行文件

![](http://img.kingtous.cn/img/20200717220521.png)

### 配置网络

#### 使用wpa_supplicant配置

- 编写`wpa_supplicant.conf`

```bash
ctrl_interface=DIR=/var/run/wpa_supplicant
update_config=1
country=CN

network={
        ssid="xxx-xxx"
        psk="xxx"
        key_mgmt=WPA-PSK
        disabled=1
}

network={
        ssid="xxx-xxx"
        psk="xxx"
        key_mgmt=WPA-PSK
}

```

#### 使用iw、iwconfig连接无密码网络/WEP网络

- `iwconfig`与`iw`都不支持现在主流的`WPA/WPA2`加密
- 使用教程此处略，网上很多