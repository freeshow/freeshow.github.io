---
title: Hexo主题Yelee介绍
date: 2016-07-23 16:47:03
tags: Hexo
categories:
---

# Yelee —— 简而不减 Hexo 双栏博客主题 #

本主题基于主题 [Hexo-Theme-Yilia](https://github.com/litten/hexo-theme-yilia) 修改而来，在此再次感谢原作者 Litten。修复了一些 bugs，改变了大量的样式，添加了不少特性。对原主题百般折腾后，发觉变动越来越大，索性就发布个新主题了，主题随我微博名 "夜Yelee" 。个人喜欢简洁的样式，重视内容的浏览，同时希望作为个人网站的博客，能稍微凸显出博主个性。各种修改折腾大抵基于以上考虑。

<!-- more -->


## GitHub链接
GitHub: [https://github.com/MOxFIVE/hexo-theme-yelee](https://github.com/MOxFIVE/hexo-theme-yelee)

## Yelee主题使用说明

Yelee主题使用说明: [http://moxfive.coding.me/yelee/](http://moxfive.coding.me/yelee/)

## 我的Blog简单使用介绍

按照上面的`Yelee主题使用说明`配置。

### 主菜单设置

1.标签云设置：
使用 Hexo 命令新建一个名为 tags 的页面即可
```
hexo new page tags
```
2.关于我页面

使用 Hexo 命令新建一个名为 about 的页面即可
```
hexo new page about
```
>该页面内容在文件 \hexo\source\about\index.md 中修改

然后，将上面两个在下面配置中建立链接关系。

```
menu:
  主页: /
  所有文章: /archives/
  标签云: /tags/
  关于我: /about/
```

注意：
```
上面标签云设置，已经包含了tags和cataloges了，
如果你在使用hexo new page catalages创建一个catalages分类，会和about页面一样，只显示一个网页。
```

## 写新的Blog

将`\freeshow.github.io\scaffolds`下的post.md模板文件修改为如下：
```
---
title: {{ title }}
date: {{ date }}
tags:
categories:
---
```
当写文件时，就可以填写所属tags或categories了。
但是，tags和categories都会显示在 `标签云` 中.






