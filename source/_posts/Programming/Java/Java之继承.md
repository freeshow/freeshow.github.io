---
title: Java之继承
date: 2016-07-23 23:15:20
tags: Java
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

**8.1 继承的基础知识**

	Java支持单继承，但不支持多继承。
	
	尽管子类包含超类的所有成员，但是子类不能访问超类中被声明为私有的所有成员。
	
**8.2 使用super关键字**

```
	class Box
	{
		doubel width;
		doubel height;
		doubel depth;
		
		Box(doubel w,doubel h,doubel d)
		{
			width = w;
			height = h;
			depth = d;
		}
	}
	
	class BoxWeight extents Box
	{
		doubel weight;
		
		BoxWeight(doubel w,doubel h,doubel d,doubel m)
		{
			width = w;  //当超类中width是私有成员时，就不能这样赋值，只能使用super()进行赋值。
			height = h;
			depth = d;
			weight = m;
		}
	}
```
	
	但是，有时候希望创建只有自己才知道实现细节的超类（也就是说将超类的数据成员保存为私有）。对于这种情况，子类就不能直接访问或初始化这些变量。
	因为封装是OOP的主要特性，所以Java为这一个问题提供了一个解决的方案是很正常的。
	无论何时，当子类需要引用它的直接超类时，都可以使用关键字super.
	
	super有两种一般用法：第一种用于调用超类的构造函数；第二种用于访问超类中被子类的某个成员隐藏的成员。
	
	(1)使用super调用超类的构造函数。
	
	格式：super(arg-list);
	其中，arg-list是超类中构造函数中需要的全部参数，并且super()总是子类的构造函数中执行的第一条语句。
	例如：
```
	class BoxWeight
	{
		BoxWeight(doubel w,doubel h,doubel d,doubel m)
		{
			super(w,h,d);
			weight = m;
		}
	}
```
	其中，BoxWeight()使用参数w、h和d调用super(),这会调用Box类的构造函数，使用这些值初始化width、height和depth变量。
	BoxWeight自身不在初始化这些值，而是只需要初始化自身特有的值：weight.这样一来，Box类就可以将这些变量声明为私有的。
	
	总结：当子类调用super()时，会调用直接超类的构造函数。因此，super()总是引用调用类的直接超类。即使在多层次继承中也是如此。
		  此外，super()总是子类构造函数中执行的第一句。
		  
	(2)使用super访问超类中被子类的某个成员隐藏的成员。
	
	super的这种一种用法和this有些类似，只不过super总是引用超类。
	格式：super.member
	其中，member既可以是方法，也可以是实例变量。
	
	最常用的形式情况是,子类的成员名称隐藏了超类中的同名成员。
	例如：
```
	class A 
	{
		int i;
	}
	
	class B extents A
	{
		int i;
		
		B(int a,int b)
		{
			super.i = a;
			i = b;
		}
		
		void show()
		{
			System.out.println("i in superclass:"+super.i);
			System.out.println("i in subclass:"+i);
		}
	}
	
	class SueSuper
	{
		public static void main(String[] args)
		{
			B subOb = new B(1,2);
			subOb.show();
		}
	}
```
	输出：
	i in superclass: 1
	i in subclass: 2
	
8.4 构造函数的调用时机

	答案是：在类层次中，从超类到子类按照继承的顺序调用构造函数。
	
**8.5 方法重写**

	在类层次中，如果子类的一个方法和超类的一个方法具有相同的名称和类型签名，那么称子类中的这个方法重写了超类中相应的那个方法。
	当子类中调用被重写的方法时，总是调用有子类定义的版本，由超类定义的版本会被隐藏。
	例如：
```
	class A
	{
		int i,j;
		
		A(int a,int b)
		{
			i = a;
			j = b;
		}
		
		void show()
		{
			System.out.println("i and j:"+i+" "+j);
		}
	}
	
	class B extents A
	{
		int k;
		
		B(int a,int b,int c)
		{
			super(a,b);
			k = c;
		}
		
		void show()
		{
			System.out.println("k:"+k);
		}
	}
	
	class Override
	{
		public static void main(String[] args)
		{
			B subOb = new B(1,2,3);
			
			subOb.show();
		}
	}
```
	输出：
	k:3
	当在类型B的对象上调用show()方法时，使用的是B中定义的版本。也就是说，类B中的show()版本覆盖了在类A中声明的版本。
	
	如果希望访问到超类中被重写的方法，可以通过super完成该操作。例如将上面的类B改成如下形式：
