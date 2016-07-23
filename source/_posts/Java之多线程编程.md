---
title: Java之多线程编程
date: 2016-07-23 23:12:43
tags: Java
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

	在单核系统中，并发执行的线程共享CPU，每个线程得到一片时间周期。所以，在单核系统中，两个或更多个线程不是真正的同时运行，但是
空闲时间被利用了。然而，在多核系统中，两个或更多个线程可能是真正同步执行。在许多情况下，这会进一步提高程序的效率并提高特定操作的速度。

**11.1.1 线程优先级**

	Java为每个线程都指定了优先级，优先级决定了相对于其他线程应当如何处理某个线程。线程的优先级用于决定何时从一个运行的线程切换到写一个。这种称为上下文切换(context switch)。决定上下文切换发生时机的规则比较简单：
	
(1)线程自愿的放弃控制。线程显示的放弃控制、休眠或在I/O之前阻塞，都会出现这种情况。在这种情况下，检查所有其他线程，并且准备运行的线程中优先级高的那个线程会获取CPU资源。

(2)线程被允许优先级更高的线程取代。对于这种情况，没有放弃控制权的低优先级线程不管正在做什么，都会被高优先级线程取代。基本上，只要高优先级 的线程希望运行，它就会取代低优先级的线程，这种称为抢占式多任务处理(preemptive multitasking)。 

   如果具有相同优先级的两个线程竞争CPU资源，这种情况有些复杂。对于Windows这类操作系统，优先级相等的线程以循环方式自动获取CPU资源。对于其他操作系统，优先级相等的线程必须自愿的向其他线程放弃控制权，否则其他线程就不能运行。

**11.1.2 同步**

	此处讲得同步其实是操作系统中的互斥。
	例如，链表，就需要以某种方式确保它们之间不会发生冲突。也就是说，当一个线程正在读取该数据结构时，必须阻止另一个线程向该数据结构写入数据。
	为此，Java以监视器这一年代久远的进程间同步模型为基础，实现了一种巧妙的方案。监视器最初由C.A.R.Hoare定义的一种控制机制，可以将监视器看作非常小的只能包含一个线程的盒子。一旦某个线程进入监视器，其他所有线程必须等待，直到该线程退出监视器。通过这种方式，可以将监视器用于保护共享的资源，以防止多个线程同时对资源进程操作。

	Java没有提供"Monitor"类；相反，每个对象都有自己隐式的监视器。如果调用对象的同步方法，就会自动进入对象的隐式监视器。一旦某个线程位于一个同步方法中，其他线程就不能调用同一对象的任何其他同步方法。因为语言本身内置了同步支持，所以可以编写出非常清晰并且简明的多线程代码。

**11.1.3 消息传递**

	将程序分隔到独立的线程之后，需要定义它们之间相互通信的方式。当使用某些其他语言编程时，必须依赖操作系统建立线程之间的通信。当然，这会增加系统开销。相反，通过调用所有对象都具有的预定义的方法，Java为两个或更多个线程之间的相互通信提供了一种简洁的低成本方法。Java的消息传递系统允许某个线程进入对象的同步方法，然后进行等待，直到其他线程显示地通知这个线程退出为止。

**11.1.4**

	Java的多线程系统是基于Thread类、Thread类的方法及其伴随接口Runnable而构建的。Thread类封装了线程的执行。因为不能直接引用正在运行的线程的细微状态，所以需要通过代理进行处理，Thread实例就是线程的代理。为了创建新线程，程序可以扩展Thread类或实现Runnable接口。
	Thread类定义了一些用于帮助管理线程的方法。
	getName():获取线程的名称。
	getPriority()：获取线程的优先级。
	isAlive()：确定线程是否仍然运行。
	join():等待线程终止。
	run():线程的入口点。
	sleep():挂起线程一段时间。
	start():通过调用线程的run()方法启动线程。
	
**11.2 主线程**

	当Java程序启动时，会立即开始运行一个线程，因为它是程序开始时执行的线程，所以这个线程通常称为程序的主线程。主线程很重要，有以下两个原因：
	(1)其他线程都是从主线程产生的。
	(2)通常，主线程必须是最后才结束执行的线程，因为它要执行各种关闭动作。
	
	尽管主线程是在程序启动时自动创建的，但是可以通过Thread对象对其进行控制。为此，必须调用currentThread()方法获取对主线程的一个引用。该方法是Thread类的公有静态成员，它的一般形式如下：
	static Thread currentThread()
	这个方法返回对调用它的线程的引用。一旦得到对主线程的引用，就可以像控制其他线程那样控制主线程了。
	
	sleep()方法使线程从调用时挂起，暂缓执行指定的时间间隔(毫秒数),它的一般形式如下：
	static void sleep(long milliseconds) throws InterruptedException
	sleep()方法可能会抛出InterruptedException异常。如果其他线程试图中断这个正在睡眠的线程，就会抛出这个异常。
	
