---
layout: post
title: 解决Flutter下iOS deployment target的问题
subtitle: minimum deploy target error.
author: IOS-Kingtous
date: 2020-11-09 08:27:30
header-img: img/post-bg-top.jpg
catalog: True
tags:
- Flutter
- Xcode
---

> Xcode 12.1，iOS 14.1

#### 1. build时出现target error的问题

> plugin xxx target is 11.0, but minimum target is 9.0...

不知道是`Flutter`没考虑到，还是说插件本身可能强制使用了某些target

解决办法：在`Podfile`尾部加入：

```shell
>platform :ios, '11.0'

...
post_install do |installer|
  installer.pods_project.targets.each do |target|
    flutter_additional_ios_build_settings(target)
>   target.build_configurations.each do |config|
>     config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '11.0'
    end
  end
end
```

#### 2. iOS虚拟机调试问题

> building for iOS simulator, but linking in dylib built for iOS

![](http://img.kingtous.cn/img/20201109083035.png)

如果是进行虚拟机调试时，则可以暂时屏蔽相应图片中出现的冲突架构

1. 进行Excluded Architectures

![](http://img.kingtous.cn/img/20201109092621.png)

2. 搜索VALID_ARCHS，添加x86_64架构

![](http://img.kingtous.cn/img/20201109092647.png)