---
layout: post
title: macOS Catalina启用NTFS-3G并实现自动挂载的正确方式
subtitle: NTFS Yes！
author: Kingtous
date: 2020-03-06 09:27:10
header-img: img/about-bg.jpg
catalog: True
tags:
- macOS
---

> NTFS-3G是一个开源的支持NTFS读写的系统小工具
>
> 相比NTFS For Mac等一些专业工具来说，NTFS-3G的驱动读写性能大约为专业工具的**30%**
>
> 请谨慎考虑是否安装

链接：[NTFS-3G Wiki](https://github.com/osxfuse/osxfuse/wiki/NTFS-3G#auto-mount-ntfs-volumes-in-read-write-mode-on-macos-1015-catalina)

### macOS Catalina启用NTFS-3G并实现自动挂载的正确方式

#### 安装NTFS-3G

需要安装`osxfuse`和`ntfs-3g`

```shell
brew cask install osxfuse
brew install ntfs-3g
```

国内安装`osxfuse`可能较慢，可用`proxychains-ng` 代理终端

#### 解锁apfs分区

```shell
 kingtous@localhost  ~  diskutil list
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *251.0 GB   disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:                 Apple_APFS Container disk1         250.8 GB   disk0s2

/dev/disk1 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +250.8 GB   disk1
                                 Physical Store disk0s2
   1:                APFS Volume Macintosh HD -数据      175.9 GB   disk1s1
   2:                APFS Volume Preboot                 82.6 MB    disk1s2
   3:                APFS Volume Recovery                526.6 MB   disk1s3
   4:                APFS Volume VM                      1.1 GB     disk1s4
   5:                APFS Volume Macintosh HD            11.0 GB    disk1s5
```

我们获取到`Macintosh HD -数据`的标号为`disk1s1`

在终端输入该标号进行解锁

```shell
diskutil apfs unlockVolume disk1s1
```

---

**到此NTFS-3G**可以正常使用了，可以手动调用NTFS-3G进行挂载，挂载指令如下：

```shell
sudo /usr/local/bin/ntfs-3g /dev/disk2s1 /Volumes/NTFS -olocal -oallow_other -o auto_xattr
```

指令中，`/Volumes/NTFS`如果不存在的话先`mkdir`一下，`/dev/disk2s1`这个是你NTFS分区的disk号，可以通过`diskutil list`查看。

**若要实现自动挂载，还需要更改系统挂载脚本**

---

### 实现自动挂载

原理：替换`/Volumes/Macintosh HD/sbin`下的`mount_ntfs`脚本

#### 关闭DIP保护模式

- 在macOS Recovery的Terminal下输入`csrutil disable`

- 重启至正常模式

#### 重命名mount_ntfs

```shell
cd "/Volumes/Macintosh HD/sbin"
mv mount_ntfs mount_ntfs.orig
```

> 注意：在macOS Catalina下由于启用了新机制，在关闭DIP保护模式时，目录"/Volumes/Macintosh HD/sbin"**仍然为read-only状态**，*这点大部分博客都没写*
>
> **需要临时挂载为可写**（重启后失效）
>
> ```shell
> sudo mount -uw /
> ```

#### 软连接NTFS-3G的连接脚本

```shell
ln -s "/Volumes/Macintosh HD/usr/local/sbin/mount_ntfs" mount_ntfs
```

#### 开启DIP保护模式（可选）

> 如果不需要保护模式可以开启

- 在macOS Recovery的Terminal下输入`csrutil enable`

- 重启至正常模式

---

#### 检查是否生效

只需查看`New Folder`是否亮起即可，如图：

![](http://img.kingtous.cn/img/20200306095916.png)