**11.3 创建线程**

	在最通常情况下，通过实例化Thread类型的对象创建线程。Java定义了创建线程的两种方法：
	(1)实现Runnable接口 (重载Runnalbe接口中的run()方法)
	(2)扩展Thread类本身 (需要从Java.lang.Thread类派生一个新的线程类，重载它的run()方法)
	
**11.3.1 实现Runnable接口**

	创建线程最简单的方式是创建实现了Runnable接口的类。Runnable接口抽象了一个可执行代码单元。可以依托任何实现了Runnable接口的对象来创建线程。
为了实现Runnable接口，类只需要实现run()方法，该方法的声明如下：
	

> 	public void run()

	
	在run()方法内部，定义组成新线程的代码。run()方法可以调用其他方法，使用其他类，可也以声明变量，就像main线程那样，理解这一点很重要。唯一的区别是：run()方法为程序中另外一个并发线程的执行建立了入口点。当run()方法返回时，这个线程将结束。

	在创建实现了Runnable接口的类之后，可以在类中实例化Thread类的对象。Thread类定义了几个构造函数。我们使用的那个构造函数如下所示：
	
	Thread(Runnable threadOb,String threadName)
	
	在这个构造函数中，threadOb是实现了Runnable接口的类的实例，这定义了从何开始执行线程。新线程的名字有threadName指定。
	
	 在创建了新线程之后，只有调用线程的start()方法，线程才会运行，该方法是在Thread类中声明的。本质上，start()方法执行对run()方法的调用。
	 start()方法的声明如下所示：
	 
	 void start()
	 
	 例如：
```
	 class NewThread implements Runnable
	 {
		Thread t;
		
		NewThread()
		{
			//Create a new thread.
			t = new Thread(this,"Demo Thread");
			System.out.println("Child thread: "+t);
			//Start thread 
			t.start();
		}
		
		public void run()
		{
			try
			{
				for(int i = 5; i>0; i--)
				{
					System.out.println("Child Thread: "+i);
					Thread.sleep(500);
				}
				
			}
			catch(InterruptedException e)
			{
				System.out.println("Child interrupted.");
			}
				
			System.out.println("Exiting child thread.");
		}	
	 }
	 
	 class ThreadDemo 
	 {
		public static void main(String[] args)
		{
			new NewThread();
			
			try
			{
				for(int i = 5; i>0; i--)
				{
					System.out.println("Main Thread: "+i);
					Thread.sleep(1000);
				}
			}
			catch(InterruptedException e)
			{
				System.out.println("Main Thread interrupted.");
			}
			
			System.out.println("Main thread exiting.");
		}
	 }
```
	
	在NewThread类的构造函数中，通过下面这条语句创建了一个新的Thread对象:
	
	t = new Thread(this,"Demo Thread");
	
	传递this作为第一个参数，以表明希望新线程调用this对象的run()方法。接下来调用start()方法，从run()方法开始启动线程的执行。这会导致开始执行子线程的for循环。调用完start()方法之后，NewThread类的构造函数返回到main()方法。当恢复主线程时，会主线程的for循环。这两个线程继续运行，在单核系统中它们会共享CPU，直到它们的循环结束。

	另外提供一种，比较符合人们编程方式的上面代码的书写方式，即不在实现了Runnable接口的类中创建/启动线程。
	
```
	class NewThread implements Runnable
	 {
	 
		public void run()
		{
			try
			{
				for(int i = 5; i>0; i--)
				{
					System.out.println("Child Thread: "+i);
					Thread.sleep(500);
				}
				
			}
			catch(InterruptedException e)
			{
				System.out.println("Child interrupted.");
			}
				
			System.out.println("Exiting child thread.");
		}	
	 }
	 
	 class ThreadDemo 
	 {
		public static void main(String[] args)
		{
			Runnable r = new NewThread();
			Thread t = new Thread(r,"Demo Thread");
			t.start();
			
			try
			{
				for(int i = 5; i>0; i--)
				{
					System.out.println("Main Thread: "+i);
					Thread.sleep(1000);
				}
			}
			catch(InterruptedException e)
			{
				System.out.println("Main Thread interrupted.");
			}
			
			System.out.println("Main thread exiting.");
		}
	 }
```
	
**13.2 扩展Thread类**

	扩展类必须重写run()方法，run()方法是新线程的入口点。扩展类还必须调用start()方法以开始新线程的执行。
	使用扩展Thread类对上面程序就行就该：
	
