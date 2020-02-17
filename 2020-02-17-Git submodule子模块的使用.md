---
layout: post
title: Git submodule子模块的使用
subtitle: submodule(git 2.25)
author: Kingtous
date: 2020-02-17 13:34:40
header-img: img/about-bg.jpg
catalog: True
tags:
- Git

---

### 本文介绍：

#### 1.把一个仓库的子目录变换为submodule，并完整记录提交记录

步骤：

- 清除与子目录提交无关的提交
- 推送至新仓库
- 克隆原仓库，以`submodule`的形式引用

**效果：**

![image.png](https://i.loli.net/2020/02/17/LpqcYyRemNM6SvA.png)

#### 1.1 清除与子目录提交无关的提交

- 创建一个新分支（此处分支名为`post`，可以自定义）

```shell
git checkout post
```

- 清除所有无关提交

```shell
git filter-branch --tag-name-filter cat --prune-empty --subdirectory-filter _posts -- --all
```

- 清除无关缓存

```shell
git for-each-ref --format="%(refname)" refs/original/ | xargs -n 1 git update-ref -d
git reflog expire --expire=now --all
git gc --aggressive --prune=now
```

#### 1.2 推送至新仓库

> 注意：新仓库与原仓库需要在同一条分支才可以同步

- 删除原仓库远程链接

```shell
git remote rm origin
```

- 添加新远程仓库链接

```shell
git remote add origin https://github.com/USER/new.git
```

- 将上述`post`分支转为`master`分支

```shell
git branch -d master
git branch master
```

#### 1.3 克隆原仓库，以`submodule`的形式引用

- `clone`原仓库

```shell
git clone https://github.com/USER/old.git
```

- 删除要引用的文件夹

```shell
git rm -r NEED_Submodule/
```

- 添加`submodule`

```shell
git submodule add -b master https://github.com/USER/new.git NEED_Submodule
git submodule init
git submodule update
```

- 推送至原仓库即可

---

> 注意：原仓库必须有`submodule`的访问权限

#### 2.子模块的提交与原仓库的更新

- 进入每一个子模块，commit
- 针对每一个子模块，push

```shell
git submodule foreach git pull orgin master
```

#### 3.Pull含有子模块的仓库

- Pull主仓库

```shell
git pull old.git
```

- 针对每一个子模块，分别进行pull
    - 假设每个本地分支都是master，远程分支都是origin

```shell
git submodule foreach git pull orgin master
```

