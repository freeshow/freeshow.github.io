---
title: Cookie介绍
date: 2016-07-24 20:53:18
tags: Crawler
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## Cookie数据长什么样

- Request:

```

Cookie:bid=PHjUxRzrHNk; _vwo_uuid_v2=56A954C0557184C73BBB3DF5C8D30C1D|409597a19056d473ebee60708893e9b8; ap=1; _pk_ref.100001.8cb4=%5B%22%22%2C%22%22%2C1465517885%2C%22https%3A%2F%2Fwww.google.com.hk%2F%22%5D; _pk_id.100001.8cb4=cbb9346c7bb2e22f.1465354092.2.1465517885.1465354092.; __utma=30149280.2019919087.1465354115.1465354115.1465517888.2; __utmc=30149280; __utmz=30149280.1465517888.2.2.utmcsr=google|utmccn=(organic)|utmcmd=organic|utmctr=(not%20provided)
```

- Response

```
Set-Cookie:ll="118221"; path=/; domain=.douban.com; expires=Sat, 10-Jun-2017 04:39:13 GMT
```

---


## Cookie的格式

- 客户端发送Cookie时：

    ```
    Cookie: key1=value1;key2=value2;key3=value3
    ```

- 服务器保存Cookie时:

    ```
    Set Cookie: key1=value1;path=/;domain=xxx
    Set Cookie: key2=value2;path=/;domain=xxx
    ```

- Cookie属性：

    Domain and Path:定义Cookie的作用域。当指定domain时，这个domain及其子域名都包含这个Cookie.
    
    Exprires:定义cookie的生命周期。
    
    HttpOnly:禁用脚本访问。

---

## Cookie的用途：

- 登录信息：判断用户是否已经登录
- 购物车：保存用户购买的商品列表


---
## Cookie小结：

- 服务器在客户端存储信息

- 请求时，客户端需要把未超时的cookies发回给服务器。
- 应答时，服务器会把新的cookies发给客户端，以便下次请求时带上这些cookies.

---


## 从登录行为看Cookie

- 第一次登录用户名成功时，服务器会向Cookie中写入用户登录成功的信息，服务器并将登录成功的Cookie发回客户端。
- 当客户端在此登录时，客户端会将登录成功的Cookie(在生命周期范围内)发送给服务器，这是服务器就认为客户端已经登录了。


## Cookie可能会引发什么安全问题？
自己搜索。




