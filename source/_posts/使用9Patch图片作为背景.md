---
title: 使用9Patch图片作为背景
date: 2016-07-23 23:26:11
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

将图片作为View组件的背景时，当View中要呈现的文字内容太多时，Android会自动缩放整张图片，以保证背景图片能覆盖整个View。但这种缩放图片的效果可能并不好。可能存在的情况是我们只想缩放图片的某个部分，这样才能保证View的视图效果。

为了实现只缩放图片中的某个部分效果，我们需要借助于9Patch图片来实现。9Patch图片是一种特殊的PNG图片，这种图片以.9.png结尾，它在原始图片四周各添加一个宽度为1像素的线条，这4条线就决定了该图片的缩放规则、内容显示规则。

左侧和上侧的直线共同决定了图片的缩放区域：以左边直线为左边界绘制矩形，它覆盖的区域可以在纵向上缩放；以上面直线为上边界绘制矩形，它覆盖的区域可以水平缩放；它们两者的交集可以在两个方向上缩放。

右侧和下侧的直线共同决定图片的内容显示区域：以右边直线为右边界绘制矩形，以下边直线为下边界绘制矩形，它们二者的交集就是图片内容显示区域。

Android为制作9Patch图片提供了draw9patch工具，该工具位于Android SDK安装路径的tools目录下，进入该目录双击draw9patch.bat文件，即可启动该工具。

启动draw9patch工具之后，通过工具主菜单上的"File"-"Open 9-patch"菜单项打开一张PNG图片，然后通过该工具定义图片的缩放区域和内容显示区域。

如下图两个类似于QQ显示聊天信息的消息文本：

显示左侧聊天信息：

![这里写图片描述](http://img.blog.csdn.net/20151210193514465)

显示右侧聊天信息：
![这里写图片描述](http://img.blog.csdn.net/20151210193619800)

这样就可以定义聊天信息的布局文件了：
左侧：

```
<TextView
    android:id="@+id/textview_chat_text_left"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    //chat_text_left.9.png为上面的左侧9Patch图片
    android:background="@drawable/chat_text_left"
    android:gravity="center_vertical"
    android:maxWidth="270dp"
    android:text="Chat_Left_Text"
    android:textAppearance="?android:attr/textAppearanceLarge"
/>
```


