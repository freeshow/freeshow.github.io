---
title: Java之控制语句
date: 2016-07-23 23:18:27
tags: Java
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

**1.switch语句**

```
switch(expression)
{
	case value1:
		//...
		break;
	case value2:
		//...
		break;
	....
	default:
		//...
		break;
}
```

对于JDK7以前的Java版本，expression必须是byte、short、int、char或枚举类型。从JDK7开始，expression可以是String类型。
在case语句中，指定的每个数值必须是唯一的常量表达式。case不允许重复。每个数值的类型必须与express的类型兼容。

例如：

```
class StringSwitch
{
	public static void main(String[] args)
	{
		String str = "two";
		switch(str)
		{
			case "one":
				System.out.println("one");
				break;
			case "two":
				System.out.println("two");
				break;
			default:
				System.out.println("no match");
				break;
		}
	}
}
```

输出为： two

**2.for语句**

```
for(initialization; condition; iteration)
{
	//body
}
```

for循环的执行过程如下：当第一次开始循环时，执行循环的initialization部分。初始化表达式只执行一次。
接下来就执行condition表达式，然后执行body,在执行iteration.完成一次循环。往后的循环就不执行initialization初始化表达式了。
执行condition-->body-->iteration循环执行。直到condition的值为false.

注意：当在for循环内部或initialization初始化变量时，当for语句结束时，变量的作用域也随之结束。也就是说变量的作用域局限于for循环。
在for循环之外，变量将不再存在。如果需要在程序的其他地方使用循环控制变量，就不能再for循环内部声明它。

**3.for-each风格的for循环(增强版for循环)**

> for(type var : collection) statement;

type制定了类型，var制定了迭代变量的名称，迭代变量用于接收来自collection集合的元素，从开始到结束，每次接收一个。collection指定了要遍历的集合。
对于循环的每次迭代，会检索出集合中的下一个元素，并存储在var中。循环会重复执行，直到得到集合中的所有元素。

关于for-each风格的for循环,有重要的一点需要声明：迭代变量var是“只读的”，因为迭代变量与背后的数组关联在一起。对迭代变量的一次赋值不会影响背后的数组。
换句话说，不能通过为迭代变量指定一个新值来改变数组的内容。


对多维数组进行迭代：

增强版for循环当对多维数组进行迭代时，因为每次迭代获取的是下一个数组，而不是单个元素。此外，for循环中迭代变量必须和获取的数组类型兼容。
例如，对于二维数组，迭代变量必须是对一维数组的引用。

```
class ForEach
{
	public static void main(String[] args)
	{
		int sum = 0;
		int[][] nums = new int[3][5];
		
		for(int i = 0; i < 3; i++)
		{
			for(int j = 0; j < 5; j++)
			{
				nums[i][j] = (i+1) * (j+1);
			}
		}
		
		for(int[] x : nums)
		{
			for(int y : x)
			{
				System.out.println("Value is: "+y);
				sum += y;
			}
		}
		System.out.println("Summation: "+sum);
	}
}
```

**4.跳转语句**

Java支持3中跳转语句：break语句、continue语句和return语句。

（1）Break语句
三种用途：
第一种：终止switch语句
第二种：退出循环
内层循环的break语句只会导致内层循环终止，对外层循环没有影响。
一个循环中可以出现多条break语句。但请小心，过多的break语句可能会破坏代码的结构。
在某条switch语句中使用的break语句，只会影响该switch语句，不会结束任何外层循环。
第三种：用作goto语句的"文明"形式

Java没有提供goto语句，因为goto语句可以随意的进入另一个程序的分支，并且是一种非结构化的方法。
使用goto语句会是代码难于理解和维护，还会妨碍特定的编译器优化。然而，在某些情况下，goto语句
对于流程控制很有价值并且结构合法。例如，当退出深度嵌套的一系列循环时，goto语句就很有用。
为了处理这种情况，Java定义了break语句的一种扩展形式。例如。通过使用这种形式的break语句，可以
中断一个或多个代码块。这些代码块不必是某个循环或switch语句的一部分，它们可以是任何代码块。此外，
可以精确指定准备在什么位置继续执行，因为这种形式的break语句使用标签进行工作。正如即将看到的，
break语句提供了goto语句的优点，而没有goto语句的问题。

使用标签的break语句的一般形式如下：
break label;
最常见的情况是，label是标识一个代码块的标签的名称。它既可以是一个独立的代码块，也可以是作为另一个语句的目标代码块。
当执行这种形式的break语句时，程序的执行控制会跳出由标签命名的代码块。具有标签的代码块必须包含break语句，但是不必立即包含break语句。
这意味着可以使用带有标签的break语句退出一系列的嵌套的代码，但是不能使用break语句将控制转移出没有包含break语句的代码块。

为了命名代码块，可以在代码块之前放置一个标签。标签可以是任意合法的Java标识符，后面跟随着一个冒号。一旦命名代码块，
就可以使用命名标签作为break语句的目标。这样一来，就可以在标识的代码块的末端恢复执行。
例如：

```
class Break
{
	public static void main(String[] args)
	{
		boolean t = true;
		first:
		{
			second:
			{
				third:
				{
					System.out.println("Before the break.");
					if(t) break second; //break out of second block;
					System.out.println("This won't execute");
				}
				System.out.println("This won't execute");
			}
			System.out.println("This is after second block.")
		}
	}
}
```

输出为：
Before the break.
This is after second block

带有标签的break语句的最常用用途之一是退出嵌套的循环。例如下面的程序中，外层循环只执行一次：

```
case Break
{
	public static void main(String[] args)
	{
		outer:for(int i = 0;i < 3;i++)
		{
			System.out.println("Pass"+i+":");
			for(int j = 0;j < 100;j++)
			{
				if(j == 10) break outer;
				System.out.print(j+" ");
			}
			System.out.println("This will not print");
		}
		System.out.println("Loops complete.");
	}
}
```

输出为：
Pass 0: 0 1 2 3 4 5 6 7 8 9 Loops complete.

可以看出，当内层循环中断外层循环时，两个循环都终止了。
注意，这个例子为for语句添加了标签，有一个代码块作为该for语句的标签。

要牢记，程序的执行控制不能跳至为没有包含break语句的代码块定义的标签。
例如下面程序是无效的，不能通过编译：

```
//This program contains an error
class BreakErr
{
	public static void main(String[] args)
	{
		one:for(int i = 0;i < 3;i++)
		{
			System.out.print("Pass"+i+":")
		}
		
		for(int j = 0;j < 100;j++)
		{
			if(j == 10) break one; //WRONG
			System.out.print(j+" ");
		}
	}
}
```

因为具有标签one的循环没有包含break语句，所以不能从该代码块中转移出程序的执行控制。

出自：《Java 8编程参考官方教程(第9版)》