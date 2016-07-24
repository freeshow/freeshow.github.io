---
title: Python爬虫之urllib2介绍
toc: false
date: 2016-07-24 21:01:36
tags: Crawler
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 一、urllib与urllib2的区别
- urllib2提供了比urllib更丰富的功能。
- urllib2.Request - 提供http hander定制功能。
- 提供更强大的功能，包括cookie处理，鉴权，可定制话等。
- urllib2能不能完全替代urllib? --不能，需要用到urllib.encoding()函数。

### 1.urllib2.urlopen()


```
urlopen(url, data=None, timeout=<object object>, cafile=None, capath=None, cadefault=False, context=None)
```
比urllib.urlopen()中多了一个timeout参数。

timeout：超时时间

比如，设置timeout为3秒，则3秒之内连接不到服务器，则报timeout错误。


###  2.urllib2.Request()

```
class Request:
    Methods defined here:
    __getattr__(self, attr)
    __init__(self, url, data=None, headers={}, origin_req_host=None,         unverifiable=False)
    add_data(self, data)
    add_header(self, key, val)
    add_unredirected_header(self, key, val)
    get_data(self)
    get_full_url(self)
    get_header(self, header_name, default=None)
    get_host(self)
    get_method(self)
    get_origin_req_host(self)
    get_selector(self)
    get_type(self)
    has_data(self)
    has_header(self, header_name)
    has_proxy(self)
    header_items(self)
    is_unverifiable(self)
    set_proxy(self, host, type
```

参数：
- url
- data - optional
- headers: 字典

使用Request添加或修改http头：
- Accept: application/json
- Content-Type: application/json
- User-Agent: Chrome

---


实例：

```
# -*- coding: utf-8 -*-
import urllib2

def urlopen():
    #网址不存在，则请求超时。
    url = 'http://blog.kamidox.com/no-exits'
    try:
        s = urllib2.urlopen(url,timeout=3)
    except urllib2.HTTPError,e:
        print e
    else:
        print s.read(100)
        s.close()

def request():
    url = 'http://blog.kamidox.com'
    #定制http头
    headers = {'User-Agent':'Mozilla/5.0','x-my-hander':'my value'}
    req = urllib2.Request(url,headers=headers)
    response = urllib2.urlopen(req)
    print response.read(100)
    response.close()

if __name__ == '__main__':
    urlopen()
    request()
```


---

### 3.urllib2.build_openner


```
1.build_opener(*handlers)
Create an opener object from a list of handlers.
 
The opener will use several default handlers, including support
for HTTP, FTP and when applicable, HTTPS.
 
If any of the handlers passed as arguments are subclasses of the
default handlers, the default handlers will not be used.

2.install_opener(opener)
```
- 可以定制http的行为。

- 参数：Handler列表
- 返回OpenerDirector

#### (1)BaseHandler及其子类
- HTTPHandler
- HTTPSHandler
- HTTPCookieProcessor

#### (2)当调用urllib2.urlopen()时，默认会创建的Handler链：
urllib2.build_opener()不传参数时，也会创建默认的Handler链。
- ProxyHandler(如果设置了代理)
- UnknownHandler
- HTTPHandler
- HTTPDefaultErrorHandler
- HTTPRedirectHandler
- FTPHandler
- HTTPErrorProcessor
- HTTPSHandler(如果安装了ssl模块)


#### 实例1:打印Http调试信息

