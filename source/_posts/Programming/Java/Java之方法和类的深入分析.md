---
title: Java之方法和类的深入分析
date: 2016-07-23 23:16:19
tags: Java
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

**7.1 重载方法**

	在Java中，可用在同一个类中定义两个或多个共享相同名称的方法，这要他们的参数声明不同即可。
	当出现这种情况时，这些方法被称为重载的(overloaded)，并且这一过程被称为方法重载(method overloading).
	方法重载是Java支持多态性的方式之一。
	
	当调用重载方法时，Java使用参数的类型或数量决定实际调用哪个版本。因此，重载方法在参数类型或数量方面必须有所区别。
	虽然重载方法可以返回不同的类型，但是单靠返回类型不足以区分方法的多个版本。当Java遇到对重载方法的调用时，简单的执行
	方法形参与调用中所使用的实参相匹配的版本。
	
	当调用重载方法时，Java在用来调用方法的实参和形参之间查找匹配。然而，这个匹配并不需要总是精确的，当找不到精确的匹配对象时候、，
	Java的自动类型转换在重载版本的判断中可以发挥作用。
	例如，如果没有定义test(int)而定义了test(double),则当执行test(8)时，则会执行test(double).
	即，在没有找到test(int)版本时，Java会将整型int i，自动提升为double类型，并调用test(double).
	当然，如果定义了test(int)版本，就会调用该版本。只有当没有找到精确的匹配时，Java才会使用自动类型转换。
	
**7.3 参数传递的深入分析**

	对于计算机语言来说，向子例程传递参数的方式通常有两种。
	第一种方式是值调用(call-by-value),这种方式将实参的值复制到子例程的形参中。所以，对子例程参数进行的修改不会影响实参。
	第二种方式是引用调用(call-by-reference)，在这种情况下，将对实参的引用(而不是实参的值)传递给形参。在子例程中，
	该引用用于访问在调用中标识的实参。这意味着对子例程参数进行的修改会影响用于调用子例程的实参。
	
	尽管Java使用值调用传递所有的实参，但是根据所传递的是基本类型还是引用类型，精确效果是不同的。
	
	(1)当为方法传递基本数据类型，使用值传递，因此会得到实参的副本，并且对接收实参的形参进行操作，对方法外部没有影响。
	例如：
```
	class Test
	{
		void meth(int i, int j)
		{
			i *= 2;
			j /= 2;
		}
	}
	
	class CallByValue
	{
		public static void main(String[] args)
		{
			Test ob = new Test();
			int a = 15,b = 20;
			System.out.println("a and b before call: "+a+" "+b);
			ob.meth(a,b);
			System.out.println("a and b after call "+a+" "+b);
		}
	}
```
	输出：
	a and b before call: 15 20
	a and b after call: 15 20
	
	
	(2)当方法传递对象时，情况就有所不同了，因为对象时通过引用对用传递的。请牢记，当创建类变量时，只是创建指向对象的引用。
	因此，当将引用传递给方法时，接收引用的形参所引用的对象与实参引用的是同一个对象。这意味着对象就好像是通过引用调用传递给方法的。
	在方法内修改对象会影响到作为实参的对象。
	例如：
```
	class Test
	{
		int a ,b;
		Test(int a,int b)
		{
			a = i;
			b = j;
		}
		void meth(Test o)
		{
			o.a *= 2;
			o.b /= 2;
		}
	}
	
	class PassObjRef
	{
		public static void main(String[] args)
		{
			Test ob = new Test(15,20);
			System.out.println("ob.a and ob.b before call: "+ob.a+" "+ob.b);
			ob.meth(ob);
			System.out.println("ob.a and ob.b after call: "+ob.a+" "+ob.b);
		}
	}
```
	
	输出：
	ob.a and ob.b before call: 15 20
	ob.a and ob.b after call: 30 10
	
	请记住：
	当对象引用传递给方法时，引用本身是使用值调用传递的。但是，由于传递的值引用同一个对象，
	因此，值的副本仍然引用相应实参指向的同一个对象。
	
