---
title: Python爬虫实例：登录豆瓣并修改签名
date: 2016-07-24 21:23:12
tags: 
- Crawler
- Python
categories:
- 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 功能
- 登录豆瓣
- 修改签名

## 一、登录流程分析
- 向哪个url发送请求
- 发送哪些数据
- 有哪些特殊的头字段
- 验证码问题如何解决

1.抓取豆瓣登录流程：

使用账号：xxxxxx 密码：xxxxxx 抓取得Network如下：

豆瓣登录界面网址：`https://www.douban.com/accounts/login`

```
General

Request URL:https://accounts.douban.com/login
Request Method:POST
Status Code:302 Moved Temporarily
Remote Address:211.147.4.32:443

---------------------------------------------------------------------------

Response Headers

Cache-Control:must-revalidate, no-cache, private
Connection:keep-alive
Content-Length:65
Content-Type:text/plain
Date:Sat, 11 Jun 2016 02:48:18 GMT
Expires:Sun, 1 Jan 2006 01:00:00 GMT
Keep-Alive:timeout=30
Location:https://www.douban.com
P3P:CP="IDC DSP COR ADM DEVi TAIi PSA PSD IVAi IVDi CONi HIS OUR IND CNT"
Pragma:no-cache
Server:dae
Set-Cookie:ue="877646746@qq.com"; domain=.douban.com; expires=Sun, 11-Jun-2017 02:48:18 GMT; httponly
Set-Cookie:dbcl2="146925119:/crpdV7NiKQ"; path=/; domain=.douban.com; httponly
Set-Cookie:as="deleted"; max-age=0; domain=.douban.com; expires=Thu, 01-Jan-1970 00:00:00 GMT
Strict-Transport-Security:max-age=15552000;
X-Content-Type-Options:nosniff
X-DAE-App:accounts
X-DAE-Node:sindar15a
X-Douban-Mobileapp:0
X-Frame-Options:SAMEORIGIN
X-Xss-Protection:1; mode=block

----------------------------------------------------------------------------------

Request Headers

Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Encoding:gzip, deflate, br
Accept-Language:en-US,en;q=0.8,zh-CN;q=0.6,zh;q=0.4
Cache-Control:max-age=0
Connection:keep-alive
Content-Length:138
Content-Type:application/x-www-form-urlencoded
Cookie:bid=PHjUxRzrHNk; _vwo_uuid_v2=56A954C0557184C73BBB3DF5C8D30C1D|409597a19056d473ebee60708893e9b8; ap=1; ll="118221"; __utmt=1; ps=y; __utma=30149280.2019919087.1465354115.1465606255.1465612975.3; __utmb=30149280.2.10.1465612975; __utmc=30149280; __utmz=30149280.1465612975.3.3.utmcsr=baidu|utmccn=(organic)|utmcmd=organic; dbcl2="146925119:KHEcD+nREDs"; ck=9R18
Host:accounts.douban.com
Origin:https://accounts.douban.com
Referer:https://accounts.douban.com/login
Upgrade-Insecure-Requests:1
User-Agent:Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.84 Safari/537.36
-------------------------------------------------------------
Form Data

ck:9R18
source:None
redir:https://www.douban.com
form_email:877646746@qq.com
form_password:song@3345616
login:登录
```
> 
> 即登录时，我们只需要模拟Request Headers中的头和Form Data中的post参数就可以登录了。
> 
> 如果登录时，需要图片中的验证码，我们需要抽取验证码图片，然后手动填写上去。（半自动化方式）

> 当然，如果需要全自动化的方式，则需要用到机器学习中的知识，爬取所有验证码图片，然后训练模型，用机器学习的方法自动识别出验证码图片中的验证码。


## 二、修改签名流程分析
- 向哪个url发送请求
- 发送哪些数据
- 有哪些特殊的头字段
- 返回值长什么样


```
General

Request URL:https://www.douban.com/j/people/146925119/edit_signature
Request Method:POST
Status Code:200 OK
Remote Address:211.147.4.31:443

--------------------------------------------------------------------------
Response Headers

Cache-Control:must-revalidate, no-cache, private
Connection:keep-alive
Content-Length:47
Content-Type:application/json; charset=utf-8
Date:Sat, 11 Jun 2016 06:06:37 GMT
Expires:Sun, 1 Jan 2006 01:00:00 GMT
Keep-Alive:timeout=30
Pragma:no-cache
Server:dae
Strict-Transport-Security:max-age=15552000;
X-DAE-App:sns
X-DAE-Node:sindar25b
X-Douban-Mobileapp:0
X-Xss-Protection:1; mode=block

-----------------------------------------------------------------------
Request Headers

Accept:application/json, text/javascript, */*; q=0.01
Accept-Encoding:gzip, deflate, br
Accept-Language:en-US,en;q=0.8,zh-CN;q=0.6,zh;q=0.4
Connection:keep-alive
Content-Length:54
Content-Type:application/x-www-form-urlencoded
Cookie:bid=PHjUxRzrHNk; _vwo_uuid_v2=56A954C0557184C73BBB3DF5C8D30C1D|409597a19056d473ebee60708893e9b8; ll="118221"; ps=y; ue="877646746@qq.com"; dbcl2="146925119:/crpdV7NiKQ"; ck=vkO3; ap=1; _pk_ref.100001.8cb4=%5B%22%22%2C%22%22%2C1465624694%2C%22https%3A%2F%2Fwww.baidu.com%2Flink%3Furl%3DjUjRq0ldEsr3DVgcsr-2j6hhjW72VMHrsETjWL2QAee%26wd%3D%26eqid%3Dc07ebf420008142f00000003575b7a83%22%5D; __utmt=1; push_noty_num=0; push_doumail_num=0; _pk_id.100001.8cb4=cbb9346c7bb2e22f.1465354092.4.1465624911.1465613335.; _pk_ses.100001.8cb4=*; __utma=30149280.2019919087.1465354115.1465612975.1465624696.4; __utmb=30149280.4.10.1465624696; __utmc=30149280; __utmz=30149280.1465612975.3.3.utmcsr=baidu|utmccn=(organic)|utmcmd=organic; __utmv=30149280.14692
Host:www.douban.com
Origin:https://www.douban.com
Referer:https://www.douban.com/people/146925119/
User-Agent:Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.84 Safari/537.36
X-Requested-With:XMLHttpRequest

----------------------------------------------------------------------------
Form Data

ck:vkO3   
signature:顶顶顶顶
```


