---
title: Python爬虫实例：用requests重构豆瓣热播电影爬虫
toc: false
date: 2016-07-24 21:20:50
tags: Crawler
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 功能：
- 用requests重新实现豆瓣热播电影（原先用的是urllib,urlib2）
- 增加功能：下载每一个电影的海报图片



## 分析海报图片在HTML代码中的格式

```
<li id="2131940" class="list-item" data-title="魔兽" data-score="8.2" data-star="40" data-release="2016" data-duration="124分钟" data-region="美国 中国大陆 加拿大" data-director="邓肯·琼斯" data-actors="崔维斯·费米尔 / 托比·凯贝尔 / 宝拉·巴顿" data-category="nowplaying" data-enough="True" data-showed="True" data-votecount="63034" data-subject="2131940">
    <ul class="">
        <li class="poster">
            <a href="https://movie.douban.com/subject/2131940/?from=playing_poster" class="ticket-btn" target="_blank" data-psource="poster">
                <img src="https://img1.doubanio.com/view/movie_poster_cover/mpst/public/p2345947329.jpg" alt="魔兽" rel="nofollow" class="">
            </a>
        </li>

    </ul>
```

每个电影的海报图片url在爬取的其电影用的<li>标签，的内部。


## 实例


```python
# -*- coding: utf-8 -*-
import urllib2
import json
from HTMLParser import HTMLParser
import requests

class MovieParser(HTMLParser):
    def __init__(self):
        HTMLParser.__init__(self)
        self.movies=[]
        self.in_movies = False
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
            self.in_movies = True

        if tag == 'img' and self.in_movies:
            src = _attr(attrs,'src')
            movie = self.movies[len(self.movies)-1]
            #将电影的海报url添加到movie中
            movie['poster-url'] = src
            #根据海报的url下载图片
            _download_poster_image(movie)
            #下载完后将self.in_movies属性置为False
            self.in_movies = False

#根据海报图片的url下载海报图片
def _download_poster_image(movie):
    #"poster-url":"https://img3.doubanio.com/view/movie_poster_cover/mpst/public/p2354707516.jpg",
    src = movie['poster-url']
    r = requests.get(src)
    fname = src.split('/')[-1]
    #将图片下如文件
    with open(fname,'wb') as f:
        f.write(r.content)
        movie['poster-path'] = fname


#获取热播电影信息
def nowplaying_movies(url):
    headlers = {'User-Agent':'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.84 Safari/537.36'}

    # req = urllib2.Request(url,headers=headlers)
    # response = urllib2.urlopen(req)
    # #定义解析器MovieParser继承自HTMLParser
    # parser = MovieParser()
    # #将response.read()喂给解析器，
    # # 供解析器的handle_startendtag(self, tag, attrs)解析
    # parser.feed(response.read())
    # response.close()
    # return parser.movies

    #使用requests重构上面注释的代码
    r = requests.get(url,headlers)
    p = MovieParser()
    p.feed(r.content)
    return p.movies

if __name__ == '__main__':
    url = 'https://movie.douban.com/nowplaying/qingdao/'
    movies = nowplaying_movies(url)

    #把movies以json格式打印出来
    print '%s' % json.dumps(movies,sort_keys=True,indent=4,separators=(',',':'))

```



