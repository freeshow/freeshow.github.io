---
title: Java之异常处理
date: 2016-07-23 23:13:39
tags: Java
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

异常时运行时在代码序列中引起的非正常情况。换句话说，异常是运行时错误。

**10.1 异常处理的基础知识**

	Java异常是用来描述在一段代码中发生的异常情况(也就是错误)的对象。当出现引起异常的情况时，
就会创建用来表示异常的对象，并在引起错误的方法中抛出异常对象。方法可以选择自己处理异常，
也可以继续传递异常。无论采用哪种形式，在某一点都会捕获并处理异常。有Java抛出的异常与那些违反
Java语言规则或Java执行环境约束的基础性错误有关。手动生成的异常通常用于向方法的调用者报告某些错误条件。

	Java异常处理通过5个关键字进行管理；try、catch、throw、throws以及final.
	在try代码块中封装可能发生异常的程序语句，对这些语句进行监视。如果在try代码块中发生异常，就会将异常抛出。
代码可以(使用catch)捕获异常，并以某些理性方式对其进行处理。系统生成的异常由Java运行时系统自动抛出。
为了手动抛出异常，需要使用throw关键字。从方法抛出的异常都必须通过一条throws子句进行指定。在try代码块结束之后
必须执行的所有代码都需要放入finally代码块中。

	下面是异常处理代码块的一般形式：
```
	try
	{
	
	}
	catch(ExceptionType1 e)
	{
	
	}
	catch(ExceptionType2 e)
	{
	
	}
	finally
	{
	
	}
```
	其中，ExceptionType是已经发生异常的类型。
	
	注意：从JDK 7开始，try语句增加了一种支持自动资源管理的新形式，这种形式的try被称为带资源的try,
	将在文件管理部分进行介绍，因为文件是最常用的资源。
	
**10.2 异常类型**

	所有异常类型都是内置类Throwable的子类。因此，Throwable位于异常层次中的顶部。紧跟Throwable之下的是两个子类，它们将异常分为两个不同的分支。
一个分支是Exception类，这个类既可以用于用户程序应当捕获的异常情况，也可用于创建自定义异常类型的子类。Exception有一个重要的子类，
名为RuntimeException。对于你编写的程序而言，这种类型的异常是自动定义的，包括除零和无效数组索引这类情况。

	另一个分支是Error类，该类定义了在常规环境下不希望由程序捕获的异常。Error类型的异常由Java运行时系统调用，以指示运行时环境本身出现了某些错误。
堆栈溢出是这类错误的一个例子。本章不会处理Error类型的异常，因为它们通常是为了响应灾难性的失败而创建的，你的程序通常不能处理这类异常。


**10.3 未捕获的异常**

	学习如何在程序中处理异常之前，先看一看如何不处理异常会发生什么情况。
	下面的程序包含一个故意引起除零错误的表达式：
```
	class Exc0
	{
		public static void main(String[] args)
		{
			int d = 0;
			int a = 42 / d;
		}
	}
```
	当Java运行时检测到试图除以零时，它会构造一个新的异常对象，然后抛出这个异常。这会导致Exc0终止执行，因为一旦抛出异常，
就必须有一个异常处理程序捕获该异常，并立即进行处理。

	在这个例子中，没有提供任何自己的处理异常的程序，所以该异常会由Java运行时系统提供的默认处理程序捕获。
没有被程序捕获的所有异常，最终都将由默认处理程序进行处理。默认处理程序会显示一个描述异常的字符串，输出
异常发生点的堆栈踪迹并终止程序。
	下面是执行这个程序发生的异常：
	java.lang.ArithmeticException: / by zero
		at Exc0.main(Exc0.java:5)
	注意类名(Exc0)、方法名(main)、文件名(Exc0.java)以及行号(5)，是如何被包含到这个简单的堆栈踪迹中。
	此外，注意抛出的异常类型是Exception的子类ArithmeticException,该类更具体的描述了发生的错误类型。
