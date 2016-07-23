---
title: Java运行时多态性：继承和接口的实现
date: 2016-07-24 00:01:48
tags: Java
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

转载地址：[Java运行时多态性：继承和接口的实现](http://developer.51cto.com/art/200909/153887.htm)
Java做为一个面向对象语言的最强大机制：运行时多态性。两个实现方式分别是继承和接口。

Java是面向对象的语言，而运行时多态性是面向对象程序设计代码重用的一个最强大机制，动态性的概念也可以被说成“一个接口，多个方法”。Java实现运行时多态性的基础是动态方法调度，它是一种在运行时而不是在编译期调用重载方法的机制，下面就继承和接口实现两方面谈谈java运行时多态性的实现。

**一、通过继承中超类对象引用变量引用子类对象来实现**
举例说明：

```
//定义超类superA  
class superA  
{  
	int i = 100;  
	void fun()  
	{  
		System.out.println(“This is superA”);  
	}  
}  

//定义superA的子类subB  
class subB extends superA  
{  
	int m = 1;  
	void fun()  
	{      
		System.out.println(“This is subB”);  
	}  
}  

//定义superA的子类subC  
class subC extends superA  
{  
	int n = 1;  
	void fun()  
	{      
		System.out.println(“This is subC”);  
	}  
}  

class Test  
{  
	public static void main(String[] args)  
	{  
		//定义超类对象引用变量
		superA a;
		//子类对象  
		subB  b = new subB();  
		subC  c = new subC();
		//超类对象引用变量 引用 子类对象  
		a=b;  
		a.fun();  (1)  
		a=c;  
		a.fun();  (2)  
	}  
}
```
运行结果为：

This is subB
This is subC

上述代码中subB和subC是超类superA的子类，我们在类Test中声明了3个引用变量a, b, c，通过将子类对象引用赋值给超类对象引用变量来实现动态方法调用。也许有人会问：“为什么(1)和(2)不输出：This is superA”。**java 的这种机制遵循一个原则：当超类对象引用变量引用子类对象时，被引用对象的类型而不是引用变量的类型决定了调用谁的成员方法，但是这个被调用的方法必须是在超类中定义过的，也就是说被子类覆盖的方法。**

另外，如果子类继承的超类是一个抽象类，虽然抽象类不能通过new操作符实例化，但是可以创建抽象类的对象引用指向子类对象，以实现运行时多态性。具体的实现方法同上例。

不过，抽象类的子类必须覆盖实现超类中的所有的抽象方法，否则子类必须被abstract修饰符修饰，当然也就不能被实例化了。


----------
**二、通过接口类型变量引用实现接口的类的对象来实现**

接口的灵活性就在于“规定一个类必须做什么，而不管你如何做”。我们可以定义一个接口类型的引用变量来引用实现接口的类的实例，当这个引用调用方法时，它会根据实际引用的类的实例来判断具体调用哪个方法，这和上述的超类对象引用访问子类对象的机制相似。

举例说明：

```
//定义接口InterA  
interface InterA  
{  
	void fun();  
}  

//实现接口InterA的类B  
class B implements InterA  
{  
	public void fun()  
	{      
		System.out.println(“This is B”);  
	}  
}  

//实现接口InterA的类C  
class C implements InterA  
{  
	public void fun()  
	{      
		System.out.println(“This is C”);  
	}  
}  

class Test  
{  
	public static void main(String[] args)  
	{  
		//定义接口
		InterA  a;
		//接口类型变量引用实现接口的类的对象 
		a= new B();  
		a.fun();   
		a = new C();   
		a.fun();   
	}  
}
```
输出结果为：

This is B
This is C

上例中类B和类C是实现接口InterA的两个类，分别实现了接口的方法fun()，通过将类B和类C的实例赋给接口引用a而实现了方法在运行时的动态绑定，充分利用了“一个接口，多个方法”展示了Java的动态多态性。

需要注意的一点是：Java在利用接口变量调用其实现类的对象的方法时，该方法必须已经在接口中被声明，而且在接口的实现类中该实现方法的类型和参数必须与接口中所定义的精确匹配。


----------


**结束语**：以上就是java运行时多态性的实现方法，大家在编程过程中可以灵活运用，但是在性能要求较高的代码中不提倡运用运行时多态，毕竟Java的运行时动态方法调用较之普通的方法调用的系统开销是比较大的。
