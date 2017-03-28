---
title: SIP路由字段和机理
date: 2016-07-23 20:24:14
tags: SIP
categories: 网络通信
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## From 
如果一个SIP消息中没有Contact或者Record-Route头域，那么callee就会根据From头域产生后续的Request。

比如,如果Alice打一个电话给Bob,From头域的内容是 From:Alice<sip:alice@example.org>。那么Bob打给Alice时就会使用 sip:alice@example.org作为To头域和Request-URI头域的内容。 

## Contact 
后续Request将根据Contact头域的内容决定目的地的地址，同时将Contact头域的内容放到Request-URI中。它还可以用来指示没有在Record-Route头域中记录的Proxies的地址。同时它还可以被用在Redirect servers和REGISTER requests 和responses。

## Record-Route
Record-Route字段实际上用于帮助UA建立Route Set，当UA发送Request时会用Route Set来设置上面提到的Route字段。当一个Request消息经过Proxy Server时，如果该Proxy Server希望通知UA相关的后续消息都能通过它来转发，此时它就会在消息中添加Record-Route字段，内容为自己的地址信息。当UAS发送Resposne消息时它将复制Request中的Record-Route字段，而UAC在Response消息中检测到Record-Route字段时，它就会用该字段的路由信息更新自己的Route Set 。

## Via 
Via头域是被服务器插入request中，用来检查路由环的，并且可以使response根据via找到返回的路。<font color=red>它不会对未来的request 或者是response造成影响。 </font>

总的来说，如果有Route，request就应该根据Route发送，如果没有就根据Contact头域发送，如果连Contact都没有，就根据From头域发送。

## Service-Route
Service-Route在S-CSCF向UE发送REGISTER成功应答时设置，作用和Record-Route类似，用于帮助UE建立Route Set，这样UE注册后的消息（例如INVITE）通过设置Route字段无需经过I-CSCF可直接送达S-CSCF。

## Path 
只能用于用户向注册服务器发送的Register请求。
1.如果某代理服务器希望发往用户的任何后续请求仍能经过自己，就可以在Register请求中插入一个Path字段并赋值为自身的URI。
2.如果要求拓扑隐藏，经过I-CSCF的时候要把这个 I 添加到Path字段中.

## To
To 字段总是包含被呼叫方的地址（通过sip代理时是公用地址，点对点时是真实ip），要注意的是区别该标题头和sip消息请求行中的Request-URI。To在信令路径中不会被代理改变，然而Request-URI包含的是信令路径中下一跳的地址，因此在路途中被每个代理改变。



总的来说，SIP中存在两种路由场景：
1.请求消息的路由
2.响应消息的路由 

其中，响应消息的路由非常简单，就是完全依靠Via来完成的。
【说明】一个SIP消息每经过一个Proxy（包括主叫），都会被加上一个Via头域，当消息到达被叫后，Via头域就记录了请求消息经过的完整路径。被叫将这些Via头域原样copy到响应消息中（包括各Via的参数，以及各Via的顺序），然后下发给第一个Via中的URI，每个Proxy转发响应消息前都会把第一个Via（也就是它自己添加的Via）删除，然后将消息转发给新的第一个Via中的URI，直到消息到达主叫。

**下面谈SIP请求消息的路由。**
 
首先我们要搞清楚什么是严格路由和松散路由。

## 严格路由（Strict Routing）
可以理解为比较“死板”的理由机制，这种路由机制在SIP协议的前身RFC 2534中定义，其机制非常简单。<font color=red>要求接收方接收到的消息的request-URI必须是自己的URI(即发送方的Request-URI always contained URI of the next hop)，然后它会把第一个Route头域“弹”出来，并把其中的URI作为新的request-RUI，然后把该消息路由给该URI。</font>

## 松散路由（Louse Routing，lr）：
该路由机制较为灵活，也是SIP路由机制的灵魂所在，在RFC 3261中定义。
下面介绍一下一个松散路由的Proxy的路由决策过程：
 
1，Proxy首先会检查消息的request-URI是不是自己属于自己所负责的域。如果是，它就会通过定位服务将该地址“翻译”成具体的联系地址并以此替换掉原来的request-URI；否则，它不会动request-URI。
 
2，Proxy检查第一个Route头域中的URI是不是自己的，如果是，则移除之。
 