正如后面所讨论的，Java提供了一些能与可能产生的各种运行是错误相匹配的内置异常类型。

	堆栈踪迹总是会显示导致错误的方法调用序列。例如：
```
	class Exc1
	{
		static void subroutine()
		{
			int d = 0;
			int a = 10 / d;
		}
		
		public static void main(String[] args)
		{
			Exc1.subroutine();
		}
	}
```
	默认异常处理程序产生的堆栈踪迹显示了整个调用堆栈过程，如下所示：
	java.lang.ArithmeticException: / by zero
		at Exc1.subroutine(Exc1.java:6)
		at Exc1.main(Exc1.java:11)
		
	可以看出，堆栈底部是main()方法中的第11行，该行调用subroutine()方法，该方法在第6行引起了异常。
调用堆栈对于调试非常有用，因为可以精确定位导致错误发生的步骤序列。

**10.4 使用try和catch**

	尽管对于调试而言，Java运行时系统提供的默认异常处理程序很有用，但是通常希望自己处理异常。
	自己处理异常有两个优点：
	第一，允许修复错误；
	第二，阻止程序自动终止；
	
	如果程序停止运行并且无论何时发生错误都输出堆栈踪迹，会让大多数用户感到困惑!幸运的是，阻止这种情况发生很容易。
	
	为了防止并处理运行时错误，可以简单的在try代码块中封装希望监视的代码。紧随try代码块之后，提供一条catch子句，
指定希望捕获的异常类型。例如：

```
	class Exc2
	{
		public static void main(String[] args)
		{
			int d,a;
			
			try
			{
				d = 0;
				a = 42 / d;
				System.out.println("This will not be print.");
			}
			catch(ArithmeticException e)
			{
				System.out.println("Devision by zero.");
			}
			
			System.out.println("After catch statement.");
		}
	}
```
	输出如下：
	Devision by zero.
	After catch statement.
	
	注意，对try代码块中println()方法的调用永远都不会执行。一旦抛出异常，程序控制就会从try代码块中转移出来，进入catch代码块中。
话句话说，不是”调用”catch，所以执行控制永远不会从catch代码块“返回“到try代码块。因此，不会显示"This will not be print."这一行。
执行完catch语句后，程序控制就会继续进入到程序中整个try/catch代码块的下一行。

	try及catch构成了一个单元。catch子句的作用域被限制在由之前try语句指定的那些语句内。catch语句不能把捕获由另一条try语句抛出
的异常(这种情况的一个例外是嵌套的try语句，稍后对此进行介绍)。由try保护的语句必须使用花括号括起来，也就是说，它们必须位于一个代码块中。

	大部分设计良好的catch子句，都应该能够分辨出异常情况，然后继续执行，就好像错误根本没有发生一样。例如下面的程序，for循环的
每次迭代都会获取两个随机数。将两个整数彼此相除，之后用12345除以结果。将最终结果保存到变量a中。只要任何一个除法操作引起除零错误，
就会捕获该错误，并将a设置为0，然后程序继续执行。例如：
	
	**import java.util.Random
	
	class HandleError
	{
		public static void main(String[] args)
		{
			int a = 0,b = 0,c = 0;
			Random r = new Random();
			
			for(int i = 0; i<3200; i++)
			{
				try
				{
					b = r.nextInt();
					c = r.nextInt();
					a = 12345 / (b/c);
				}
				catch(ArithmeticException e)
				{
					System.out.println("Devision by zero.");
					a = 0;
				}
				
				System.out.println("a: "+a);
			}
		}
	}**
	
	
	显示异常的描述信息
	
	Throwable重写了(由Object定义的)toString()方法，从而可以返回一个包含异常描述的字符串。可以使用println()语句显示这个描述，
为此，只需要简单的将异常作为参数传递给println()方法。
	将上面的程序中catch块改成如下形式：
```
	catch(ArithmeticException e)
	{
		System.out.println("Exception: "+e);
		a = 0;
	}
```
	输出为：
	Exception: java.lang.ArithmeticException: / by zero
	
	这样做虽然在这个例子中不是很有价值，但是显示异常描述的能力对于其他情况却是很有价值——特别是当对异常进行检验或进行调试时。
	