**7.5 递归**

	递归是根据自身定义内容的过程。就Java而言，递归是一个允许方法调用自身的特性。调用的自身的方法被称为递归方法。
	
	许多例程的递归版本，它们的执行速度比与之等价的迭代版本要更慢一些，因为增加了额外的函数调用负担。
	递归方法的主要优点是，对于某些算法，使用递归可以创建比迭代版本更清晰并且更简单的版本。例如，某些与人工智能相关的算法类型，使用递归方案最容易实现。
	
	当编写递归方法时，在某个地方必须有一条if语句，用于强制方法返回而不再执行递归调用。如果没有这么做，一旦调用该方法，就永远不会返回。
	
**7.6 访问控制**

	Java的访问修饰符包括public、private以及protected.Java还定义了默认访问级别。只有涉及到继承时才会应用protected.
	
	如果类的某个成员使用public进行修饰，那么该成员可以被任何代码访问。
	如果累的某个成员被标识为private,那么该成员只能被所属类的其他成员访问。
	
	这就是为什么main()方法之前总是带有public修饰符了。因为main()方法要有程序之外的代码访问，也就是Java运行时访问。
	如果没有使用访问修饰符，那么类成员在它自己的包中默认是公有的，但是在包外不能访问。
	
**7.7 理解static**

	有时候希望定义能够独立于类的所有对象进行使用的成员，即可以创建能够由类本身使用的成员，而不需要通过特定实例的引用。
	为了创建这种成员，需要在成员声明的前面使用关键字static.如果成员被声明为静态的，就可以在创建类的任何对象之前访问该成员，
	并不需要使用任何对象的引用。方法和变量都可以声明为静态的。main()方法是常见的静态成员的例子。main()方法被声明为静态的，
	是因为需要在创建所有对象之前调用该方法。
	
	被声明为静态的实例变量，在本质上是实例变量。当声明类的对象时，不会生成静态变量的副本。相反，类的所有实例共享相同的静态变量。
	
	声明静态的方法有几个限制：
	它们只能直接调用其他静态方法。
	它们只能直接访问静态数据。
	它们不能以任何方式引用this或super关键字。
	
	为了初始化静态变量，如果需要进行计算，可以声明静态代码块。静态代码块只执行一次，当第一次加载类时执行。
	例如：
```
	Class UseStatic
	{
		static int a =3;
		static int b;
		
		static void meth(int x)
		{
			System.out.println("x ="+x);
			System.out.println("a ="+a);
			System.out.println("b ="+b);
		}
		
		//因为b需要计算得到值，故需要放到静态代码块中，如果b不需要计算进行初始化，则可以完成向初始化a一样。
		static
		{
			System.out.println("Static block initialized.");
			b= a * 4;
		}
		
		public static void main(String[] args)
		{
			meth(42);
		}
	}
```
	输出：
	Static block initialized.
	x = 42
	a = 3
	b = 12
	
	只要加载UseStatic类，就会运行所有使用static声明的语句。首先，a被设置为3，然后执行静态代码块，输出一条消息，并将b初始化为a*4，即12.
	然后调用main()方法，该方法调用meth()方法，将42传递给参数x。
	
	在定义静态方法和静态变量的类的外部，不依赖于任何对象就可以使用这些静态成员。为此，只需要指定类的名称后面跟随点运算符。
	
**7.8 final介绍**
	
	可以将变量声明为final.这么做可以防止修改变量的内容，本质上是将变量变成为常量。这意味着fianl变量必须在声明时进行初始化。
	可以通过两种方式之一完成这个工作：
	(1)第一种方式：最为常见
	final int FINAL_VALUE = 1;
	final变量名全部大写，这是一种常见的编码约定。
	(2)可以在构造函数中为其赋值。
	
	除了可以将变量声明为final之外，方法参数和局部变量也可以声明为final.
	将参数声明为final，可以防止在方法中修改参数。将局部变量声明为final,可以防止多次为其赋值。
	
	关键字final也可以应用于方法，但是含义与应用于变量有本质上的区别。对final的这种补充用法，在后面介绍。
	
