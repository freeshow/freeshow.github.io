---
title: Shell编程（一）
date: 2016-07-23 19:52:38
tags: Linux
categories:
---


参考自：[Linux Shell脚本教程：30分钟玩转Shell脚本编程](http://c.biancheng.net/cpp/shell/)

<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## Shell编程的特点
### Shell有两种执行命令的方式：

- 交互式（Interactive）：解释执行用户的命令，用户输入一条命令，Shell就解释执行一条。
- 批处理（Batch）：用户事先写一个Shell脚本(Script)，其中有很多条命令，让Shell一次把这些命令执行完，而不必一条一条地敲命令。

Shell脚本和编程语言很相似，也有变量和流程控制语句，但Shell脚本是解释执行的，不需要编译，Shell程序从脚本中一行一行读取并执行这些命令，相当于一个用户把脚本中的命令一行一行敲到Shell提示符下执行。

### 脚本语言的特点：
脚本语言又被称为扩建的语言，或者动态语言，是一种编程语言。

脚本语言是为了缩短传统的编写-编译-链接-运行过程而创建的计算机编程语言。

一个脚本通常是解释运行而非编译。

1.脚本语言是一种解释性的语言,例如Python、Shell等等,它不象c\c++等可以编译成二进制代码,以可执行文件的形式存在，脚本语言不需要编译，可以直接用，由解释器来负责解释。

2.脚本语言一般都是以文本形式存在,类似于一种命令。

3.相对于编译型计算机编程语言：用脚本语言开发的程序在执行时，由其所对应的解释器（或称虚拟机）解释执行。系统程序设计语言是被预先编译成机器语言而执行的。脚本语言的主要特征是：程序代码即是脚本程序，亦是最终可执行文件。脚本语言可分为独立型和嵌入型，独立型脚本语言在其执行时完全依赖于解释器，而嵌入型脚本语言通常在编程语言中（如C，C++，VB，Java等）被嵌入使用。

4.和系统程序设计语言相比：不同是脚本语言是被解释而系统程序设计语言是被编译。被解释的语言由于没有编译时间而提供快速的转换，通过允许用户运行时编写应用程序，而不需要耗时的编译/打包过程。解释器使应用程序更加灵活，脚本语言的代码能够被实时生成和执行。脚本语言通常都有简单、易学、易用的特性，目的就是希望能让程序设计师快速完成程序的编写工作。


----------

## 第一个Shell脚本

打开文本编辑器，新建一个文件，扩展名为sh（sh代表shell），扩展名并不影响脚本执行，见名知意就好，如果你用php写shell 脚本，扩展名就用php好了。

输入一些代码：

```
#!/bin/bash
echo "Hello World !"
```
“#!” 是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种Shell。echo命令用于向窗口输出文本。

### 运行Shell脚本有两种方法：

**1.作为可执行程序**

将上面的代码保存为test.sh，并 cd 到相应目录：

```
chmod +x ./test.sh  #使脚本具有执行权限
./test.sh  #执行脚本
```
<font color=red>注意，一定要写成./test.sh，而不是test.sh。</font>运行其它二进制的程序也一样，直接写test.sh，linux系统会去PATH里寻找有没有叫test.sh的，而只有/bin, /sbin, /usr/bin，/usr/sbin等在PATH里，你的当前目录通常不在PATH里，所以写成test.sh是会找不到命令的，要用./test.sh告诉系统说，就在当前目录找。

通过这种方式运行bash脚本，第一行一定要写对，好让系统查找到正确的解释器。

这里的"系统"，其实就是shell这个应用程序（想象一下Windows Explorer），但我故意写成系统，是方便理解，既然这个系统就是指shell，那么一个使用/bin/sh作为解释器的脚本是不是可以省去第一行呢？是的。

**2.作为解释器参数**

这种运行方式是，直接运行解释器，其参数就是shell脚本的文件名，如：

```
/bin/sh test.sh
```

这种方式运行的脚本，不需要在第一行指定解释器信息，写了也没用。

再看一个例子。下面的脚本使用 read 命令从 stdin 获取输入并赋值给 PERSON 变量，最后在 stdout 上输出：

```
#!/bin/bash

# Author : mozhiyan
# Copyright (c) http://see.xidian.edu.cn/cpp/linux/
# Script follows here:

echo "What is your name?"
read PERSON
echo "Hello, $PERSON"
```

运行脚本：

```
chmod +x ./test.sh
$./test.sh
What is your name?
mozhiyan
Hello, mozhiyan
$
```


----------


## Shell变量

Shell支持自定义变量。


----------


### 变量定义

**格式：**变量名=变量值

注意，变量名和等号之间不能有空格，这可能和你熟悉的所有编程语言都不一样。同时，变量名的命名须遵循如下规则：

- 首个字符必须为字母（a-z，A-Z）。
- 中间不能有空格，可以使用下划线（_）。
- 不能使用标点符号。
- 不能使用bash里的关键字（可用help命令查看保留关键字）。

eg:

```
myName="ZhangSan"
myNum=100
```


----------


### 使用变量

使用一个定义过的变量，只要在变量名前面加美元符号（$）即可，如：

```
your_name="ZhangSan"
echo $your_name
echo ${your_name}
```
变量名外面的花括号是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界，比如下面这种情况：

```
for skill in Ada Coffe Action Java 
do
    echo "I am good at ${skill}Script"
done
```
如果不给skill变量加花括号，写成echo "I am good at \$skillScript"，解释器就会把$skillScript当成一个变量（其值为空），代码执行结果就不是我们期望的样子了。

<font color=red>推荐给所有变量加上花括号，这是个好的编程习惯。</font>


----------


### 重新定义变量

已定义的变量，可以被重新定义，如：

```
myName="ZhangSan"
echo ${myName}

myName="LiSi"
echo ${myName}
```


----------
### 只读变量

使用 readonly 命令可以将变量定义为只读变量，只读变量的值不能被改变。

下面的例子尝试更改只读变量，结果报错：

```
#!/bin/bash

myName="ZhangSan"
readonly myName
myName="LiSi"
```
运行脚本，结果如下：

> ./test.sh: line 5: myName: readonly variable


----------
### 删除变量

使用 unset 命令可以删除变量。语法：

```
unset variable_name
```

变量被删除后不能再次使用；unset 命令不能删除只读变量。

举个例子：

```
#!/bin/bash

myName="ZhangSan"
unset myName
echo ${myName}
```
上面的脚本没有任何输出。


----------
### 变量类型

运行shell时，会同时存在三种变量：

#### 1.用户变量(局部变量)

用户自己定义的变量。
定义格式： 变量名=变量值

局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。

#### 2.环境变量
所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。

修改shell环境变量的方法大致分为两种，一种是使用export命令，一种是修改配置文件。

(1)export命令，该方式只对该次交互式非登录shell有效，退出shell再次进入后修改的内容丢失。

格式：export 变量名=变量值

例如将/home/xxxx/bin添加到PATH中，可以使用命令：
export PATH=$PATH:/home/xxxx/bin。

这条命令可以直接在shell中执行，也可以放在脚本中，但放在脚本中需要使用source命令来执行脚本。

修改后可以使用命令：
echo $PATH或者export
查看是否修改成功。如果输出的PATH中含有/home/xxxx/bin则表明修改成功。

(2)修改配置文件
如果想在两个交互式非登录shell中都访问到变量，则可以将变量添加到~/.bashrc中，因为每打开一个交互式非登录shell都会重新加载~/.bashrc文件，故都会访问到。
当然，如果把变量加载到/etc/profile ，所有登陆shell都更能访问的到了。

#### 3.shell变量
shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行。

**常用的shell变量**
HOME  ：保存注册目录 
TERM  ：终端类型（xterm 图形终端 ，linux 文本终端） 
UID  ：当前用户的标识符，取值是由数字构成的字符串 
PWD  ：当前工作目录的绝对路径名，该变量的取值随 cd 命令的使用而 
变化 
PS1  ：主提示符，“  # ”    “ $ ” 
PS2:    辅助提示符，在输入行末尾“ \”  ，输出该提示符 
IFS  ：  shell 指定的缺省域分隔符（空格  table   ） 
LOGNAME  ：此变量保存登录名 
SHELL  ：保存缺省 shell 
RANDOM  ：产生随机数
PATH  ：保存用冒号分隔的目录路径名 

echo $PATH
/usr/local/sbin:/usr/local/bin/:/usr/sbin:/usr/bin:/sbin:/bin/:/usr/games
在shell中执行命令 ls等于执行/bin/ls 即当在shell中执行命令ls时，会在PATH变量的目录中寻找有没有ls这个文件，当找到/bin/ls就执行文件/bin/ls.


----------
### Shell特殊变量

前面已经讲到，变量名只能包含数字、字母和下划线，因为某些包含其他字符的变量有特殊含义，这样的变量被称为特殊变量。

例如，$ 表示当前Shell进程的ID，即pid，看下面的代码：

```
$ echo $$
```
运行结果：

> 29949


----------


#### **特殊变量列表:**
| 变量 | 含义 | 
| :-----: |:-------------| 
| \$0 | 当前脚本的文件名 | 
| \$n | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。 | 
| $# | 传递给脚本或函数的参数个数。 |
|\$*| 传递给脚本或函数的所有参数。|
|\$@|传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，下面将会讲到。|
|\$?|上个命令的退出状态，或函数的返回值。|
|\$$|当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。|


#### **命令行参数**

运行脚本时传递给脚本的参数称为命令行参数。命令行参数用 $n 表示，例如，$1 表示第一个参数，$2 表示第二个参数，依次类推。

请看下面的脚本：

```
#!/bin/bash

echo "File Name: $0"
echo "First Parameter : $1"
echo "First Parameter : $2"
echo "Quoted Values: $@"
echo "Quoted Values: $*"
echo "Total Number of Parameters : $#"
```

运行结果：

> $./test.sh Zara Ali
File Name : ./test.sh
First Parameter : Zara
Second Parameter : Ali
Quoted Values: Zara Ali
Quoted Values: Zara Ali
Total Number of Parameters : 2


----------


####  \$* 和 \$@ 的区别

$* 和 $@ 都表示传递给函数或脚本的所有参数，不被双引号(" ")包含时，都以"$1" "$2" … "$n" 的形式输出所有参数。

但是当它们被双引号(" ")包含时，"$*" 会将所有的参数作为一个整体，以"$1 $2 … $n"的形式输出所有参数；"$@" 会将各个参数分开，以"$1" "$2" … "$n" 的形式输出所有参数。

下面的例子可以清楚的看到 $* 和 $@ 的区别：

```
#!/bin/bash
echo "\$*=" $*
echo "\"\$*\"=" "$*"

echo "\$@=" $@
echo "\"\$@\"=" "$@"

echo "print each param from \$*"
for var in $*
do
    echo "$var"
done

echo "print each param from \$@"
for var in $@
do
    echo "$var"
done

echo "print each param from \"\$*\""
for var in "$*"
do
    echo "$var"
done

echo "print each param from \"\$@\""
for var in "$@"
do
    echo "$var"
done
```

执行 ./test.sh "a" "b" "c" "d"，看到下面的结果：

```
$*=  a b c d
"$*"= a b c d  #"a b c d" 是一个整体
$@=  a b c d
"$@"= a b c d
print each param from $*
a
b
c
d
print each param from $@
a
b
c
d
print each param from "$*" #"$*"是一个整体输出参数。
a b c d
print each param from "$@"
a
b
c
d
```


----------


#### 退出状态
$? 可以获取上一个命令的退出状态。所谓退出状态，就是上一个命令执行后的返回结果。

退出状态是一个数字，一般情况下，大部分命令执行成功会返回 0，失败返回 1。

不过，也有一些命令返回其他值，表示不同类型的错误。

下面例子中，命令成功执行：

```
$./test.sh Zara Ali
File Name : ./test.sh
First Parameter : Zara
Second Parameter : Ali
Quoted Values: Zara Ali
Quoted Values: Zara Ali
Total Number of Parameters : 2
$echo $?
0
$
```

$? 也可以表示函数的返回值，后续将会讲解。


