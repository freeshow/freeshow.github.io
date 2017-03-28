---
title: Java之I/O以及其它主题
date: 2016-07-23 23:07:45
tags: Java
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

**13.1 I/O的基础知识**
```
在前面的实例程序中没有大量使用I/O。实际上，除了print()和println()方法之外，基本上没有使用其他的I/O方法。原因很简单：大多数实际的Java应用程序不是基与文本的控制台程序。相反，要么是面向图形的程序——这类程序依赖于Java的GUI框架(如Swing、AWT或JavaFX)来进行用户交互，要么是Web应用程序。尽管作为教学演示，基于文本的控制台程序是很优秀的，但是对于使用Java编写实际的程序，它们并不重要。此外，Java对控制台I/O的支持也是有限的，即使是简单的示例程序，使用它们也有些蹩脚。基于文本的控制台I/O对于实际的Java编程确实用处不大。
```

**13.1.1 流**

	Java程序通过流执行I/O。流是一种抽象，要么产生信息，要么使用信息。流通过Java的I/O系统链接到物理设备。所有流的行为方式都是相同的，尽管与它们链接的物理设备是不同的。因此，可以为任意类型的设备应用相同的I/O类和方法。这意味着可以将许多不同类型的输入————磁盘文件、键盘或网络socket，抽象为输入流。与之对应，输出流可以引用控制台、磁盘文件或网络连接。流是输入/输出的一种清晰的方式，例如，代码中的所有部分都不需要理解键盘和网络之间的区别。流是Java在有java.io包定义的类层次中实现的。

	注意：除了在java.io包中定义的基于流的I/O外，Java还提供了基于缓冲和基于通道的I/O,它们是在java.nio及其子包中定义的。
	
**13.1.2 字节流和字符流**

	Java定义了两种类型的流：字节流和字符流。
	字节流为处理字节的输入和输出提供了方法。例如，当读取和写入二进制数据时，使用的就是字节流。
	字符流为处理字符的输入和输出提供了方便的方法。它们使用Unicode编码，所以可以被国际化。此外，在某些情况下，字符流比字节流高效。
	
	另外一点：在最底层，所有I/O仍然是面向字节的。基于字符的流只是为了处理字符提供了一种方便和高效的方法。
	
	1.字节流
	
	字节流是通过两个类层次定义的。在顶级是两个抽象类：InputStream和OutputStream。每个抽象类都有几个处理各种不同设备的具体子类，例如磁盘文件、网络连接甚至磁盘缓冲区。
	
	抽象类InputStream和OutputStream定义了一些其他流类实现的一些关键方法。其中最重要的两个方法是read()和write(),这两个方法分别分别读取和写入字节数据。每个方法都有抽象形式，派生的流类必须重写这两个方法。
	
	2.字符流类
	
	字符流是通过两个类层次定义的。在顶层是两个抽象类：Reader和Writer。这两个抽象类处理Unicode字符流。
	
	
**13.1.3 预定义流**

	所有Java程序都自动导入java.lang包。这个包定义了System类，该类封装了运行时环境的某些方法。System还包含了3个预定义的流变量：in、out以及err.这些变量在System类中被声明为public、static以及final。这意味着程序中其他任何部分都可以使用它们，而不需要引用特定的System对象。

	System.out引用标准的输出流，默认情况下是控制台。System.in引用标准的输入流，默认情况下是键盘。System.err引用标准的错误流，默认情况下也是控制台。但是，这些流可以被重定向到任何兼容的I/O设备。

	System.in是InputStream类型的对象；System.out和System.err是PrintStream类型的对象。这些都是字节流，尽管他们通常用于从控制台读取字符以及向控制台写入字符。可以看出，如果愿意的话，可以在基于字符的流中封装这些流。

