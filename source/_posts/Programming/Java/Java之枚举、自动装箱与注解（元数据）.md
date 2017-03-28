---
title: Java之枚举、自动装箱与注解（元数据）
date: 2016-07-23 23:11:47
tags: Java
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

**12.1 枚举**

	形式最简单的枚举，看起来和其它语言中的枚举类似。但是，这种相似性只是表面上的。因为在Java中，枚举定义了一种类类型，极大的扩展了枚举
的功能。例如在Java中，枚举可以具有构造函数、方法以及实例变量。

**12.1.1	枚举的基础知识**

	创建枚举需要使用关键字enum.例如，下面是一个简单的枚举，其中列出了各种苹果的品种：
	
```
	enum Apple
	{
		Jonathan, GoldenDel, RedDel, Winesap, Cortland
	}
```
	
	标识符Jonathan、GoldenDel等被称为枚举常量。每个枚举常量被隐式声明为Apple的公有、静态final成员。此外，枚举常量的类型是声明它们的枚举类型。因此在Java语言中，这些常量被称为是"自类型化的"。 

	定义了枚举后，可以创建枚举类型的变量。但是，尽管枚举定义了类类型，却不能使用new实例化枚举。反而，枚举变量的声明和使用方式在许多方面与基本类型相同。例如：
	
	Apple ap;
	ap = Apple.RedDel;
	
	可以使用关系运算发"=="比较两个 枚举常量的相等性。例如：
	
	if(ap == Apple.GoldenDel) //...
	
	枚举值也可以用于控制switch语句。例如：
	
```
	switch(ap)
	{
		case Jonathan:
			//...
		case Winesap:
			//...
			
	}
```
	
	注意在case语句中，枚举常量的名称没有使用枚举类型的名称进行限定。也就是说，使用的是Winesap而不是Apple.Winesap。这是因为switch表达式中枚举类型已经隐式的指定了case常量的枚举类型，所以case语句中不需要使用枚举类型的名称对常量进行限制。实际上，如果试图这么做的话，会造成编译时错误。

	当显示枚举常量时，例如在println()语句中，会输出常量的名称。例如：
	
	System.out.println(Apple.Winesap);
	
	会显示名称"Winesap"。 
	
**12.1.2 values()和valueOf()方法**

	所有枚举都包含两个预定义的方法：values()和valueOf()。它们的一般形式如下所示：
	
	public static enum-type[] values()
	public static enum-type valueOf(String str)
	
	values()方法返回一个包含枚举常量列表的数组，valueOf()方法返回与传递到参数str的字符串相对应的枚举常量。对于这两个方法enum-type是枚举类型。例如，对于前面的Apple枚举，Apple.valueOf("Winesap")的返回类型是Winesap.

	例如：
	
```
	enum Apple
	{
		Jonathan,GoldenDel,RedDel,Winesap,Cortland
	}
	
	class EnumDemo2
	{
		public static void main(String[] args)
		{
			Apple ap;
			
			System.out.println("Here are all Apple constants:");
			
			Apple[] apples = Apple.values();
			for(Apple a : apples)
			{
				System.out.println(a);
			}
			
			System.out.println();
			
			ap = Apple.valueOf("Winesap");
			System.out.println("ap contains "+ap);
		}
	}
```
	
	输出如下：
	Here are all Apple constants:
	Jonathan
	GoldenDel
	RedDel
	Winesap
	Cortland
	
	ap constants Winesap
	
**12.1.3 Java 枚举是类类型**

	Java枚举是类类型，虽然不能使用new实例化枚举，但是枚举却有许多和其它类相同的功能。枚举定义了类，这为Java枚举提供了超乎寻常的功能。例如可以为枚举提供构造函数、添加实例变量和方法，甚至可以实现接口。

	需要理解的重要一点是：每个枚举常量都是所属枚举类型的对象(故枚举不许要使用new实例化枚举，创建枚举类型时，已经实例化了枚举)。因此，如果为枚举定义了构造函数，那么当创建每个枚举常量时都会调用该构造函数。此外，对于枚举定义的实例变量，每个枚举常量都有它自己的副本。例如：