```
	class NewThread extends Thread
	{
		NewThread()
		{
			super("Demo Thread");
			System.out.println("Child thread: "+this);
			start(); //start the thread 
		}
		
		public void run()
		{
			try
			{
				for(int i=5; i>0; i--)
				{
					System.out.println("Child Thread: "+i);
					Thread.sleep(500);
				}
			}
			catch(InterruptedException e)
			{
				System.out.println("Child interrupted.");
			}
			
			System.out.println("Exiting child thread.");
		}
	}
	
	class ExtendThread
	{
		public static void main(String[] args)
		{
			new NewThread();
			
			/**
			*	当NewThread类中的构造函数中没有start()方法时，可以这样书写，比较符合一般人的习惯。
			*	Thread t = new NewThread();
			*	t.start()
			*/
			
			try
			{
				for(int i = 5; i>0; i--)
				{
					System.out.println("Main Thread: "+i);
					Thread.sleep(1000);
				}
			}
			catch(InterruptedException e)
			{
				System.out.println("Main Thread interrupted.");
			}
			
			System.out.println("Main thread exiting.");
		}
	}
```
	
	
**11.3.3 选择一种创建方式**	
	
	为什么Java要提供两种方法来创建线程呢？它们都有哪些区别？相比而言，哪一种方法更好呢？
	在Java中，类仅支持单继承，也就是说，当定义一个新的类的时候，它只能扩展一个外部类.这样，如果创建自定义线程类的时候是通过扩展 Thread类的方法来实现的，那么这个自定义类就不能再去扩展其他的类，也就无法实现更加复杂的功能。因此，如果自定义类必须扩展其他的类，那么就可以使用实现Runnable接口的方法来定义该类为线程类，这样就可以避免Java单继承所带来的局限性。

	还有一点最重要的就是使用实现Runnable接口的方式创建的线程可以处理同一资源，从而实现资源的共享.
	
	假设一个影院有三个售票口，分别用于向儿童、成人和老人售票。影院为每个窗口放有100张电影票，分别是儿童票、成人票和老人票。三个窗口需要同时卖票，而现在只有一个售票员，这个售票员就相当于一个CPU，三个窗口就相当于三个线程。通过程序来看一看是如何创建这三个线程的。

	（1）通过扩展Thread类来创建多线程
	
```
	class MutliThread extends Thread
	{
		private int ticket=100;//每个线程都拥有100张票

			MutliThread(String name)
			{
				super(name);//调用父类带参数的构造方法
			}

			public void run()
			{
				while(ticket>0)
				{
					System.out.println(ticket--+" is saled by "+Thread.currentThread().getName());
					ticket--;
				}
			}
	}
	
	public class MutliThreadDemo 
	{
		public static void main(String [] args)
		{
			MutliThread m1=new MutliThread("Window 1");
			MutliThread m2=new MutliThread("Window 2");
			MutliThread m3=new MutliThread("Window 3");
			
			m1.start();
			m2.start();
			m3.start();
		}
	}
```
	
	程序中定义一个线程类，它扩展了Thread类。利用扩展的线程类在MutliThreadDemo类的主方法中创建了三个线程对象，并通过start()方法分别将它们启动。
	
	从结果可以看到，每个线程分别对应100张电影票，之间并无任何关系，这就说明每个线程之间是平等的，没有优先级关系，因此都有机会得到CPU的处理。但是结果显示这三个线程并不是依次交替执行，而是在三个线程同时被执行的情况下，有的线程被分配时间片的机会多，票被提前卖完，而有的线程被分配时间片的机会比较少，票迟一些卖完。

	可见，利用扩展Thread类创建的多个线程，虽然执行的是相同的代码，但彼此相互独立，且各自拥有自己的资源，互不干扰。
	
   （2）通过实现Runnable接口来创建多线程
	
```
	class MutliThread implements Runnable
	{
		private int ticket=100;//每个线程都拥有100张票
		private String name;
		
		MutliThread(String name)
		{
			this.name=name;
		}
		
		public void run()
		{
			while(ticket>0)
			{
				System.out.println(ticket--+" is saled by "+name);
				ticket--;
			}
		}
	}
	
	public class MutliThreadDemo2 
	{
		public static void main(String [] args)
		{
			MutliThread m1=new MutliThread("Window 1");
			MutliThread m2=new MutliThread("Window 2");
			MutliThread m3=new MutliThread("Window 3");

			Thread t1=new Thread(m1);
			Thread t2=new Thread(m2);
			Thread t3=new Thread(m3);

			t1.start();
			t2.start();
			t3.start();
		}
	}
```
	
	由于这三个线程也是彼此独立，各自拥有自己的资源，即100张电影票，因此程序输出的结果和（1）结果大同小异。均是各自线程对自己的100张票进行单独的处理，互不影响。
	可见，只要现实的情况要求保证新建线程彼此相互独立，各自拥有资源，且互不干扰，采用哪个方式来创建多线程都是可以的。因为这两种方式创建的多线程程序能够实现相同的功能。

   （3）通过实现Runnable接口来实现线程间的资源共享
   
   现实中也存在这样的情况，比如模拟一个火车站的售票系统，假如当日从A地发往B地的火车票只有100张，且允许所有窗口卖这100张票，那么每一个窗口也相当于一个线程，但是这时和前面的例子不同之处就在于所有线程处理的资源是同一个资源，即100张车票。如果还用前面的方式来创建线程显然是无法实现的，这种情况该怎样处理呢？看下面这个程序，程序代码如下所示：

