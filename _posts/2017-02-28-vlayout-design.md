---

layout: post
title:  Tangram 的基础 —— vlayout（Android）
author: Longerian

---

## 前言

vlayout 是手机天猫 Android 版内广泛使用的一个基础 UI 框架项目 提供了一个用于RecyclerView的自定义的LayoutManger，可以实现不同布局格式的混排，目标是支撑客户端native页面的快速开发。它也是 [Tangram 框架](http://pingguohe.net/2016/12/20/Tangram-design-and-practice.html)的基础模块，现已开源，欢迎移步到 [github](https://github.com/alibaba/vlayout) 上指教。

## 简介

### 背景

Android中UI性能消耗主要来自于两个方面：

1. 布局层次嵌套导致多重measure/layout
2. View控件的创建和销毁

除了从在实践中注意消除嵌套布局，Android官方也提供了ListView/GirdView/RecyclerView等基础空间来处理View的回收与复用。

但很多时候我们都会碰到视觉需要在一个长列表下做多种类型的布局来分配各种元素, 特别是电商业务各类首页，频道等页面，元素结构复杂多样。

这种时候实现的选择有不用复用，直接用各个组件进行拼接，但这样会损失性能；选择一个主要的复用容器, 如ListView或者RecyclerView+LinearLayoutManager等，然后在其中使用嵌套等方式对其他的布局方式进行处理，这样一个是减少了复用的能力，另一个是如果需要嵌套无法兼容的布局的时候，需要处理嵌套滑动的情况。

既然RecyclerView提供了基础的回收复用功能，也支持LayoutManager的扩展，那么能不能用一个LayoutManager就完成所有的布局类型呢？ 感觉的这是一个不错的方向，目前在 github 上也能找到类似的项目，但是这些之前也埋有不少bug, 大部分都是因为在一些特殊场景下和RecyclerView相关的其他的类一起使用时出现问题。 为了避免掉入bug大坑，我们决定基于LinearLayoutManager来做改造。

### 特性

1.	自定义了一个VirtualLayoutManager，它继承自 LinearLayoutManager；引入了 LayoutHelper 的概念，它负责具体的布局逻辑；VirtualLayoutManager管理了一系列LayoutHelper，将具体的布局能力交给LayoutHelper来完成，每一种LayoutHelper提供一种布局方式，框架内置提供了几种常用的布局类型，包括：网格布局、线性布局、瀑布流布局、悬浮布局、吸边布局等。这样实现了混合布局的能力，并且支持扩展外部，注册新的LayoutHelper，实现特殊的布局方式。

## 架构

整体的设计方案和思路如下：
![](https://gw.alicdn.com/tps/TB1aGAlPFXXXXa6XVXXXXXXXXXX-1574-914.png)

1.	RecyclerView是整个页面的主体，它的运行需要绑定一个Adapter和LayoutManager，在我们的设计里自定义了VirtualLayoutAdapter和VirtualLayoutManager来绑定到RecyclerView。

### 初始化

![](https://gw.alicdn.com/tps/TB1MScdPFXXXXaJXVXXXXXXXXXX-1112-1095.png)

在使用vlayout的时候，首先做初始化工作，对业务使用方来说，和使用普通的 RecyclerView + LayoutManager 初始化流程基本一致。对于框架流程上来说，前前后后涉及了6个角色，基本流程如下：


![](https://gw.alicdn.com/tps/TB1VCZCPFXXXXcoXpXXXXXXXXXX-1104-1630.png)

当完成前面的初始化工作，将数据和LayoutHelper都绑定到vlayout内部之后，紧接着就可以开始布局流程了。这里无论是刚打开页面第一次布局，还是用户滑动页面，进行一次新的布局，流程都是一致的。


demo动效

![](http://img3.tbcdn.cn/L1/461/1/1b9bfb42009047f75cee08ae741505de2c74ac0a)

实战效果
![](https://gw.alicdn.com/tps/TB1ofsVPFXXXXX0XXXXXXXXXXXX-288-512.png)

本文着重介绍 vlayout 的设计思路和原理，如果要进一步熟悉其细节，最好是到 [github](https://github.com/alibaba/vlayout) 上下载源码阅读，结合本文的说明，效果会更佳。如果想要尝试使用 vlayout 搭建页面，也可以到 [github](https://github.com/alibaba/vlayout) 上下载 demo，阅读使用文档和样式属性说明文档。

> 『很惭愧，就做了一点微小的工作』