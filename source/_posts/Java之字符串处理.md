---
title: Java之字符串处理
date: 2016-07-23 23:05:18
tags: Java
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

	在Java中，字符串也是一连串的的字符。但是，不像其他语言作为字符数组实现字符串，Java将字符串实现为String类型的对象。
	
	当创建String对象时，创建的字符串不能修改。也就是说，一旦创建一个String对象，就不能改变这个字符串中包含的字符。乍一看，这好像是一个严重的限制。但是，情况并非如此。仍然可以执行各种字符串操作。区别是，当每次需要已存在字符串的修改版本时，会创建包含修改后内容的新String对象。原始字符串保持不变。使用这种方式的原因是：实现固定的、不能修改的字符串与实现能够修改的字符串相比效率更高。对于那些需要能够修改的字符串的情况，Java提供了两个选择：
	StringBuffer和StringBuilder。这两个类用来保存在创建之后可以进行修改的字符串。
	
	String、StringBuffer和StringBuilder类都是在java.lang中定义的。因此，所有程序自动都可以使用它们。所有这些类被声明为final，这意味着这些类不能有子类。这使得对于通用的字符串操作，可以采取特定的优化以提供性能。这3个类实现了
	CharSequence接口。
	
	最后一点：所谓String类型对象中的字符串是不可改变的，是指创建了String实例后不能修改String实例的内容。但是，在任何时候都可以修改String引用变量，使其指向其他String对象。
	
**16.1 String类的构造函数**

	String类支持几个构造函数：
	
```
	String()
	
	String(chars[])
	String(chars[],int startIndex,int numChars)
	
	String(String strObj)
	
	String(byte[] chars)
	String(byte[] chars,int startIndex,int numChars)
	
	String(StringBuffer strBufObj)
	String(StringBuilder strBuildObj)
```
	
	注意：
	无论何时，从数组创建String对象都会复制数组的内容。在创建字符串之后，如果改变数组内容，String对象不会发生改变。
	
**16.2 字符串的长度**

	字符串的长度是指字符串包含字符的数量。为了获取这个值，调用length()方法，如下所示：
	
	int length()
	
	下面的代码输出"3"，因为在字符串s中3个字符：
```
	char[] chars = {'a','b','c'};
	String s = new String(chars);
	System.out.println(s.length());
```
	
**16.3 特殊的字符串操作**

**(1)字符串转换和toString()方法** 
	当Java在执行连接操作期间，将数据转换成相应的字符串表示形式时，是通过调用String定义的字符串转换方法valueOf()的某个重载版本来完成的。valueOf()方法针对所有基本类型以及Object类型进行了重载。对于基本类型，valueOf()方法返回一个字符串，该字符串包含与调用值等价的人类可以阅读的形式。对于对象，valueOf()方法调用对象的toString()方法。在本章后面将详细分析valueOf()方法。在此，首先分析toString()方法，因为，该方法决定了所创建类对象的字符串表示形式。
	
	每个类都实现了toString()方法，因为该方法是由Object定义的。然而，toString()方法的默认实现很少能够满足需求。对于自己创建的大多中重要类，你会希望重写toString()方法，并提供自己的字符串表示形式。幸运的是，这很容易完成。
	toString()方法的一般形式如下：
	
	String toString()
	
	为创建的类重写toString()方法，可以将其完全集成到Java开发环境中。例如，可以把他们用于print()和println()语句，还可以用于字符串连接表达式中。下面的程序通过为Box类重写toString()方法，演示了这一点：
	
```
	class Box
	{
		double width;
		double height;
		double depth;
		
		Box(double w,double h,double d)
		{
			width = w;
			height = h;
			depth = d;
		}
		
		public String toString()
		{
			return "Dimensions are "+width+" by "+depth+" by "+height+".";
		}
	}
	
	class toStringDemo
	{
		public static void main(String[] args)
		{
			Box b = new Box(10,12,14);
			String s = "Box b: "+b;
			
			System.out.println(b);
			System.out.println(s);
		}
	}
```
	
	输出：
	Dimensions are 10.0 by 14.0 by 12.0
	Box b: Dimensions are 10.0 by 14.0 by 12.0
	
	可以看出，在连接表达式或println()调用中使用Box对象时，会自动调用Box的toString()方法。 

**(2)比较字符串**

	为了比较两个字符串是否相等，可以使用equals()方法，它的一般形式如下：
	
	boolean equals(Object str)
	
	其中，str是将要与调用String对象进行比较的String对象。如果字符串以相同的顺序包含相同的字符，该方法返回true，否则返回false。比较是大小写敏感的。
	
	为了执行忽略大小写区别的比较，可以调用equalsIgnoreCase()。
	