3、Loose Router首先会检查Request URI是否为自己：如果不是，则不作处理；如果是，则取出Route字段的最后一个地址作为Request URI地址，并从Route字段中删去最后一个地址。

4、Loose Router其次会检查下一跳是否为Strict Router：如果不是，则不作处理；如果是，则将Request URI添加为Route的最后一个字段，并用下一跳Strict Router的地址更新Request URI。

可以看到步骤3、4其实是Loose Router为了兼容Strict Router而做的额外工作。

前面都是准备工作，下面该进行真正的路由了。如果还有Route头域，则Proxy会把消息路由给该头域中的URI，否则就路由给request-URI。
 
对于前面的规则，可以简单总结为一句话：<font color=red>Route的优先级高于request-URI的。</font>

好，了解了两种路由机制，我们再来了解一下Route和Record-Route。

<font color=red>如果说Via是为了给一个请求消息的响应消息留后路，那么Record-Route就是为了给该请求消息之后的请求消息留后路。</font>

而在一个请求消息的传输过程中，Proxy也可能（纯粹自愿，如果它希望还能接收到本次会话的后续请求消息的话）会添加一个Record-Route头域，这样当消息到达被叫后里面就有会有0个或若干个Record-Route头域。被叫会将这些Record-Route头域并入路由集，并并入自己的路由集，随后被叫在发送请求消息时就会使用该路由集构造一系列Route头域，以便对消息进行路由。然后，被叫会像上面对待Via头域一样，将Record-Route头域全部原样copy到响应消息中返回给主叫。
    
主叫收到响应消息后也会将这些Record-Route头域并入路由集，只是它会将其反序。该会话中的后续请求消息的Route头域就会通过路由集构造。

<font color=red>【注意】</font>Record-Route头域不用来路由，而只是起到传递信息的作用。
Record-Route头域不是路由集的唯一来源，路由集还可以通过手工配置等方式得到。


只是描述还是比较抽象，下面就以RFC 3261中的两个实例来解释一下。
 
路由示例1：
 
场景：                 
两个UE间有两个Proxy，U1 -> P1 -> P2 -> U2，并且两个Proxy都乐意添加Record-Route头域。
U1( u1.example.com)
P1(p1.example.com)
P2(p2.domain.com)
U2(u2.domain.com)
 　　　　　　　　　　　 　　　
消息流：
【说明】由于我们在此只关心SIP路由机制，因此下面消息中跟路由机制无关的头域都省略了。

U1发出一个INVITE请求给P1（P1是U1的外拨代理服务器）：
      

```
(U1-->P1)
INVITE sip:callee@domain.com SIP/2.0
Contact: sip:caller@u1.example.com
```

 
P1不负责域domain.com，消息中也没有Route头域，因此通过DNS查询得到负责该域的Proxy的地址并且把消息转发过去。这里P1在转发前就添加了一个Record-Route头域，里面有一个lr参数，说明P1是一个松散路由器，遵循RFC3261中的路由机制。

```
(P1--->P2)
INVITE sip:callee@domain.com SIP/2.0
Contact: sip:caller@u1.example.com
Record-Route: <sip:p1.example.com;lr>
```

<font color=red>P2负责域domain.com，因此它通过定位服务得到callee@domain.com 对应的设备地址是callee@u2.domain.com ，因此用新的URI重写request-URI。</font>消息中没有Route头域，因此它就把该消息转发给request-URI中的URI，转发前它也增加了一个Record-Route头域，并且也有lr参数。

```
(P2-->U2)
INVITE sip:callee@u2.domain.com SIP/2.0
Contact: sip:caller@u1.example.com
Record-Route: <sip:p2.domain.com;lr>
Record-Route: <sip:p1.example.com;lr>
```

位于u2.domain.com的被叫收到了该INVITE消息，并且返回一个200 OK响应。其中就包括了INVITE中的Record-Route头域。

```
SIP/2.0 200 OK
Contact: sip:callee@u2.domain.com
Record-Route: <sip:p2.domain.com;lr>
Record-Route: <sip:p1.example.com;lr>
```

被叫此时也就有了自己的路由集：

```
(<sip:p2.domain.com;lr>,<sip:p1.example.com;lr>)
```