```
	class MutliThread implements Runnable
	{
		private int ticket=100;//每个线程都拥有100张票
		
		public void run()
		{
			while(ticket>0)
			{
				System.out.println(ticket--+" is saled by "+name);
				ticket--;
			}
		}
	}
	
	public class MutliThreadDemo2 
	{
		public static void main(String [] args)
		{
			MutliThread m=new MutliThread();

			Thread t1=new Thread(m,"Window 1");
			Thread t2=new Thread(m,"Window 2");
			Thread t3=new Thread(m,"Window 3");

			t1.start();
			t2.start();
			t3.start();
		}
	}
```
	
	结果正如前面分析的那样，程序在内存中仅创建了一个资源，而新建的三个线程都是基于访问这同一资源的，并且由于每个线程上所运行的是相同的代码，因此它们执行的功能也是相同的。

	可见，如果现实问题中要求必须创建多个线程来执行同一任务，而且这多个线程之间还将共享同一个资源，那么就可以使用实现Runnable接口的方式来创建多线程程序。而这一功能通过扩展Thread类是无法实现的，读者想想看，为什么？
	答案：通过扩展Thread类可以实现，只需将ticket设置成静态static类型，读写的时候进行加锁即可。但是，实现ruanable接口的这种方式，明显是定义一个类，创建一个对象，然后跟着创建三个线程，他们共享该对象的成员变量。相比第一种继承Thread，要创建三个对象，然后创建三个线程，最后按照你们的方法创建一个static的变量，再加锁，这个方式，并不是很好。

	实现Runnable接口相对于扩展Thread类来说，具有无可比拟的优势。这种方式不仅有利于程序的健壮性，使代码能够被多个线程共享，而且代码和数据资源相对独立，从而特别适合多个具有相同代码的线程去处理同一资源的情况。这样一来，线程、代码和数据资源三者有效分离，很好地体现了面向对象程序设计的思想。因此，几乎所有的多线程程序都是通过实现Runnable接口的方式来完成的。

**11.5 使用isAlive()和join()方法** 

	前面提到，通常希望主线程在最后结束。在前面的例子中通过sleep()方法，并指定足够长的延迟时间来确保所有子线程在主线程之前终止。但是，这完全不是一个令人满意的方案，并且还能造成一个更大的问题：线程如何知道另一个线程何时结束？幸运的是，Thread类提供了一种能够解决这个问题的方法。

	有两种方法可以确实线程是否已经结束。
	首先，可以为线程调用isAlive()方法。这个方法是由Thread类定义的，它的一般形式如下所示：
	
	final boolean isAlive()
	
	如果线程仍在运行，isAlive()方法就会返回true,否则返回false.
	
	虽然isAlive()方法很有用，但是通常使用join()方法来等待线程结束，如下所示：
	
	final void join() throws InterruptedException
	
	该方法一直等待，直到调用线程终止。如此命名该方法的原因是：调用线程一直等待，直到指定的线程加入(join)其中为止。join()方法的另外一种形式，允许指定希望等待指定线程终止的最长时间。

	下面是前面例子的改进版本，该版本使用join()方法确保主线程在最后结束，另外演示了isAlive()方法的使用：
	
```
	class NewThread implements Runnable
	{
		String name; //name of thread
		Thread t;
		
		NewThread(String threadName)
		{
			name = threadName;
			t = new NewThread(this,name);
			System.out.println("New Thread: "+t);
			t.start();
		}
		
		public void run()
		{
			try
			{
				for(int i=5; i>0; i--)
				{
					System.out.println(name+": "+i);
					Thread.sleep(1000);
				}
				catch(InterruptedException e)
				{
					System.out.println("name"+" interrupted");
				}
				
				System.out.println(name+" exiting");
			}
		}
	}
	
	class DemoJoin
	{
		public static void main(String[] args)
		{
			NewThread ob1 = new NewThread("One");
			NewThread ob2 = new NewThread("Two");
			NewThread ob3 = new NewThread("Three");
			
			System.out.println("Thread One is alive: "+ob1.t.isAlive());
			System.out.println("Thread Two is alive: "+ob2.t.isAlive());
			System.out.println("Thread Three is alive: "+ob3.t.isAlive());
			
			//waiting for threads to finish
			try
			{
				System.out.println("Waiting for threads to finish.");
				ob1.t.join();
				ob2.t.join();
				ob3.t.join();
			}
			catch(InterruptedException e)
			{
				System.out.println("Main Thread interrupted");
			}
			
			System.out.println("Thread One is alive: "+ob1.t.isAlive());
			System.out.println("Thread Two is alive: "+ob2.t.isAlive());
			System.out.println("Thread Three is alive: "+ob3.t.isAlive());
			
			System.out.println("Main Thread exiting");
		}
	}
```
	
	可以看出，在join()方法调用返回之后，线程停止执行。
	
