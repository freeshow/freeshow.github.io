---
title: Python爬虫实例：豆瓣热播电影（urllib+urllib2）
date: 2016-07-24 21:11:57
tags: 
- Crawler
- Python
categories:
- 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 第1步：热播电影格式
1. 使用Chrome打开也爬取的网页，打开Chrome的开发者选项，点击下图中的按钮!![](http://i.imgur.com/kmaPcJH.png)
，选中要爬取的区域，然后查看html代码，查看抽取内容的格式。

通过上面方法找到热播电影的格式为:

```
 <div class="mod-bd">
            <ul class="lists">
                    <li
                        id="2131940"
                        class="list-item"
                        data-title="魔兽"
                        data-score="8.2"
                        data-star="40"
                        data-release="2016"
                        data-duration="124分钟"
                        data-region="美国 中国大陆 加拿大"
                        data-director="邓肯·琼斯"
                        data-actors="崔维斯·费米尔 / 托比·凯贝尔 / 宝拉·巴顿"
                        data-category="nowplaying"
                        data-enough="True"
                        data-showed="True"
                        data-votecount="59747"
                        data-subject="2131940"
                    >
                    <li
                        id="25786060"
                        class="list-item"
                        data-title="X战警：天启"
                        data-score="8.2"
                        data-star="40"
                        data-release="2016"
                        data-duration="144分钟"
                        data-region="美国"
                        data-director="布莱恩·辛格"
                        data-actors="詹姆斯·麦卡沃伊 / 迈克尔·法斯宾德 / 詹妮弗·劳伦斯"
                        data-category="nowplaying"
                        data-enough="True"
                        data-showed="True"
                        data-votecount="82158"
                        data-subject="25786060"
                    >
                    。。。。。。
```
因此，我们只需要拿到所有<li>标签中属性data-category="nowplaying"的data-title属性，就可获得热播电影。


## HTMLParse简介
- feed：向解析器(HTMLParse)中喂数据，可以分段提供
- handler_starttag：处理html的开始标签
    - tag：标签名称
    - attrs：标签属性列表
- handler_data：处理标签里的数据体
- data：数据文本

## 第2步通过HTMLParser解析器解析网页html代码，获取所有信息


```python
# -*- coding: utf-8 -*-
import urllib2
import json
from HTMLParser import HTMLParser

class MovieParser(HTMLParser):
    def __init__(self):
        HTMLParser.__init__(self)
        self.movies=[]
    #重载父类方法
    #循环处理feed进来的所有的tag,以及标签对应的attrs
    def handle_starttag(self, tag, attrs):
        #给定tag的属性名attrname，获取属性的值
        def _attr(attrList,attrname):
            for attr in attrList:
                if attr[0] == attrname:
                    return attr[1]
            return None
        
        if tag == 'li' and _attr(attrs,'data-title') \
                and _attr(attrs,'data-category') == 'nowplaying':
            movie = {}
            movie['title'] = _attr(attrs,'data-title')
            movie['score'] = _attr(attrs,'data-score')
            movie['director'] = _attr(attrs,'data-director')
            movie['actors'] = _attr(attrs,'data-actors')
            self.movies.append(movie)
            
#获取热播电影信息
def nowplaying_movies(url):
    headlers = {'User-Agent':'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.84 Safari/537.36'}
    req = urllib2.Request(url,headers=headlers)
    response = urllib2.urlopen(req)
    #定义解析器MovieParser继承自HTMLParser
    parser = MovieParser()
    #将response.read()喂给解析器，
    # 供解析器的handle_startendtag(self, tag, attrs)解析
    parser.feed(response.read())
    response.close()
    return parser.movies

if __name__ == '__main__':
    url = 'https://movie.douban.com/nowplaying/qingdao/'
    movies = nowplaying_movies(url)

    #把movies以json格式打印出来
    print '%s' % json.dumps(movies,sort_keys=True,indent=4,separators=(',',':'))

```