**13.2 读取控制台输入**

	在Java1.0中，执行控制台输入的唯一方法是使用字节流。现在仍然可以使用字节流读取控制台输入。但是，对于商业应用程序，读取控制台输入最好方法是使用面向字符的流。使用面向字符的流可以使程序更容易国际化和维护。

	在Java中，控制台输入是通过从System.in读取完成的。为了获得与控制台关联的基于字符的流，可以在BufferedReader对象中封装System.in。BufferedReader支持缓冲的输入流。通常使用的构造函数如下：

	BufferedReader(Reader inputReader)
	
	其中inputReader是与即将创建的BufferedReader实例链接的字符流。而System.in是字节流，故需要先将字节流转换为字符流。Reader是抽象类，InputStreamReader是它的一个具体子类，该类将字节转换为字符。为了获得与System.in链接的InputStreamReader对象，使用下面的构造函数：

	InputStreamReader(InputStream inputStream)
	
	因为System.in引用InputStream类型的对象，所以可以用作inputStream参数。将这些内容结合起来，下面的代码创建了一个与键盘链接的BufferReader对象：
	
	BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
	
	执行这条语句之后，br就通过System.in与控制台链接的基于字符的流。
	
**13.2.1 读取字符**

	int read()throws IOException
	
	每次调用read()方法都会从输入流读取一个字符，并将之作为整形值返回。如果达到流的末尾，就返回-1；
	
	例如：
	
```
import java.io.*;
	
class BRRead
{
	public static void main(String[] args) throws IOException
	{
		char c;
		BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
			System.out.println("Enter characters, 'q' to quit.");
			
		do
		{
			c = (char)br.read();
			System.out.println(c);
		}while(c != 'q');
	}
}
```
	
	下面是一次示例运行：
	Enter characters, 'q' to quit.
	123abcq
	1
	2
	3 
	a 
	b 
	c 
	q 
	
	System.in是按行缓冲的。这意味着在按下Enter之前，实际上没有输入被传递到程序。
	
**13.2.2 读取字符**

	从键盘中读取字符串可以使用BufferReader类的readLine()方法，该方法的一般形式如下所示;
	
	String readLine() throws IOException
	
	可以看出该方法返回一个String字符串。
	
	例如：
	
```
import java.io;
	
class BRReadLines
{
	public static void main(String[] args) throws IOException
	{
		BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
			
		String str;
		System.out.println("Enter lines of text.");
		System.out.println("Enter 'stop' to quit.");
			
		do
		{
			str = br.readLine();
			System.out.println(str);
		}while(! str.equals("stop"));
	}
}
```
	
	
**13.3 向控制台写输出**

	在前面提到过，使用print()和println()是向控制台写输出的最容易的方式。这个方法是有PrintStream类(也就是System.out引用的对象的类型)定义的。System.out
尽管是字节流，但是对于简单的程序仍然何以使用。

	因为PrintStream是派生自OutputStream的输出流，所以还实现了低级的write()方法。因此，可以使用write()向控制台输出。
	
	void write(int byteval);
	
	例如：
	
```
class WriteDemo
{
	public static void main(String[] args)
	{
		int b;
		b = 'A';
		System.out.write(b);
		System.out.write('\n');
	}
}
```
	
	通常不会使用write()执行控制台输出(尽管在某些情况下这么做是有用的)，因为print()和println()确实更容易使用。
	

**13.4 PrintWriter类**

	尽管使用System.out向控制台输出可以接受，但是最好将其用于调试或者用于实例程序。对于实际的程序，使用Java向控制台输出的推荐方法是通过PrintWriter流。
PrintWriter是基于字符的流，为控制台输出使用基于字符的流，可以使程序国际化更容易。

	PrintWriter类定义了几个构造函数，在此使用的构造函数如下所示：
	
	Printwrite(OutputStream outputStream,boolean flushingOn)
	
	其中，flushingOn控制Java是否在每次调用println()方法时刷新输出流。如果flushingOn为true,就自动刷新；如果为false，那么不会自动刷新;
	
	PrintWriter类支持print()和println()方法，因此，可以使用与System.out相同的方式使用它们。如果参数不是简单类型，PrintWriter的方法会调用对象的toString()方法，然后输出。例如，下面这行代码是创建一个连接到控制台输出的PrintWriter；

	PrintWriter pw = new PrintWriter(System.out,true);
	
	例如：
	
```
import.java.io.*;
	
class PrintWriterDemo
{
	public static void main(String[] args)
	{
		PrintWriter pw = new PrintWriter(System.out,true);
			
		pw.println("This is a string");
		int i = -7;
		pw.println(i);
			
	}
}
```
	
	输出如下：
	This is a string 
	-7
	
	