**11.6 线程优先级** 

	为了设置线程的优先级，需要使用setPriority()方法，它是Thread类的成员，下面是该方法的一般形式：
	
	final void setPriority(int level)
	
	其中，level指定了为调用线程设置的新优先级，默认level == 5.level的值必须在MIN_PRIORITY和MAX_PRIORITY之间选择。目前，这些值分别是1和10.
如果希望线程设置为默认优先级，可以使用NORM_PRIORITY，目前的值是5.这些优先级在Thread类中作为static final变量定义的。

	可以通过调用Thread类的getPriority()方法获取当前的优先级，该方法如下所示：
	
	final int getPriority()
	
**11.7 同步** 

	当两个或多个线程需要访问共享资源时，它们需要以某种方式确保每一次只有一个线程使用资源。实现这一目的的过程称为同步。
	
	同步的关键是监视器的概念，前面已经叙述。监视器是用作互斥锁的对象。在给定时刻，只有一个线程可以拥有监视器。
	
	可以使用两种方法同步代码。这两种方法都要用到synchronized关键字，下面分别介绍这两种方法。
	
**11.7.1  使用同步方法**
	
	在Java中进行同步很容易，因为所有对象都有与他们自身关联的隐式的监视器。为了进入对象的监视器，只需要调用synchronized关键字修饰过
的方法。当某个线程进入同步方法中，调用同一实例的该同步方法(或任何其他同步方法)的所有其他线程都必须等待。为了退出监视器并将对象的控制权
交给下一个等待线程，监视器的拥有者只需要简单的从同步方法中返回。

	下面介绍一个应当使用但是没有使用同步的例子：
	
```
	//This program is not synchronized
	 
	 class Callme
	 {
		void call(String msg)
		{
			System.out.println("["+msg);
			
			try
			{
				Thread.sleep(1000);
			}
			catch(InterruptedException e)
			{
				System.out.println("Interrupted");
			}
			
			System.out.println("]")
		}
	 }
	 
	 class Caller implements Runnable
	 {
		String msg;
		Callme target;
		Thread t;
		
		Caller(Callme targ,String s)
		{
			target = targ;
			msg = s;
			t = new Thread(this);
			t.start();
		}
		
		void run()
		{
			target.call(msg);
		}
	 }
	 
	 class Synch
	 {
		public static void main(String[] args)
		{
			Callme target = new Callme();
			
			Caller th1 = new Caller(target,"Hello");
			Caller th2 = new Caller(target,"Java");
			Caller th3 = new Caller(target,"Android");
			
			//waiting for threads to end
			try
			{
				th1.t.join();
				th2.t.join();
				th3.t.join();
			}
			catch(InterruptedException e)
			{
				System.out.println("Interrupted");
			}
		}
	 }
```
	 下面该程序生成的输出：
	 Hello[Java[Android]
	 ]
	 ]
	 
	 可以看出，通过调用sleep()方法，call()方法允许执行切换到另一个线程，这会导致混合输出3个消息字符串。在这个程序中，没有采取什么方法以阻止3个线程在相同的时间调用同一对象的同一方法，这就是所谓的竞态条件(race condition),因为3个线程相互竞争以完成方法。这个例子中使用了sleep()方法，使得效果可以重复并且十分明显。在大多数情况下，竞态条件会更加微妙并且不可预测，因为不能确定何时会发生线程上下文切换。这会造成程序在某一次运行正确，而在下一次运行错误。

	为了修复前面的程序，必须按顺序调用call()方法。也就是说，必须限制每次只能由一个线程调用call()方法。为此，只需要简单的在call方法定义的前面添加关键字synchronized，如下所示：

```
	class Callme
	{
		synchronized void call(String msg)
		{}
	}
```
	
	当一个线程使用call()方法时，这会阻止其他线程进入该方法。将synchronized关键字添加到call()方法之后，程序输出如下：
	[Hello]
	[Java]
	[Android]
	
	在多线程情况下，如果有一个或一组方法来操作对象的内部状态(对象中的数据)，那么每次都应当使用synchronized关键字，以保证状态不会进入竞态条件。
请记住，一旦线程进入了一个实例的同步方法，所有其他线程就都不能再进入相同实例的任何同步方法。但是，仍然可以继续调用同一实例的非同步部分。

**11.7.2 synchronized语句**

	虽然在类中创建同步方法是一种比较容易并且行之有效的实现同步的方式，但并不是在所有情况下都可以使用这种方式。为了理解其中的原因，我们分析下面的内容。假设某个类没有针对多线程访问进行设计，即类没有使用同步方法，而又希望同步对类的访问。进一步讲，类不是由你创建的，而是由第三方创建的，并且你不能访问类的源代码。因此，不能为类中的合适方法添加synchronized修饰符。如何同步访问这种类的对象呢？幸运的是，这个问题的解决方案很容易：可以简单的将对这种类定义的方法的调用放到synchronized代码块中。
	
	下面是synchronized语句的一般形式：
	
