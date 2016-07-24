---
title: Python爬虫之requests介绍
toc: false
date: 2016-07-24 21:05:18
tags: Crawler
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 一、基本介绍
[requsets官网地址](http://requests.readthedocs.io/en/latest/)

#### 和urllib、urllib2的区别：
- requests不是标准库
- 最好用的http库，pythonic风格

#### 安装：pip install requests


## 二、requests请求

### 1.requests.request(method, url, **kwargs)


```
requests.request(method, url, **kwargs)
    Constructs and sends a Request. Returns Response object.
```

**参数：**
- method -- method for the new Request object.(get/post/head/put/delete)
url -- URL for the new Request object.
- params -- (optional) Dictionary or bytes to be sent in the query string for the Request.(请求参数)
- data -- (optional) Dictionary, bytes, or file-like object to send in the body of the Request.
- json -- (optional) json data to send in the body of the Request.
- headers -- (optional) Dictionary of HTTP Headers to send with the Request.
- cookies -- (optional) Dict or CookieJar object to send with the Request.
- files -- (optional) Dictionary of 'name': file-like-objects (or {'name': file-tuple}) for multipart encoding upload. file-tuple can be a 2-tuple ('filename', fileobj), 3-tuple ('filename', fileobj, 'content_type') or a 4-tuple ('filename', fileobj, 'content_type', custom_headers), where 'content-type' is a string defining the content type of the given file and custom_headers a dict-like object containing additional headers to add for the file.
- auth -- (optional) Auth tuple to enable Basic/Digest/Custom HTTP Auth.
- timeout (float or tuple) -- (optional) How long to wait for the server to send data before giving up, as a float, or a (connect timeout, read timeout) tuple.
- allow_redirects (bool) -- (optional) Boolean. Set to True if POST/PUT/DELETE redirect following is allowed.
- proxies -- (optional) Dictionary mapping protocol to the URL of the proxy.
- verify -- (optional) whether the SSL cert will be verified. A CA_BUNDLE path can also be provided. Defaults to True.
- stream -- (optional) if False, the response content will be immediately downloaded.
- cert -- (optional) if String, path to ssl client cert file (.pem). If Tuple, ('cert', 'key') pair.(验证证书)


**Return**：requests.Response

**Usage**:

```
>>> import requests
>>> req = requests.request('GET', 'http://httpbin.org/get')
<Response [200]>
```

### 2.requests.get()

```
requests.get(url, params=None, **kwargs)
    Sends a GET request.
```

### 3.requests.post()

```
requests.post(url, data=None, json=None, **kwargs)
    Sends a POST request.
```


### 4.requests.head()

### 5.requests.put()

## 二、requests应答

- status_code：状态码
- headers：应答得http头
- json：应答得json数据
- text：应答得Unicode编码的文本
- content：应答得字节流数据
- cookies：应担的cookies，自动处理。

## 三、基本用法

```
# -*- coding: utf-8 -*-
import requests

def get_json():
    response = requests.get('https://api.github.com/events')
    print response.status_code
    #print response.headers
    #print response.content
    #print response.text
    print response.json()

def get_querystring():
    #http://httpbin.org是专门测试http的网站
    url = 'http://httpbin.org/get'
    params = {'qs1':'value1','qs2':'value2'}
    r = requests.get(url,params=params)
    print r.status_code
    print r.content

def get_custom_headers():
    url = 'http://httpbin.org/get'
    headers = {'x-header1':'value1','x-header2':'value2'}
    r = requests.get(url,headers=headers)
    print r.status_code
    print r.content

def get_cookie():
    url = 'http://www.douban.com'
    headers = {'User-Agent':'Chrome'}
    r = requests.get(url,headers=headers)
    print r.status_code
    print r.cookies
    #输出：<RequestsCookieJar[<Cookie bid=rnxfxUZLSxA for .douban.com/>,
    # <Cookie ll="118221" for .douban.com/>]>
    print r.cookies['bid'] #输出：zkXS0p3Zars

if __name__ == '__main__':
    #get_json()
    #get_querystring()
    #get_custom_headers()
    get_cookie()
```

## 四、高级用法
下面这些高级用法：自己看requests文档。
- Session：同一个会话内参数保持一致，且会重用TCP连接。也会尽量保持连接，也会提高性能
- SSL证书认证：开启、关闭、自定义CA证书
- 上传普通文件和复杂结构的文件
- 代理访问