```
	class B extents A 
	{
		int k;
		
		B(int a,int b,int c)
		{
			super(a,b);
			k = c;
		}
		
		void show()
		{
			super.show(); //this calls A's show()
			System.out.println("k:"+k);
		}
	}
```
	输出为：
	i and j: 1,2
	k:3
	
	只有当两个方法的签名(方法名称和参数列表的顺序和类型)相同时，才发生重写。如果不相同，那么这两个方法只是简单的重载关系。
	
**8.6 动态方法调度**

	方法重写形成了动态方法调度(dynamic method dispatch)的基础，动态方法调度是Java中最强大的功能之一。动态方法调度是一种机制，
	通过这种机制可以在运行时，而不是在编译时解析对重写方法的调用。动态方法调度很重要，因为这是Java实现运行时多态的机理所在。
	
	首先在此声明一个重要的原则：超类引用变量可以指向子类对象。Java利用这一事实，在运行时解析对重写方法的调用。
	下面是实现原理：当通过超类引用调用重写的方法时，Java根据在调用时所引用对象的类型来判断调用哪个版本的方法。
	因此，这个决定是在运行时做出的。如果引用不同的对象，就会调用不同版本的重写方法。换句话说，是当前正在引用的对象
	的类型（而不是引用变量的类型）决定了要执行哪个版本的重写方法。
	例如：
```
	class A
	{
		void callme()
		{
			System.out.println("Inside A's callme method")
		}
	}
	
	class B extents A 
	{
		void callme()
		{
			System.out.println("Inside B's callme method");
		}
	}
	
	class C extents A 
	{
		void callme()
		{
			System.out.println("Inside C's callme method");
		}
	}
	
	class DisPatch
	{
		public static void main(String[] args)
		{
			A a = new A();
			B b = new B();
			C c = new C();
			A r;
			
			r=a;
			r.callme():
			
			r=b;
			r.callme();
			
			r=c;
			r.callme();
		}
	}
```
	输出：
	Inside A's callme method
	Inside B's callme method
	Inside C's callme method
	
**8.7 使用抽象类**

	下面的程序创建了一个超类Figure,该类存储二维对象的尺寸。该类还定义了一个area()方法，该方法计算对象的面积。
	这个程序从Figure类派生了两个子类。第一个子类是Rectangle(矩形),第二个子类是Triangle(三角形)，每个子类都重写了area()方法，
	从而可以相应的返回矩形和三角形的面积。
	
```
	class Figure
	{
		double dim1;
		double dim2;
		
		Figure(double a, double b)
		{
			dim1 = a;
			dim2 = b;
		}
		
		doubel area()
		{
			System.out.println("Area for Figure is undefined.")
			return 0;
		}
	}
	
	class Rectangle extents Figure
	{
		Rectangle(double a, double b)
		{
			super(a,b);
		}
		
		double area()
		{
			System.out.println("Inside Area for Rectangle.");
			return dim1*dim2;
		}
	}
	
	class Triangle extents Figure
	{
		Triangle(double a,double b)
		{
			super(a,b);
		}
		
		double area()
		{
			System.out.println("Inside Area for Triangle.");
			return dim1*dim2/2;
		}
	}
	
	class FindAreas
	{
		public static void main(String[] args)
		{
			Figure f = new Figure(10,10);
			Rectangle r = new Rectangle(9,5);
			Triangle t = new Triangle(10,8);
			Figure figure;
			
			figure = f;
			System.out.println("Area is"+figure.area());
			
			figure = r;
			System.out.println("Area is"+figure.area());
			
			figure = t;
			System.out.println("Area is"+figure.area());
		}
	}
```
	输出：
	Area for Figure is undefined.
	Area is 0
	Inside Area for Rectangle.
	Area is 45
	Inside Area for Triangle.
	Area is 40
	
	有时候会希望创建这样一种超类——只定义被所有子类共享的一般形式，而让每个子类填充细节。这种类决定了子类必须实现的方法的本质。
	例如，分析Triangle类，如果没有定义area()方法，它将没有意义。对于这种情况，你会希望有某些方式能够确保子类确实、真正的重写了所有必需的方法。
	Java对这个问题提供的解决方法是抽象方法(abstract method).
	
	可以通过abstract类型修饰符，要求特定的方法必须被子类重写。这些方法有时被称做子类责任(subclass responsibility).
	因为在超类中没有提供实现。因此，子类必须重写他们————不能简单的使用在超类中定义的版本。
	
	为了声明抽象方法，需要使用下面的一般形式：
	abstract type name(parameter-list);
	
	任何包含一个或多个抽象方法的类都必须被声明为抽象。为了声明抽象类，只需要简单的在类声明的开头、在class关键字前面使用关键字abstract。
	对于抽象类不存在对象。也就是说，不能使用new运算符直接实例化抽象类。这种类的对象是无用的，因为抽象类的定义是不完整的。
	此外，不能声明抽象的构造方法，也不能声明抽象的静态方法。抽象类的所有子类，要么实现超类中的所有抽象方法，要么自己也声明为抽象类。
	抽象类中可以包含非抽象方法。
	
	将上面的例子，写成抽象的形式：