```
	enum Apple
	{
		//使用下面自己定义的构造函数初始化枚举常量。
		Jonathan(10),GoldenDel(9),RedDel(12),Winesap(15),Cortland(8);
	
		private int price; //price of each apple 
	
		//Constructor
		Apple(int p){price = p;}
	
		int getPrice(){return price;}
	}

	class EnumDemo3
	{
		public static void main(String[] args)
		{
			Apple ap;
			
			//Display price of Winesap.
			System.out.println("Winesap costs "+Apple.Winesap.getPrice()+"cents.\n");
			
			//Display all apples and prices.
			System.out.println("All apple prices:");
			for(Apple a : Apple.values())
			{
				System.out.println(a+" costs "+a.getPrice()+"cents.");
			}
		}
	}
```
	
	输出如下：
	Winesap costs 15 cents.
	
	All apple prices:
	Jonathan costs 10 cents.
	GoldenDel costs 9 cents.
	RedDel costs 12 cents.
	Winesap costs 15 cents.
	Cortland costs 8 cents.
	
	
	尽管前面的例子只包含一个构造函数，但是枚举可以提供两种甚至更多种重载形式，就像其他类那样。例如下面版本的Apple提供了一个默认构造函数，可以将price变量初始化为-1，表明不能获得价格数据：

```
	enum Apple
	{
		Jonathan(10),GoldenDel(9),RedDel,Winesap(15),Cortland(8);
		
		private int price;
		
		Apple(int p){price = p;}
		
		Apple(){price = -1;}
		
		int getPrice(){return price;}
	}
```
	
	注意在这个版本中，没有为RedDel提供参数。这意味着会调用无参构造函数，并将RedDel的Price变量设置为-1。
	
	枚举有两条限制：第一，枚举不能继承其他类；第二，枚举不能是超类。这意味着枚举不能扩展。在其他方面，枚举和其他类很相似。
	需要记住的关机是:每个枚举常量都是定义它的类的对象。
	
	
**12.1.4 枚举继承自Enum类**

	尽管声明枚举时不能继承超类，但是所有枚举都自动继承超类java.lang.Enum,这个类定义了所有枚举都可以使用的一些方法。后面将详细介绍Enum类，这里先讨论它的3个方法。

	可以获取用于指示枚举常量在常量列表中位置的值，这称为枚举常量的序列值。通过ordinal()方法可以检索序列值，该方法的声明如下所示：
	
	finil int ordianl()
	
	该方法返回调用常量的序列值，序列值从0开始。
	
	可以使用compareTo()方法比较相同类型的两个枚举常量的序列值，该方法的一般形式如下：
	
	finil int compareTo(enum-type e)
	
	可以使用equals()方法来比较枚举常量和其它对象的相等性。
	

	
**12.2 类型封装器**

	我们已经知道，Java使用基本类型来保持语言支持的基本数据类型。处于性能考虑，为这些数据使用基本类型而不是对象。为这些数据使用对象，即使最简单的计算也会增加不可接受的开销。因此，基本类型不是对象层次的组成部分，它们不能继承Object类。

	虽然基本类型提供了性能方面的好处，但是有时会需要对象这种表示形式。例如，不能通过引用为方法传递基本类型。此外，Java实现的许多标准数据结构是针对对象进行操作的，这意味着不能使用这些结构存储基本类型。为了处理这些情况，Java提供了类型封装器，用来将基本类型封装到对象中。类型封装器是类。

	类型封装器包括Double、Float、Long、Interger、Short、Byte、Character以及Boolean。这些类提供了大量的方法，通过这些方法可以完全将基本类型集成到Java的对象层次中。


**12.2.1 Character封装器**

	Character是char类型的封装器。Character的构造函数为：
	
	Character(char ch);
	
	其中，ch指定了由即将创建的Character对象封装的字符。
	
	为了获取Character对象中的char值，可以调用charValue()方法，如下所示：
	
	char charValue()
	
	该方法返回封装的字符。
	
**12.2.2 Boolean封装器**

	Boolean是用来封装布尔值的封装器，其中定义了以下构造函数：
	
	Boolean(boolean boolValue)
	Boolean(String boolString)
	
	在第一个版本中,boolValue必须是true或false。在第二个版本中，如果boolString包含字符串"true"(大写或小写形式都可以)，那么新的Boolean对象将为true，否则将为false。

	为了从Boolean对象获取布尔值，可以使用BooleanValue()方法，如下所示：
	
	boolean booleanValue()
	
	该方法返回与调用对象等价的布尔值。
	
	
