---
title: Python爬虫之urllib介绍
toc: false
date: 2016-07-24 20:57:41
tags: Crawler
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>


## 一、urllib.urlopen(url,[data])

```python
urlopen(url, data=None, proxies=None, context=None)
Create a file-like object for the specified URL to read from.
```

- url: scheme(http: / file: )
- data: 如果有，则变成post方法，数据格式必须是application/x-wwww-form-urlencoded
- 返回类文件句柄

### 类文件句柄的常方法：
- read(size): size=-1/None
- readline():
- readlines():
- close():
- getcode():

## 二、探求HttpMessage的方法：
HTTPMessage没有官方文档，如何找出其有用方法？

- info():返回httplib.HTTPMessage实例(urllib.urlopen().info())
- httplib.HTTPMessage:
    - headers
    - gettype()
    - getheader()/getheaders()
    - items()/keys()/values()
    

## 三、urllib.urlretrieve()

```
urlretrieve(url, filename=None, reporthook=None, data=None, context=None)
```
功能：将请求的url响应保存在filename中。

- url: 远程地址
- filename: 要保存到本地的文件
- reporthook: 下载状态报告
- data: 如果有，则变成post方法，数据格式必须是application/x-wwww-form-urlencoded
- 返回：(filename,HTTPMessage)

### reporthook:
- 参数1：当前传输的块数
- 参数2：块大小
- 参数3：数据总大小
- 需要注意：content-length不是必须的

监控下载进度：
参数1*参数2就是当前下载的大小，然后除以参数3就是下载的进度。


## 实例：

```python
# -*- coding: utf-8 -*-
import urllib

def print_list(list):
    for i in list:
        print(i)

def demo():
    s = urllib.urlopen('http://blog.kamidox.com')
    print s.read()
    print s.getcode()

    #获取HTTPMessage实例
    msg = s.info()

    #获取HTTPMessage方法
    print_list(dir(msg))

    #使用HTTPMessage方法
    print_list(msg.headers)
    print msg.getheader('Content-Type')

def retrieve():
    fname,msg = urllib.urlretrieve('http://blog.kamidox.com','index.html',
                                   reporthook=progress)
    print fname
    print_list(msg.items())

# 显示下载进度
# 0/16176 - 0.00%
# 8192/16176 - 50.64%
# 16384/16176 - 101.29%:超出100的原因可能是total_size只包含了body，
# 没有包含http headers
def progress(blk,blk_size,total_size):
    print '%d/%d - %.02f%%' % (blk*blk_size,total_size,
                               (float)(blk*blk_size)*100/total_size)


if __name__ == '__main__':
    demo()
    retrieve()
```


---
## urllib.urlencode()

```
urlencode(query, doseq=0)
Encode a sequence of two-element tuples or dictionary into a URL query string.
 
If any values in the query arg are sequences and doseq is true, each
sequence element is converted to a separate parameter.
 
If the query arg is a sequence of two-element tuples, the order of the
parameters in the output will match the order of parameters in the
input.
```
- 把字典数据转化为url编码
- 用途：
    - 对url参数进行编码
    - 对post上的form数据进行编码
    

## urlparse.parse_qs()

```
parse_qs(qs, keep_blank_values=0, strict_parsing=0)
Parse a query given as a string argument.

Arguments:
 
qs: percent-encoded query string to be parsed
 
keep_blank_values: flag indicating whether blank values in
    percent-encoded queries should be treated as blank strings.
    A true value indicates that blanks should be retained as
    blank strings.  The default false value indicates that
    blank values are to be ignored and treated as if they were
    not included.
 
strict_parsing: flag indicating what to do with parsing errors.
    If false (the default), errors are silently ignored.
    If true, errors raise a ValueError exception.
```

> 把url编码转化为字典数据


## 实例：

