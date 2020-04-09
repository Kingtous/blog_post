---
layout: post
title: ESP8266 Arduino-天气预报小助手Demo
subtitle: esp8266 ssd1306OLED
author: Kingtous
date: 2020-04-09 12:23:43
header-img: img/about-bg.jpg
catalog: True
tags:
- 嵌入式
---

#### Arduino ESP8266 天气预报小助手程序

项目地址：[https://github.com/Kingtous/esp8266_arduino_weather_reporter](https://github.com/Kingtous/esp8266_arduino_weather_reporter)

### 适配配置

- 芯片：`ESP8266 (NodeMcu、CH340)`
- 显示器：`SSD1306OLED SW I2C`
- 编程语言：C++ 17

### 功能

- **设置功能**

    - 未连接成功，自动开始AP和WebServer，访问 `192.168.4.1` 即可设置WiFi、天气
    - 连接成功，输入分配给ESP8266的IP地址也可以设置

    效果图：

    ![](http://img.kingtous.cn/img/20200409122631.png)

    City Code为9位，引用的是：[https://github.com/baichengzhou/weather.api/blob/master/src/main/resources/citycode-2019-08-23.json](https://github.com/baichengzhou/weather.api/blob/master/src/main/resources/citycode-2019-08-23.json)

- **显示功能**

    - 连接成功，LED灯常亮。正在连接，LED灯闪烁。连接失败开启AP，LED灯熄灭
    - 连接成功后自动获取天气并显示

### 目前使用的第三方库

> 请下载第三方库项目的`src`文件夹，更名后放入`lib`文件夹

- [U8g2](https://github.com/olikraus/u8g2) (用于显示屏输出)
- [U8g2-wqy](https://github.com/larryli/u8g2_wqy) (适合 u8g2 的中文字体)