```
	abstract class Figure
	{
		double dim1;
		double dim2;
		
		Figure(double a,double b)
		{
			dim1 = a;
			dim2 = b;
		}
		
		abstract double area();
	}
	
	class Rectangle extents Figure
	{
		Rectangle(double a,double b)
		{
			super(a,b);
		}
		
		double area()
		{
			System.out.println("Inside Area for Rectangle.");
			return dim1*dim2;
		}
	}
	
	class Triangle extents Figure
	{
		Triangle(double a,double b)
		{
			super(a,b);
		}
		
		double area()
		{
			System.out.println("Inside Area For Triangle.");
			return dim1*dim2/2;
		}
	}
	
	class AbstractAreas
	{
		public static void main(String[] args)
		{
			Figure f;
			Rectangle r = new Rectangle(10,10);
			Triangle t = new Triangle(10,8);
			
			f = r;
			System.out.println("Area is "+f.area());
			
			f = t;
			System.out.println("Area is "+f.area());
		}
	}
```
	尽管不能创建Figure类型的对象，但是可以创建Figure类型的引用变量。因为，Java运行时多态是通过使用超类引用实现的。
	因此，必须能够创建指向抽象类的引用，从而可以用于指向子类对象。
	
**8.8 在继承中使用final关键字**

	关键字final有三个用途。
	首先，可以用于创建自己命名常量的等价物。
	final关键字的另外两个用途是用于继承。
	
	(1)使用final关键字阻止重写
	
	虽然方法重写是Java中最强大的特性之一，但是有时候希望阻止这种情况的发生。为了禁止重写方法，可以在方法中声明的开头使用final作为修饰符。
	使用final声明的方法不能被重写。
	
	经方法声明为final，有时可以提高性能：编译器可以自由的内联对这类方法的调用，因为编译器知道这些方法不能被子类重写。当调用小的fanal方法时，
	Java编译器通常可以复制子例程的字节码，直接和调用方法的编译代码内联到一起，从而可以消除方法调用所需要的开销。
	
	内联是final方法才有的一个选项。通常Java在运行时动态分析对方法的调用，这称为后期绑定。
	但是，因为final方法不能被重写，所以对final方法的调用可以在编译时解析，这称为早期绑定。
	
	(2)使用final关键字阻止继承
	
	有时候希望阻止类的继承。为此，可以在类声明的前面使用final关键字。将类声明为final,就隐式的将类的所有方法声明为final.
	因此，将类同时声明为abstract和final是非法的，因为抽象类是不完整的，它依赖子类来提供完整的实现。
	
**8.9 Object类**

	有一个特殊的类，即Object，该类由Java定义的。所有其他类都是Object的子类。也就是说，Object是所有其他类的超类。这意味着Object
	类型的引用变量可以引用其他类型的对象。此外，因为数组也是作为类实现的，所以Object类型的变量也可以引用任何数组。
	
	Object类定义了如下方法，这意味着所有对象都可以使用这些方法：
	
	Object clone():		创建一个和将要复制对象完全相同的新对象。
	boolean equals(Object object):		判定一个对象是否和另一个对象相等。
	void finalize():		在回收不在使用的对象之前调用。
	Class<?> getClass():		在运行时获取对象所使用的类
	int hashCode():			返回与调用对象相关联的散列值。
	void notify():		恢复执行在调用对象上等待的某个线程。
	void notifyAll():		恢复执行在调用对象上等待的所有线程。
	String toString(): 		返回一个描述对象的字符串。
	
	void wait():
	void wait(long milliseconds):
	void wait(long milliseconds, int nanoseconds): 		等待另一个线程的执行。
	
	方法getClass()、notify()、notifyAll()以及wait()被声明为final.你可以重写其他方法。
	
	
	出自：《Java 8编程参考官方教程(第9版)》