**13.5 读/写文件**

	对于读/写文件，两个最常用的流类是FileInputStreamReader和FileOutputStream，这两个创建与文件链接的字节流。为了打开文件，只需要简单的这些类中某个类的对象，指定文件名作为构造函数的参数即可。尽管两个类也支持其他构造函数，但在此将使用一下方式：
	
	FileInputStream(String fileName) throws FileNotFoundException
	FileOutputStream(String fileName) throws FileNotFoundException
	
	其中，fileName指定希望打开的文件名。当创建输入流时，如果文件不存在，就会抛出FileNotFoundException异常。对于输出流，如果不能打开文件或不能创建文件，也会抛出FileNotFoundException异常。FileNotFoundException是IOException的子类。当打开输出文件时，先前存在的同名文件将被销毁。

	文件使用完成之后必须关闭文件。关闭文件是通过close()方法完成的，FileInputStream和FileOutputStream都实现了该方法。该方法的声明如下：
	
	void close() throws IOException
	
	关闭文件会释放为文件分配的系统资源，从而允许其他文件使用这些资源。关闭文件失败会导致"内层泄露"，因为未使用的资源没有被释放。
	
	注意：从JDK 7开始，close()方法是由java.lang包中的AutoCloseable接口指定的。java.io包中的Closeable接口继承了AutoCloseable接口，所有流类都实现了这两个接口，包括FileInputStream和FileOutputStream。

	可以使用两种方式关闭文件：
	第一种是传统方法，当在不需要文件时显示调用close()方法。这是JDK 7之前所有Java版本使用的方式，所以在所有遗留代码中可以看到这种方式。
	第二种是使用带资源的try语句，这种try语句是由JDK 7新增的，当不在需要文件时自动关闭文件。在这种方式写没有显示调用close()方法。因为JDK 7以后close()函数由AutoCloseable接口指定。故可以隐身自动关闭文件。

	下面先介绍关闭文件的传统方式，后面介绍带资源的try语句。
	
	为了读取文件，可以使用在FileInputStreamReader中定义的read()版本。
	
	int read() throws IOException
	
	每次调用read()方法时，都会从文件读取一个字节，并作为整型值返回。当到达文件末尾时，read()方法返回-1.
	
	下面的程序使用read()方法读取和显示文件的内容，该文件包含ASCII文本。文件名是作为命令行参数指定的。
	
```
	import java.io.*;
	
	class ShowFile
	{
		public static void main(String[] args)
		{
			int i;
			FileInputStream fin;
			
			if(args.length != 1)
			{
				System.out.println("Usage: ShowFile fileName");
				return;
			}
			
			//打开文件
			try
			{
				fin = new FileInputStream(args[0]);
			}
			catch(FileNotFoundException e)
			{
				System.out.println("Cannot Open File.");
				return;
			}
			
			//访问文件
			try
			{
				do
				{
					i = fin.read();
					if(i != -1) System.out.print((char)i);
				}while(i != -1);
			}
			catch(IOException e)
			{
				System.out.println("Error reading file.");
			}
			
			//关闭文件 
			try
			{
				fin.close();
			}
			catch(IOException e)
			{
				System.out.println("Error Closing File.");
			}
		}
	}
```
	
	尽管前面的例子中在读取文件后关闭了文件流，但是另个种处理方式通常很有用，就是在finally代码块中调用close()方法。在这种方式下，访问文件的所有方法都被包含在try代码块中，并使用finally代码块关闭文件。对于这种方式，不管try代码块是如何终止的，文件都会被关闭。例如：

```
	try
	{
		do
		{
			i = fin.read();
			if(i != -1) System.out.print(i);
		}while(i != -1);
	}
	catch(IOException e)
	{
		System.out.println("Error Reading File.");
	}
	finally
	{
		try
		{
			fin.close();
		}
		catch(IOException e)
		{
			System.out.println("Error Closing File.");
		}
	}
```
	
	尽管这么做对于这个例子没有什么问题，但一般来说，这种方式的一个优点是：即使访问文件的代码因为与I/O无关的异常而终止，finally代码块也仍然会关闭文件。
	
	有时在单个try代码块中(而不是独立的两个代码块)中封装打开文件和访问文件的代码，然后使用finally代码块关闭文件更容易一些。例如：
	
