---
layout: post
title: Linux Shell-批量重命名
subtitle: Shell
author: Kingtous
date: 2020-07-23 10:48:14
header-img: img/post-bg-unix-linux.jpg
catalog: True
tags:
- Shell
---

#### 需求分析

- 一个文件夹下有`0.jpg`、`1.jpg`、`2.jpg`、`3.jpg`...需要从某个基数开始重新命名
    - 比如基数为`5`，则重命名为`5.jpg`、`6.jpg`、`7.jpg`、`8.jpg`...

#### 解决方法

- 使用数字进行排序，切分为数组，并使用`expr`指令做加法。最后调用`mv`重命名。

#### 代码解决

```bash
# argv
path=$1
base_num=1501
# entering
#echo "entering $path"
cd $path
# 1350-1125 = 225
result=`ls | sort -n | xargs`
array=(`echo $result`)
#array=(${result// /})
#echo "$result"
for i in ${array[@]}; 
do
	number=`echo $i | tr -cd "[0-9]"`
	number=`expr $number + $base_num`
	mv $i $number.jpg
done
```

