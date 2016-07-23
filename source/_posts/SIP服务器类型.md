---
title: SIP服务器类型
date: 2016-07-23 20:34:19
tags: VoIP
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

有一些不同类型的SIP服务器。根据你的应用，你可以使用它们其中的一种或所有类型来解决你的问题。OpenSIPS可以作为代理服务器、重定向服务器、背靠背用户代理或者注册服务器。

## The proxy server(代理服务器)
在 SIP 代理模式下，所有的 IP 消息都要经过 SIP 代理。这种行为在向诸如计费（ billing）的过程中帮助很大，而且迄今为止，这也是一种最普遍的选择。但是它的缺点就是在会话建立过程中的所有的 SIP 交互中，服务器造成的额外开销也是客观的。要记住的是，即使服务器作为 SIP 代理在工作时， RTP 包也总是直接从一端传送到另一端，而不会经过服务器。

![proxy server](http://img.blog.csdn.net/20160404151303707)


----------
## The redirect server(重定向服务器)
SIP 代理可以运行在 SIP 重定向模式。在这种模式下， SIP 服务器的处理量是相当巨大的，因为它不需要保持事务处理的状态。在对 INVITE 消息进行初始化后，仅仅向 UAC 回复一条―302 Moved Temporarily‖消息就可以离开 SIP 对话（ dialog）了。在这种模式下的 SIP 代理，即使只是利用非常少的资源也可以每小时传送上百万的通话。当你需要的规模很大并且不需要对通话计费的情况下，这种模式通常会被使用。

![The redirect server](http://img.blog.csdn.net/20160404151756662)


----------
## The B2BUA server(背靠背用户代理服务器)
The server can also work as a Back-to-Back User Agent(B2BUA). B2BUAs are normally applied to hide the topology of the network(网络的拓扑结构). They are also useful to support buggy clients unable to route SIP requests correctly based on record routing. Many PBX systems such as Asterisk, FreeSwitch, Yate, and others work as B2BUAs.

想了解B2BUA的可以看《FreeSWITCH权威指南》一书，上面做了详细介绍。

![B2BUA](http://img.blog.csdn.net/20160404152922995)

注意：图中左边的Call-ID和右边的Call-ID不同(B2BUA Two Legs)。


----------