**10.5 多条catch子句**

	在有些情况下，单块代码块可能会引发多个异常。为了处理这种情况，可以指定两条或多条catch子句，每条catch子句捕获不同类型的异常。
当抛出异常时，按顺序检查每条catch语句，并执行类型和异常能够匹配的第一条catch子句。执行了一条catch语句之后，会忽略其他catch语句，
并继续执行try/catch代码块后面的代码。下面的例子捕获两种不同的异常类型：

```
	class MultipleCatches
	{
		public static void main(String[] args)
		{
			try 
			{
				int a = args.length;
				System.out.println("a = "+a);
				int b = 42 / a;
				int[] c = {1};
				c[42] = 99;
			}
			catch(ArithmeticException e)
			{
				System.out.println("Devision by 0:"+e);
			}
			catch(ArrayIndexOutOfBoundsException e)
			{
				System.out.println("Array index oob: "+e);
			}
			
			System.out.println("After try/catch blocks.");
		}
	}
```
	如果在启动程序时没有提供命令行参数，那么上述程序会引起除零异常，因为a会等于0.如果提供命令行参数，那么会将a设置为大于0的数值，
程序会继续进行除法运算。但是会引起ArrayIndexOutOfBoundsException异常，因为int型数组c的长度为1,但是程序试图为c[42]赋值。

	当使用多条catch语句时，要重点记住异常子类必须位于所有超类之前，因为使用了某个超类的catch语句会捕获这个超类及其所有子类的异常。
因此，如果子类位于超类之后的话，永远也不会到达子类。此外在Java中，不可到达的代码被认为是错误。

**10.6 嵌套的try语句**

	可以嵌套try语句。也就是说，一条try语句可以位于另一条try语句中。每次遇到try语句时，异常的上下文就会被推入到堆栈中。如果内层的
try语句没有为特定的异常提供catch处理程序，堆栈就会弹出该try语句，检查下一条try语句的catch处理程序，查看是否匹配异常。这个过程会
一直持续下去，直到找到一条匹配的catch语句，或直到检查完所有嵌套的try语句。如果没有找到匹配的catch语句，Java运行时系统会处理异常。
例如：

```
	class NestTry
	{
		public static void main(String[] args)
		{
			try
			{
				int a = args.length;
				int b = 42/a;
				
				System.out.println("a = "+a);
				
				try
				{
					if(a == 1) a = a/(a-a);
					
					if(a == 2)
					{
						int[] c = {1};
						c[42] = 99;
					}
				}
				catch(ArrayIndexOutOfBoundsException e)
				{
					System.out.println("Array index out-of-bounds: "+e);
				}
			}
			catch(ArithmeticException e)
			{
				System.out.println("Divide by 0: "+e);
			}
		}
	}
```
	可以看出，该程序在一个try代码块中嵌套另一个try代码块。该程序的工作过程如下：
	如果执行程序时没有提供命令行参数，那么外层的try代码块会产生除零异常。如果执行程序时提供一个命令行参数，那么会从嵌套的try代码块中
产生除零异常。因为内层的try代码块没有捕获该异常，所以会将其传递到外层的try代码块，在此对除零异常进行处理。如果执行程序时提供了两个命
令行参数，那么会从内层的try代码块中产生数组越界异常。

	当涉及方法调用时，可能会出现不那么明显的try语句嵌套。例如，可能在一个try代码块中包含对某个方法的调用，而在该方法内部又有另外一条
try语句。对于这种情况，方法内部的try语句仍然被嵌套在调用该方法的外层try代码块中。例如：

```
	class MethNestTry
	{
		static void nesttry(int a)
		{
			try
			{
				if(a == 1) a = a/(a-a);
				
				if(a ==2)
				{
					int[] c = {1};
					c[42] = 99;
				}
			}
			catch(ArrayIndexOutOfBoundsException e)
			{
				System.out.println("Array index out-of-bounds: "+e);
			}
		}
		
		public static void main(String[] args)
		{
			try
			{
				int a = args.length;
				int b = 42/a;
				System.out.println("a = "+a);
				
				nesttry(a);
			}
			catch(ArithmeticException e)
			{
				System.out.println("Divide by 0: "+e);
			}
		}
	}
```
	该程序的输出和上面的程序完全相同。
	
