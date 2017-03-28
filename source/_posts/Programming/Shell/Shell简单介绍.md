---
title: Shell简单介绍
date: 2016-07-24 00:18:46
tags: Shell
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

Shell是UNIX/Linux 系统的重要组成部分。在UNIX/Linux 下，Shell扮演了一个双重角色。虽然它表面上和Windows 的命令提示符相似，但是它却具备完成执行复杂程序的强大功能。用户不仅可以通过它执行命令、调用Linux 工具，还可以把Shell作为一种编程语音，编写自己的程序。


----------


## Shell简介
用户登录Linux 系统时，可以进入基于XWindow的图形界面系统，如KDE 或GNOME。当然，很多工作可以在图形界面环境下完成，但是在服务器应用环境的很多情况下，需要远程连接到服务器进行管理配置，而使用命令行模式进行管理更加方便和简单。

### 什么是Shell
Shell 本身是一个用C 语言编写的程序，它是用户使用Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。作为命令语言，它互动式地解释和执行用户输入的命令；作为程序设计语言，它定义了各种变量和参数，并提供了许多在高阶语言中才具有的控制结构，包括循环和分支。它虽然不是Linux 系统内核的一部分，但它调用了系统内核的大部分功能来执行程序、创建文档并以并行的方式协调各个程序的运行。

利用shell程序设计语言我们可以编写出来功能强大且代码简单的shell脚本程序，特别是通过把相关的linux命令和shell有机结合，我们可实现linux系统的自动化管理，大大简化我们的工作。

### 常见 Shell 的种类
Linux Shell的种类很多，目前流行的Shell包括ash、bash、ksh、csh、zsh等，用户可以通过查看/etc/shells 文件中的内容来查看自己主机中当前有哪些种类的Shell，命令如下（下面是在笔者Linux 主机中查看信息的结果）：