**(3)equals()与==**

	equals()方法与"=="运算符执行不同的操作，理解这一点很重要。equals()方法比较String对象中的字符。"=="运算符比较两个对象的引用，查看它们是否指向相同的实例。例如：
	
```
	class EqualsNotEqualTo
	{
		public static void main(String[] args)
		{
			String s1 = "Hello";
			String s2 = new String(s1);
			
			System.out.println(s1+" equals "+ s2+" -> "s1.equals(s2));
			System.out.println(s1+" == "+s2+" -> "+(s1==s2));
		}
	}
```
	
**16.7 修改字符串**

	因为String对象是不可改变的，所以当希望修改String对象时，必须将之复制到StringBuffer或StringBuilder对象中，或者使用String类提供的方法来构造字符串修改后的新副本。
	
**(1) substring()**

	使用substring()方法可以提取子串，它有两种形式：
	
```
	String substring(int startIndex)
	String substring(int startIndex,int endIndex)
```
	
**(2) concat()**

	可以使用concat()方法连接两个字符串，如下所示：
	
	String concat(String str)
	
	该方法创建一个新对象，这个新对象包含调用字符串并将str的内容追加到结尾。concat()与"+"执行功能相同。

**(3) replace()**

```
	String replace(char original,char replacement)
	String replace(CharSequence original,CharSequence replacement)
```
	
**(4) trim()**

	String trim()
	
	该方法返回调用字符串的副本，并移除开头和结尾的所有空白字符。
	
**16.8 使用valueOf()转换数据**

	valueOf()方法将数据从内部格式转换成人类可以阅读的方式。valueOf()是静态方法，String针对所有Java内置类型对该方法进行了重载，从而使你创建的所有类类型的对象都可以作为valueOf()方法的参数(请记住，Object是所有类的超类)。下面是valueOf()方法的几种形式：
	
```
	static String valueOf(double num)
	static String valueOf(long num)
	static String valueOf(Object ob)
	static String valueOf(char[] chars)
```
	
	前面讨论过，当需要其他某类型的数据的字符串表示形式时会调用valueOf()方法，例如在连接操作期间。可以使用任何数据类型直接调用直接调用valueOf()方法，从而得到可读的字符串表示形式。所有简单类型都被转换成他们通用的字符串表示形式。传递给valueOf()方法的所有对象都将返回调用对象的toString()方法的结果。实际上，可以直接调用toString()方法得到相同的结果。
	
**16.9 改变字符串中字符的大小写**

	String toLowerCase() //将字符串的所有字符从大写改成小写
	String toUpperCase() //将字符串的所有字符从小写改成大写
	
**16.10 连接字符串**

	JDK 8为String类添加了一个新方法join()，用于连接两个或更多个字符串，并使用分隔符分隔各个字符串，如空格或逗
	号。join()方法有两种形式，第一种形式如下所示：
	
	static String join(CharSequence delim,CharSequence ...strs)
	
	其中，delim指定了分隔符，用于分隔strs指定的字符序列。因为String类实现了一个CharSequence接口，所以strs可以是一个字符串列表。
	
	
**16.12 StringBuffer类**

	StringBuffer支持可修改的字符串。String表示长度固定、不可修改的字符序列。与之对应，StringBuffer表示可增长、可写入的字符序列。StringBuffer允许在中间插入字符和子串，或者在末尾追加字符和子串。StringBuffer能够自动增长，从而为这类添加操作准备空间，并且通常预先分配比实际需要更多的字符空间，以允许空间增长。
	
**16.12.1 StringBuffer类的构造函数**

	StringBuffer类定义了以下4个构造函数：
	
```
	StringBuffer()
	StringBuffer(int size)
	StringBuffer(String str)
	StringBuffer(CharSequence chars)
```
	
	默认构造函数(没有参数的那个)预留16个字符的空间，不要在分配。
	第2个版本接受一个显示设置缓冲区大小的整型参数。
	第3个版本接受一个设置StringBuffer对象初始化内容的String参数，并额外预存16个字符的空间，不需要再分配。
	第4个版本构造函数创建包含字符序列的对象，并额外预留16个字符的空间，包含的字符序列是由chars指定的。
	如果没有要求特定的缓冲区长度，StringBuffer会为16个附加字符分配空间，因为在分配空间是很耗时的操作。此外，频繁的分配空间会产生内存碎片。通过为一部分二外字符分配空间，StringBuffer减少了在此分配空间的次数。
	
