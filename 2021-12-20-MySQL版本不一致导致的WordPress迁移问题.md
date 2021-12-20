---
layout: post
title: MySQL版本不一致导致的WordPress迁移问题
subtitle: MySQL 5.5 vs 5.7
author: Kingtous
date: 2021-12-20 11:32:32
header-img: img/about-bg-walle.jpg
catalog: True
tags:
- WordPress
- MySQL
- vim
- sed
---

# 2021-12-20-MySQL版本不一致导致的WordPress迁移问题的解决

迁移WordPress博客发现，MySQL数据库始终迁移不过去，打开phpmyadmin发现报错：

```shell
unknown collection "utf8mb4_unicode_520_ci"
```

看到发现是编码错误，猜测可能是MySQL版本问题。

后来查看发现原始服务器使用的是MySQL 5.7，但是目标服务器使用的是MySQL 5.5，**而MySQL 5.5没有utf8mb4_unicode_520_ci**，只有"utf8mb4_unicode_ci"。

## 解决思路

在备份文件中将`utf8mb4_unicode_520_ci`替换为`utf8mb4_unicode_ci`，目前没发现问题。

附vim全局替换命令：
```shell
:%s/源字符串/目标字符串/g
```

附sed全局替换命令：

- `-i`表示文件中直接替换，否则输出到控制台
- `g`表示全局替换，否则只替换第一个匹配字符

```shell
sed 's/源字符串/目标字符串/g'
sed -i "s/源原字符串/目标字符串/g"
```