```
	import java.io.*;
	
	class ShowFile
	{
		public static void main(String[] args)
		{
			int i;
			FileInputStream fin = null;
			
			if(args.length != 1)
			{
				System.out.println("Usage: ShowFile fileName");
				return;
			}
			
			try
			{
				fin = new FileInputStream(args[0]);
				
				do
				{
					i = fin.read();
					if(i != -1) System.out.println((char) i);
				}while(i != -1);
			}
			catch(FileNotFoundException e)
			{
				System.out.println("File Not Found.");
			}
			catch(IOException e)
			{
				System.out.println("An I/O Error Occurred");
			}
			finally
			{
				try
				{
					if(fin != null) fin.close();
				}
				catch(IOException e)
				{
					System.out.println("Error Closing File");
				}
			}
		}
	}
```
	
	对于前面的例子，还可以更紧凑的使用try/catch语句。因为FileNotFoundException是IOException的子类，所以这种异常不需要单独捕获。例如，下面的语句进行了重新编写以消除捕获FileNotFoundException异常，在这种情况下，会显示错误的标准异常消息。

```
			try
			{
				fin = new FileInputStream(args[0]);
				
				do
				{
					i = fin.read();
					if(i != -1) System.out.println((char) i);
				}while(i != -1);
			}
			catch(IOException e)
			{
				System.out.println("I/O Error: "+e);
			}
			finally
			{
				try
				{
					if(fin != null) fin.close();
				}
				catch(IOException e)
				{
					System.out.println("Error Closing File");
				}
			}
```
			
	为了向文件中写入内容，可以使用FileOutputStream定义的write()方法，该方法最简单的形式如下：
	
	void write(int byteval) throws IOException
	
	例如，使用write()方法复制文件：
	
```
	import java.io.*;
	
	class CopyFile
	{
		public static void main(String[] args)
		{
			int i;
			FileInputStream fin;
			FileOutputStream fout;
			
			if(args.length != 2)
			{
				System.out.println("Usage: CopyFile from to");
				return;
			}
			
			try
			{
				fin = new FileInputStream(args[0]);
				fout = new FileOutputStream(args[1]);
				
				do
				{
					i = fin.read();
					if(i != -1) fout.write(i);
				}
			}
			catch(IOException e)
			{
				System.out.println("I/O Error: "+e);
			}
			finally
			{
				try
				{
					if(fin != null) fin.close();
				}
				catch(IOException e2)
				{
					System.out.println("Error Closing Input File");
				}
				
				try
				{
					if(fout != null) fout.close();
				}
				catch(IOException e2)
				{
					System.out.println("Error Closing Output File");
				}
			}
		}
	}
```
	
	
**13.6 自动关闭文件**
	
	在前面，一旦不在需要文件，示例程序就显示调用close()方法以关闭文件。这是JDK 7以前的Java版本使用的关闭文件方式。JDK 7之后增加了一个新特性，该特性提供了另外一种资源管理(例如文件流)的方式，这种方式能自动关闭文件。这个特性有时被称为自动资源管理，该特性以try语句的扩展板为基础。自动资源管理的主要优点是：当不在需要文件(或其他资源)时，可以防止无意中释放它们。

	自动资源管理基于try语句的扩展形式，它的一般形式如下所示：
	
```
	try(resource-specification)
	{
		//use the resource
	}
```
	
	其中，resource-specification是用来声明和初始化资源(例如文件流)的语句。该语句包含一个变量声明，在该变量声明中使用将被管理的对象引用初始化变量。当try语句块结束时，自动释放资源。
	当然，这种形式的try语句也可以包含catch和finally子句。新形式的try语句被称为带资源的try语句。
	
	对于那些实现了AutoCloseable接口的资源，才能使用带资源的try语句。AutoCloseable接口由java.lang包定义，该接口定义了close()方法。java.io包中的Closable接口继承自AutoCloseable接口。所有流类都实现了这两个接口。因此，当使用流时————包括文件流，可以使用带资源的try语句。

	下面是ShoeFile程序的改写版，该版本使用自动文件关闭功能：
		
