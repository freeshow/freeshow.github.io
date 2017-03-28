---
title: Shell编程（三）---字符串、数组
date: 2016-07-23 20:06:22
tags: Shell
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

转载自：[Linux Shell脚本教程：30分钟玩转Shell脚本编程](http://c.biancheng.net/cpp/shell/)，只用于自己学习Shell编程。
## Shell字符串

字符串是shell编程中最常用最有用的数据类型（除了数字和字符串，也没啥其它类型好用了），字符串可以用单引号，也可以用双引号，也可以不用引号。单双引号的区别跟PHP类似。

### 单引号

```
str='this is a string'
```

单引号字符串的限制：

- 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；
- 单引号字串中不能出现单引号（对单引号使用转义符后也不行）。

### 双引号

```
your_name='qinjx'
str="Hello, I know your are \"$your_name\"! \n"
```
输出结果：

```
Hello, I know your are "qinjx"! 
```

双引号的优点：

- 双引号里可以有变量
- 双引号里可以出现转义字符

### 拼接字符串

```
your_name="qinjx"
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"

echo $greeting $greeting_1
```
输入结果：

```
hello, qinjx ! hello, qinjx !
```

### 获取字符串长度

```
string="abcd"
echo ${#string} #输出 4
```

### 提取子字符串

```
string="alibaba is a great company"
echo ${string:1:4} #输出liba
```

### 查找子字符串

```
string="alibaba is a great company"
echo `expr index "$string" y`
```
输出结果为：26 
即字符串从1开始，y字符在第26位置上。


----------
## 数组

Shell在编程方面比Windows批处理强大很多，无论是在循环、运算。

bash支持一维数组（不支持多维数组），并且没有限定数组的大小。类似与C语言，数组元素的下标由0开始编号。获取数组中的元素要利用下标，下标可以是整数或算术表达式，其值应大于或等于0。

### 定义数组
在Shell中，用括号来表示数组，数组元素用“空格”符号分割开。定义数组的一般形式为：
    array_name=(value1 ... valuen)
    
例如：

```
array_name=(value0 value1 value2 value3)
```
或者

```
array_name=(
value0
value1
value2
value3
)
```

还可以单独定义数组的各个分量：

```
array_name[0]=value0
array_name[1]=value1
array_name[2]=value2
```
<font color=red>可以不使用连续的下标，而且下标的范围没有限制。</font>


----------
### 读取数组
读取数组元素值的一般格式是：
    ${array_name[index]}
    
例如：

```
valuen=${array_name[2]}
```
举个例子：

```
#!/bin/sh

NAME[0]="Zara"
NAME[1]="Qadir"
NAME[2]="Mahnaz"
NAME[3]="Ayan"
NAME[4]="Daisy"
echo "First Index: ${NAME[0]}"
echo "Second Index: ${NAME[1]}"
```
运行脚本，输出：

```
$./test.sh
First Index: Zara
Second Index: Qadir
```
使用@ 或 * 可以获取数组中的所有元素，例如：

```
${array_name[*]}
${array_name[@]}
```

举个例子：

```
#!/bin/sh

NAME[0]="Zara"
NAME[1]="Qadir"
NAME[2]="Mahnaz"
NAME[3]="Ayan"
NAME[4]="Daisy"
echo "First Method: ${NAME[*]}"
echo "Second Method: ${NAME[@]}"
```
输出结果：

```
First Method: Zara Qadir Mahnaz Ayan Daisy
Second Method: Zara Qadir Mahnaz Ayan Daisy
```


----------
### 获取数组的长度
获取数组长度的方法与获取字符串长度的方法相同，例如：

```
# 取得数组元素的个数
length=${#array_name[@]}
# 或者
length=${#array_name[*]}
# 取得数组单个元素的长度
lengthn=${#array_name[n]}
```


