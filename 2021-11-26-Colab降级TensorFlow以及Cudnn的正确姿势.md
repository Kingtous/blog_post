---
layout: post
title: Google Colab降级TensorFlow以及Cudnn的正确姿势
subtitle: Google Colab
author: Kingtous
date: 2021-11-26 23:55:01
header-img: img/about-bg-walle.jpg
catalog: True
tags:
- Colab
- Linux
---

# Google Colab升/降级TensorFlow以及cudnn的正确姿势

## 背景

跑leaf-audio论文的时候，代码用到了lingvo库，当前lingvo库的最新版本是0.10.0，并且与tensorflow-gpu版本强依赖（2.6，而colab是2.7）。

直接按要求pip tensorflow-gpu 2.6后，又发现cudnn版本不对。也就是说，tensorflow 2.6版本是基于cudnn 8.1编译的，需要8.1以上的cudnn才行。但是系统里面的cudnn是8.0.5。

经过n天的摸索，试了各种方法....最终确定了只有一种方案可行：升级系统内部的cudnn。

## 下载cudnn

到Nvidia官网下载cudnn即可。我这里下载的是ubuntu 18.04的cudnn的xz包（deb包安装后感觉没用）。

我这里下载好了一份8.5的cudnn，[可以到这里下载](https://drive.google.com/file/d/1-0BGQa2-0ixK78duTXYYBVoKlNJ5kYP1/view?usp=sharing)，或者直接link到自己的Google Drive即可。

## 进入CoLab Jupyter

我们需要一些操作，这里直接列举出来，方便大家copy。

### 切换到tmp分区

```python
import os
os.chdir('/tmp')
```

### 解压并复制cudnn文件到指定目录

如果是你自己准备的文件，这里根据你自己的cudnn的路径，改一下/tmp/xxx/lib中间的xxx即可。

```python
!tar -xvf "/content/drive/MyDrive/cuda11.5.tar.xz" -C /tmp/
!sh -c "mv /tmp/cudnn-linux-x86_64-8.3.1.22_cuda11.5-archive/lib/* /usr/local/cuda/lib64/" 
!sh -c "mv /tmp/cudnn-linux-x86_64-8.3.1.22_cuda11.5-archive/include/* /usr/local/cuda/include/" 
```

### 给文件赋权

```python
!sudo chmod a+r /usr/local/cuda/include/cudnn*.h
!sudo chmod a+r /usr/local/cuda/lib64/libcudnn*
```

#### 刷新系统链接设置（重要！！）

一定要使用ldconfig刷新系统链接配置，否则无效

```python
!ldconfig
```

到这里所有的配置都OK了。

> 我不知道为什么，网上配置这些环境要如此复杂，甚至写长篇论文一样讲述自己的过程。
>
> 其实在我看来，只有两种原因：
>
> - 语言不够精炼，突出不了重点，废话太多
> - 凑字数或者装X

## 效果

直接执行自己的代码，在TensorFlow降级到2.6.2时，成功启动了cuda+cudnn

![](https://tva1.sinaimg.cn/large/008i3skNly1gwrutxtdtej31vd0u0qeh.jpg)

## 题外话

Windows 10现在可以联合wsl2使用CUDA了，效果还不错，有兴趣的小伙伴可以尝试一下。

步骤：

- 下载Windows 10的wsl cuda的专版显卡驱动
- 进入wsl2，按照n卡官网wsl文档，下载特制的cuda和公版的cudnn，配置即可。
