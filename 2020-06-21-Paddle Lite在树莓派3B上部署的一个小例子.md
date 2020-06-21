---
layout: post
title: Paddle Lite Demo在树莓派3B上部署的一个小例子
subtitle: Paddle Lite Demo
author: KIngtous
date: 2020-06-21 10:51:55
header-img: img/post-bg-universe.jpg
catalog: True
tags:
- Linux
- 人工智能
---

#### 总流程

- 下载源码
- 下载预编译的模型以及编译支持库
- 修改配置文件为树莓派3B
- 编译代码并执行

#### 安装依赖

涉及到源码编译

```shell
sudo apt-get install gcc g++ make wget unzip libopencv-dev pkg-config
```

#### 下载Paddle Lite Demo

直接`clone`下网友上传到Gitee上的源码

```shell
git clone https://gitee.com/irving_gao/Paddle-Lite-Demo.git
```

#### 下载Paddle Lite Demo的预训练模型以及支持库

- 进入`Paddle-Lite-Demo/PaddleLite-armlinux-demo`

- ```shell
    ./download_models_and_libs.sh
    ```

- 等待下载即可

#### 修改run.sh编译运行Demo

树莓派3B+架构为`armv7`，注释掉第四行的`armv8`设置，取消注释第5行.

![](http://img.kingtous.cn/img/20200621110527.png)



修改最后的执行指令，删除导入照片等指令，最终如下图所示.

![](http://img.kingtous.cn/img/20200621110638.png)

#### 效果

运行视频流检测，效果如下图所示.

```shell
./run.sh armv7hf
```

![效果](http://img.kingtous.cn/img/20200621105339.png)