**12.2.3 数值类型封装器**

	到目前为止，最常用的类型封装器是那些表示数值的封装器，包括Byte、Short、Integer、Long、Float以及Double。所有这些数值类型封装器都继承自抽象类Number。Number声明了以不同数字格式从对象返回数值的方法，如下所示：

	byte byteValue()
	short shortValue()
	int intValue()
	long longValue()
	float floatValue()
	double doubleValue()
	
	所有数值类型封装器都定义了用于从给定数值或数值的字符串表示形式构造对象的构造函数。例如下面是为Integer定义的构造函数：
	
	Integer(int num)
	Integer(String str)
	
	如果str没有包含有效的数值，就会抛出NumberFormatException异常。
	
	所有类型封装器都重写了toString()方法，用来返回封装器所包含数值的人类可以阅读的形式，从而可以通过将封装器对象传递给println()方法来输出数值。例如，不必将之转换为基本类型。

```
	class Wrap
	{
		public static void main(String[] args)
		{
			Integer iOb = new Integer(100);
			
			int i = iOb.intValue();
			
			System.out.println(i+" "+iOb); //display 100  100 
		}
	}
```
		
	将数值封装到对象中的过程称为装箱。因此在这个程序中，下面这行代码将数值100装箱到一个Integer对象中：
	
	Integer iOb = new Integer(100);
	
	从类型封装器中抽取数值的过程称为拆箱。例如，这个程序使用下面这条语句拆箱iOb中的数值：
	
	int i = iOb.intValue();
	
	
	上面程序使用的装箱和拆箱数值的一般过程，在Java原始版本中就已经提供了。但是，自JDK 5 以后，Java通过自动装箱从根本上改进了这一过程，下面对此进行介绍。


**12.3 自动装箱/拆箱**

	自JDK 5 开始，Java增加了两个重要特性：自动装箱和自动拆箱。
	自动装箱：无论何时，只要需要基本类型的对象，就自动将基本类型自动封装(装箱)到与之等价的类型封装器中，而不需要显示的构造对象。
	自动拆箱：当需要时自动抽泣(拆箱)以封装对象的数值的过程。不需要调用intValue()或doubleValue()这类方法。
	
	自动装箱/拆箱特性极大的简化了一些算法的编码，移除了单调乏味的手动装箱和拆箱数值的操作。它们还有助于防止错误的发生。此外，它们对于泛型非常重要，因为泛型只能操作对象。最后，集合框架需要利用自动装箱的特性进行工作。

	其实，自动装箱/拆箱类似于数据类型的隐式类型转换。基本数据类型和基本数据对象之间互相进行隐式的类型转换。
	
	有了自动装箱特性，封装基本类型将不必在手动创建对象。只需要将数值赋值给类型封装器的引用即可，Java会自动创建对象：
	
	Integer iOb = 100;
	
	注意没有使用new显示的创建对象。Java会自动处理这个过程。
	
	为了拆箱对象，可以简单的将对象引用赋值给基本类型变量。例如：
	
	int i = iOb;
	
	自动装箱/拆箱实例：
	
```
	class AutoBox
	{
		public static void main(String[] args)
		{
			Integer iOb = 100;
			
			int i = iOb;
			
			System.out.println(i+" "+iOb); display 100  100 
		}
	}
```
	
**12.3.1 自动装箱与方法**

	除了赋值这种简单的情况外，无论何时，如果必须将基本类型转换为对象，就会发生自动装箱；无论何时，如果对象必须转换为基本类型，就会自动拆箱。因此，当向方法传递参数或从方法返回数值时，都可能发生自动拆箱/装箱。例如，下面的程序：

```
	class AutoBox2
	{
		static int m(Integer v)
		{
			return v;
		}
		
		public static void main(String[] args)
		{
			Integer iOb = m(100);
			
			System.out.println(iOb);
		}
	}
```
	
	输出: 100 
	
	在表达式计算中也会出现自动装箱/拆箱的情况。
	
	通常，应当限制类型封装器的使用，只有当需要基本类型的对象表示形式时才应当使用。提供的自动装箱/拆箱不是用来作为消除基本类型的"后门"。
	
		
**12.4 注解(元数据)**

	从JDK 5开始，Java支持在源文件中嵌入补充信息，这类信息称为注解(annotation)。注解不会改变程序的动作，因此也就不会改变程序的语义。但是在开发和部署期间，各种工具可以使用这类信息。

	由于设计注解的目的主要是用于其他的开发和部署工具，故在此先不做介绍，当用到时在写着部分。
	//...................................................

出自：《Java 8编程参考官方教程(第9版)》