```
	import java.io.*;
	
	class ShowFile
	{
		public static void main(String[] args)
		{
			int i;
			
			if(args.length != 1)
			{
				System.out.println("Usage: ShowFile fileName");
				return;
			}
			
			try(FileInputStream fin = new FileInputStream(args[0]))
			{
				do
				{
					i = fin.read();
					if(i != -1) System.out.println((char) i);
				}while(i != -1);
			}
			catch(FileNotFoundException e)
			{
				System.out.println("File Not Found.");
			}
			catch(IOException e)
			{
				System.out.println("An I/O Error Occurred");
			}
		}
	}
```
	
	在这个程序中，请特别注意在try语句中打开文件的方式：
	
	try(FileInputStream fin = new FileInputStream(args[0])){}
	
	注意try语句的资源约定部分声明了一个FileImputStream类型的变量fin,然后将由FileInputStream类构造函数打开的文件的引用赋值给该变量。因此，该程序的这个版本中，变量fin局限于try代码块，当进入try代码块时创建。当离开代码块时，会隐式的调用close()方法以关闭与fin关联的流。不需要显示的调用close()方法，这意味着不可能忘记关闭文件。这是使用带资源的try语句的关键好处。

	try语句中声明的资源被隐式声明为final，理解这一点很重要。这意味着在创建资源变量后，不能将其他资源赋值给该变量。此外，资源的作用域局限于带资源的try语句。

	可以在一条try语句中管理多个资源。为此，只需要简单的使用分号分隔每个资源约定即可。下面的程序显示了一个例子。该程序对前面显示的CopyFile进行了改写：
	
```
	import java.io.*;
	
	class CopyFile
	{
		public static void main(String[] args)
		{
			int i;
			
			if(args.length != 2)
			{
				System.out.println("Usage: CopyFile from to");
				return;
			}
			
			try(FileInputStream fin = new FileInputStream(args[0]);
				FileOutputStream fout = new FileOutputStream(args[1]))
			{		
				do
				{
					i = fin.read();
					if(i != -1) fout.write(i);
				}
			}
			catch(IOException e)
			{
				System.out.println("I/O Error: "+e);
			}
		}
	}
```
	

**13.7 applet的基础知识**

	applet好像已经过时了，这里先不写了。
	
**13.9 使用instanceof运算符**

	运算符instanceof德尔一般形式如下所示；
	
	objref instanceof type
	
	其中,objref是对类实例的引用，type是类类型。如果objref是指定的类型或者可以可以被转化为这种指定类型，那么instanceof运算符的求值结果为true;否则为false。
	
**13.11 本地方法**

	偶尔可能希望调用使用非Java语言编写的子例程，尽管这种情况很少见。典型的，这类子例程是作为可以执行的代码(对于正在使用的CPU和环境而言是可执行的代码，即本地代码)而存在的。但是，因为Java程序被编译为字节码，然后由Java运行时系统解释执行(或及时编译),所以看起来不可能从Java程序中调用本地代码子例程。幸运的是，这个结论是错误的。Java提供了native关键字，用于声明本地代码方法。一旦方法使用native进行声明，就可以从Java程序内部调用这些方法，就像调用其他Java方法一样。

	为了声明本地方法，在方法声明前使用native修饰符，但不能为方法定义任何方法体。例如：
	
	public native int meth();
	
	在声明了本地方法后，必须编写本地方法，并且需要遵循复杂的步骤序列，将本地方法和Java代码链接起来。
	
	大多数本地代码都是使用C语言编写的。将C代码集成到Java程序中的机制称为Java本地接口(Java Native Interface,JNI)。对于JNI的详细描述超出了本书的范围，不过下面提供的信息对于简单应用程序来说是足够的。

	注意：需要遵循的准确步骤根据不同的Java环境可能有所不同，它们还依赖于实现本地代码的语言。后续讨论假定使用Windows环境，使用C语言实现本地代码。另外，这里介绍的方法使用动态链接库，但是从JDK 8开始，也可以创建静态链接库。

	要理解这个过程，最容易的方法是通过一个例子。首先，输入下面的简短程序，该程序使用了本地方法test():
	
