---
layout: post
title: 使用LLVM库编译出现错误的一些解决办法
subtitle: LLVM
author: Kingtous
date: 2020-10-05 23:07:32
header-img: img/home-bg-art.jpg
catalog: True
tags:
- LLVM
- 编译原理
---

### 1.编译成功，提示Option注册了多次

效果图：

- 提示`Option xxx registerd more than once! LLVM ERROR: inconsistency in registerd CommandLine options`

![](http://img.kingtous.cn/img/20201005231105.png)

#### 原因分析：

如果使用的是CMake进行查找依赖的话，如`llvm_map_components_to_libnames`，可能是因为：

- 安装了多个LLVM版本，导致查找时得到了多次相同的`Option`

![](http://img.kingtous.cn/img/20201005231212.png)

#### 解决方法：

- 只保留一个LLVM版本
- 尝试将后面链接处改成`LLVM`而不是之前找到的`llvm_libs`（如上图）



### 2. 编译失败，提示-lxxx not found

如图：

> connot find -ledit
>
> connot find -lz

![](http://img.kingtous.cn/img/20201005231927.png)

#### 原因分析：

缺少lib库

#### 解决方法：

安装`libedit-dev`以及`zlib1g-dev`即可



### 3. CMake配置失败，找到LLVM但提示yaml-bench不存在

如图：

![](http://img.kingtous.cn/img/20201005232131.png)

#### 原因分析：

测试在`LLVM 9.0.0`中不存在这个软件包，但在`9.0.1`中测试包含

#### 解决方法：

- 由于该文件不是很必须的文件，所以可以尝试`touch`该文件

- 尝试安装`libclang-common-9-dev`

![](http://img.kingtous.cn/img/20201005232321.png)