---
title: Java之数据类型、变量和数组
date: 2016-07-23 23:20:02
tags: Java
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

**1.基本数据类型**

基本类型被定义为具有明确的范围和数学行为。C和C++这类语言允许整数的大小随着执行环境的要求而变化。然而，Java与之不同。因为Java需要具备可移植性，所有数据类型都具有严格定义的范围。例如，无论在哪种特定的平台上，int总是32位的，因而可以编写出不经修改就能确保在任何体系结构的计算机上都能运行的程序。虽然严格指定整数的范围在某些环境中可能会造成一些性能损失，但为了实现可移植性这么做是必须的。

Java基本数据类型都是有符号的、正的或负的整数。Java不支持无符号的、只是正值得整数。

Java中布尔型只有两个逻辑值——true和false。True和false不能转化为任何数字表示形式。在Java中，字面值true不等于1,字面值false不等于0;
Java字符串需要重点指出的是，他们的开头和结尾必须位于同一行中。与其他某些语言不通，在Java中没有续行的转义字符。

**2.变量的作用域和生命周期**

Java允许在任何代码块中声明变量。代码块以开花括号开始并以闭花括号结束。代码块决定了作用域。因此，每当开始一个新的代码块时就创建一个新的作用域。作用域决定了对象对程序其他部分的可见性，并且决定了这些对象的生命周期。

在Java中，两种主要的作用域分别是由类和方法定义的。
由方法定义的作用域从方法的开括号开始。然而，如果方法具有参数，那么它们也会被包含到方法的作用域中。

作为通用规则，在作用域中声明的变量，对于在作用域之外定义的代码是不可见的(即不可访问的)。因此，当在某个作用域中声明变量时，就局部化了该变量，并保护它免受未授权的访问和修改。实际上，作用域规则为封装提供了基础。

作用域是可以嵌套的。当遇到这种情况时，外层的作用域包围了内层的作用域。这意味着在外层作用域中声明的对象对于内层作用域中的代码是可见的。然而，反过来就不是这样了，在内层作用域中声明的对象，在内层作用域之外是不可见的。

当进入变量的作用域时创建变量，当离开他们的作用域时销毁变量。这意味着一旦离开作用域，变量就不会再保持原来的值。所以对于在方法中声明的变量来说，在两次调用该方法之间，变量不会保持他们的值。此外，在对于代码库中声明的变量来说，当离开代码块时会丢失它们的值。因此，变量的生存期被限制在作用域之内。

如果变量声明包含了初始或器，那么每当进入声明变量的代码块时都会重新初始化变量。

尽管可以嵌套代码块，但是在内层代码块中不能声明与外层代码块具有相同名称的变量。例如，下面的程序是非法的：

```
class ScopeErr
{
	public static void main(String[] args)
	{
		int bar = 1;
		{
			int bar =2;
		}

	}
}
```

**3.类型提升规则**

在进行运算时，所有byte、short和char类型的值都会被提升为int类型。然后，如果有一个操作数是long类型，就将整个表达式提升为long类型；如果有一个操作数是float类型，就将整个表达式提升为float类型；如果任何一个操作数为double类型，结果将为double类型。

**4.数组**

(1)一维数组

```
int[] temp = new int[3];
temp[0] = 1;
temp[1] = 2;
temp[2] = 3;
```

使用new分配一个数组，必须执行要分配元素的类型和数量。通过new分配的数组，其元素会自动初始化为0(对于数值型)、false(对于布尔类型)或null(对于引用类型)。

当声明数组时，可以对其进行初始化。数组初始化器是一个位于花括号中由逗号分隔的表达式列表。用逗号分隔数组元素的值。Java会自动创建足够大的数组，以容纳数组初始化器中指定的数组元素的数量，这是需要使用new运算符。
int[ ] temp = {1,2,3};

2）二维数组
当为多维数组分配内存时，只需要为第一维分配内存。可以单独为余下的维分配内存。

```
int twoD[][] = new int[3][];
twoD[0] = new int[1];
twoD[1] = new int[2];
twoD[2] = new int[3];
```

使用初始化器初始化二维数组：

```
int twoD[][] = 
{
	{1},
	{2,3},
	{4,5,6}
};
```

出自：《Java 8编程参考官方教程(第9版)》