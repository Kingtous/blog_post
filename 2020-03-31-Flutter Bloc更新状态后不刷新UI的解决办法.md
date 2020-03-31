---
layout: post
title: Flutter Bloc更新状态后不刷新UI的一个解决办法
subtitle: Flutter
author: Kingtous
date: 2020-03-31 10:51:21
header-img: img/post-bg-miui6.jpg
catalog: True
tags:
- Flutter
---

## Flutter Bloc更新状态后不刷新UI的一个解决办法

> Flutter 1.12.13+hotfix.8 • channel stable 
>
> 使用`Equatable`创建Bloc

在用Bloc框架写一个项目时，发现在`mapEventToState`中`yield`一个`state`后发现`UI`并没有刷新，明明改变了关注状态。

究竟是怎么回事呢？

![](http://img.kingtous.cn/img/20200331095933.png)

上原代码

```dart
class ...

TopicInfoEntity _entity;

...
            _entity.data.subscribeStatus = 0;
            yield GetTopicDetailState(_entity);

...
```

`Github`上搜寻结果发现，问题出在`Equatable`上

```dart
abstract class TopicDetailState extends Equatable {
  const TopicDetailState();
}

class GetTopicDetailState extends TopicDetailState {
  final TopicInfoEntity infoEntity;

  GetTopicDetailState(this.infoEntity);

  @override
  List<Object> get props => [infoEntity];
}
```

我们知道，`Bloc`需要判断一个新`state`是否需要刷新原UI，需要判断二者`state`是否相等，如果相等则不刷新。

而我们使用的是需要判断二者是否相等，而`Equatable`接口便实现的是这个功能。它通过`props`传入的参数来判断二者是否相等。

那为什么两个有着不同参数的`Entity`会被判断相等呢？

我们来看`Equatable`的接口代码

```dart
@immutable
abstract class Equatable {
  List<Object> get props;
  bool get stringify => false;
  const Equatable();
  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is Equatable &&
          runtimeType == other.runtimeType &&
          equals(props, other.props);
  @override
  int get hashCode => runtimeType.hashCode ^ mapPropsToHashCode(props);
  @override
  String toString() =>
      stringify ? mapPropsToString(runtimeType, props) : '$runtimeType';
}
```

注意！接口重载了`==`判断符，判断相同的条件满足以下2点之一即可

- `identical(this, other)`，即二者属于相同的`Object`
- `other`（这里为传入的`state`）是实现`Equatable`的，且两个`state`的运行类型一样，且**他们的`props`相同**

经过分析，我们发现上述问题出在`props`上，于是我们调试进入`equals`方法。

![](http://img.kingtous.cn/img/20200331101306.png)

果不其然，调试过程中，代码一路执行，走到了**true**

所以二者立刻的相等了，则不刷新`UI`。

我们来看`equals`里面判断了什么：

判断`List`二者不相等的条件可以为：

- 二者至少有一方是`null`
- props的`List`长度不一致
- 二者为`Iterable`的，或者为`Map`（如列表等），同时`unit`内部的元素也满足前面所述的`equals`元素（这个equals函数会针对不同数据类型做不同的判断，详情可查看源代码）

其他情况则直接为`true`

---

`list`中有我们的一个`entity`

- 11行，两个`list`不相同，因为每创建一个`state`，一个空列表都会在`state`中实例化一次
- 12行，二者都不为`null`
- ...以此类推

我们发现，判断两个`entity`是否相等，重点在与`24`行：

在`24`行判断二者相等时，判断二者相等，因为：

>  `==` returns true if two objects are the same instance.

回到我们之前，原来！我们传入`state`是`Bloc`中全局变量：相同的`_entity`，而我们只是改变了其中的一个值而已，两个`object`还是相等的，因为二者为**引用的同一个实例关系**。

****

#### 正确的写法

既然我们的`Entity`不支持`Equatable`类型...那我们可以另辟蹊径，新建一个元素，把值`copy`一遍

好在插件提供了以下方法可以达到`copy`的效果

```dart
...
            var nEntity = topicInfoEntityFromJson(
                TopicInfoEntity(), _entity.toJson());
            nEntity.data.subscribeStatus = 0;
            yield GetTopicDetailState(nEntity);
            _entity = nEntity;
...
```

改完后就能正常刷新啦~

调试我们也发现正常返回了`false`

![](http://img.kingtous.cn/img/image-20200331103852871.png)

![](http://img.kingtous.cn/img/20200331103922.png)