**10.7 throw** 

	到目前为止，捕获的都是有Java运行时系统抛出的异常。但是，你的程序也可以使用throw语句显示的抛出异常。
	throw的一般形式如下：
	throw ThrowableInstance;
	其中，ThrowableInstance必须为Throwable或其子类类型的对象。可以通过两种方式获得Throwable对象：在catch子句中使用参数
或者使用new运算符创建Throwable对象。

	throw语句之后的执行流会立即停止，所有后续语句都不会执行。检查最近的try代码块，查看是否存在和异常类型相匹配的catch语句。
如果没有，就检查下一条try语句，这个过程一直重复下去。如果没有找到匹配的catch语句，那么默认的异常处理程序会终止程序并输出堆栈踪迹。

	下面是一个创建并抛出异常的实例程序。捕获异常的处理程序将异常再次抛出到外层的异常处理程序中：
```
	class ThrowDemo
	{
		static void demoproc()
		{
			try
			{
				throw new NullPointerException("demo");
			}
			catch(NullPointerException e)
			{
				System.out.println("Caught inside demoproc.");
				//将异常从新抛出
				throw e;
			}
		}
		
		public static void main(String[] args)
		{
			try
			{
				demoproc();
			}
			catch(NullPointerException e)
			{
				System.out.println("Recaught: "+e);
			}
		}
	}
```
	输出：
	Caught inside demoproc.
	Recaught: java.lang.NullPointerException: demo
	
	许多内置的Java运行时异常至少有两个构造函数：一个不带参数，另一个带有一个字符串参数。如果使用第二种形式，那么参数指定了用来描述
异常的字符串。当将对象用做print()或println()方法的参数时，会显示该字符串。还可以通过调用getMessage()方法获取这个字符串，该方法由Throwable定义的。

**10.8 throws** 

	如果方法可能引发自身不进行处理的异常，就必须指明这种行为，以便方法的调用者能够保卫它们自己以防备上述异常。可以通过在方法声明中提供
throws子句列出了方法可能抛出的异常类型。除了Error和RuntimeException及其子类类型的异常外，对于所有其他类型的异常这都是必须的。方法可能抛出
的所有其他异常都必须在throws子句中进行声明。如果没有这么做，就会产生编译时错误。
	因为RuntimeException及其子类属于为经检查的异常，这类异常编译器不检查方法是否处理或抛出这些异常。因此，当方法中引发这类异常并不进行处理时，
并不需要使用throws子句列出方法抛出的这类异常。
	
	Java什么情况下必须用throws抛出异常？
	答：在程序中抛出了非RuntimeException异常却没有对其处理(用try catch块处理)的情况下，必须在方法头throws该异常。
	
	"异常机制"中还有一种特殊情况――RuntimeException"异常类"，这个"异常类"和它的所有子类都有一个特性，就是"异常"对象一产生就被Java虚拟机直接
处理掉，即在方法中出现throw子句的地方便被虚拟机捕捉了。因此凡是抛出这种"运行时异常"的方法在被引用时，不需要有try…catch语句来处理"异常"。
而，当方法包含经检查的异常时，该方法被引用时，需要有try/catch语句来处理异常。

	下面是包含throws子句的方法声明的一般形式：
```
	type method-name(parameter-list) throws exception-list
	{
		//body of method 
	}
```
	其中，exception-list是方法可能抛出的异常列表，它们由逗号隔开。
	
	下面是一个错误程序，该程序试图抛出无法匹配的异常。因为程序没有指定throws子句来声明这一事实，所以程序无法编译。
