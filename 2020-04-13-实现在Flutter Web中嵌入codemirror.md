---
layout: post
title: 实现在Flutter Web中嵌入codemirror
subtitle: dart codemirror -> flutter codemirror
author: Kingtous
date: 2020-04-13 08:14:10
header-img: img/post-bg-js-version.jpg
catalog: True
tags:
- Flutter
---

## 实现在Flutter Web中嵌入codemirror

> 博文的Flutter版本
>
> Flutter 1.17.0 • channel beta • https://github.com/flutter/flutter.git
> Framework • revision d3ed9ec945 (6 天前) • 2020-04-06 14:07:34 -0700
> Engine • revision c9506cb8e9
> Tools • Dart 2.8.0 (build 2.8.0-dev.18.0 eea9717938)

> CodeMirror 是一款“Online Source Editor”，基于Javascript，短小精悍，实时在线代码高亮显示，他不是某个富文本编辑器的附属产品，他是许多大名鼎鼎的在线代码编辑器的基础库。

`codemirror`目前只有Google团队推出的`dart web`项目，但是未在`flutter web`中实现

我们知道，在`flutter_web`中使用原生`html`插件，需要使用`HtmlElementView`组件支持，并在`initState`中注册组件并配置组件...以下是痛苦的踩坑之路

### 1. 声明全局变量于Stateful Widget中

- `code-mirror`的html ID
- `DivElement`，用于存放
- `CodeMirror`，处理回调
- `HtmlElementView`，UI
- `options`，`codemirror`的配置

```dart
  // UI变量
  String _codemirrorId = "code-edit";
  html.DivElement _codeContent = html.DivElement();
  CodeMirror _codeMirror;
  HtmlElementView _codeView;
  Map options = {'mode': 'clike', 'theme': 'monokai'};
```

### 2. `initState`中注册并生成`HtmlElementView`

> 注意1：这里的`ui`是`dart:ui`
>
> > `import 'dart:ui' as ui;`
>
> 注意2：`// ignore: undefined_prefixed_name`用于忽略`registerViewFactory`未定义问题

```dart
  @override
  void initState() {
    super.initState();
    // 注册codemirror标签
    // ignore: undefined_prefixed_name
    ui.platformViewRegistry
        .registerViewFactory(_codemirrorId, (int viewId) => _codeContent);
    _codeView = HtmlElementView(
      viewType: _codemirrorId,
    );
  }
```

### 3. `build`方法中放入`_codeView`

> 注意：一定要设置`Container`的宽高，否则出现白屏

```dart
 body: Container(
        alignment: Alignment.center,
        child: Column(
          mainAxisAlignment: MainAxisAlignment.start,
          children: <Widget>[
            FlatButton(onPressed: showCodeEditor, child: Text("show")),
            Container(
              decoration: BoxDecoration(color: Colors.grey),
              width: 400,
              height: 400,
>>>>>>>>>>    child: _codeView, <<<<<<<<<<<<<<<<<<<<<<<<<<<<
            ),
          ],
        ),
      ),
```

### 4. 在`_codeView`中导入`CodeMirror`

**注意：`_codeView`在对应的html中，使用了`shadowRoot`进行隔离，在`index.html`的`header`中添加CSS、JS是无效的，无法作用在`shadowRoot`内部，所以我们要另辟蹊径！**

目前有两种方案：

- 使用`iFrame element`导入一个完整的`html`页面，然后通过访问`iFrame`的`contentDocument`变量的`querySelector`方法拿到`html body`里我们定义好的一个空`div`，然后使用`codemirror.fromElement`方法填充进去

    ❎不行，因为`Dart SDK 2.8.0`还没有将`contentDocument`开放出来...[GitHub Issue链接](https://github.com/dart-lang/sdk/issues/34103)给出的解决方案是转化为`JsObject`，但是这样就没法使用`codemirror`的构造函数进行构造了，而且`JsObject`生涩难用，成本实在太高

    ![](http://img.kingtous.cn/img/20200412234355.png)

- 直接在`shadowRoot`中配置！（以下操作为该步骤）

#### 4.1 拿到_codeView的`shadowRoot`

**注意！_codeView的`shadowRoot`没法通过`codeView.shadowRoot`方法（无此方法）！只能通过DOM数查找，所有的`HtmlView`在DOM树种都是flt-platform-view，开发时如果有多个，这个下标注意更换为对应数字**

```dart
var node = html.document.getElementsByTagName("flt-platform-view")[0] as html.HtmlElement;
```

#### 4.2 在`DivElement`中实例化`codemirror`

```dart
_codeMirror = CodeMirror.fromElement(_codeContent, options: options);
```

#### 4.3 导入要使用的第三方CSS、JS

> 这里使用`cdn.bootcss.com`的CSS和JS文件

注意`innerHTML`和`OuterHtml`的区别，我们要使用的是`setInnerHtml`方法。**注意：需要传入一个支持`tag`和`attributes`的验证器，否则header无法添加**

```dart
var headElement = html.HeadElement();
// node 验证器
final NodeValidatorBuilder _htmlValidator=new NodeValidatorBuilder.common()
    ..allowElement('link',attributes: ["rel","href"])
    ..allowElement('script',attributes: ["src"]);
// 设置header
    headElement.setInnerHtml('<link href="https://cdn.bootcss.com/codemirror/5.52.2/codemirror.min.css" rel="stylesheet"><script src="https://cdn.bootcss.com/codemirror/5.52.2/codemirror.min.js"></script><script src="https://cdn.bootcss.com/codemirror/5.52.2/mode/clike/clike.min.js"></script><script src="https://cdn.bootcss.com/codemirror/5.52.2/addon/selection/active-line.min.js"></script><link href="https://cdn.bootcss.com/codemirror/5.52.2/theme/monokai.min.css" rel="stylesheet">',validator: _htmlValidator);
```

#### 4.4 将Header添加进`shadowRoot`的`children`内

```dart
node.shadowRoot.children.insert(0, headElement);
```

---

至此，如果配置正确，将成功实现`codemirror`嵌入`flutter_web`了，效果如下

![](http://img.kingtous.cn/img/20200412234217.png)

但是仍然有些问题！`空格`和`Tab`按键会和`Flutter`有些许冲突，这个之后日益完善吧~