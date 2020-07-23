---
layout: post
title: PyGame-带手柄的物体移动Demo
subtitle: Python World.
author: Kingtous
date: 2020-07-23 09:01:14
header-img: img/about-bg.jpg
catalog: True
tags:
- Python
---

**注：截止到2020年07月23日，PyGame 1.9.6版本在macOS上无法显示图片等，改用2.0.0.dev10后显示正常**

使用`PyGame`可以很方便的制作小游戏，快速支持手柄操作、键盘鼠标操作以及动画实现、物体绘制。

- 使用`pygame.joystick`驱动手柄操作。

#### 导入必要的包

```python
'''
@Author: Kingtous
@Date: 2020-07-22 22:36:40
@LastEditors: Kingtous
@LastEditTime: 2020-07-22 22:41:37
@Description: Kingtous' Code
'''

import pygame
from pygame.event import Event
from pygame.joystick import Joystick
from pygame import image
```

#### 初始化游戏基本设置

- 初始化`pygame`、`pygame.joystick.init()`、`clock`帧率

```python
# GAME BASIC CONFIG
pygame.init()
pygame.joystick.init()
joysticks = [pygame.joystick.Joystick(x) for x in range(pygame.joystick.get_count())]
clock = pygame.time.Clock()
```

#### 定义窗口

```python
# WINDOW
screen_size = screen_width, screen_height = [600, 600]
screen = pygame.display.set_mode(screen_size)
pygame.display.set_caption("cat movement")
done = False
```

#### 定义物体、背景

- 可用`get_rect`方法获取物体大小
- 使用`pygame.transform.scale`重新定义物体分辨率

```python
# PROPERTY CAT
speed = [-2, 1]
object_cat = image.load("cat.png")
object_cat = pygame.transform.scale(object_cat, (200, 200))
position = object_cat.get_rect()
# object_cat_width = object_cat.get_width()
# object_cat_height = object_cat.get_height()
# object_cat_x = screen_width / 2 - object_cat_width / 2
# object_cat_y = screen_height - object_cat_height
# PROPERTY CAT ENDED

# PROPERTY BG
img_bg = image.load("bg.jpg")
screen.blit(img_bg, (0, 0))
# PROPERTY BG END
```

#### 初始化事务Event

- 此处实现手柄控制物体移动
- 使用`position.move`完成物体移动
- 使用`screen.blit`方法放置物体，`display.update()`更新窗口
- 使用`pygame.event.get()`获取当前事务

```python
if __name__ == '__main__':

    print("joysticks:", joysticks)
    if len(joysticks) > 0:
        joystick: Joystick = joysticks[0]
        joystick.init()
        print("joystick name:", joystick.get_name())
        while not done:
            for event in pygame.event.get():
                event: Event
                # JOYAXISMOTION = 7
                # JOYBALLMOTION = 8
                # JOYBUTTONDOWN = 10
                # JOYBUTTONUP = 11
                # JOYHATMOTION = 9
                if event.type == pygame.JOYHATMOTION:
                    speed = [num*2 for num in list(event.value)]
                    speed[1] = -speed[1]
                    #print("speed", speed)
                elif event.type == pygame.QUIT:
                    done = True
                    break
            # print("refresh")
            screen.fill((255, 255, 255))
            position = position.move(speed)
            if position.left < 0 or position.right > 600 or position.top < 0 or position.bottom > 600:
                position = object_cat.get_rect()
            #print("position now", position)
            # update cat
            screen.blit(object_cat, position)
            # update ui
            pygame.display.update()
            # delay
            pygame.time.delay(10)
            clock.tick(60)

    pygame.quit()
```

#### 效果

- 执行`.py`文件，效果如下

![效果图](http://img.kingtous.cn/img/20200723090252.png)