```
	synchronized(objRef)
	{
		//statements to be synchronized
	}
```
	
	其中，objRef是对被同步对象的引用。synchronized代码块确保对objRef对象的成员方法的调用，只会在当前线程成功进入objRef的监视器之后发生。
	
	下面是前面例子的另一个版本，该版本在run()方法中使用synchronized代码块：
	
```
	//This program uses a synchronized block
	
	class Callme
	 {
		void call(String msg)
		{
			System.out.println("["+msg);
			
			try
			{
				Thread.sleep(1000);
			}
			catch(InterruptedException e)
			{
				System.out.println("Interrupted");
			}
			
			System.out.println("]")
		}
	 }
	 
	 class Caller implements Runnable
	 {
		String msg;
		Callme target;
		Thread t;
		
		Caller(Callme targ,String s)
		{
			target = targ;
			msg = s;
			t = new Thread(this);
			t.start();
		}
		
		void run()
		{
			//uses synchronized block
			synchronized(target)
			{
				target.call(msg);
			}
		}
	 }
	 
	 class Synch
	 {
		public static void main(String[] args)
		{
			Callme target = new Callme();
			
			Caller th1 = new Caller(target,"Hello");
			Caller th2 = new Caller(target,"Java");
			Caller th3 = new Caller(target,"Android");
			
			//waiting for threads to end
			try
			{
				th1.t.join();
				th2.t.join();
				th3.t.join();
			}
			catch(InterruptedException e)
			{
				System.out.println("Interrupted");
			}
		}
	 }
```
	 
	 在此，没有使用synchronized修饰call()方法。反而在Caller类的run()方法中使用了synchronized语句，这会使该版本的输出和前面版本的相同。
	 
	 
**11.8 线程间通信**

	这里的线程间通信其实操作系统中所讲的同步，同步首先互斥。例如，生产者和消费者问题。
	
	Java通过提供wait()、notify()以及notifyAll()方法，提供了一种巧妙的进程间通信机制，这些方法在Object中是作为final方法实现的，因此所有类都具有这些方法。所有这3个方法都只能在同步上下文中调用：
	(1)wait()方法通知调用线程放弃监视器并进入休眠，直到其他一些线程进入同一个监视器并调用notify()方法或notifyAll()方法。 
	(2)notify()方法唤醒调用相同对象的wait()方法的线程。
	(3)notifyAll()方法唤醒调用相同对象的wait()方法的所有线程，其中的一个线程将得到访问授权。
	
	这些方法都是在Object类中定义的，如下所示：
	
	final void wait() throws InterruptedException
	final void notify()
	final void notifyAll()
	
	wait()方法还有梁歪一种形式，允许指定等待的时间间隔。
	
	在通过例子演示线程通信之前，还有重要的一点需要指出。尽管在正常情况下，wait()方法会等待直到调用notify()或notifyAll()方法，但是还有一种几率很小却可能发生的情况，等待线程由于假唤醒(spurious wakeup)而被唤醒。对于这种情况，等待线程也会被唤醒。然而却没有调用notify()或notifyAll()方法(本质上，线程在没有什么明显理由的情况下就被恢复了)。因为存在这种极小却可能发生的情况，Orical推荐应当在一个检查线程等待条件的循环中调用wait()方法。

	首先分析下面的示例程序，该例以不正确的方式实现了一个简单形式的生产者/消费者问题。该例包含4个类：类Q是试图同步的队列；类Producer是产生队列条目的线程对象；类Customer是使用队列条目的线程对象；类PC是一个小类型，用于创建类Q、Producer和Customer的实例。

```
	class Q
	{
		int n;
		
		synchronized int get()
		{
			System.out.println("Got: "+n);
			return n;
		}
		
		synchronized void put(int n)
		{
			this.n = n;
			System.out.println("Put: "+n);
		}
	}
	
	class Producter implements Runnable
	{
		Q q;
		
		Producter(Q q)
		{
			this.q = q;
			new Thread(this,"Producter").start();
		}
		
		public void run()
		{
			int i = 0;
			
			while(true)
			{
				q.put(++i);
			}
		}
	}
	
	class Consumer implements Runnable
	{
		Q q;
		
		Consumer(Q q)
		{
			this.q = q;
			new Thread(this,"Consumer").start();
		}
		
		public void run()
		{
			while(true)
			{
				q.get();
			}
		}
	}
	
	class PC
	{
		public static void main(String[] args)
		{
			Q q = new Q();
			new Producter(q);
			new Consumer(q);
			
			System.out.println("Press Contrlo-C to stop.");
		}
	}
```
	
	输出如下(根据处理器的速度和加载任务，实际输出可能不同)：
	
	put: 1 
	get: 1 
	get: 1 
	get: 1 
	get: 1 
	get: 1 
	put: 2 
	put: 3
	put: 4
	put: 5 
	put: 6
	put: 7
	get: 7
	可以看出，生产者在将1放入队列之后，消费者开始运行，并且连续5次获得相同的数值1.然后，生产者恢复执行，并产生数值2到7，而不让消费者有机会使用它们。
	
	使用Java编写这个程序的正确方式是使用wain()和notify()方法在两个方向上发信号，如下所示：
	