<font color=red>并且它本次会话的远端目的地址设置为INVITE中Contact中URI：caller@u1.example.com，此后被叫在该会话中的请求消息就发到这个URI。同样，被叫在200 OK响应中也携带了自己的联系地址，主叫收到该响应消息后也会把本次会话的远端目的地址设置为：callee@u2.domain.com，此后主机在该会话中的请求消息就发到这个URI。</font>
同样，主叫也有了自己的路由集，只是跟被叫的是反序的：

```
(<sip:p1.example.com;lr>,<sip:p2.domain.com;lr>)
```

通话完毕后，我们架设主叫先挂机，则主叫发出BYE请求：
 

```
BYE sip:callee@u2.domain.com SIP/2.0
Route: <sip:p1.example.com;lr>,<sip:p2.domain.com;lr>
```

可以看到，BYE的Route头域正是主机的路由集构造来的。
由于p1在第一个Route中，因此BYE首先发给P1。
 
P1收到该消息后，发现request-URI中的URI不属于自己负责的域，而消息有Route头域，并且第一个Route头域中的URI正是自己，因此删除之，并且把消息转发给新的第一个Route头域中的URI，也就是P2：

```
BYE sip:callee@u2.domain.com SIP/2.0
Route: <sip:p2.domain.com;lr>
```

P2收到该消息后，发现request-URI中的URI不属于自己负责的域（P2负责的是domain.com，而不是u2.domain.com），第一个Route头域中的URI正是自己，因此删除之，此时已经没有Route头域了，因此就转发给了request-URI中的URI。
 
被叫就会收到BYE消息：
  

```
BYE sip:callee@u2.domain.com SIP/2.0
```


----------
路由示例2：
如果说上面的示例主要关注的是路由流程，那么本示例关注的则是严格路由与松散路由的区别。
 
场景：
U1->P1->P2->P3->P4->U2
其中，P3是严格路由的，其余Proxy都是松散路由的，并且4个Proxy都很乐意增加Record-Route头域。
 
消息流：
我们直接给出了到达被叫的INVITE消息：

```
INVITE sip:callee@u2.domain.com SIP/2.0
Contact: sip:caller@u1.example.com
Record-Route: <sip:p4.domain.com;lr>
Record-Route: <sip:p3.middle.com>
Record-Route: <sip:p2.example.com;lr>
Record-Route: <sip:p1.example.com;lr>
```

 
这中间的其他消息我们就不过问了，直接看一下被叫最后发出的BYE消息大概是什么样子：

```
BYE sip:caller@u1.example.com SIP/2.0
Route: <sip:p4.domain.com;lr>
Route: <sip:p3.middle.com>
Route: <sip:p2.example.com;lr>
Route: <sip:p1.example.com;lr>
```

 
因为P4在第一个Route里，因此被叫将BYE消息发给了P4。

P4收到该消息后，发现自己不负责域u1.example.com，但是第一个Route头域中的URI正是自己，因此删除之。<font color=red>P4还发现新的第一个Route头域中的URI是一个严格路由器，因此它把request-URI中的URI添加到最后一个Route的位置，并且将第一个Route“弹出”并且覆盖原来的request-URI。然后将消息转发给当前的request-URI，也就是P3。</font>

```
BYE sip:p3.middle.com SIP/2.0
Route: <sip:p2.example.com;lr>
Route: <sip:p1.example.com;lr>
Route: <sip:caller@u1.example.com>
```

 
P3收到该消息后，直接把消息作出如下变换并且发给P2：

```
BYE sip:p2.example.com;lr SIP/2.0
Route: <sip:p1.example.com;lr>
Route: <sip:caller@u1.example.com>
```

P2收到该消息后，发现消息中的request-URI是自己的，因此在进一步处理先首先对消息做如下变换：
 

```
BYE sip:caller@u1.example.com SIP/2.0
Route: <sip:p1.example.com;lr>
```

然后，P2发现自己不负责域u1.example.com，第一个Route中的URI也不是自己的，因此将消息转发给该URI，也就是P1。
 
P1收到该消息后，发现自己不负责域u1.example.com，但是第一个Route头域中的URI正是自己，因此删除之。消息变成下面的样子：

```
BYE sip:caller@u1.example.com SIP/2.0
```

 转载自：[SIP路由字段和机理](http://blog.csdn.net/dingpeng1978/article/details/2652380)