> Form Data
> 
> ck:vkO3    
> signature:顶顶顶顶

当不知道post data中的值如何获得时，往往需要到操作页面的html源码中去寻找，如上面的
> ck:vk03
如要的操作页面的html的代码中寻找，然后把它解析出来。


## 实例：

注意：

本实例是基于登录时有图片验证码的，现在登录豆瓣好像不需要图片验证码了；

如果登录不需要验证码，则把验证码部分去掉即可。


```
# -*- coding: utf-8 -*-
from HTMLParser import HTMLParser
import requests


def _attr(attrs, attrname):
    for attr in attrs:
        if attr[0] == attrname:
            return attr[1]
    return None

#获得验证码信息
def _get_captcha(content):
    class CaptchaParser(HTMLParser):
        def __init__(self):
            HTMLParser.__init__(self)
            self.captcha_id = None
            self.captcha_url = None

        def handle_starttag(self, tag, attrs):
            if tag == 'input' and _attr(attrs,'type') == 'hidden' and _attr(attrs,'name') == 'captcha_id':
                self.captcha_id = _attr(attrs,'value')
            if tag == 'image' and _attr(attrs,'id') == 'captcha_image' and _attr(attrs,'class') == 'captcha_image':
                self.captcha_url == _attr(attrs,'src')

    p = CaptchaParser()
    p.feed(content)
    return p.captcha_id, p.captcha_url

#获得ck属性的值
def _get_ck(content):
    class CKParser(HTMLParser):
        def __init__(self):
            HTMLParser.__init__(self)
            self.ck = None

        def handle_starttag(self, tag, attrs):
            if tag == 'input' and _attr(attrs,'type') == 'hidden' and _attr(attrs,'name') == 'ck':
                self.ck = _attr(attrs,'value')

    p =CKParser()
    p.feed(content)
    return p.ck


class DoubanClient(object):
    def __init__(self):
        object.__init__(self)
        headers = {'User-Agent':'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.84 Safari/537.36',
                   'origin':'http/www.douban.com'}
        #create requests session
        self.session = requests.session()
        #对session的头进行定制，这样以后，以后所有的请求都会包含上面headers中的数据
        self.session.headers.update(headers)

    #登录豆瓣
    def login(self,username,password,source='index_nav',
              redir = 'http://www.douban.com/',login = '登录'):
        url = 'https://www.douban.com/accounts/login'
        #access login page to get captcha
        #湖区登录界面中的验证码图片
        #r = requests.get(url)
        #应为登录和修改签名在同一个session中，故使用session.get(url)的方式登录
        r = self.session.get(url)
        (captcha_id,captcha_url) = _get_captcha(r.content)
        if captcha_id:
            captcha_solution = raw_input('please input solution for [%s]' % captcha_url)

        #post login request
        data = {'from_email':username,'from_passwd':password,'source':source,
                'redir':redir,'login':login}
        #将验证信息加入到post data中
        if captcha_id:
            data['captcha_id'] = captcha_id
            data['captcha_url'] = captcha_url

        headers = {'referer':'http://www.douban.com/accounts/login?source=main',
                   'host':'accounts.douban.com'}
        #r = requests.post(url,data=data,headers=headers)
        r = self.session.post(url,data=data,headers=headers)
        print self.session.cookies.items()
    
    #编辑签名
    def edit_signature(self,username,signature):
        #access user's homepage
        url = 'https://www.douban.com/people/%s/' % username
        r  = self.session.get(url)
        #从操作页面的HTML代码中获取post data数据中参数ck的值
        ck = _get_ck(r.content)

        #post request to change signature
        url = 'https://www.douban.com/j/people/%s/edit_signature' % username
        headers = {'referer':url,'host':'www.douban.com',
                 'x-requested-with':'XMLHTTPRequest'}
        data = {'ck':ck,'signature':signature}
        r = self.session.post(url,data=data,headers=headers)
        print r.content

if __name__ == '__main__':
    c = DoubanClient()
    c.login('877646746@qq.com','song@3345616')
    c.edit_signature('146925119','Hello')
```


## 四、作业
- 登录知乎
- 修改个人简介



