---
title: 更改Yelee主题标签云为球形标签云
date: 2016-07-23 18:06:17
tags: Hexo
categories: 综合
---

首先介绍球形标签云官方js插件地址：[http://www.goat1000.com/tagcanvas.php](http://www.goat1000.com/tagcanvas.php)
里面包含了详细说明，看例子应该很容易明白，利用html5的Canvas绘图。

这里我使用的是独立版本的 [tagcanvas.js](http://www.goat1000.com/tagcanvas.js?2.8)，不知道为什么jquery版本不行- -

<!-- more -->

## 第一步 下载tagcanvas.js

下载上述 [tagcanvas.js](http://www.goat1000.com/tagcanvas.js?2.8)，放到主题文件夹`\...\themes\yelee\source\js`下



## 第二步 修改配置文件
修改主题布局文件 `\themes\yelee\layout\_partial\tag-cloud-page.ejs`，参考如下:

```javascript
<hr>
<br>
<%- list_categories() %>
<script src="/js/tagcanvas.js" type="text/javascript"></script>
<div class="tags" id="myTags">
 <canvas width="350" height="350" id="my3DTags">
    <p>Anything in here will be replaced on browsers that support the canvas element</p>
</canvas>
</div>
<div class="tags" id="tags">
 <ul>
  <%- tagcloud({
      min_font: 16,
      max_font: 35,
      amount: 999,
      color: true,
      start_color: 'blue',
      end_color: 'red',
  }) %>
 </ul>
</div>
<style type="text/css">
    .category-list li, .tags li{
        display: inline;
        font-size: 1.2em;
        margin-right: 1em;
        line-height: 60px;
        border: 1px solid lightgray;
        padding: 6px;
    }
    .category-list a {
        color: gray;
    }
    .category-list:hover a {
        color: gray;
        text-decoration: none;
        font-weight: bold;
    }
    .category-list-count {
        margin-left: 2px;
        font-size: .9em;
    }
    .article-entry ul li:before{
        display: none;
    }
    .article-inner  {
        text-align: center;
    }
    .article-meta {
        display: none;
    }
    .article-header {
        padding-right: 35px;
    }
    #container .article .article-title {
        padding-right: 0;
    }
    .tags {
        max-width: 40em;
        margin: 2em auto;
        margin-top: 0em;
    }
    .tags a {
        margin-right: 1em;
        line-height: 65px;
        border-bottom: 1px solid gray;
    }
    .tags a:hover {
        font-weight: bold;
        text-decoration: none;
    }
    .category-list-child {
        display: none;
    }
</style>
    <script type="text/javascript">
      window.onload = function() {
        try {
          TagCanvas.Start('my3DTags','tags',{
            textFont: 'Georgia,Optima',
            textColour: null,
            outlineColour: '#ff00ff',
            weight: true,
            reverse: true,
            depth: 0.8,
            maxSpeed: 0.05,
            bgRadius: 1,
            freezeDecel: true
          });
        } catch(e) {
          // something went wrong, hide the canvas container
          document.getElementById('myTags').style.display = 'none';
        }
      };
    </script>
```

配置参数在TagCanvas.Start();参数表中，这里有具体配置选项：[Options](http://www.goat1000.com/tagcanvas-options.php)

## 第三步 修改 tagcloud.js
接着到博客根目录下，找到/node_modules/hexo/lib/plugins/helper，修改tagcloud.js，找到如下代码:
```javascript
result.push(
'<a href="' + self.url_for(tag.path) + '" style="' + style + '">' +
(transform ? transform(tag.name) : tag.name) +
'</a>'
);
```

加上`<li></li>`标签，如下:

```javascript
result.push(
  '<li><a href="' + self.url_for(tag.path) + '" style="' + style + '">' +
  (transform ? transform(tag.name) : tag.name) +
  '</a></li>'
);
```

大功告成，剩下的就是部署了。


转载自：[http://www.netcan666.com/2015/12/15/Hexo个性化球形标签云/](http://www.netcan666.com/2015/12/15/Hexo个性化球形标签云/)