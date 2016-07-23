---
title: openSIPS路由类型
date: 2016-07-23 20:29:18
tags: VoIP
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

OpenSIPS路由规则使用几种类型的路由。每种路由是被一种特定事件触发，并且允许你处理一种确定类型的消息（请求或者应答）。

## 1.route(主路由)

请求路由块。它包含对SIP请求采取的一些列动作。

触发条件：SIP请求

处理：SIP请求

类型：无状态的初始请求，可以通过TM模块函数变为有状态的。

默认动作：如果请求没有被转发或者应答，路由会在最后把路由简单的丢弃。

主路由块通过代码route{...}或route[0]{...}(route[relay])来定义，用来处理每个SIP请求。

The implicit action after execution of the main route block is to drop the SIP request. To send a reply or forward the request, explicit actions must be called inside the route block.
执行完主路由块之后的隐式动作是(drop the SIP request是啥意思？)放弃该SIP请求。为了发送一个应答或者转发该请求，明确的动作（其实就是应答或者转发请求）必须在主路由中被调用。

使用示例:

```
route 
{
    if(is_method("OPTIONS")) 
    {
        # send reply for each options request
        sl_send_reply("200", "ok");
        exit();
    }
    route(1);
}

route[1] 
{
    # forward according to uri
    forward();
}
```


注意：如果route(X)在branch_route[Y]中调用，route[X]只是处理每一个单独的分支，而不像发生在主路由中一样，处理所有的分支。


----------
## 2.branch_route(分支路由)

请求的分支路由块。它包含对SIP请求的每个分支采取的一些列动作。

触发条件：准备一个请求的新分支，这个分支已经完整但并未被发送出去

处理：带有分支特征的SIP请求，比如requset URI，分支标志等

类型：有状态

默认动作 : if the branch is not dropped (via "drop" statement), the branch will be automatically sent out.

It is executed only by TM module after it was armed
 via t_on_branch("branch_route_index").

使用示例:
```
    route 
    {
        lookup("location");
        t_on_branch("1");
        if(!t_relay()) 
        {
            sl_send_reply("500", "relaying failed");
        }
    }
    
    branch_route[1] 
    {
        if(uri=~"10\.10\.10\.10") 
        {
            # discard branches that go to 10.10.10.10
            drop();
        }
    }
```


----------
## 3.failure_route(失败路由)

错误处理路由快。它包含对所有分支的每个事务收到错误（>=300）应答时所采取的一系列动作。
Triggered by : receiving or generation(internal) of a negative reply that completes the transaction (all branches are terminated with negative replies)
触发条件：收到或生成错误(>=300)应答

处理：被发送出去的初始SIP请求

类型：有状态

默认动作：如果里面没有新的分支生成（走新的分支路由），或者没有应答被强制结束，将获得的回复发送回UAC。

The 'failure_route' is executed only by TM module after it was armed via t_on_failure("failure_route_index").

Note that inside the 'failure_route', the request that initiated the transaction is being processed, and not its reply.

使用示例:

```
    route 
    {
        lookup("location");
        t_on_failure("1");
        if(!t_relay()) 
        {
            sl_send_reply("500", "relaying failed");
        }
    }
    
    failure_route[1] 
    {
        if(is_method("INVITE")) 
        {
             # call failed - relay to voice mail
             t_relay("udp:voicemail.server.com:5060");
        }
    }
```


----------
## onreply_route(应答路由)
应答路由块，它包含对SIP应答采取的一系列动作。

触发条件：接收到应答。

处理：接收到的应答。

类型：如果绑定到事务就是有状态的，如果是全局应答则是无状态的。

默认动作: 如果应答没有被丢弃（只有临时应答可能会），它将会由事务引擎来处理。

有三种应答路由类型：

- global(全局应答路由) 
它捕获所有被Opensips接收的应答， 不需要特殊操作（简单定义就行)。
使用onreply_route {...}或onreply_route[0] {...}.

