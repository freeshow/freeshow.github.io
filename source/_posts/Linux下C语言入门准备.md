---
title: Linux下C语言入门准备
date: 2016-07-24 00:23:13
tags: Linux
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 目录规范
目录结构：

- src：	存放源文件 *.c  *.cpp
- include：存放头文件 *.h
- obj	：存放目标文件 *.o (中间产物)
- lib：存放库文件 *.so  *.a *.so.1
- bin	：存放二进制可执行程序 


----------
## 预处理指令

\#开始的命令
例如：
1.#include

头文件搜索方式:

- \#include ""	//非标准头文件(用户的),优先从用户指定的头文件路径进行搜索	
- \#include <>	//标准头文件(系统的),优先从系统的默认头文件路径进行搜索

2.#if
```
#if 0   在预处理期代码会被去除(等同与注释，注释在预处理期会被去除)
注释一
 /*

	注释二
 
 */
#endif
```

3.\#ifndef ：用于编写头文件，避免头文件重复定义

例如：

```
a.h
	int i;
-----------------
b.h
	#include "a.h"
------------------
c.h
	#include "a.h"  -->  int i;
	#include "b.h"  -->  int i;
则在c.h中重复定义了a.h，是不允许的
```
则可以用#ifndef,避免头文件的重复定义：

```
a.h
#ifndef __A_H__		//如果没有定义a.h则执行下面的代码
#define __A_H__
	int i;
#endif
-----------------
b.h
	#include "a.h"
------------------
c.h
	#include "a.h"		//没有定义a.h，则定义a.h  
	#include "b.h"		//因为a.h已经定义了，故在a.h中#ifndef下面的代码不执行了，故造不成a.h头文件的重复定义

```

----------


## gcc

-o 指定生成的文件名
-I 指定头文件路径，包括标准头文件(系统的)和*非标准头文件(用户的)*
-L 指定库文件路径
-l 指定库文件名

gcc的编译流程分为了四个步骤，分别为：

1.预处理期：

1. 去除注释
2. 替换头文件	#include
3. 条件编译		#if...
4. 宏展开		#define 
//不检查语法错误

2.编译阶段：
生成预处理后的源代码的汇编代码

3.汇编阶段:
汇编代码进行汇编,生成了*.o的目标文件

4.链接阶段：
将*.o的目标文件链接生成二进制的可执行文件

例如：

include/print.h 
```
#ifndef 	__PRINT_H__
#define		__PRINT_H__

extern void hello();

#endif
```

src/print.c

```
#include<stdio.h>

void hello()
{
	printf(“hello world\n”);
}
```

src/main.c

```
#include   “print.h”

int main()
{
	hello();
	return 0;
}
```
gcc编译：生成中间代码
> gcc -c -o obj/print.o src/print.c -I include
gcc -c -o obj/main.o src/main.c -I include

生成：

> obj/print.o
obj/main.o

gcc编译：生成二进制文件(可执行文件)

> gcc -o bin/main obj/print.o obj/main.o

生成：

> bin/main

或者一步生成可执行文件：

> gcc -o  bin/main  src/print.c src/main.c  -I include 

路径选项 
 
 - -I dir ：头文件目录路径 
 - -L dir ：库文件目录路径 
 - -l 库名 ：链接库的使用 


----------
## 库文件
可以在程序运行时被载入的文件,以调用某个特定的模块，也可以在编译时被直接编译进可执行文件中。


**静态和共享(动态的)库：**

- 静态：在编译程序时被导入可执行文件中
- 共享：在运行程序时被导入可执行文件中(即在执行main函数的入口时导入)
- 动态:   程序中语句需要使用时载入库代码

**共享库与动态库调用时机：**
```
int main(void)   	共享库代码被载入
{
	a.....

	b.....

	c.....

	d....  (调用库内的代码)		 动态库载入

}
```

**制作库文件:**

1. 编码  -->  单独的xxx.c
		同一个模块
2. 编译生成目标文件  xxx.o

3. 生成库文件(lib开头)

- 动态：
gcc -fPCI -o libxxx.so xxx.o main.o
- 共享：
gcc -shared -fPCI -o libxxx.so xxx.o
- 静态：
		ar -rc libxxx.a xxx.o

4. 生成需要调用库文件的可执行文件
		gcc -o bin/main obj/main.o -Llib -lxxx	
	//-Llib -lxxx :告诉编译器库文件放在哪里

5. 告诉内核库文件在哪(共享) (即程序运行时，告诉内核库文件在哪)
		方法1.配置环境变量
				export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:lib/
		方法2.将库文件放到系统库文件路径下
		  		系统库文件路径有：	/lib/    /usr/lib/
				系统路径(root权限)
				//将生成的库文件放入系统库文件路径下
				sudo cp  lib/libxxx.so /usr/lib/ 

