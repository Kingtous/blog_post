---
layout: post
title: QML Bootstrap开发学习
subtitle: Qt+Bootstrap
author: Kingtous
date: 2020-08-02 22:45:28
header-img: img/post-bg-web.jpg
catalog: True
tags:
- Qt
- C++
---

#### QML Bootstrap

Link：[Github仓库地址](https://github.com/brexis/qml-bootstrap)



#### 列表List

- 效果图

    ![](http://img.kingtous.cn/img/20200802225156.png)

- 代码

    ```xml
    DefaultListView{
            id: listView
            hasDivider: true
            anchors.fill: parent
            onItemClicked: {
                listView.model.get(index).text = "Item clicked";
            }
    
            model: ListModel{
                ListElement{
                    text: "Item 1 default"
                    divider: "Divider 1"
                }
                ListElement{
                    text: "Item 2 light"
                    divider: "Divider 1"
                    class_name: "light"
                }
                ListElement{
                    text: "Item 3 stable"
                    divider: "Divider 1"
                    class_name: "stable"
                }
                ListElement{
                    text: "Item 4 calm"
                    divider: "Divider 2"
                    class_name: "calm"
                }
                ListElement{
                    text: "Item 5 positive"
                    divider: "Divider 2"
                    class_name: "positive"
                }
                ListElement{
                    text: "Item 5 assertive"
                    divider: "Divider 2"
                    class_name: "assertive"
                }
                ListElement{
                    text: "Item 5 balanced"
                    divider: "Divider 3"
                    class_name: "balanced"
                }
                ListElement{
                    text: "Item 5 royal"
                    divider: "Divider 3"
                    class_name: "royal"
                }
                ListElement{
                    text: "Item 5 dark"
                    divider: "Divider 3"
                    class_name: "dark"
                }
            }
    
        }
    ```



#### 带图标的列表

- 效果图

    ![](http://img.kingtous.cn/img/20200802234335.png)

- 代码