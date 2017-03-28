---
title: Android之AdapterView及子类
date: 2016-07-23 23:42:29
categories: 编程语言
tags: Android
---

AdapterView具有如下特征：

 - AdapterView继承了ViewGroup，它的本质是容器。
 - AdapterView可以包括多个“列表项”以合适的形式显示出来。
 - AdapterView显示的多个“列表项”由Adapter提供。调用AdapterView的setAdapter(Adapter)方法设置Adapter即可。

AdapterView及其子类的继承关系图如下图所示：
![AdapterView及其子类继承关系图](http://img.blog.csdn.net/20151130092808596)

从上图中不难看出，AdapterView派生了3个子类：ABSListView、ABSSpinner和AdapterViewAnimator，这3个子类依然是抽象的，实际使用时往往采用它们的子类。

注意：由于Gallery是一个已经过时的API，Android推荐使用HorizontalScrollView来代替它。

参考自《疯狂Android讲义》