- per request/transaction(每个请求/事务应答路由) 
它捕获一个特定事务的应答路由，需要在请求的时候通过t_on_reply(）来武装, 而实际则调用onreply_route[N] {...}。

- per branch(每个分支应答路由)
它捕获一个特定分支的应答路由，需要在请求的时候通过t_on_reply(）来武装, 而实际则调用onreply_route[N] {...}。


使用示例：
```
route 
{
        seturi("sip:bob@opensips.org");  # first branch
        append_branch("sip:alice@opensips.org"); # second branch

        t_on_reply("global"); # the "global" reply route
                              # is set the whole transaction
        t_on_branch("1");

        t_relay();
}

branch_route[1] 
{
        if ($rU=="alice")
                t_on_reply("alice"); # the "alice" reply route
                                      # is set only for second branch
}

onreply_route 
{
        xlog("OpenSIPS received a reply from $si\n");
}

onreply_route[global] 
{
        if (t_check_status("1[0-9][0-9]")) 
        {
                setflag(1);
                log("provisional reply received\n");
                if (t_check_status("183"))
                        drop;
        }
}

onreply_route[alice] 
{
        xlog("received reply on the branch from alice\n");
}

```


----------
## 5. error_route(错误路由)

当在SIP请求处理过程中发生一个解析错误(parsing error)时，或当一个脚本断言失败时，错误路由会自动执行。在这种错误情况下它允许管理员来决定做什么。

触发条件：解析错误(parsing error)

处理：失败的请求

状态：无状态

默认动作：丢弃请求

在错误路由中，下面的伪变量(pseudo-variables)可以获得错误详情：

- $(err.class) - 错误的类型 (now is '1' for parsing errors)
- $(err.level) - 错误的严重级别
- $(err.info) - 错误的信息
- $(err.rcode) - recommended reply code
- $(err.rreason) - recommended reply reason phrase

```
  error_route 
  {
     xlog("--- error route class=$(err.class) level=$(err.level) info=$(err.info) rcode=$(err.rcode) rreason=$(err.rreason) ---\n");
     xlog("--- error from [$si:$sp]\n+++++\n$mb\n++++\n");
     sl_send_reply("$err.rcode", "$err.rreason");
     exit;
  }
```


----------
## 6.local_route(本地路由)

当一个新的SIP请求被TM模块(而非UAC)产生时，本地路由会被自动执行。它用来做消息检查，账户认证和更新消息头。路由和信令函数不能在这里面执行。

触发条件：TM模块生成一个分支的新请求

处理：新的请求

类型：有状态

默认动作：发情请求
使用示例：
```
  local_route 
  {
     if (is_method("INVITE") && $ru=~"@foreign.com") 
     {
        append_hf("P-hint: foreign request\r\n");
        exit;
     }
     if (is_method("BYE") ) 
     {
        acc_log_request("internally generated BYE");
     }
  }
```


----------
## 7.startup_route(启动路由)

当OpenSIPS启动后并且在处理SIP消息开始前，启动路由只执行一次。这些是有用的，如果需要一些初始化动作，像加载一些数据到内存来缓和未来的处理。

注意：启动路由和其它路由相比，他不是在接收到消息时触发，能在启动路由中调用的函数不能够处理SIP消息。

触发条件：OpenSIPs启动后，监听进程运行前。

处理：初始化函数

使用示例：
```
  startup_route 
  {
    avp_db_query("select gwlist where ruleid==1",$avp(i:100));
    cache_store("local", "rule1", "$avp(i:100)");
  }
```


----------
## 8.timer_route(定时路由)

定时路由，顾名思义，在它配置的时间间隔内定时执行。和启动路由一样，它不处理SIP消息。你可以定义多个定时路由。

触发条件：定时器

处理：做刷新动作的函数

使用示例：
```
  timer_route[gw_update, 300] 
  {
    avp_db_query("select gwlist where ruleid==1",$avp(i:100));
    $shv(i:100) =$avp(i:100);
  }
```


----------
## 9.event_route(事件路由)

当一个事件触发时，事件路由被OpenSIPS Event Interface用来执行脚本。路由的名字是被路由处理的事件。路由自动订阅该事件。自动OpenSIPS 2.1开始，事件处理的方法可以从路由定义中指定，就像下面要展示的实例一样。如果没有方法处理指定的事件，将使用默认方法。关键字"sync"和"async"可以被使用。

触发条件：当一个事件被OpenSIPS Event Interface引发时，由event_route模块触发。

处理：该触发的事件

类型：无状态

默认动作： 当事件引发时，不执行任何脚本
使用示例：
```
 event_route[E_PIKE_BLOCKED] 
 {
    xlog("The E_PIKE_BLOCKED event was raised\n");
 }
 
 event_route[E_PIKE_BLOCKED, async] 
 {
    xlog("The E_PIKE_BLOCKED event was raised\n");
 }
```


----------