![这里写图片描述](http://img.blog.csdn.net/20160322093659004)

使用下面的命令来查看Linux 当前正在使用的Shell 类型：

```
echo $SHELL
```
显示/bin/bash(Ubuntu下)

$SHELL是一个环境变量，它记录了Linux 当前用户所使用的Shell类型。用户可以通过直接输入各种Shell的二进制文件名（因为这些二进制文件本身是可以被执行的），来进入到该Shell下，比如进入bash可以直接输入：

```
$ /bin/bash
```
这个命令为用户又启动了一个Shell，这个Shell在最初登录的那个Shell之后，称为下级的Shell或子Shell。使用命令：

```
$ exit
```
可以退出这个子Shell。使用不同的Shell 的原因在于它们各自都有自己不同的特点，下面简单介绍Linux 下各种不同的Shell类型的特点。

**1.ash**
ash Shell是由Kenneth Almquist编写的，是Linux 中占用系统资源最少的一个小Shell，它只包含24个内部命令，因而使用起来很不方便。

**2. bash**
bash是Linux系统默认使用的Shell，它由Brian Fox 和Chet Ramey共同完成，是BourneAgain Shell的缩写，内部命令一共有40 个。Linux 使用它作为默认的Shell是因为它具有以下特色：

可以使用类似DOS下面的doskey的功能，用上下方向键查阅和快速输入并修改命令。

自动通过查找匹配的方式，给出以某字串开头的命令。

包含了自身的帮助功能，你只要在提示符下面键入help就可以得到相关的帮助信息。

**3. ksh**
ksh是Korn Shell的缩写，由Eric Gisin编写，共有42 条内部命令。该Shell最大的优点是几乎和商业发行版的ksh 完全相容，这样就可以在不用花钱购买商业版本的情况下尝试商业版本的性能了。

**4. csh**
csh 是Linux 比较大的内核，它由以William Joy 为代表的共计47 位作者编成，共有52个内部命令。该Shell其实是指向/bin/tcsh这样的一个Shell，也就是说，csh其实就是tcsh。

**5. zch**
zch是Linux 最大的Shell之一，由Paul Falstad完成，共有84 个内部命令。如果只是一般的用途，没有必要安装这样的Shell。

### Shell 命令的两种形式
1.内部命令
内部命令内置于Shell源码中，即存在于内存中，一般比较简短，代码量很少，执行起来速度快，经常会使用，比如cd、echo。它与shell本身处在同一进程内（它就写在Shell这个程序里面）,当打开Shell时，操作系统会将Shell程序放入内存 。

2.外部命令
外部命令一般功能比较强大，包含的代码量也较大，所以在系统加载时并不随系统一起被加载到内存中，而是在需要时才调用，它们是存在于文件系统中某个目录下的单独的程序，当执行外部命令时，会到文件系统中文件的目录中寻找，例如 cp 、rm。 

3.查看命令类型
type  command

例如：

```
$ type cd 
cd is a shell builtin(内部命令)
```

```
$ type cp
cp is /bin/cp(外部命令)
```

### Shell初始化
shell启动时将运行初始化文件来初始化自己，而具体选取哪些初始化文件，系统依据shell的类型来确定。

**Shell类型：**
登陆shell、交互式非登陆shell（通过命令bash启动一个子shell）、非交互式shell 。

**Shell类型的理解：**
登陆shell和非登陆shell 、 交互式shell与非交互式shell，首先这两个是从不同维度来划分的，一个是否交互式，一个是否登陆。

交互式模式就是shell等待你的输入，并立即执行你提交的命令，之所以被称为交互式是因为shell与用户进行了交互；

非交互式（执行shell脚本）：shell不与你进行交互，而是读取存放在文件中的命令，并执行他们，当读到文件尾时，shell结束。

#### **1.登录Shell**
指的是当用户登录系统时所用的那个shell（Linux系统启动时候输入用户名、密码，然后会启动第一个shell）。

然后这个登录shell 将查找几个不同的启动文件来处理其中的命令（它的作用是初始化linux系统相关配置）, bash shell 处理文件的顺序如下：
1）系统登录后，shell首先执行/etc/profile中的命令(超级用户)。设置这个文件后，可以为系统内所有的bash用户建立默认的特征（不同版本的Linux在此文件放置的命令不尽相同）；

/etc/profile :系统级的初始化文件,如里面代码有：如果是超级用户则提示符用#，如果是普通用户则提示符用$.

2）当某个用户定路后，shell依次查找~/.bash_profile、~/.bash_login、～/.profile这几个文件，并执行它找到的第一个文件中的命令，可以将命令放在这些文件中，以重写/etc/profile文件中默认的设置；

3）当用户注销时，bash执行文件~/.bash_logout中的命令，这个文件包含了退出会话时执行的清理命令，退出等，如：exit退出。

#### 2.交互式非登录shell 
交互式非登录shell 就是指你在当前图形界面中打开“终端”出来的那种窗口程序，和登录shell相比，它是“非登录”的，你并不需要输入用户名和码；和非交互式shell相比，这是“交互式”的，就像你说的那它：你输入什么，它就解释出什么。

并不执行前面提到的初始化文件中的命令，继承了登录 shell 设置的 shell 变量。

在交互式非登录shell里，只读取/etc/bash.bashrc和~/.bashrc（该本件包含了专用于你的bash shell的shell信息，当登录时或者每次打开新shell时，该文件即被读取

#### **3.非交互式Shell**

非交互式shell指的是以shell script(脚本)方式运行。在这种模式下运行时shell 并不与用户进行交互（除非在运行时需要用户指定运行参数），而是读取存放在文件中的命令并执行它们。当它读到文件的结尾，shell 也就终止了。这些shell从登陆时就继承了由这些启动文件设置的shell变量。

并不执行前面描述的初始化文件中的命令，继承了登录 shell 设置的 shell 变量。

**启动交互式shell:**
$/bin/sh filename或bash filename

**退出交互式shell:**
文件执行完自动退出

**运行shell脚本**
(1)编写一个脚本文件
文件名为：firsttest.sh ,vi firsttest.sh
内容为：
\#！/bin/bash
data
who
(2)添加可执行权限
$chmod +x firsttext.sh  //因为刚创建的文件权限为0644,没有可执行权限
(3)运行脚本
执行方式一：

```
$ ./firsttext.sh //使用当前shell执行脚本
```
执行方式二：

```
$ bash firsttext.sh 或 /bin/sh firsttext.sh 
//使用新的shell(/bin/sh)执行脚本
```

#### 为什么要分系统级初始化文件和用户级初始化文件？
/etc/profile:系统级初始化文件
~/.bashrc:用户级初始化文件

因为linux是多用户操作系统，系统级文件只初始化shell的大体。
而~/.bashrc每一个用户都可以配置它，不同的用户配置的~/.bashrc,则shell有不同的功能（即用户可以进行个性化设置）。