```
# -*- coding: utf-8 -*-
import urllib
import urlparse

def urlencode():
    params = {'score':100, 'name':'爬虫基础','content':'very good'}
    qs = urllib.urlencode(params)
    # 输出：content=very+good&score=100&name=%E7%88%AC%E8%99%AB%E5%9F%BA%E7%A1%80
    print qs
    #输出：{'content': ['very good'], 'score': ['100'],
    # 'name': ['\xe7\x88\xac\xe8\x99\xab\xe5\x9f\xba\xe7\xa1\x80']}
    print urlparse.parse_qs(qs)

def parse_qs():
    url = 'https://www.baidu.com/s?' \
          'wd=%E9%AD%94%E5%85%BD%E4%B8%96%E7%95%8C&' \
          'rsv_spt=1&rsv_iqid=0x93e1c64900082ad5&' \
          'issp=1&f=8&rsv_bp=0&rsv_idx=2&ie=utf-8&' \
          'tn=baiduhome_pg&rsv_enter=1&rsv_sug3=3&' \
          'rsv_sug1=3&rsv_sug7=101&sug=%E9%AD%94%E5%85%BD%E4%B8%96%E7%95%8C&' \
          'rsv_n=1'
    result = urlparse.urlparse(url)
    print result
    #输出：ParseResult(scheme='https', netloc='www.baidu.com', path='/s',
    # params='', query='wd=%E9%AD%94%E5%85%BD%E4%B8%96%E7%95%8C&rsv_spt=1&
    # rsv_iqid=0x93e1c64900082ad5&issp=1&f=8&rsv_bp=0&rsv_idx=2&ie=utf-8&
    # tn=baiduhome_pg&rsv_enter=1&rsv_sug3=3&rsv_sug1=3&rsv_sug7=101&
    # sug=%E9%AD%94%E5%85%BD%E4%B8%96%E7%95%8C&rsv_n=1', fragment='')

    #提取url中的params，即ParseResult中的query.
    params = urlparse.parse_qs(result.query)
    print params
    #输出：{'sug': ['\xe9\xad\x94\xe5\x85\xbd\xe4\xb8\x96\xe7\x95\x8c'], 
    # 'wd': ['\xe9\xad\x94\xe5\x85\xbd\xe4\xb8\x96\xe7\x95\x8c'], 'rsv_spt': ['1'],
    # 'rsv_iqid': ['0x93e1c64900082ad5'], 'f': ['8'], 'rsv_enter': ['1'], 
    # 'rsv_bp': ['0'], 'rsv_idx': ['2'], 'tn': ['baiduhome_pg'], 'rsv_sug7': ['101'], 
    # 'rsv_sug1': ['3'], 'issp': ['1'], 'rsv_sug3': ['3'], 'rsv_n': ['1'], 'ie': ['utf-8']}

if __name__ == '__main__':
    urlencode()
    parse_qs()
```

---

## 一个简单的爬虫实例：从雅虎财经获取股票数据
雅虎财经股票数据接口：

1.股票数据(历史上所有日交易数据)：
- 深市数据链接：`http://table.finance.yahoo.com/table.csv?s=000001.sz`
    - s=000001.sz:股票代码
- 上市数据链接：`http://table.finance.yahoo.com/table.csv?s=600000.ss`


```
Date,Open,High,Low,Close,Volume,Adj Close
2016-06-09,10.50,10.50,10.50,10.50,000,8.6225
2016-06-08,12.648,12.648,12.552,12.60,26818800,10.347
2016-06-07,12.624,12.636,12.576,12.624,22928200,10.36671
2016-06-06,12.612,12.636,12.54,12.612,33833200,10.35685
......
```
- Data：时间
- Open:开盘价
- High：最高价
- Low：最低价
- Close：收盘价
- Volume：成交量




2.获取指定时间段的交易数据：
- 时间参数：
    - a(月) b(日) c(年)：开始时间
    - d()月 e(日) f(年)：结束时间
    - s：股票代码

例如，取深市2012年1月1日至2012年4月19日的数据：

`http://table.finance.yahoo.com/table.csv?a=0&b=1&c=2012&d=3&e=19&f=2012&s=s=000001.ss`




```
# -*- coding: utf-8 -*-
import urllib
import datetime

#获取股票全部数据
def download_socket_data(stock_list):
    for sid in stock_list:
        url = 'http://table.finance.yahoo.com/table.csv?s=' + sid
        fname = sid + '.csv'
        print 'downloading %s from %s' % (fname,url)
        urllib.urlretrieve(url,fname)
        
#获取股票指定时间的数据
def download_socket_in_period(stock_list,start,end):
    for sid in stock_list:
        params = {'a':start.month-1,'b':start.day,'c':start.year,
                  'd':end.month-1,'e':end.day,'f':end.year,
                  's':sid}
        url = 'http://table.finance.yahoo.com/table.csv'
        qs = urllib.urlencode(params)
        url = url + qs
        fname = '%s_%d%d%d_%d%d%d.csv' % (sid,start.year,start.month,start.day,
                                          end.year,end.month,end.day)
        print 'downloading %s from %s' % (fname,url)
        urllib.urlretrieve(url,fname)

if __name__ == '__main__':
    stock_list = ['300001.sz','300002.sz']
    download_socket_data(stock_list)

    start = datetime.date(year=2015,month=11,day=17)
    end = datetime.date(year=2015,month=12,day=17)
    download_socket_in_period(stock_list,start,end)
```