**16.12.3 ensureCapacity()**

	在创建了StringBuffer对象后，如果希望为特定数量的字符预先分配空间，可以使用ensureCapacity()方法设置缓冲区的大小。如果事先知道将要向StringBuffer对象追加大量的小字符串，这个方法时有用的。
	ensureCapacity()方法的一般形式为：
	
	void ensureCapacity(int minCapacity)
	
	其中，minCapacity指定了缓冲区的最下尺寸(出于效率考虑，可能会分配比minCapacity更大的缓冲区)
	
	备注：缓冲区即上面创建StringBuffer对象时预留的16个字符空间，因为默认缓冲器的大小为16。
	
**16.12.2 length()和capacity()**

	通过length()方法可以获得StringBuffer对象的当前长度，而通过capacity()方法可以获得已分配的容量。
	这两个方法的一般形式如下：
	
	int length()
	int capacity()
	
	下面是一个例子：
	
```
	class StringBufferDemo
	{
		public static void main(String[] args)
		{
			StringBuffer sb = new StringBuffer("Hello");
			
			System.out.println("buffer = "+sb);
			System.out.println("length = "+sb.length());
			System.out.println("capacity = "+sb.capacity());
		}
	}
```
	
	输出：
	buffer = Hello
	length = 5
	capacity = 21
	
	因为sb在创建时是使用字符串"Hello"初始化的，所以它的长度是5。因为自动添加了16个附加字符的空间，所以它的容量是21.当sb的长度超过这个capacity容量时，capacity会自动扩展、增大。
	
**16.12.4 setLength()**

	可以使用setLength()方法设置StringBuffer对象中字符串的长度，一般形式为：
	
	void setLength(int len)
	
	其中，len指定字符串的长度，值必须非负。
	
	当增加字符串的大小时，会向末尾添加null字符。如果调用setLength()方法时，使用的值小于length()方法返回的当前值，那么超出新长度的字符将会丢失。
	
**16.12.5 charAt()与setCharAt()**

	通过charAt()方法可以从StringBuffer获取单个字符的值，使用setCharAt()方法可以设置StringBuffer对象中某个字符的值。这两个方法的一般形式如下：
	
```
	char charAt(int where)
	void setCharAt(int where,char ch)
```
	
**16.12.6 getChars()**

	可以使用getChars()方法将StringBuffer对象的子串复制到数组中，一般形式为：
	
	void getChars(int sourceStart,int sourceEnd,char target[],int targetStart)
	
**16.12.7 append()**

	append()方法将各种其他类型数据的字符串表示形式连接到调用StringBuffer对象的末尾。给方法有一些重载版本，下面是其中的几个：
	
```
	StringBuffer append(String str)
	StringBuffer append(int num)
	StringBuffer append(Object obj)
```
	
	通常调用String.valueOf()来获取每个参数的字符串的表示形式，结果将被添加到当前StringBuffer对象的末尾。
	
**16.12.8 insert()**

	insert()方法将一个字符串插入到另一个字符串中。
	
	下面是其中的几种重载形式：
	
```
	StringBuffer insert(int index,String str)
	StringBuffer insert(int index,char ch)
	StringBuffer insert(int index,Object obj)
```
	
	
**16.12.9reverse()**
	
	可以使用reverse()方法颠倒StringBuffer对象中的字符，如下所示：
	
	StringBuffer reverse()
	
	该方法返回调用对象的反转形式。
	
**16.12.10 delete()与deleteCharAt()**

	使用delete()和deleteCharAt()方法可以删除StringBuffer对象中的字符。这些方法如下所示：
	
	StringBuffer delete(int startIndex,int endIndex)
	StringBuffer deleteCharAt(int loc)
	
**16.12.11 replace()**

	通过调用replace()方法可以使用一个字符集替换StringBuffer对象中的另个一字符集。该方法的签名如下所示：
	
	StringBuffer replace(int startIndex,int endIndex,String str)
	
**16.12.12 substring()**

	通过调用substring()方法可以获得StringBuffer对象的一部分。该方法有以下两种重载形式：
	
```
	String substringI(int startIndex)
	String substring(int startIndex,int endIndex)
```
	
**16.3 StringBuilder类**

	StringBui类是由JDK 5引入的，以增强Java的字符串处理能力。StringBuilder与StringBuffer类似，只有一个重要的区
	别：StringBuilder不是同步的，这意味着它不是线程安全的。StringBuilder的优势在于能得到更高的性能。但是，如果可以修改的字符串将被多个线程修改，并且没有使用其他同步措施的话，就必须使用StringBuffer，而不能使用StringBuilder。

出自：《Java 8编程参考官方教程(第9版)》