**静态和共享库的优缺点：**

	静态:在编译程序时被导入可执行文件中(即可执行文件中已经包含库文件的内容)
		优点：在程序执行时不依赖库文件
			  在编译完成后,.a的库文件可删除
		缺点：导致可执行文件较大
			  不利于项目的维护和更新
	共享:在运行程序时被导入可执行文件中(即可执行文件中不包含库文件的内容)
		优点:主程序体积较小
		     便于发布和更新
			 项目结构清晰
		缺点:丢失库文件后将导致主程序无法启动

**链接库文件：**
-lprint

libprint.so 	//共享库文件或动态库文件
libprint.a	//静态库文件
当共享库文件和静态库文件同时在./lib下时，先执行共享库文件，如果想执行静态库文件，则需删除共享库文件。


----------
## Makefile
Makefile 文件：批量编译和处理项目文件

**格式：**

```
target:depends
	commands
```

**例如：**

```
target	:	depends		//目标:桌子，依靠：木板,钉子 ，命令：组合

桌子：木板  钉子
	组合(commands)       
木板:树
	切割
钉子:铁
	打造
```

根据把上面gcc的例子，编写Makefile文件：

```
bin/main:obj/main.o obj/print.o
	gcc -o bin/main obj/main.o obj/print.o
obj/main.o:src/main.c  include/print.h
	gcc -c -o obj/main.o src/main.c -I include
obj/print.o:src/main.c include/print.h
	gcc -c -o obj/print.o src/print.c -I include
```

make： 默认执行Makefile
make -f：指定Makefile 文件名
//当存在Makefile和Makefile_shared两个makefile文件时，make默认执行Makefile。如果想要执行Makefile_shared,则执行命令：make -f Makefile_shared

**清理步骤：**

命令：make clean 

需要在Makefile中添加clean

```
clean:
	rm -fr obj/*
	rm -fr bin/*
	rm -fr lib/* 
```


----------
## 调试：

找出程序中错误的代码或逻辑并修正

**错误error分类：**

1. 编译时错误(编译器)
		a.根据编译器的错误提示debug
		b.添加删除程序法(注释法)，即用//或/*  */注释掉部分代码缩小范围查找错误

2. 运行时错误(内核)
		a.添加删除程序法
		b.标记法（在代码周围用printf()打印一句话，可以通过打印的话，缩小范围）
		c.使用调试工具(gdb)

**警告warning(可被忽视)：**
gcc警告提示

1. Wall 类警告 
 	-Wall 打开所有类型语法警告,建议使用 
 	-Wchar-subscripts 如果数组使用 char 类型变量作为下标值,则发出警告 
 	-Wcomment 当 ’ /*’ 出现在 ’ /*......*/’ 注释中,或则 ’ \’ 出现在 ’ //......’ 注释 结尾	处时, 使用 -Wcomment 会给出警告,它可能会影响程序的运行结果

2. 非 Wall 类警告提示 
 	-ansi 强制 GCC 生成标准语法所要求的告警信息。 
 	-pedantic 允许发出 ANSI C 标准所列的全部警告信息


**版本引发的问题(如C98和C99之间的问题)：** 

1989第一个标准版本：ANSC C89 = ISO  C90
1999：c99
当出现因为版本不同时， 可在编译时加   -std=c99  就可以了


----------
## gdb调试

**gdb调试步骤：**
gdb调试工具 

1. 编译gdb调试文件： 
	gcc -g src/sum.c -o bin/sum  //-g把代码增加到可执行文件中 

2. 开启gdb对文件的调试： 
	方法1：gdb ./bin/sum 
	方法2：gdb 
	       file ./bin/sum 
3.  ! command :执行外部命令 

4. 先设断点，后执行。即先break/b后run/r 
	如果不设断点就执行，则程序就会快速执行完毕，设置断点后程序就会在断点处暂停，如需往下执行就可以手动往下执行。 


**调试命令:** 

```
		file --> 指定调试文件 
		list/l --> 查看原代码(默认10行) 
		list n --> 列出n附近的10行代码 
		run/r --> 运行 
		断点：在程序执行时在某个代码行处暂停 
		break/b --> 设置断点 如: b main  b 10 
		info breakpoints/i b --> 查看所有的断点 
		delete breakpoints  number--> 删除断点 
		next/n --> 单步运行下一行代码(遇到函数调用不会进入函数内运行) 
		step/s --> 单步运行下一行代码(遇到函数调用会进入函数内运行) 
		回车-->执行上一次操作 
		print/p	 变量-->查看变量的值 
		display 变量-->追踪变量的值 
		set variable 变量名=值-->修改变量的值，使程序跳到变量获得结果处。 
		until/u 26 --> 跳到某行 
		finish --> 跳出函数(finish不能简写成f) 
		quit/q --> 退出gdb 


		display /i $pc --> 同时显示c代码和汇编代码 

		disassemble 函数名 --> 查看指定函数的反汇编代码 
```

**objdump工具** 
	
	objdump -xd sum.o 
	objdump -xd sum 
	将.o目标文件或可执行文件进行反汇编