```
	class Q
	{
		int n;
		//刚开始valueSet == 0,表示队列为空，没有数据。生产者可以向队列中放入数据，将valueSet = true,表示队列中存入了数据，消费者可以从
		//队列中取出数据。取出数据后valueSet = 0，生产者可以放入数据，依次循环。
		boolean valueSet = false;
		
		synchronized int get()
		{
			while(!valueSet)
			{
				try
				{
					wait();
				}
				catch(InterruptedException)
				{
					System.out.println("InterruptedException caught.");
				}
			}
			
			System.out.println("Got: "+n);
			valueSet = false;
			notify();
			return n;
		}
		
		synchronized void put(int n)
		{
			while(valueSet)
			{
				try
				{
					wait();
				}
				catch(InterruptedException e)
				{
					System.out.println("InterruptedException caught.");
				}
			}
			
			this.n = n;
			valueSet = true;
			System.out.println("Put: "+n);
			notify();
		}
	}
	
	class Producter implements Runnable
	{
		Q q;
		
		Producter(Q q)
		{
			this.q = q;
			new Thread(this,"Producter").start();
		}
		
		public void run()
		{
			int i = 0;
			
			while(true)
			{
				q.put(++i);
			}
		}
	}
	
	class Consumer implements Runnable
	{
		Q q;
		
		Consumer(Q q)
		{
			this.q = q;
			new Thread(this,"Consumer").start();
		}
		
		public void run()
		{
			while(true)
			{
				q.get();
			}
		}
	}
	
	class PC
	{
		public static void main(String[] args)
		{
			Q q = new Q();
			new Producter(q);
			new Consumer(q);
			
			System.out.println("Press Contrlo-C to stop.");
		}
	}
```
	
	输出如下：
	put: 1
	get: 1 
	put: 2 
	get: 2 
	put: 3 
	get: 3 
	put: 4 
	get: 4 
	put: 5 
	get: 5
	

死锁：

	例如，当两个线程循环依赖一对同步对象时，会发生死锁现象。
	假设一个线程进入对象X的监视器，另一个线程进入对象Y的监视器。如果X中的线程试图调用对象Y中的任何同步方法，同时Y中的线程也试图调用X中的任何同步方法，则X中的线程和Y中的线程同时被阻塞，两个线程则会永远等待下去。

	例如： 
```
	class A 
	{
		synchronized void foo(B b)
		{
			String name = Thread.currentThread.getName();
			
			System.out.println(name+" entered A.foo");
			
			try
			{
				Thread.sleep(1000);
			}
			catch(InterruptedException e)
			{
				System.out.println("A interrupted");
			}
			
			System.out.println(name+" trying to call B.last()");
			b.last();
		}
		
		synchronized void last()
		{
			System.out.println("Inside A.last");
		}
	}
	
	class B 
	{
		synchronized void bar(A a)
		{
			String name = Thread.currentThread().getName();
			System.out.println(name+" entered B.bar");
			
			try
			{
				Thread.sleep(1000);
			}
			catch(InterruptedException e)
			{
				System.out.println("B Interrupted");
			}
			
			System.out.println(name+" trying to call A.last()");
			a.last();
		}
		
		synchronized void last()
		{
			System.out.println("Inside A.last");
		}
	}
	
	class Deadlock implements Runnable
	{
		A a = new A();
		B b = new B();
		
		Deadlock()
		{
			Thread.currentThread().setName("MainThread");
			Thread t = new Thread(this,"RacingThread");
			t.start();
			
			a.foo(b);
			System.out.println("Back in main thread");
		}
		
		public void run()
		{
			b.bar(a);
			System.out.println("Back in other thread");
		}
		
		public static void main(String[] args)
		{
			new Deadlock();
		}
	}
```
	
	当运行这个程序，会看到如下输出：
	MainThread entered A.foo
	RacingThread entered B.bar
	MainThread trying to call B.last()
	RacingThread trying to call A.last()
	
	可以看出程序出现了死锁线程。当等待a的监视器时，RacingThread拥有b的监视器。同时，MainThread拥有a,并且在等待获取b。这个程序永远不会结束。正如该程序所演示的那样，如果多线程程序偶尔被锁住，那么首先应当检查是否是由于死锁造成的。