```
	public class NativeDemo
	{
		int i;
		
		public static void main(String[] args)
		{
			NativeDemo ob = new NativeDemo();
			
			ob.i = 10;
			System.out.println("This is ob.i before the native method: "+ob.i);
			
			ob.test();
			System.out.println("This is ob.i after the native method: "+ob.i);
		}
		
		//declare native method
		public native void test();
		
		//load DLL that contains static method 
		//只有加载进动态链接库DLL(NativeDemo.dll)，才能调用本地方法，因为，本地方法打包在了动态链接库中
		static
		{
			//加载动态链接库NativeDemo.dll,注意加载链接库时不需要写后缀名.dll 
			System.loadLibrary("NativeDemo");
		}
	}
```
	
	注意test()方法被声明为native并且没有方法体。稍后将使用C语言实现这个方法。还请注意static代码块。在前面已经解释过，static代码块只执行一次，当程序开始时执行(或更精确的说，当第一次加载它的类时执行)。这个例子中，static代码块用于加载包含test()本地实现的动态链接库。

	动态链接库是由loadLibrary()方法加载的，该方法是System类的一部分，它的一般形式如下所示：
	
	static void loadLibrary(String filename)
	
	其中，filename是标识库文件名的字符串。对于Windows环境，假定库文件具有.DLL扩展名。
	
	输入程序代码后，编译代码，生成NativeDemo.class。接下来便是生成动态链接库NativeDemo.DLL。步骤如下：
	
	(1)必须使用javah.exe(javah.exe包含于JDK中)生成文件NativeDemo.h。
	   
	  在test()方法实现中将包含此头文件NativeDemo.h。
	  
	  为了生成NativeDemo.h的头文件，使用下面的命令：
	  
	  javah -jni NativeDemo
	  
	  这个命令将生成文件名为NativeDemo.h的头文件。在实现test()方法的C文件中必须包含这个头文件。这个命令生成的输出如下所示：
	  
	  
```
	/* DO NOT EDIT THIS FILE - it is machine generated */
	#include <jni.h>
	/* Header for class NativeDemo */

	#ifndef _Included_NativeDemo
	#define _Included_NativeDemo
	#ifdef __cplusplus
	extern "C" {
	#endif
	/*
	* Class:     NativeDemo
	* Method:    test
	* Signature: ()V
	*/
	JNIEXPORT void JNICALL Java_NativeDemo_test
	(JNIEnv *, jobject);

	#ifdef __cplusplus
	}
	#endif
	#endif
```

	请特别注意下面的代码，它定义了将要创建test()函数的原型：

	JNIEXPORT void JNICALL Java_NativeDemo_test(JNIEnv *, jobject);
	
	注意函数的名称是Java_NativeDemo_test(),必须使用它作为要实现的本地函数的名称。也就是说，不是创建名为test()的C函数，而是创建名为Java_NativeDemo_test()的函数。NativeDemo部分是添加的前缀，因为test()方法是作为NativeDemo类的一部分定义的。记住，另外一个类也可能定义属于自己的test()本地方法，这和NativeDemo声明的test()本地方法完全不同。通过在前缀中包含类名，可以提供一种区分不同版本的手段。作为一般规则，为本地函数提供的名称将包含声明它们的类的名称。

	在生成必要的头文件之后，可以编写test()方法的实现代码，并将它们保存在NativeDemo.c文件中：
	
