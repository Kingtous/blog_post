---
layout: post
title: 巧用树莓派+MotionEyeOS+PushPlus实现摄像头监控系统，支持微信推送
subtitle: Respberry Pi WebCam System
author: Kingtous
date: 2020-05-16 10:22:47
header-img: img/neuq-sky.png
catalog: True
tags:
- Linux
- 树莓派
---

#### 1.烧录MotionEyeOS镜像

使用balenaEtcher烧录到一张SD卡中。启动树莓派

- 在`config.txt`中加入`ssh`文件开启SSH Server，配置`wpa_supplicant.conf`设置默认WiFi热点

#### 2.设置时间服务器

内置的好像在中国没有用，时间初始为`1970-01-01`，这里我们换成阿里云提供的NTP时间服务器

```shell
# NTP协议
ntp1.aliyun.com
```

![](http://img.kingtous.cn/img/20200516102519.png)

#### 3.设置动作捕捉

实测`Frame Change Threshold`设置为`3%`左右为最佳

其他设置见下图

![](http://img.kingtous.cn/img/20200516102750.png)



#### 4.开启监控串流

- 如果在公网中，Authentication Mode建议开启basic auth进行验证

> 别忘记最后点击`apply`保存所有设置

![](http://img.kingtous.cn/img/20200516103051.png)



#### 5. 实现动作捕捉，微信通知

- 去`Push+`注册一个`token`

- 使用内置的`curl`指令对接`push+`提供的API接口

```shell
curl http://pushplus.hxtrip.com/customer/push/send -X POST -H "Content-Type:application/json" -d '{"title":"监控系统提示","content":"在%Y年%m月%d日-%H时%M分%S秒，摄像头画面出现波动，请关注","token":"xxx","topic":"homing"}'
```

---

配置完，测试结果如图

![](http://img.kingtous.cn/img/20200516103458.jpg)