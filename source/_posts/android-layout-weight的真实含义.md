---
title: 'android:layout_weight的真实含义'
date: 2016-07-23 23:28:12
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

首先声明只有在Linearlayout中，该属性才有效。

> android:layout_weight的真实含义是:一旦View设置了该属性(假设有效的情况下)，那么该
> View的宽度等于原有宽度(android:layout_width)加上剩余空间的占比！

很多人不知道剩余空间是个什么概念，下面 我先来说说剩余空间。
假设手机屏幕的高度为L。

1.当layout_hieght="wrap_content"时：
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:divider="#f40808">

    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="One"/>
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Two"/>
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Three"/>
</LinearLayout>

```
上面布局的剩余空间的高度为：
![这里写图片描述](http://img.blog.csdn.net/20151210153945916)

系统给上面的3个EditText分配它们的高度值为wrap_content(高度足以包含它们的内容"One"、"Two"、"Three"即可),则剩余空间为：L-3*wrap_content。

2.当layout_hieght="match_parent"时：
上面三个EditText的每个高度为 L，则剩余空间的高度为L-3*L = -2L。


----------


下面我就来讲，Layout_weight这个属性的真正的意思：
(1)当layout_hieght="wrap_content"时:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal"
    android:divider="#f40808">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="#f20303"
        android:layout_weight="1"
        android:text="One"/>
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="#064df2"
        android:layout_weight="2"
        android:text="Two"/>
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="#f5f909"
        android:layout_weight="3"
        android:text="Three"/>
</LinearLayout>

```
效果图：
![这里写图片描述](http://img.blog.csdn.net/20151210160833528)

我们假设手机屏幕的宽度为L，则剩余空间的宽度freeWidth为L-3*wrap_content，
![这里写图片描述](http://img.blog.csdn.net/20151210161353581)

根据计算公式：
TextView(One)的宽度为：wrap_content+(1/6)*freeWidth.
TextView(Two)的宽度为：wrap_content+(2/6)*freeWidth.
TextView(Three)的宽度为：wrap_content+(3/6)*freeWidth.

2.当layout_hieght="match_parent"时：
则剩余空间的宽度freeWidth为：L-3*L = -2L；
根据计算公式：
TextView(One)的宽度为：L+(1/6)*freeWidth = (4/6)L.
TextView(Two)的宽度为：L+(2/6)*freeWidth = (2/6)L.
TextView(Three)的宽度为：L+(3/6)*freeWidth = 0L.

效果如下：
![这里写图片描述](http://img.blog.csdn.net/20151210162348455)

(3)当没有设置 Layout_weight这个属性时，它默认值是0：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal"
    android:divider="#f40808">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="#f20303"
        android:text="One"/>
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="#064df2"
        android:layout_weight="1"
        android:text="Two"/>
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="#f5f909"
        android:text="Three"/>
</LinearLayout>

```

效果图：
![这里写图片描述](http://img.blog.csdn.net/20151210162742080)

因为TextView(One)和TextView(Two)没有设置Layout_weight属性所以默认为0,则将剩余空间全部分配给TextView(Two);

总结：
Google官方推荐，当使用weight属性时，将width设为0dip即可，效果跟设成wrap_content是一样的。这样weight就可以理解为占比了！

参考自：[Android：Layout_weight的深刻理解](http://www.android100.org/html/201310/22/4536.html)