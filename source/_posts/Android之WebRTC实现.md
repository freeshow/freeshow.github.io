---
title: Android之WebRTC实现
date: 2016-07-23 22:18:54
tags: WebRTC
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

### APK功能介绍：

1. 实现消息发送
2.  实现视频通话

效果图：

1.登录界面：
![这里写图片描述](http://img.blog.csdn.net/20160125201404382)

2.主界面(联系人列表界面)
![这里写图片描述](http://img.blog.csdn.net/20160125201515560)

3. 发送消息界面
![这里写图片描述](http://img.blog.csdn.net/20160125201538779)

只实现了功能，界面做的很丑，哈哈~~~~

点击联系人可以发送消息，单击电话按钮可以视频通话。

### 主要技术介绍

1.消息发送技术，主要采用了XMPP协议(openfire服务器+Smack API 实现)
如果不懂，可以看我后面准备要写的关于XMPP博客。

2.视频通话技术，即使用的WebRTC的技术，我转载的那几篇关于WebRTC的博客感觉已经讲的很清楚了。

### 代码实现
等过段时间，就会上传代码，稍等 哈哈