```
	class ThrowsDemo
	{
		static void throwOne()
		{
			System.out.println("Inside throwOne.");
			throw new IllegalAccessException("demo");
		}
		
		public static void main(String[] args)
		{
			throwOne();
		}
	}
```
	
	为了使这个例子能够编译，有两种方法，第一种是将throwOne()中的代码放入try/catch代码块中，这种方法上面已经讲过，故不做叙述。
第二种，使用throws语句，需要进行两处修改。首先，需要声明throwOne()方法抛出IllegalAccessException异常。其次，main()方法必须
定义捕获该异常的try/catch块。
	改正后的例子如下所示：
```
	class ThrowsDemo
	{
		static void throwOne throws IllegalAccessException
		{
			System.out.println("Inside throwOne.");
			throw new IllegalAccessException("demo");
		}
		
		public static void main(String[] args)
		{
			try
			{
				throwOne();
			}
			catch(IllegalAccessException e)
			{
				System.out.println("Caught "+e);
			}
		}
	}
```
	输出： 
	Inside throwOne.
	Caught java.lang.IllegalAccessException: demo
	
**10.9 finally**

	当抛出异常后，方法中的执行流会采用一种非常突然的、非线性的路径，这将改变方法的正常执行流。根据方法的编码方式，异常甚至可能使方法比
预计时间更早的返回。对于某些方法这可能是一个问题。例如，如果方法在开始时打开一个文件，并在结束时关闭这个文件，那么你可能不希望关闭文件
的代码绕过异常处理机制。关键字finally就是为解决这种可能情况而设计的。

	使用finally可以创建一个代码块，该代码块会在执行try/catch代码块之后，并在执行try/catch代码块后面的代码之前执行。不管是否有异常抛出，
都会执行finally代码块。如果抛出了异常，那么即使没有catch语句能匹配异常，finally代码块也仍将执行。只要try/catch代码块内部返回到调用者，
不管是通过未捕获的异常还是使用显示的返回语句，都会在方法返回之前执行finally语句。对于关闭文件句柄以及释放在方法开始时进行分配，并在方法
返回之前进行处理的所有其他资源来说，finally子句都是很有用的。finally子句是可选的。但是，每条try子句至少需要一条catch子句或finally子句。

	下面的实例程序显示了以各种方式退出的三个方法，所有这些方法都执行它们各自的finally子句。
```
	class FinallyDemo
	{
		//Throw an exception out of the method
		static void procA()
		{
			System.out.println("Inside procA.");
			throw new RuntimeException("demo");
		}
		finally
		{
			System.out.println("procA's finally.");
		}
		
		//Return from within a try block
		static void procB()
		{
			try
			{
				System.out.println("Inside procB.");
				return;
			}
			finally
			{
				System.out.println("procB's finally.");
			}
		}
		
		//Extcute a try block normally
		static void procC()
		{
			try
			{
				System.out.println("Inside procC.");
			}
			finally
			{
				System.out.println("procC's finally.");
			}
		}
		
		public static void main(String[] args)
		{
			try
			{
				procA();
			}
			catch(Exception e)
			{
				System.out.println("Exception caught.");
			}
			
			procB();
			procC();
		}
	}
```
	输出： 
	Inside procA.
	procA's finally.
	Exception caught.
	Inside procB.
	procB's finally.
	Inside procC.
	procC's finally.
	
	在这个例子中，方法procA()通过抛出异常过早的跳出try/catch代码块，在退出之后执行finally子句。方法procB()通过return语句退出try代码块，
在procB()方法返回前执行finally子句。在方法procC()中，try语句正常执行，没有错误，但是仍然会执行finally代码块。

	请记住：如果finally代码块和某个try代码块相关联，那么finally代码块会在这个try代码块结束后执行。
	
**10.10 Java的内置异常**

	java里面异常分为两大类:checked exception(检查异常)和unchecked exception(未检 查异常)。
	对于未检查异常也叫RuntimeException(运行时异常)，即RuntimeException及其子类的异常。对于运行时异常，java编译器不要求你一定要把它捕获
或者一定要继续抛出，在所有方法的throws列表中不需要包含这些异常。但是对checked exception(检查异常)要求你必须要在方法里面或者捕获或者继续抛出.

	例如：
