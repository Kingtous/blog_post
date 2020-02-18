---
layout: post
title: React Router DOM
subtitle: Router路由设置
author: Kingtous
date: 2020-02-18 16:10:55
header-img: img/post-bg-js-version.jpg
catalog: True
tags:
- React
---

#### React Router DOM路由无效显示白屏的解决方案

> 版本5.1.2

网上的各种方案都没用

大多数都是：

```react
<Router>
  <div>
    <Switch>
      <Route exact path="/" Component={IndexPage}/>
      <Route path="/user/login" Component={LoginPage}/>
      <Route path="/user/register" Component={RegisterPage}/>
    </Switch>
  </div>
</Router>
```

但是在我这没用

经过认真研读官方Documentation后，发现官方用的不是`Component`而是在子标签里设置：

```react
<Router>
  <div>
    <Switch>
      <Route exact path="/"><IndexPage/></Route>
      <Route path="/user/login"><LoginPage/></Route>
      <Route path="/user/register"><RegisterPage/></Route>
    </Switch>
  </div>
</Router>
```

---

效果图！！！终于出来了

![效果图](http://img.kingtous.cn/img/20200218161434.png)