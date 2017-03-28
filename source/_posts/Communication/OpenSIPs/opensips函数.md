---
title: opensips函数
date: 2016-07-23 20:28:01
tags: SIP
categories: 网络通信
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## append_branch()

Similarly to t_fork_to, it extends destination set by a new entry. The difference is that current URI is taken as new entry.

Without parameter, the function copies the current URI into a new branch. Thus, leaving the main branch (the URI) for further manipulation.(保留主分支（URI）做进行进一步的处理。)

With a parameter, the function copies the URI in the parameter into a new branch（这个函数将函数参数作为URI复制到新的分支）. Thus, the current URI is not manipulated.(当前URI不被操纵。)

Note that it's not possible to append a new branch in "on_failure_route" block if a 6XX response has been previously received (it would be against RFC 3261).

Example of usage:

```
    # if someone calls B, the call should be forwarded to C too.
    #
    if (method=="INVITE" && uri=~"sip:B@xx.xxx.xx ")
    {
        # copy the current branch (branches[0]) into
        # a new branch (branches[1])
        append_branch();
        # all URI manipulation functions work on branches[0]
        # thus, URI manipulation does not touch the 
        # appended branch (branches[1])
        seturi("sip:C@domain");

        # now: branch 0 = C@domain
        #      branch 1 = B@xx.xx.xx.xx

        # and if you need a third destination ...

        # copy the current branch (branches[0]) into
        # a new branch (branches[2])
        append_branch();

        # all URI manipulation functions work on branches[0]
        # thus, URI manipulation does not touch the 
        # appended branch (branches[1-2])
        seturi("sip:D@domain");

        # now: branch 0 = D@domain
        #      branch 1 = B@xx.xx.xx.xx
        #      branch 2 = C@domain

        t_relay();
        exit;
    };

    # You could also use append_branch("sip:C@domain") which adds a branch with the new URI:

    if(method=="INVITE" && uri=~"sip:B@xx.xxx.xx ") 
    {
        # append a new branch with the second destination
        append_branch("sip:user@domain");
        # now: branch 0 = B@xx.xx.xx.xx
        # now: branch 1 = C@domain

        t_relay();
        exit;
	}
```

##revert_uri()

Set the R-URI to the value of the R-URI as it was when the request was received by server (undo all changes of R-URI).

Example of usage:

```
    revert_uri();
```

## prefix(string)

Add the string parameter in front of username in R-URI.

Example of usage:
 

```
   prefix("00");
```

## rewritehost() / sethost()

Rewrite the domain part of the R-URI with the value of function's parameter. Other parts of the R-URI like username, port and URI parameters remain unchanged.

Example of usage:
   

```
 rewritehost("1.2.3.4");
```

## rewritehostport() / sethostport()

Rewrite the domain part and port of the R-URI with the value of function's parameter. Other parts of the R-URI like username and URI parameters remain unchanged.

Example of usage:

```
    rewritehostport("1.2.3.4:5080");
```

## rewriteuri(str) / seturi(str)

Rewrite the request URI.

Example of usage:
  

```
  rewriteuri("sip:test@opensips.org");
```

## t_relay()
Relay a message statefully to a fixed destination. The destination is specified as “[proto:]host[:port]”. If a destination URI “$du” for this message was set before the function is called then this value will be used as the destination instead of the function parameter.

The function may take as parameter an optional set of flags for controlling the internal behaviour - for details see the above “t_relay([flags])” function.

This functions can be used from REQUEST_ROUTE, FAILURE_ROUTE.

Example 1.20. t_relay usage


```
t_relay("tcp:192.168.1.10:5060");
t_relay("mydomain.com:5070","0x1");
t_relay("udp:mydomain.com");
```