```
# -*- coding: utf-8 -*-
import urllib2
import urllib

def request_post_debug():
    url = 'http://www.douban.com'
    #POST
    data = {'username':'kamidox','password':'xxxxxxxx'}
    headlers = {'User-Agent':'Mozilla/5.0'}
    req = urllib2.Request(url,data=urllib.urlencode(data),headers=headlers)
    
    #打印http调试信息
    #debuglevel会把交互的信息都打印出来
    opener = urllib2.build_opener(urllib2.HTTPHandler(debuglevel=1))
    response = opener.open(req)
    print response.read(100)
    response.close()

if __name__ == '__main__':
    request_post_debug()
    
#输出：
#send: 'POST / HTTP/1.1\r\nAccept-Encoding: identity\r\nContent-Length: 34\r\n
# Host: www.douban.com\r\nContent-Type: application/x-www-form-urlencoded\r\n
# Connection: close\r\nUser-Agent: Mozilla/5.0\r\n\r\nusername=kamidox&
# password=xxxxxxxx'
# reply: 'HTTP/1.1 301 Moved Permanently\r\n'
# header: Date: Fri, 10 Jun 2016 12:57:26 GMT
# header: Content-Type: text/html
# header: Content-Length: 178
# header: Connection: close
# header: Location: https://www.douban.com/
# header: Server: dae

# <!DOCTYPE HTML>
# <html lang="zh-cms-Hans" class="">
# <head>
# <meta charset="UTF-8">
# <meta name="descrip

```

#### (3)将自己创建的opener保存为默认：
urllib2.install_opener(opener):会将自己创建的opener安装到urllib2中，相当于用自己创建的opener替换了urllib2中的默认Handler,当使用urllib2.urlopen()时，使用的是
install_openner(opener)中自己定义的openner.

上面实例1:打印Http调试信息，相当于创建了一个局部的opener,而urllib2.install_openner(opener)相当于创建了一个全局的opener.

#### 实例：

```
# -*- coding: utf-8 -*-
import urllib2
import urllib

def request():
    url = 'http://blog.kamidox.com'
    #定制http头
    headers = {'User-Agent':'Mozilla/5.0','x-my-hander':'my value'}
    req = urllib2.Request(url,headers=headers)
    response = urllib2.urlopen(req)
    print response.read(100)
    response.close()

def request_post_debug():
    url = 'http://www.douban.com'
    #POST
    data = {'username':'kamidox','password':'xxxxxxxx'}
    headlers = {'User-Agent':'Mozilla/5.0'}
    req = urllib2.Request(url,data=urllib.urlencode(data),headers=headlers)

    #打印http调试信息
    #debuglevel会把交互的信息都打印出来
    opener = urllib2.build_opener(urllib2.HTTPHandler(debuglevel=1))
    response = opener.open(req)
    print response.read(100)
    response.close()

def install_debug_handler():
    opener = urllib2.build_opener(urllib2.HTTPHandler(debuglevel=1),
                                  urllib2.HTTPSHandler(debuglevel=1))
    urllib2.install_opener(opener)

if __name__ == '__main__':
    #request_post_debug()
    install_debug_handler()
    #request()会使用urllib2.install_opener(opener)中的opener
    request()

```

### 4.cookies处理
cookies由服务器生成，并发送给客户端保存；当客户端再次发送请求时，会连保存的cookies一起发送。

#### (1)cookielib.CookieJar
提供解析并保存cookie的接口。

#### (2)urllib2.HTTPCookieProcessor 继承自BaseHandler
提供自动处理cookie的功能

#### (3)实例：

```
# -*- coding: utf-8 -*-
import urllib2
import cookielib

def handler_cookie():
    cookiejar = cookielib.CookieJar()
    cookieHandler = urllib2.HTTPCookieProcessor(cookiejar=cookiejar)
    opener = urllib2.build_opener(cookieHandler,urllib2.HTTPHandler(debuglevel=1))
    response = opener.open('http://www.douban.com/')
    print response.read(100)
    response.close()

    #请求之后cookiejar中就包含了服务器中返回的cookies信息
    print cookiejar._cookies

    #当再次发送请求时，会在请求中添加cookies信息。
    response = opener.open('http://www.douban.com')
    response.close()

if __name__ == '__main__':
    handler_cookie()

```

#### (4)问题思考
- 用cookie来模拟登陆？
- 验证码问题？




