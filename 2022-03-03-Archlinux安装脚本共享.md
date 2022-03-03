---
layout: post
title: Archlinux安装脚本共享
subtitle: Linux Setup
author: Kingtous
date: 2022-03-03 10:28:32
header-img: img/about-bg-walle.jpg
catalog: True
tags:
- Linux 
---


可以参考[Archlinux Wiki](https://wiki.archlinux.org/title/installation_guide)

# 安装时

## 连接网络

界面上有英文提示，按要求连接WiFi等。

## 给磁盘分区

使用`fdisk`完成，可以使用`fdisk -l`查看磁盘号。分好区按目的地挂载一下即可

- EFI分区(1)
  - 要使用`parted`工具`set 1 boot on`设置成可引导
  - `fdisk`后`-t`指定1(efi partition)
- Swap分区、home分区、其他分区等

分完区使用`mkfs.vfat`格式化fat分区，`mkfs.ext4`格式化ext4分区

如果设置了swap可以启用：
```shell
swapon /dev/swap_partition
```

## 挂载后进入arch-chroot

以`/mnt`为例。

```shell
arch-chroot /mnt
```

## 安装部分系统

```shell
pacstrap /mnt base base-devel linux linux-firmware
```

## 设置时区

```shell
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
```

## 本地化

编辑 `/etc/locale.gen`保留要的语言后，输入
```shell
locale-gen
```

## 网络名称

创建一个包含自己网络名称的文件到`/etc/hostname`

## 创建用户

```shell
useradd -m xxx (m指定在home下创建用户目录)
passwd xxx
```
另外还需要给用户赋权限。编辑/etc/sudoers，在root ALL什么的下面复制一行
```shell
xxx ALL...
```

## 安装引导

```shell
pacman -S grub efibootmgr
grub-install /dev/sda
mkinitcpio -P
grub-mkconfig -o /boot/grub/grub.cfg
```


# 安装后操作

## 修改国内镜像

编辑 /etc/pacman.d/mirrorlist，在文件的最顶端添加：

```shell
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```

## 添加Archlinuxcn源

```shell
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```

## 安装桌面（KDE）

```shell
pacman -S xorg plasma kde-applications
systemctl enable xorg 
systemctl enable NetworkManager 
```