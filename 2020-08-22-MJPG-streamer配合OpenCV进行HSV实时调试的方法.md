---
layout: post
title: MJPG-streamer配合OpenCV进行HSV实时调试的方法
subtitle: Realtime Adjustment
author: Kingtous
date: 2020-08-22 18:18:33
header-img: img/post-bg-2015.jpg
catalog: True
tags:
- OpenCV
- Linux
---

**PS:**MJPG streamer为实时显示视频流提供了思路，并提供了丰富的API供调用，此处使用MJPG streamer提供的HTTP API对接Python OpenCV实现HSV调节

#### 1. 启动MJPG streamer

- 此处未使用base auth验证，公网使用请注意安全

```shell
./mjpg_streamer -i "/home/kingtous/github/mjpg-streamer/mjpg-streamer-experimental/input_uvc.so -d /dev/video0 -n -f 30" -o "/home/kingtous/github/mjpg-streamer/mjpg-streamer-experimental/output_http.so -l 0.0.0.0 -p 8080 -w /home/kingtous/github/mjpg-streamer/mjpg-streamer-experimental/www"
```

- input_uvc.so表示使用uvc摄像头
  - -d 指定device
  - -f 表示30帧
- -o表示输出
  - output_http.so表示启动http服务器
    - -l 监听地址，`0.0.0.0`表示内外网都监听
    - -p 端口号
    - -w HTTP服务器根目录

#### 2.Python HSV编辑器实现

整体分为两部：

- 请求API下载实时帧
  - API地址：`http://localhost:8080/?action=snapshot`
- OpenCV显示图片

整体代码实现：

```python
# -*- coding:utf-8 -*-

import cv2
import numpy as np
from urllib import request

"""
功能：读取一张图片，显示出来，转化为HSV色彩空间
     并通过滑块调节HSV阈值，实时显示
"""

#cv2.imshow("BGR", image) # 显示图片

hsv_low = np.array([25, 75, 165])
hsv_high = np.array([40, 255, 255])

def h_low(value):
    hsv_low[0] = value
    pass

def h_high(value):
    hsv_high[0] = value
    pass

def s_low(value):
    hsv_low[1] = value
    pass

def s_high(value):
    hsv_high[1] = value
    pass

def v_low(value):
    hsv_low[2] = value
    pass

def v_high(value):
    hsv_high[2] = value
    pass

def binary_HSV(img):
    hsv_low = np.array([25, 75, 165])
    hsv_high = np.array([40, 255, 255])
    hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV) # BGR转HSV
    dst = cv2.inRange(hsv, hsv_low, hsv_high) # 通过HSV的高低阈值，提取图像部分区域
    cv2.show("Binary HSV Outpt:",dst)
    return dst


cv2.namedWindow('image',cv2.WINDOW_FREERATIO)
cv2.createTrackbar('H low', 'image', 0, 255, h_low) 
cv2.createTrackbar('H high', 'image', 0, 255, h_high)
cv2.createTrackbar('S low', 'image', 0, 255, s_low)
cv2.createTrackbar('S high', 'image', 0, 255, s_high)
cv2.createTrackbar('V low', 'image', 0, 255, v_low)
cv2.createTrackbar('V high', 'image', 0, 255, v_high)


url = "http://localhost:8080/?action=snapshot"

def downloadImg():
    global url
    with request.urlopen(url) as f:
        data = f.read()
        img1 = np.frombuffer(data, np.uint8)
        #print("img1 shape ", img1.shape) # (83653,)
        img_cv = cv2.imdecode(img1, cv2.IMREAD_ANYCOLOR)
        return img_cv

while True:
    # image = downloadImg() 
    image = downloadImg() #cv2.imread('1.jpg') # 根据路径读取一张图片

    dst = cv2.cvtColor(image, cv2.COLOR_BGR2HSV) # BGR转HSV
    dst = cv2.inRange(dst, hsv_low, hsv_high) # 通过HSV的高低阈值，提取图像部分区域
    cv2.imshow('output', dst)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
    
cv2.destroyAllWindows()
```