```
	#<jin.h>  
	#include "NativeDemo.h"
	#include<studio.h>


	JNIEXPORT void JNICALL Java_NativeDemo_test(JNIEnv *env, jobject obj)
	{
		jclass cls;
		jfieldId fid;
		jint i;

		printf("Starting the native method.\n");
		cls = (*env)->GetObjectClass(env,obj);
		fid = (*env)->GetFieldID(env,cls,"i","I");

		if(fid == 0)
		{
			printf("Could not get field id.\n");
			return;
		}

		i = (*env)->GetIntField(env,obj,fid);
		printf("i = %d\n",i);
		(*env)->SetIntField(env,obj,fid,2*i);
		printf("Ending the native method.\n");
	}
```

	注意这个文件包含了jni.h头文件，该头文件包含接口信息。这个头文件是由Java编译器提供的。头文件NativeDemo.h之前有Javah创建。
	
	在这个函数中，GetObjectClass()方法用于获取一个包含有关NativeDemo类信息的C结构。GetFieldID()方法返回一个C结构，该结构包含于类名为"i"的域变量有关的信息。GetIntField()方法该域变量的初始值，SetIntField()方法在该域变量中保存一个更新后的值(用于处理其他类型数据的其他方法请查看jni.h头文件)。

	在创建了NativeDemo.c文件后，必须进行编译并创建DLL。要通过Microsoft C/C++编译器完成该工作，需要使用下面的命令行(可能需要指定jni.h以及其次级文件jin_md.h的路径)：
	
	CI /LD NativeDemo.c 
	
	这会生成文件NativeDemo.dll。一旦该工作完成，就可以执行Java程序了，生成的输出如下：
	
	
	This is ob.i before the native method: 10 
	Starting the native method.
	i = 10 
	Ending the native method.
	This is ob.i after the native method: 20.
	
	为什么要创建动态链接库的原因：因为Java代码中执行本地函数meth()需要向动态链接库中查找函数，而不是在NativeDemo.c文件中查找函数。类似于Java调用函数，需要向类的.class文件中查找函数，引入第三方库也是类似，调用的函数需要向第三方库中查找，而不是向源代码中查找。

	本地方法存在的问题：
	(1)潜在的安全性风险：因为本地方法执行的实际的机器代码，所有可以访问宿主系统的所有部分。也就是说，本地代码可以离开Java执行环境。
	(2)可移植性丢失：本地代码因为包含在DLL中，所以必须存在于执行Java程序的机器上。而且，本地方法依赖于CPU和操作系统，所以DLL本身是不可移植的。
	   因此，使用本地方法的Java程序只能运行于已经安装了兼容DLL的机器上。
	   
	 因此，应当限制使用本地代码，因为它们会使Java程序不可移植，并且会造成严重的安全性风险。
	 

**13.12 使用assert**

	assert关键字有两种形式。第一种形式如下所示：
	
	assert condition;
	
	其中，condition是一个求值结果必须为布尔类型的表达式。如果结果为true，那么断言为真，不会发生其他动作。如果条件为false,那么断言失败并抛出默认的AssertionError对象。
	
	assert关键字的第二种形式如下所示：
	
	assert condition: expr;
	
	在这个版本中，expr是传递给AssertionError构造函数的值。这个值被转换成响应的字符串格式，并且如果断言失败，将会显示该字符串。
	
	
**13.14 通过this()调用重载的构造函数**

	当使用重载的构造函数时，在一个构造函数中调用另一个构造函数有时是有用的。在Java中，这是通过使用this关键字的另外一种形式完成的，它的一般形式如下所示
	
	this(arg-list)
	
	当执行this()时，首先执行与arg-list标识的参数列表相匹配的构造函数。然后，如果在原始构造函数中还有其他语句时，就执行这些语句。在构造函数中，对this()的调用必须是第一条语句。

	例如：下面没有使用this()的类。
	
```
	class MyClass
	{
		int a;
		int b;
		
		MyClass(int i,int j)
		{
			a = i;
			b = j;
		}
		
		MyClass(int i)
		{
			a = i;
			b = i;
		}
		
		MyClass()
		{
			a = 0;
			b = 0;
		}
	}
	
	使用this()的类，如下所示：

	class MyClass
	{
		int a;
		int b;
		
		MyClass(int i,int j)
		{
			a = i;
			b = j;
		}
		
		MyClass(int i)
		{
			this(i,i);
		}
		
		MyClass()
		{
			this(0,0);
		}
	}
```
	
	使用this()需要牢记两条限制。
	首先，在调用this()this时不能使用当前类的任何实例变量。
	其次，在同一个构造函数中不能同时使用supper()和this(),因为它们都必须是构造函数中的第一条语句。
	
	注意：this()最适合用于包含大量初始化代码的构造函数，而不适合用于那些只简单设置少了域变量值的构造函数。

出自：《Java 8编程参考官方教程(第9版)》
	
	