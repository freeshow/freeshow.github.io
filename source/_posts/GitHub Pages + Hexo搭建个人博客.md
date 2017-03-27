---
title: GitHub Pages + Hexo搭建个人博客
toc: false
date: 2017-03-27 12:01:01
tags: Hexo
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

参考自：[GitHub Pages + Hexo搭建博客](http://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/#more)
# 一、前言
这是一篇使用GitHub Pages和Hexo搭建免费独立博客的总结。

如果你厌恶了第三方博客系统的广告的繁琐的事情或想搭建自己的个性博客，我想这是一个不错的选择。

如果你是一个小小白，可以先花时间了解下以下内容：
- [Git](http://git-scm.com/book/zh/v2)
- [GitHub](https://github.com/)
- [GitHub Pages](https://pages.github.com/)
- [Hexo](https://hexo.io/zh-cn/)
- [Markdown](http://www.appinn.com/markdown/#autoescape)

# 二、必要配置

## 2.1 GitHub Pages 仓库

### 2.1.1 创建对应仓库

在自己的GitHub账号下创建一个新的仓库，命名为username.github.io（username是你的账号名)。
在这里，要知道，GitHub Pages有两种类型：User/Organization Pages 和 Project Pages，而我所使用的是User Pages。

简单来说，User Pages 与 Project Pages的区别是：
1. User Pages 是用来展示用户的，而 Project Pages 是用来展示项目的。
2. 用于存放 User Pages 的仓库必须使用username.github.io的命名规则，而 Project Pages 则没有特殊的要求。
3. User Pages 将使用仓库的 master 分支，而 Project Pages 将使用 gh-pages 分支。
4. User Pages 通过 http(s)://username.github.io 进行访问，而 Projects Pages通过 http(s)://username.github.io/projectname 进行访问。

### 2.1.2 相关资料
- [GitHub Pages Basics / User, Organization, and Project Pages](https://help.github.com/articles/user-organization-and-project-pages/)

## 2.2 Git

### 2.2.1 安装Git

在windows下安装git比较常用的有两种方式：
1. [Git 官方版本的安装](http://git-scm.com/download/win)
2. [GitHub for Windows](https://desktop.github.com/)

### 2.2.2 配置Git

当安装完Git应该做的第一件事情就是设置用户名称和邮件地址。这样做很重要，因为每一个Git的提交都会使用这些信息，并且它会写入你的每一次提交中，不可更改：
```
$ git config --global user.name "username"
$ git config --global user.email "username@example.com"
```
对于user.email，因为在GitHub的commits信息上是可见的，所以如果你不想让人知道你的email，可以Keeping your email address private:

1. 在GitHub右上方点击你的头像，选择”Settings”；
2. 在右边的”Personal settings”侧边栏选择”Emails”；
3. 选择”Keep my email address private”。

这样，你就可以使用如下格式的email进行配置：

```
$ git config --global user.email "username@users.noreply.github.com"
```

### 2.2.3 相关资料

- [安装 Git](http://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)
- [配置 Git](http://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%88%9D%E6%AC%A1%E8%BF%90%E8%A1%8C-Git-%E5%89%8D%E7%9A%84%E9%85%8D%E7%BD%AE)
- [Setting your email in Git](https://help.github.com/articles/setting-your-email-in-git/)
- [Keeping your email address private](https://help.github.com/articles/keeping-your-email-address-private/)

## 2.3 Git与GitHub

### 2.3.1 git与github的区别

这里，我们要区分清楚git与github。
git是一个版本控制的工具，而github有点类似于远程仓库，用于存放用git管理的各种项目。

### 2.3.2 与github建立联系

为了能够在本地使用git管理github上的项目，需要进行一些配置，这里介绍SSH的方法。

(1) 检查电脑是否已经有SSH keys。
```
$ ls -al ~/.ssh
# Lists the files in your .ssh directory, if they exist
```
默认情况下，public keys的文件名是以下的格式之一：id_dsa.pub、id_ecdsa.pub、id_ed25519.pub、id_rsa.pub。因此，如果列出的文件有public和private钥匙对（例如id_ras.pub和id_rsa），证明已存在SSH keys。

(2) 如果没有SSH key，则生成新的SSH key。

```
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
# Creates a new ssh key, using the provided email as a label
```
之后一路回车即可。

(3) 在GitHub添加SSH key。

首先，拷贝key到粘贴板：即将`~/.ssh/id_rsa.pub`中的内容拷贝到粘贴板。
然后，在GitHub右上方点击头像，选择”Settings”，在右边的”Personal settings”侧边栏选择”SSH Keys”。接着粘贴key，点击”Add key”按钮。

(4)相关资料
- [Generating SSH keys](https://help.github.com/articles/generating-ssh-keys/)

## 2.4 Hexo

### 2.4.1 安装Hexo
安装Hexo相当简单。在安装之前，必须检查电脑中是否已经安装下列应用程序：
- [Node.js](http://nodejs.org/)
- [Git](http://git-scm.com/)

如果您的电脑中已经安装上述必备程序，那么恭喜您！接下来只需要使用 npm 即可完成 Hexo 的安装。
```
$ npm install -g hexo-cli
```

### 2.4.2 使用Hexo建站
安装完后，在你喜欢的文件夹内（例如F：\Hexo），点击鼠标右键选择Git bash，输入以下指令：
```
$ hexo init
```
该命令会在目标文件夹内建立网站所需要的所有文件。完成后，文件夹目录如下：
```
.
├── _config.yml
├── package.json	#所需依赖包文件
├── scaffolds
├── source
|   └── _posts
└── themes
|   └── landscape
```

接下来是安装依赖包：
```
$ npm install
```
会安装`package.json`中的默认依赖包。
```
# package.json

{
  "name": "hexo-site",
  "version": "0.0.0",
  "private": true,
  "hexo": {
    "version": "3.2.2"
  },
  "dependencies": {
    "hexo": "^3.2.0",
    "hexo-generator-archive": "^0.1.4",
    "hexo-generator-category": "^0.1.3",
    "hexo-generator-index": "^0.2.0",
    "hexo-generator-tag": "^0.2.0",
    "hexo-renderer-ejs": "^0.2.0",
    "hexo-renderer-marked": "^0.2.10",
    "hexo-renderer-stylus": "^0.3.1",
    "hexo-server": "^0.2.0"
  }
}

```

这样，我们就已经搭建起本地的Hexo博客了。可以先执行以下命令（在对应文件夹下），然后再浏览器输入localhost:4000查看。
```
$ hexo g	#hexo generate
$ hexo s	#hexo server
```

### 2.4.3 相关资料
- [Hexo 官方文档](https://hexo.io/zh-cn/docs/)

# 三、一般的搭建方法

在上面，我们已经配置好了所需的所有东西，也成功地搭建了一个本地Hexo博客。现在，需要使用GitHub Pages搭建一个别人能够访问的Hexo博客了。

3.1 使用默认theme

我们继续使用上面的文件夹F:\Hexo（也可以新建一个文件夹重新生成），然后编辑该文件夹下的_config.yml。

默认生成的_config.yml：
```
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type:
```

修改后的_config.yml：
```
deploy:
  type: git
  repo: 对应仓库的SSH地址（可以在GitHub对应的仓库中复制）
  branch: 分支（User Pages为master，Project Pages为gh-pages）
```

为了能够使Hexo部署到GitHub上，需要安装一个插件：
```
$ npm install hexo-deployer-git --save
```

然后，执行下列指令即可完成部署：

```
$ hexo generate
$ hexo deploy
```

之后，可以通过在浏览器键入：username.github.io进行浏览，开心吧~

## 3.2 其他theme

如果想要使用其他主题，可以使用git clone将别人的主题拷贝到D:\Hexo\themes下，然后将_config.yml中的theme: landscape改为对应的主题名字。
详细步骤可以参考网上的指南。

landscape主题是Hexo自带的主题，不需要自己下载。

推荐两款比较好用的主题，NexT和Yelee

# 四、优化和部署
## 4.1 概述
Hexo部署到GitHub上的文件，是.md（你的博文）转化之后的.html（静态网页）。因此，当你重装电脑或者想在不同电脑上修改博客时，就不可能了（除非你自己写html o(^▽^)o ）。

其实，Hexo生成的网站文件中有.gitignore文件，因此它的本意也是想我们将Hexo生成的网站文件存放到GitHub上进行管理的（而不是用U盘或者云备份啦(╬▔皿▔)凸）。这样，不仅解决了上述的问题，还可以通过git的版本控制追踪你的博文的修改过程，是极赞的。

但是，如果每一个GitHub Pages都需要创建一个额外的仓库来存放Hexo网站文件，我感觉很麻烦（10个项目需要20个仓库(ˉ▽ˉ；)…）。
所以，我利用了分支！！！

简单地说，每个想建立GitHub Pages的仓库，起码有两个分支，一个用来存放Hexo网站的文件，一个用来发布网站。

## 4.2 我的博客搭建流程

下面以我的博客作为例子详细地讲述。

在本地电脑F盘下：
```
#第1步：
mkdir hexo	//创建hexo文件夹
cd hexo

#第2步：
hexo init

#第3步:
git@github.com:MOxFIVE/hexo-theme-yelee.git theme/yelee		#使用yelee主题
cd theme/yelee	
git remote rm origin	#删除yelee本地仓库与远程仓库的关联
rm -rf .git		#删除yelee本地仓库的`.git`文件夹，事情称为普通文件夹

#第4步
#配置主题文件等 上面已经讲了，或者参考其他资料
......

#第5步：
#修改配置文件中的deploy属性
deploy:
  type: git
  repo: 对应仓库的SSH地址（可以在GitHub对应的仓库中复制）
  branch: 分支（User Pages为master，Project Pages为gh-pages）

#第6步：
hexo d	#发布网站到GitHub中的master分支
#现在就可以登录 `http:username.github.io访问自己的网站了。

#第7步：使用backup分支备份博客文件，以便可以在其他电脑上写blog.

#在F:\hexo目录下
#将博客备份文件添加到git版本库中
git init
git add .
git commit -m "..."
#建立与远程仓库的连接
git remote add origin git@github.com:freeshow/freeshow.github.io.git

#这就是第3步中为什么将yelee仓库取消版本库的原因，因为要在它的父目录F:\hexo中创建版本库，
如果不取消yelee主题的版本库，则提交hexo仓库时，没法将其下面的yelee仓库提交到远程仓库。

#如果当yelee主题有更新时，在给yelee文件添加远程仓库连接，更新完yelee仓库时，在将yelee仓库的版本库删除即可。

#将本地仓库推送到github下的backup分支。
git push origin master:backup  #本地分支master到远程仓库backup
```

注意：
>按上面步骤搭建完成后，需要将github中的backup分支设置成默认分支，
>这样就可以在其他电脑上clone时，会默认把backup分支clone出来。

## 4.3 我的博客管理流程

### 4.3.1 日常修改
在本地对博客进行修改（添加新博文、修改样式等等）后，通过下面的流程进行管理：

依次执行
```
git add .
git commit -m "…"
git push origin master:backup
```
将推送到GitHub（此时当前分支应为backup）.

然后才执行`hexo generate -d`发布网站到master分支上。

### 4.3.2 本地资料丢失
当重装电脑之后，或者想在其他电脑上写博客，可以使用下列步骤：
1.使用git clone git@github.com:freeshow/freeshow.github.io.git拷贝仓库（默认分支为backup）；

2.在本地新拷贝的freeshow.github.io.git文件夹下通过Git bash依次执行下列指令：
```
npm install hexo --save
npm install
#npm install hexo-deployer-git --save
#在刚开始搭建博客时，已经执行了上面这条命令，则会将`hexo-deploy-get`依赖包，添加到`package.json`中去了，
#故在上面执行`npm install`时，会安装`hexo-deploy-get`依赖包。

#（记得，不需要hexo init这条指令）。hexo init会取消`freeshow.github.io.git`版本库
#因为clone出的文件中已经包含建立网站所需要的所有文件。
 
```

在其他电脑上clone出的backup分支，如果要日常修改，执行如下命令：
```
git add .
git commit -m "…"

#git push origin master:backup
git push #本来就是从backup分支clone出来的，这样就相当于上面的提交到远程仓库。
```

# 结尾
忙了一上午，才把上面的 `四、优化部署` 搞明白，现在想想主要是不太熟悉git操作导致的。

哎，祝大家能够搭建出自己个性的博客。