```
	public class ExceptionTypeTest
	{

		public void doSomething()throws ArithmeticException
		{
			System.out.println();
		}

		public static void main()
		{
			ExceptionTypeTest ett = new ExceptionTypeTest();
			ett.doSomething();
		}
	}
```
	问题1:上面的程序能否编译通过？并说明理由。 
	解答:能编译通过。
	分析:按照一般常理，定义doSomething方法是定义了ArithmeticException异常，在main方法里里面调用了该方法。那么应当继续抛出或者捕获一下。
但是ArithmeticException异常是继承RuntimeException运行时异常。java里面异常分为两大类:checked exception(检查异常)和unchecked exception(未检 查异常)，
对于未检查异常也叫RuntimeException(运行时异常),对于运行时异常，java编译器不要求你一定要把它捕获或者一定要继续抛出，但是对checked exception
(检查异常)要求你必须要在方法里面或者捕获或者继续抛出. ???? 

	问题2:上面的程序将ArithmeticException改为IOException能否编译通过？并说明理由。 
	解答:不能编译通过。
	分析:IOException extends Exception 是属于checked exception ，必须进行处理，或者必须捕获或者必须抛出 
	
	总结：java中异常分为两类:checked exception(检查异常)和unchecked exception(未检查异常),对于未检查异常也叫RuntimeException(运行时异常). ??????? 
	对未检查的异常(unchecked exception )的几种处理方式：
	1、捕获 ??????? 
	2、继续抛出 ??????? 
	3、不处理 ??????? 
	对检查的异常(checked exception，除了RuntimeException，其他的异常都是checked exception )的几种处理方式： ??????? 
	1、继续抛出，消极的方法，一直可以抛到java虚拟机来处理 ??????? 
	2、用try...catch捕获 ??????? 
	
	注意，对于检查的异常必须处理，或者必须捕获或者必须抛出
	

**10.11 创建自己的异常子类**

	尽管Java的内置异常处理了大部分常见错误，但是你可能希望创建自己的异常类型，以处理特定于应用程序的情况，这很容易完成：
只需要定义Exception的子类(当然也是Throwable的子类)即可。你的子类不许需要实际实现任何内容————只要它们存在于类型系统中，
就可以将它们用作异常。

	Exception类没有为自己定义任何方法。当然，它继承了Throwable提供的方法。因此，所有异常，包含你创建的异常，都可以获得Throwable定义的方法。
	Exception类定义了4个公有构造函数，其中两个支持链式异常(链式异常在后面描述)，另两个如下所示：
	Exception()
	Exception(String msg)
	第一种创建没有描述的异常，第二种形式可以为异常指定描述信息。
	
	虽然创建异常时指定描述通常是有用的，但是有时重写toString()方法会更好一些。原因是：Throwable定义的toString()方法版本
(Exception继承了该版本)首先显示异常的名称，之后跟一个冒号，然后显示你的一个描述。通过重写toString()方法，可以阻止显示异常
名称和冒号。这样可以使输出变得更加清晰，在有些情况下你可能希望如此。

	下面的例子声明了一个新的Exception的子类，然后在一个方法中使用这个子类标识错误条件。该子类重写了toString()方法，从而
可以仔细的修改显示的异常描述。

```
	class MyException extends Exception
	{
		private int detail;
		
		MyException(int a)
		{
			detail = a;
		}
		
		public String toString()
		{
			return "MyException["+detail+"]";
		}
	}
	
	class ExceptionDemo
	{
		static void compute(int a) throws MyException
		{
			System.out.println("Called compute("+a+")");
			
			if(a>10)
			{
				throw new MyException(a);
			}
			
			System.out.println("Normal exit.");
		}
		
		public static void main(String[] args)
		{
			try
			{
				compute(1);
				compute(20);
			}
			catch(MyException e)
			{
				System.out.println("Caught "+e);
			}
		}
	}
```
	输出：
	Called compute(1)
	Normal exit.
	Called compute(20)
	Caught MyException[20]
	
	注意：如果上面try代码块中compute(20)在compute(1)前面执行，则输出结果为：
	Called compute(20)
	Caught MyException[20]
	
