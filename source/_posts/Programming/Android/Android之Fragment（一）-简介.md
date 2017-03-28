---
title: Android之Fragment（一）--简介
date: 2016-07-24 00:05:16
tags: Android
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

Fragment是Android 3.0引入的新API。Fragment代表了Activity的子模块，因此可以把Fragment理解成Activity的片段。Fragment拥有自己的生命周期，也可以接受它自己的输入事件。

## Fragment概述 ##
Fragment必须被嵌入到Activity中才能使用，因此，虽然Fragment拥有自己的生命周期，但Fragment的生命周期会受它所在的Activity的生命周期控制。例如，当Activity暂停时，该Activity内的所有Fragment都会暂停；当Activity被销毁时，该Activity内的所有Fragment都会被销毁。只有当Activity处于活动状态时，程序员才可以通过方法独立的操作Fragment。

关于Fragment,可以归纳出如下几个特征。

 - Fragment总是作为Activity界面的组成部分。Fragment可以调用getActivity()方法获取它所在的Activity，Activity可调用FragmentManager的findFragmentById()或findFragmentByTag()方法来获取Fragment。
 - 在Activity运行过程中，可调用FragmentManager的add()、remove()、replace()方法动态的添加、删除或替换Fragment。
 - 一个Activity可以同时组合多个Fragment；反过来，一个Fragment也可以被多个Activity复用。
 - Fragment可以相应自己的输入事件，并拥有自己的生命周期，但它们的生命周期直接被其所属的Activity的生命周期控制。
 
## Fragment设计初衷 ##
 Android 3.0引入Fragment的初衷是为了使用屏幕的平板电脑，由于平板电脑的屏幕比手机屏幕更大，因此可以容纳更多的UI组件，且这些UI组件之间存在交互关系。Fragment简化了大屏幕UI的设计，它不需要开发者管理组件包换关系的复杂变化，开发者使用Fragment对UI组件进行分组、模块化管理，把一个Activity分解成不同的Fragment，就可以更方便的在运行过程中动态更新Activity的用户界面。

例如，如下图所示：
![在平板电脑中怎样把Fragment定义的两个UI模块组合到一个Activity，而在手持设备中又是如何分开例子。](http://img.blog.csdn.net/20150821193839469)

如图所示的因为浏览界面，该界面需要在屏幕左边显示新闻列表，并在屏幕右边显示新闻内容，此时可以在Activity中显示两个并排的Fragment：左边的Fragment显示新闻列表，右边的Fragment显示新闻内容。由于每个Fragment都拥有自己的生命周期，并可相应输入事件，因此可非常方面的实现：当用户单击左边列表中的指定新闻列表项时，右边的Fragment会显示相应的新闻内容。如图左边的平板电脑显示了这种UI界面。

通过上面的Fragment设计机制，可以取代传统的让一个Activity显示新闻列表，另一个Activity显示新闻内容的设计。

***由于Fragment是可复用的组件***，因此如果需要在正常尺寸的手机屏幕上运行该应用，则可改为使用两个Activity：ActivityA包含FragmentA、ActivityB包含FragmentB。其中ActivityA仅包含显示新闻列表的FragmentA，而当用户选择一个新闻项时，它会启动包含新闻内容的ActivityB。如图右边所示，由此可见，Fragment可以很好的支持如图所示的两种设计模式。