**7.10 嵌套类和内部类**

	可以在类的内部定义另一个类，这种类就是所谓的嵌套类。嵌套类的作用域被限制在包含它的类中。因此，如果类B是在类A中定义的，那么类B不能独立于类A而存在。
	嵌套类可以访问包含它的类的成员，包括私有成员。但是，包含类（包含嵌套类的类）不能访问嵌套类的成员。
	嵌套类直接在包含类中作为成员进行声明。也可以在代码块中声明嵌套类。
	
	嵌套类有两种类型：静态的和非静态的。静态的嵌套类是应用了static修饰符的嵌套类，，因为是静态的，嵌套类不能直接引用包含类的非静态成员。
	因为这个限制，所以很少使用静态的嵌套类。
	
	嵌套类最重要的类型是内部类。内部类是非静态的嵌套类，可以访问外部类的所有变量和方法，并且可以直接引用他们，
	引用方式与外部类的其他非静态成员使用的方式相同。
	例如：
```
	class Outer
	{
		int outer_x = 100;
		
		void test()
		{
			Inner inner = new Inner();
			inner.display();
		}
		
		class Inner
		{
			void display()
			{
				System.out.println("display: outer_x ="+outer_x);
			}
		}
		
		class InnerClassDemo
		{
			public static void main(String[] args)
			{
				Outer outer = new Outer();
				outer.test();
			}
		}
	}
```
	输出：
	display: outer_x = 100;
	
	被命名为Inner的内部类是在Outer类的作用域内定义的。所以，Inner类中所有代码，都可以直接访问变量outer_x。
	只能在Outer类的作用域内创建Inner类的实例，认识到这一点很重要。如果试图在Outer类之外的任何代码块中实例化Inner类，Java编译器就会生成错误。
	一般来说，必须通过封闭的作用域创建内部类的实例。
	
	内部类可以访问外部类的所有成员，但是反过来不可以。内部类的成员只能在内部类的作用域内才是已知的，并且外部类不能使用。
	例如：
```
	class Outer
	{
		int outer_x = 100;
		
		void test()
		{
			Inner inner = new Inner();
			inner.display();
		}
		
		class Inner
		{
			int y = 10;
			
			void display()
			{
				System.out.println("display: outer_x "+outer_x);
			}
		}
		
		void showy()
		{
			System.out.println(y) //error, y not known here!
		}
		
		class InnerClassDemo
		{
			public static void main(String[] args)
			{
				Outer outer = new Outer();
				outer.test();
			}
		}
	}
```
	在此，y被声明为Inner类的实例变量。因此，在Inner类的外部是不知道y,并且showy()方法也不能使用它。
	
	尽管我们一直主要关注的是，在外部类的作用域内作为成员声明的内部类，但是也可以在任何代码块的作用域内定义内部类。
	例如，可以在由方法定义的代码块中，甚至在for循环体内定义嵌套类。
	例如：
```
	class Outer
	{
		int outer_x = 100;
		
		void test()
		{
			for(int i = 0; i < 10; i++)
			{
				class Inner
				{
					void display()
					{
						System.out.println("display:outer_x ="+outer_x);
					}
				}
				
				Inner inner = new Inner();
				inner.display();
			}
		}
	}
	
	class InnerClassDemo
	{
		public static void main(String[] args)
		{
			Outer outer = new Outer();
			outer.test();
		}
	}
```
	
	输出：
	display: outer_x = 100
	display: outer_x = 100
	display: outer_x = 100
	display: outer_x = 100
	display: outer_x = 100
	display: outer_x = 100
	display: outer_x = 100
	display: outer_x = 100
	display: outer_x = 100
	display: outer_x = 100
	
	尽管嵌套类并不是对于所有的情况都适用，但是当处理事件时他们特别有用，适用内部类简化处理特定类型的事件所需要的代码，还将学习匿名内部类。
	