**10.12 链式异常**

	从JDK 1.4开始，链式异常这一特性被包含进异常子系统。通过链式异常，可以为异常关联另一个异常。第二个异常描述第一个异常的原因
(描述当前异常的原因，在后面通常称为"引发异常"或"背后异常")。
	例如，假设某个方法由于试图除零而抛出ArithmeticException异常。但是，导致问题发生的实际原因是发生了一个I/O错误，该错误导致为
除数设置了不正确的值。尽管方法必须显示的抛出ArithmeticException异常，因为这是已经发生的错误，但是你可能还希望让调用者知道背后的
原因是I/O错误，使用链式异常可以处理这种情况以及所有其他存在多层异常的情况。

	为了使用链式异常，向Throwable类添加了两个构造函数和两个方法。这两个构造函数如下所示：
	Throwable(Throwable causeExc)
	Throwable(String msg,Throwable causeExc)
	在第一种形式中，causeExc是引发当前异常的异常，即causeExc是引发当前异常的背后原因。第二种形式允许在指定引发异常的同时指定异常描述。
这两个构造方法也被添加到了Error、Exception以及RuntimeException类中。

	添加到Throwable类中的链式异常方法是getCause()和initCause().
	Throwable getCause()
	Throwable initCause(Throwable causeExc)
	getCause()方法返回引发当前异常的异常。如果不存在背后异常，就返回null。initCause()方法将causeExc和调用的异常关联到一起，并返回对
异常的引用。通常，initCause()方法用于为不支持前面描述的两个附加构造函数的遗留异常设置"背后异常"。因此，可以在创建异常之后将异常和
背后异常关联到一起。但是背后异常只能设置一次。因此，对于每个异常对象只能调用initCause()方法一次。此外，如果通过构造函数设置了背后异常，
那么就不能再使用initCause()方法进行设置。

	例如：
```
	class ChainExcDemo
	{
		static void demoproc()
		{
			NullPointerException e = new NullPointerException("top layer");
			e.initCause(new ArithmeticException("cause"));
			throw e;
		}
		
		public stati void main(String[] args)
		{
			try
			{
				demoproc();
			}
			catch(NullPointerException e)
			{
				System.out.println("Caught: "+e);
				System.out.println("Original cause: "+e.getCause());
			}
		}
	}
```
	输出如下：
	Caught: java.lang.NullPointerException: top layer 
	Original cause: java.lang.ArithmeticException: cause
	
**10.13 3个近期添加的异常特性**

	从JDK 7 开始，异常系统添加了3个有趣并且有用的特性。
	第一个特性是：当资源(例如文件)不在需要时能够自动释放。该特征的基础是try语句的扩展形式，称为带资源的try语句。
	第二个特性是：多重捕获，即一个catch 子句参数中可以写多个异常。
	的三个特性是：最后重写抛出(final rethrow)或更精确的重写抛出(more precise rethrow).
	
	为了使用多重捕获，在catch子句中使用或运算符(|)分割每一个异常。每个多重捕获参数都被隐式的声明为final.因此不能为它们赋新值。
	例如：
```
	class MultiCatch
	{
		public static void main(String[] args)
		{
			int a = 10,b = 0;
			int[] vals = {1,2,3};
			
			try
			{
				int result = a / b;
				vals[42] = 19;
			}
			catch(ArithmeticException | ArrayIndexOutOfBoundsException e)
			{
				System.out.println("Exception caught: "+e)
			}
			
			System.out.println("After multi-catch.");
		}
	}
```
	
	
	"更精确的从新抛出"异常特性会对重新抛出的异常类型进行限制，只能重新抛出满足一下条件的经检查的异常：由关联的try代码块抛出，没有被
前面的catch子句处理过，并且是参数的子类型或者超类型。显然这个功能不经常需要。

出自：《Java 8编程参考官方教程(第9版)》