**11.9 挂起、恢复与停止线程**

	有时挂起线程的执行是有用的。例如，可以使用单独的线程显示一天的时间。如果用户不想要时钟，那么可以挂起时钟线程。无论是什么情况，挂起线程都是一件简单的事情。线程一旦挂起，重新启动线程也很简单。

	Java早起版本(例如Java 1.0)和现代版本(例如Java 2.0开始)提供的用来挂起、恢复以及停止线程的机制不同。在Java 2以前，程序使用Thread类定义的suspend()、resume()和stop()方法，暂停、重新启动和停止线程的执行。

	虽然这些方法对于管理线程执行看起来是一种合理并且方便的方式，但是在新的Java程序中不能使用它们。下面是其中的原因。在几年前Java 2 不推荐使用Thread类的suspend()方法，因为suspend()方法有时会导致严重的系统故障。假定为关键数据结构加锁，如果这是线程被挂起，那么这些锁将无法释放。其他可能等待这些资源的线程会被死锁。

	方法resume()也不推荐使用。虽然不会造成问题，但是如果不使用suspend()方法，就不能使用resume()方法，它们是配对使用的。
	
	对于Thread类的stop()方法，Java 2也反对使用，因为这个方法也会造成严重的系统故障。假定线程正在向关键的重要数据结构中写入数据，并且只完成了部分发生变化的数据。如果这是停止线程，那么数据结构可能会处于损坏状态。问题是：stop()会导致释放调用线程的所有锁。因此，另一正在等待相同锁的线程可能会使用这些以损坏的数据。

	在现代Java版本中，线程必须被设计为run()方法周期性的进行检查，以确定是否应当挂起、恢复或停止线程自身的执行。通常这是通过建立用来标志线程执行状态的变量完成的。这要这个标志变量被设置为"运行"，run()方法就必须让线程继续执行。如果标志变量被设置为"挂起",线程就必须暂停。如果标志变量被设置为"停止"，线程就必须停止。当然，编写这种代码的方式有很多，但是对于所有程序中心主题是相同的。

	下面例子演示了如何使用继承自Object的wait()和notify()方法控制线程的执行。下面分析这个程序中的操作。
	NewThread类包含布尔型实例变量suspendFlag，该变量用于控制线程的执行，构造函数将该变量初始化为false。方法run()包含检查suspendFlag变量的synchronized代码块。如果该变量为true,就调用wait()方法，挂起线程的执行。mysuspend()方法将suspendFlag变量设置为true。myresume()方法将suspendFlag 设置为false，并调用notify()方法以唤醒线程。最后，对main()方法进行修改以调用myspend()和myresume()方法。 

```
	class NewThread implements Runnable
	{
		String name;
		Thread t;
		boolean suspendFlag;
		
		NewThread(String threadName)
		{
			name = threadName;
			t = new Thread(this,name);
			System.out.println("NewThread: "+t);
			suspendFlag = false;
			t.start();
		}
		
		public void run()
		{
			try
			{
				for(int i = 15; i > 0; i--)
				{
					System.out.println(name+": "+i);
					Thread.sleep(200);
					
					synchronized(this)
					{
						while(suspendFlag)
						{
							wait();
						}
					}
				}
			}
			catch(InterruptedException e)
			{
				System.out.println(name+" interrupted");
			}
			
			System.out.println(name+" exiting");
		}
		
		synchronized void mysuspend()
		{
			suspendFlag = true;
		}
		
		synchronized void myresume()
		{
			suspendFlag = false;
			notify();
		}
	}
	
	class SuspendResume
	{
		public static void main(String[] args)
		{
			NewThread ob1 = new NewThread("One");
			NewThread ob2 = new NewThread("Two");
			
			try
			{
				Thread.sleep(1000);
				ob1.mysuspend();
				System.out.println("Suspending thread One");
				
				Thread.sleep(1000);
				ob1.myresume();
				System.out.println("Resume thread One");
				
				Thread.sleep(1000);
				ob2.mysuspend();
				System.out.println("Suspending thread Two");
				
				Thread.sleep(1000);
				ob2.myresume();
				System.out.println("Resume thread Two");
			}
			catch(InterruptedException e)
			{
				System.out.println("Main Thread Interrupted");
			}
			
			try
			{
				System.out.println("Waiting for threads to finish");
				ob1.t.join();
				ob2.t.join();
			}
			catch(InterruptedException e)
			{
				System.out.println("Main thread exiting");
			}
			
			System.out.println("Main thread exiting");
		}
	}
```

	后面会看到更多使用现代线程控制机制的例子。尽管这种机制没有旧机制那么"清晰",但是不管怎样，这是确保不会发生运行错误所需要做的。
对于所有新代码，必须使用这种方式。

**11.10 获取线程的状态** 

	线程可以处于许多不同的状态。可以调用Thread类定义的getState()方法来获取线程的当前状态，该方法如下所示：
	
	Thread.State getState()
	
	该方法返回Thread.State类型的值，指示在调用该方法时线程所处的状态。State是Thread类定义的枚举类型。

出自：《Java 8编程参考官方教程(第9版)》
	
	