**7.11 String类**

	创建的每一个字符串实际上都是String类型的对象。即使是字符串常量，实际上也是String对象。
	例如：
	System.out.println("This is a String,too");
	字符串"This is a String,too"就是一个String对象。
	
	对于字符串需要理解的第二点是：String类型的对象是不可变的；一旦创建的一个String对象，其内容就不能再改变，
	尽管这看起来好像是一个严重的限制，但实际上不是，有两个原因：
	(1)如果需要改变一个字符串，总是可以创建包含修改后内容的新字符串。
	(2)Java定义了String类的对等类，分别称为StringBuffer和StringBuilder，他们允许修改字符串，所以在Java中仍然可以使用所有常规的字符串操作。
	
**7.12 使用命令行参数**

	当运行程序时，有可能希望为程序传递信息，这可以通过main()方法传递命令行参数来完成。命令行参数是执行程序时在命令行上紧跟程序名称之后的信息。
	在Java程序中访问命令行参数非常容易——他们作为字符串存储在String数组中，并传递给main()方法的args参数。
	第一个命令行参数存储在args[0]中，第二个存储在args[1]中，以此类推。
	例如：
```
	class CommandLine
	{
		public static void main(String[] args)
		{
			for(int i = 0; i < args.length; i++)
			{
				System.out.println("args["+i+"]:"+args[i]);
			}
		}
	}
```
	
	尝试向下面那样执行该程序：
	java CommandLine this is a test 100 -1
	输出：
	args[0]:this
	args[1]:is
	args[2]:a
	args[3]: test
	args[4]:100
	args[5]:-1
	
	请记住：所有命令行参数都是作为字符串传递的。必须手动将数值转换成他们的内部形式。
	
**7.13 varargs:可变长度参数**
(1)
	使用可变长度参数的方法称为可变参数方法(variable-arity method)或简称varargs方法。
	
	可变长参数通过三个句点(...)进行标识。例如：
	void varTest(int ...v);
	这种语法告诉编译器，可以使用零个或更多个参数调用varTest()方法。所以,v被隐式的声明为int[]类型数组。
	因此，在varTest()方法内部，可以使用常规的数组语法访问v.例如：
```
	class VarArgs
	{
		static void varTest(int ...v)
		{
			System.out.println("Numbers of args:"+v.length+"Contents:");
			
			for(int x:v)
			{
				System.out.println(x+" ");
			}
			
			System.out.println();
		}
		
		public static void main(String[] args)
		{
			varTest(10);	//1 arg
			varTest(1,2,3); //3 arg
			varTest();	//0 arg
		}
	}
```
	输出：
	Numbers of args: 1 Contents: 10
	Numbers of args: 3 Contents: 1 2 3 
	Numbers of args: 0 Contents:
	
	首先，在varTest()方法内部，v是作为数组进行操作的。这是因为V是一个数组。语法“..."只不过是告诉编译器将要使用可变长参数，
	并且这些参数将被存储在由v引用的数组中。
	其次，在main()方法中，使用不同数量的参数调用varTest()方法，包括根部不适用任何参数。参数被自动放进一个数组中，并传递给v。对于没有参数的情况，数组的长度为0.
	
	使用可变长度参数的方法也可以具有”常规“参数。但是，可变长度参数必须是方法最后声明的参数,并且只能有一个可变长参数。例如：
	int doIt(int a,int b,double c,int ... vals);
	
(2)重载varargs方法

	可以重载可变长参数。例如：
	static void varTest(int ... v);
	static void varTest(boolean ... v);
	static void varTest(int a, int b, double c,int ... v);
	
(3)varargs方法与模糊性

	当重载带有可变长度参数的方法时，可能会导致某些意料之外的错误。这下错误涉及模糊性，
	因为可能会为重载的varargs方法创建含糊不清的调用。例如：
	static void varTest(int ... v);
	static void varTest(boolean ... v);
	static void varTest(String ... v);
	当调用varTest()时，不知道调用上面3个函数中的哪一个。
	
	又例如：
	static void varTest(int ... v);
	static void varTestI(int n, int ... v);
	当调用varTest(1)时，不知道调用上面2个函数中的哪一个。
	
	因此类似上面显示的模糊性错误，有时需要放弃重载，并简单的使用不同的方法名。

出自：《Java 8编程参考官方教程(第9版)》