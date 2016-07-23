---
title: opensips+csipsimple出现的各种问题
date: 2016-07-23 20:22:55
tags: VoIP
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

### 1.同一路由后，无法打通电话。
经过抓包分析研究，出现这个问题，是因为由于客户端启用了ICE和stun，导致ICE候选中的列表过长，超过了标准SIP信令的长度。
解决办法：
打开csipsimple中的Use compact SIP选项。
这样客户端发出了SIP消息就会使用简称，c，m这种格式，不过可能不支持老的sip服务器。

### 2.穿越各种NAT。
有几点需要注意：

- 客户端要同时开启stun和ICE；
- opensips配置好mediaproxy模块，启动mediaproxy服务（包括dispatcher和relay）；
- opensips.cfg中添加use_media_proxy()和end_media_session()。(在invite请求时开启media proxy即可，一般不用end)

转载自：[opensips+csipsimple出现的各种问题](http://blog.csdn.net/zaker139/article/details/24311887)
