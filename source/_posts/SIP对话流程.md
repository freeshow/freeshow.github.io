---
title: SIP对话流程
date: 2016-07-23 20:35:49
tags: VoIP
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

这一节将通过一个简单的例子来介绍一些基本的 SIP 操作。先让我们来诊视下图展示的两个用户代理之间的消息顺序。你可以看到伴随这 RFC3665 描述的会话建立过程还有几个其它的流程。

![The SIP dialog flow](http://img.blog.csdn.net/20160404194016825)

我们在这些消息上标上了序号。在这个例子中用户 A 使用 IP 电话向网络上的另外一台IP电话发出通话请求。为了完成通话，使用了两个 SIP 代理。

![这里写图片描述](http://img.blog.csdn.net/20160404194214232)
此图中的Contact应该错了，Contact应该为userA。

第一行是消息的方法名（ the method name）和请求URI(request URI)。接下来是列出的头域。这个例子包含了所需要的头的最小集合。我们将在下面简要的描述这些头域。

- **Method and Request-URI**: 在第一行，request URI也可以被写作RURI。它包含了消息的当前目的地，它经常被代理服务器操作来路由请求。这是在SIP请求中最重要的域。

- **Via**: This contains the address to which userA will be waiting to receive responses to this request. It also contains a parameter called **branch**that identifies this transaction. The Via header defines the last SIP hop as IP,transport,and transaction-specific paramters. Via is used exclusively to route back the replies. Each proxy adds an additional Via header. It is a lot easier for replies to find their route back using the Via header than to go again in the location server or DNS.(这里不翻译，感觉比翻译更容易读懂)

- **To**: 它包含了名字（显示名(display name)）和最初选择的目的地的 SIP URI（这里是 sip： userB＠sip.com）。 <font color=red>To头域不被用来路由消息包</font>。

- **From**: 它包含了名字和表明主叫 ID（ caller ID）的 SIP URI（这里是 sip：userA@sip.com）。这个头域有一个 tag 参数，而这个参数包含了<font color=red>被 IP电话添加进</font>URI 的一个随机字符串。是被用来进行辨识唯一性的。 tag 参数被用在 TO 和 FROM头域中。作为一种普遍的机制用来标识对话（ dialog），对话是 CALL－ID 和两个tag的结合，<font color=red>而这两个 tag 分别来自参与对话的双方。</font>Tags 在并行派生（ parallel forking）中作用显著。

- **Call-ID**: 它包含了一个作为这通通话全局性的唯一的标识，而这个唯一标识是有一个随机字符串，来自 IP 电话的主机名或是 IP 地址结合而成的。 To， From的 tag和CALL-ID的结合完整的定义了一个端到端的 SIP 关系，这种关系就是我们所知道的 SIP 对话（ SIP dialog）.

- **CSeq**: CSEQ 或者称之为命令序列（ command sequence）包含了一个整数和一个方法名。 <font color=red>CSEQ 数对于每一个在 SIP 对话中的**新请求**都会递增</font>，是一个传统的序列数。

- **Contact**: This contains a SIP URI, which represents a direct route a contact userA, usually composed of a user name and **full qualified domain name(FQDN)**. It is usual to use the IP address instead of the FQDN in this field. <font color=red>While the **Via header field** tell the other elements where to send a response, the **Contact** tells other elements where to send future requests</font>.

- **Max-Forwards**: 它被用来限制请求在到达最终目的地的路径中被允许的最大跳数（ hops）。由一个整数构成，而这个整数在每一跳中将会递减。

- **Content-Type**: 它包含了对内容消息的描述。

- **Content-Length**: 它用来告知内容消息的字节数。

会话的一些细节，像媒体类型和编码方式并不是使用 SIP 进行描述的。而是使用叫做会话描述协议（ SDP RFC2327）来进行描述。 SDP 消息由 SIP 消承载，就像是一封电子邮件的附件一样。

话机开始并不知道用户 B 和负责域 B 的服务器的位置。因此，它向负责 sipA 域的服务器发送 INVITE 消息请求。发送地址在用户 A 的话机中进行设置或通过DNS发现。服务器 sipA.com 也就是我们知道的域 sipA.com 的 SIP 代理服务器。

过程如下：

1. 在这个例子中，代理服务器收到 INVITE 请求消息并发送―100 trying‖响应消息给用户 A，表明代理服务器已经收到了 INVITE 消息并正在转发这个求。 SIP 的响应消息使用一个三个数字组成的数字码和一条描述语句说明响应的类型。并拥有和INVITE 请求一样的 To、From、CALL－ID 和 CSEQ 等头域，以及 VIA 和其―branch参数。这就使得用户 A 的话机同发出的 INVITE 请求联系在一起。

2. 代理 A 定位代理 B 的方法是向 DNS 服务器（ SRV 记录）进行查询以找到负责sipB 的 SIP 域的服务器地址并将 INVITE 请求转发给它。在向代理B发送 INVITE 消息前，代理 A 将其自己的地址通过 VIA 头添加进 INVITE，INVITE请求在第一个Via头域汇总已经有了userA的地址。

3. 代理 B收到 INVITE 请求，给代理A返回100 Trying消息响应，表明其正在处理这个请求。 

4. 代理 B 查询自己的位置数据库以找到用户 B 的地址，然后将自己的地址也通过VIA 头域添加进 INVITE 消息发送给用户 B 的 IP 地址。<font color=red>(注：此时找到userB的真实IP地址，会将RURI中的地址替换为userB的真实IP地址)</font>。

5. 用户 B 的话机收到 INVITE 消息后开始振铃。话机为了要表明这种情况（振铃），发送回―180 Ringing‖响应消息。

6. 这个消息以相反的方向路由通过那两个代理服务器。每一个代理利用 VIA 头域来决定向哪里发送响应消息并从顶部将其自己的 VIA 头去除。结果就是， 180 Ringing 消息不需要任何的 DNS 查询，不需要定位服务的响应，也不需要任何的状态处理就能够返回到用户那里。这样的话，每一个代理服务器都能够看到由 INVITE 开始的所有消息。

7. 当用户 A 的话机收到180 Ringing响应消息后开始―回铃，表明另一端的用户正在振铃。一些话机是通过显示一些信息进行表示的。

8. 在这个例子中，用户 B 对对方发起的通话进行了响应。当用户 B 响应时，话机发送200 OK响应消息以表明通话被接起。200 OK的消息体中包含了会话的描述信息，这些信息包括指定了编码方式，端口号，以及从属于会话的所有事情。作这项工作的就是 SDP 协议。结果就是，在从 A 到 B（ INVITE）和从 B 到 A（ 200 OK）的两个阶段，双方交换了一些信息，以一种简单的**请求/响应的模式**协商了在这通通话中所需的资源和所需要的能力要求。如果用户 B 不想得到这通通话或是此刻处于忙线中， 200 OK 将不会发出，取代它的是描述这种状况（这里是 486 Busy Here）的消息。   


----------
![这里写图片描述](http://img.blog.csdn.net/20160404210114810)

 
 第一行是响应码和描述信息(OK)。接下来是头域行。 VIa， To， From， Call-ID和 CSEQ 是从 INVITE 请求中拷贝的。有三个 Via头，一个是用户 A 添加的，另一个是代理 A 添加的，最后一个则是代理 B 添加的。用户 B 的 SIP 话机在对话的双方加入了一个 **tag参数**，这个参数在这通通话的以后的请求和响应消息中都将出现。

Contact头域中包含了 URI 信息，这个 URI 信息是用户 B 能够直接被联系到他们自己的IP话机的地址。

Content-Type和Content-Length头域给出了关于 SDP头的一些信息。而 SDP 头则包含了用来建立 RTP 会话的媒体相关的参数。

当userB接听电话后将发生如下事情：

1. 在这个例子中，200 OK消息通过两个代理服务器被送回给用户 A，之后userA的话机停止回铃,表明通话被接起。

2.  最后用户 A 向用户 B 的话机发送 ACK 消息确认收到了200 OK消息。当不包含record routing时， ACK避开了两个代理服务器直接发送给用户 B。 ACK 是 SIP 中唯一不需要进行响应的消息请求。两端在 INVITE 的过程中从 Contact消息中了解双方的地址信息。也结束了 INVITE/200 OK/ACK 的过程，这个过程也就是我们所熟知的SIP三次握手。

3. 这个时候两个用户之间开始进行会话，他们以用 SDP 协议协商好的方式来发送媒体包。通常这些包是端对端进行传送的。在会话中，通话方可以通过发送一个新的 INVITE 请求来改变会话的一些特性。这叫做 re－invite。如果 re-invite 不被接受，那么488 Not Acceptable Her响应就会被发出，但是会话不会因此而失败。 

4. 要结束会话的时候，用户 B 产生 BYE 消息来中断通话。这个消息绕过两个代理服务器直接路由回用户 A 的软电话上。

5. 用户 A 发出200 OK响应消息以确认收到了 BYE 消息请求，从而结束会话。这里，不会发出 ACK。 ACK 只在 INVITE 请求过程中出现。


----------
有些情况下，在整个会话过程中，对于代理服务器来说，能够待在消息传输的中间位置来观察两端的所有消息交互是很重要的。如果代理服务器想在INVITE请求初始化完成后还待在此路径中，可以在请求消息中添加Record-Route头域。用户 B 的话机得到了这个消息，之后在其消息中也会带有这个头，并且会将消息发送回代理。记录路由（ Record Routing）在大多数的方案中都会被使用。

REGISTER 请求是代理B用来定位用户B的方法。当话机初始化的时候或是在通常的时间间隔中，软电话 B 向在域 sipB 中的一个服务器（ SIP REGISTRAR）发送 REGISTER请求。<font color=red>注册请求将 URI(< userB@sipB.com >) 与一个 IP 地址(userB的实际IP地址)联系在一起，这种绑定被存储在定位服务器上面的数据库里</font>。通常，注册服务器，定位服务器，和代理服务器在同一台物理机器上，并使用相同的软件。 OpenSIPS就能够扮演这三种角色。一个 URI 只能够在一个特定的时间内由一个单独的机器注册。

  







