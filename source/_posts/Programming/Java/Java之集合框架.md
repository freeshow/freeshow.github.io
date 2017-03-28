---
title: Java之集合框架
date: 2016-07-23 23:03:47
tags: Java
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

**18.1 集合概述**

1.集合框架在设计上需要满足几个目标。
	首先，框架必须是高性能的。基本集合(动态数组、链表、树以及哈希表)的实现是高效的。很少需要手动编写这些"数据 引擎"中的某个。
	其次，框架必须允许不同类型的集合以类似的方式进行工作，并且具有高度的互操作性。
	再次，扩展或改造集合必须易于实现。
	最后，必须添加可以将标准数组继承到集合框架中的机制。
	
2.算法是集合机制中的另一个重要部分。算法操作集合，并且被定义为Collections类中的静态方法。因此，所有集合都可以使用它们。每个集合都不需要实现特定于自己的版本，所发为操作集合提供了标准的方式。

3.于集合密切相关的另个一内容是Iterator接口。迭代器为访问集合中的元素提供了通用、标准的方式，每次访问一个元素。因此，迭代器提供了枚举集合内容的一种方式。因为每个集合都提供了迭代器，所以可以通过Iterator定义的方法访问所有集合类的元素。因此，对循环遍历集合的代码进行很小的修改，就可以将其用于循环遍历列表。

4.JDK 8添加了另一种类型的迭代器，叫做spliterator.简单的说，spliterator就是为并行迭代提供支持的迭代器。支持
spliterator的接口有Spliterator和支持基本类型的几个嵌入式接口。JDK 8还添加了用于基本类型的迭代器接口，例如
PrimitiveIterator和 PrimitiveIterator.OfDouble。

**18.3 集合接口**

	集合框架定义了一些核心接口，核心接口决定了集合类的本质特性。换句话说，具体类只是提供了标准接口的不同实现。
	
**1.Collection接口**

	Collection接口是构建集合框架的基础，因为定义集合的所有类都必须实现该接口。
	Collection接口是泛型接口，其声明如下：
	
	interface Collection<E>
	
	其中，E指定了集合将要存储的对象的类型。Collection扩展了Iterable接口，这意味着所有集合都可以使用for-each风格的for循环进行遍历(回想一下，只有实现了Iterable接口的类才能通过for循环进行遍历)。 
	
	Collection声明了所有集合都将拥有的核心方法。
	
	通过调用add()方法可以将对象添加到集合中。
	通过调用addAll()方法，可以将一个集合的所有内容添加到另一个集合中。
	
	通过使用remove()方法可以移除一个对象。
	为了移除一组对象，需要调用removeAll()方法。 
	通过调用retainAll()方法，可以移除除了指定元素之外的所有元素。
	要想移除满足某些条件的元素，可以使用removeIf()方法(Predicate是JDK 8新增的一个函数式接口)。 
	为了清空集合，可以调用clear()方法。 
	
	toArray()方法返回一个数组，其中包含调用集合中存储的元素。
	
	通过调用equals()方法可以比较两个集合的相等性。"相等"的精确含义根据集合的不同可以有所区别。
	
	另一个重要的方法是iterator()，它返回集合的一个迭代器。新的soliterator()方法返回集合的一个spliterator。当操作集合时会频繁用到迭代器。最后，stream()和parallelStream()方法返回使用集合作为元素来源的流(第29章将详细讨论新的Stream接口)。 
	
**2. List接口**

	List接口扩展了Collection，并且声明了用来存储一连串元素的集合的行为。在列表中可以使用从0开始的索引，通过元素的位置插入或访问元素。列表可以包含重复的元素。List是泛型接口，其声明如下：
	
	interface List<E>
	
	其中，E指定了将存储与列表中的对象的类型。
	
	除了Collection定义的方法外，List还定义了自己的一些方法。
	
	为了获得存储在特定位置的元素，使用对象的索引调用get()方法。为了给列表中的元素赋值，可以调用set()方法，并指定将要修改的对象的索引。为了查找对象的索引，可以使用indexOf()或laseIndexOf()方法。 
	
	通过调用subList()方法，指定子列表的开始索引和结束索引，可以获得列表的子列表。
	List定义的sort()方法是排序列表的一种方法。
	
**3. Set接口**

	Set接口定义了组(set)。它扩展了Collection接口，并且声明了集合中不允许有重复元素的组行为。所以，如果为组添加重复的元素，add()方法就会返回false。Set接口没有定义自己的其他方法。Set是泛型接口，其声明如下：
	
	interface Set<E>
	
	其中，E指定了组将包含的对象的类型。
	
**4. SortedSet接口**

	SortedSet接口扩展了Set接口，并且声明了以升序进行排序的组行为。SortedSet是泛型接口，其声明如下：
	
	interface SortedSet<E>
	
**5. NavigableSet接口**

	NavigableSet接口扩展了SortedSet接口，并且该接口声明了支持基于最接近匹配原则检索元素的集合行为。
	NavigableSet是泛型接口，其声明如下：
	
	interface NavigableSet<E>
	
**6. Queue接口**

	Queue接口扩展了Collection接口，并且声明了队列的行为，队列通常是先进先出的列表。但是，还有基于其他准则的队列类型。Queue是泛型接口，其声明如下：
	
	interface Queue<E>
	
**7. Deque接口**

	Deque接口扩展了Queue接口，并声明了双端队列的行为。双端队列既可以想标准队列那样先进先出，也可以像堆栈那样后进先出。Deque是泛型接口，其声明如下：
	
	interface Deque<E> 
	
	注意Deque接口提供了push()和pop()方法。这些方法使得Deque接口的功能与堆栈类似。
	
**18.4 集合类** 

	现在已经熟悉了集合接口，下面开始分析实现它们的标准类。其中的一些类提供了可以使用的完整实现。其他一些类是抽象的，它们提供了可以作为创建具体集合开始点的大体实现。作为一般规则，集合类不是同步的，但是在后面章节将会看到，可以获得它们的同步版本。
	
	注意：
	除了集合类之外，还有一些遗留的类，例如，Vector、Stack以及Hashtable，也进行了重新设计以支持集合。
	
**1. ArrayList类**

	ArrayList类扩展了AbstractList类并实现了List接口。ArrayList是泛型类，其声明如下：
	
	class ArrayList<E>
	
	在Java中，标准数组的长度是固定的。ArrayList类支持能够按需增长的动态数组。ArrayList就是元素为对象引用的长度可变的数组。
	
	注意：遗留类Vector也支持动态数组。
	
	ArrayList具有如下所示的构造函数：
	
	ArrayList()
	ArrayList(Collection<? extends E> c)
	ArrayList(int capacity)
	
	第1个构造函数构建了一个空的数组列表。
	第2个构造函数构建了一个数组列表，使用集合c的元素进初始化。
	第3个构造函数构建了一个初始容量为capacity的数组。
	容量是用于存储元素的数组的大小，当数组中的元素等于容量时，当在向数组列表中添加元素时，容量会自动增长。
	
	尽管存储对象时，ArrayList对象的容量会自动增长，但是可以调用ensureCapacity()方法以手动增长ArrayList对象的容量。如果事先知道将在集合中存储的元素比当前保存的元素多很多，你可能希望这么做。在开始时，一次性增加容量，从而避免以后多次重新分配内存。因为重新分配内存很耗时，阻止不必要的内存分配次数可以提高性能。
	
	ensureCapacity()方法的签名如下：
	
	void ensureCapacity(int cap)
	
	其中，cap指定集合新的最小容量。
	
	相反，如果希望减小ArrayList对象数组的大小，进而使其大小精确的等于当前容纳的元素的数量，可以调用trimToSize()方法，该方法如下所示：
	
	void trimToSize()
	
**2.	LinkedList类**

	LinkedList类扩展了AbstractSequentialList类，实现了List、Deque以及Queue接口，并且它还提供了一种链表数据结构。
	LinkedList是泛型类，其声明如下：
	
	class LinkedList<E>
	
	LinkedList具有两个构造函数，如下所示：
	
	LinkedList()
	LinkedList(Collection<? extends E> c)
	
**3.	HashSet类**

	HashSet类扩展了AbstractSet类并实现了Set接口，该类用于创建使用哈希表存储元素的集合。HashSet是泛型类，其声明如下：
	
	class HashSet<E>
	
	哈希表使用称之为散列的机制存储信息。在散列机制中，键的信息用于确定唯一的值，称为哈希码。然后将哈希码用作索引，在索引位置存储与键关联的数据。将键转换为哈希码是自动执行的————你永远不会看到哈希码本身。此外，你的代码不能直接索引哈希表。散列机制的优势是add()、contains()、remove()以及size()方法的执行时间保持不变，即使是对于比较大的数组也是如此。
	
	HashSet类定义了一下构造函数：
	
	HashSet()
	HashSet(Collection<? extends E> c)
	HashSet(int capacity)
	HashSet(int capacity,float fillRatio)
	
	HashSet不能保证元素的顺序，注意这一点很重要，因为散列处理的过程通常不创建有序的组。如果需要有序的进行存储，那么需要另外一个组，TreeSet是一个比较好的选择。
	
	下面是演示了HashSet的一个例子：
	
```
	import java.util.*;
	
	class HashSetDemo
	{
		public static void main(String[] args)
		{
			HashSet<String> hs = new HashSet<String>();
			
			hs.add("Beta");
			hs.add("Alpha");
			ha.add("Eta");
			
			System.out.println(hs);
		}
	}
```
	
	输出：
	[Eta,Beta,Alpha]
	
	正如前面所解释的，元素不是按有序的顺序存储的，精确的输出可能不同。
	
**4.	LinkedHashSet类**

	LinkHashSet类扩展了HashSet类，它没有添加自己的方法。LinkedHashSet是泛型类，其声明如下：
	
	class LinkedHashSet<E
	
	LinkedHashSet维护组中条目的一个链表。链表中条目的顺序也就是插入它们的顺序，这使得可以按照插入顺序迭代集合。换句话说，当使用迭代器遍历LinkedHashSet时，元素将以插入它们的顺序返回。
	
**5.	TreeSet类**

	TreeSet扩展了AbstractSet类并实现了NavigableSet接口，用于创建使用树进行存储的组。对象以升序存储，访问和检索速度相当快，这使得存储大量的、必须能够快速查找到的有序信息来说，TreeSet是极佳选择。
	
	TreeSet是泛型类，其声明如下：
	
	class TreeSet<T>
	
	TreeSet具有如下构造函数：
	
	TreeSet()
	TreeSet(Collection<? extends E> c)
	TreeSet(Comparator<? super E> comp)
	TreeSet(SortedSet<E> ss)
	
	第1种形式构建一个空树，将按照元素的自然顺序以升序进行存储。
	第2种形式构建一个包含集合c中元素的树。
	第3种形式构建了一个空树，将按照comp指定的比较器进行存储(比较器将在本章后面描述)
	第4种形式构建一个包含ss中元素的树。
	
	下面演示TreeSet类的一个例子：
	
```
	class TreeSetDemo
	{
		public static void main(String[] args)
		{
			TreeSet<String> ts = new TreeSet<String>();
			
			ts.add("C");
			ts.add("A");
			ts.add("B");
			ts.add("E");
			ts.add("F");
			ts.add("D");
			
			System.out.println(ts);
		}
	}
```
	
	该程序输出：
	[A,B,C,D,E,F]
	
**6.	PriorityQueue类**

	PriorityQueue扩展了AbstractQueue类并实现了Queue接口，用于创建根据队列的比较器来判定优先次序的队列。
	PriorityQueue是泛型类，其声明如下：
	
	class PriorityQueue<E>
	
	PriorityQueue是动态的、按需增长的。
	
	PriorityQueue定义了一下7个构造函数：
	
	PriorityQueue()
	PriorityQueue(int capacity)
	PriorityQueue(Comparator<? super E> comp) (JDK 8新增)
	PriorityQueue(int capacity,Comparator<? super E> comp)
	PriorityQueue(Collection<? extends E> c)
	PriorityQueue(PriorityQueue<? extends E> c)
	PriorityQueue(SortedSet<? extends E> c)
	
	第1个构造函数构建一个空队列，其实容量为11.
	第2个构造函数构建一个具有指定初始容量的的队列。
	第3个构造函数指定了一个比较器。
	第4个构造函数构建具有指定容量和比较器的队列。
	最后3个构造函数创建使用参数c传递过来的集合中的元素进行初始化的队列。
	对于由这些构造函数创建的所有队列，当添加元素时，容量都会自动增长。
	
	当创建PriorityQueue对象时，如果没有指定比较器，将使用队列中存储数据类型的默认比较器。默认比较器以升序对队列进行排序。但是，通过提供定制的比较器，可以指定不同的排序模式。
	
**7.	ArrayDeque类**

	ArrayDeque扩展了AbstractCollection类并实现了Deque接口，没有添加自己的方法。ArrayDeque是泛型类，其声明如下：
	
	class ArrayDeque<E>
	
	ArrayDeque定义了以下构造函数：
	
	ArrayDeque()
	ArrayDeque(int size)
	ArrayDeque(Collection<? extends E> c)
	
	下面演示了ArrayDeque，这里使用它来创建一个堆栈：
	
```
	import java.util.*;
	
	class ArrayDequeDemo
	{
		public static void main(String[] args)
		{
			ArrayDeque<String> ad = new ArrayDeque<>();
			
			//Use an ArrayDeque like a stack
			ad.push("A");
			ad.push("B");
			ad.push("D");
			ad.push("E");
			ad.push("F");
			
			System.out.print("Poping the stack: ");
			
			while(ad.peek() != null)
			{
				System.out.print(ad.pop()+" ");
			}
			
			System.out.println();
		}
	}
```
	
	输出：
	Poping the stack: F E D B A 
	
**8.	EnumSet类**

	EnumSet扩展了AbstractSet类并实现了Set接口，专门用于枚举类型的元素。EnumSet是泛型类，其声明如下：
	
	class EnumSet<E extends EnumSet<E>>
	
	其中，E指定了元素。注意E必须扩展Enum<E>,这强制要求元素必须是指定的枚举类型。
	
**18.5 通过迭代器访问集合**

	迭代器是实现了Iterator或ListIterator接口的对象。Iterator接口允许遍历、获取或移除元素。ListIterator接口扩展了
	Iterator接口，允许双向遍历列表，并且允许修改元素。Iterator和ListIterator是泛型接口，它们的声明如下：
	
	interface Iterator<E>
	interface ListIterator<E> 
	
1. Iterator接口声明的方法：

	void forEachRemainning(Consumer<? super E> action):对于集合中为处理的元素，执行action指定的动作(JDK 8 新增)
	boolean hasNext():如果还有更多的元素，就返回true；否则返回false。
	E next():返回下一个元素，如果不存在下一个元素，就抛出NoSuchElementException异常
	void remove():移除当前元素。如果在调用next()方法之前试图调用remove()方法，就会抛出IllegalStateException异常
	
2. ListIterator接口声明的方法：

	void add(E obj):将obj插入到列表中，新插入的元素位于下一次next()方法调用返回的元素之前。
	void forEachRemainning(Consumer<? super E> action):对于集合中为处理的元素，执行action指定的动作(JDK 8 新增)
	boolean hasNext():如果存在下一个元素，就返回true；否则返回false。
	boolean hasPrevious():如果存在前一个元素，就返回true；否则返回false。
	E next():返回下一个元素，如果不存在下一个元素，就抛出NoSuchElementException异常 
	int nextIndex():返回下一个元素的索引。如果不存在下一个元素，就返回列表的大小。
	E previous():返回前一个元素。如果不存在前一个元素，就抛出NoSuchElementException异常
	int previousIndex():返回前一个元素的索引。如果不存咋前一个元素，返回-1;
	void remove():从列表中删除当前元素。如果在调用next()或previous()方法之前调用remove()方法，
						就会抛出IllegalStateException异常。
	void set(E obj):将obj赋值给当前的元素，也就是next()或previous()方法调用最后返回的元素。
	
注意： 从JDK 8开始，也可以使用Spliterator循环遍历集合。Spliterator的工作方式与Iterator不同，稍后将会详解。

**18.5.1 使用迭代器**

	为了能够通过迭代器访问集合，首先必须获得迭代器。每个集合类都提供了iterator()方法，该方法返回一个指向集合开头的迭代器。通过使用这个迭代器对象，可以访问集合中的每个元素，每次访问一个元素。通常为了使用迭代器遍历集合的内容，需要以下步骤：
	(1)通过调用集合的iterator()方法，获取指向集合开头的迭代器。
	(2)建立一个hasNext()方法调用循环。只要hasNext()返回true，就继续迭代。
	(3)在循环中，通过调用next()方法获取每一个元素。
	
	对于实现了List接口的集合可以调用listIterator()方法以获取迭代器。列表迭代器提供了向前和向后两个方向访问集合的能力，并且允许修改元素。除此之外，ListIterator与Iterator的用法类似。
	
	例如：
	
```
	import java.util.*;
	
	class IteratorDemo
	{
		//Create an array list.
		ArrayList<String> al = new ArrayList<String>();
		
		//Add elements to the array list.
		al.add("C");
		al.add("A");
		al.add("E");
		al.add("B");
		al.add("D");
		al.add("F");
		
		//Use iterator to display contents of al.
		System.out.println("Original contents of al: ");
		Iterator<String> itr = al.iterator();
		while(itr.hasNext())
		{
			String element = itr.next();
			System.out.print(element+" ");
		}
		System.out.println();
		
		//Modify objects being iterated.
		ListIterator<String> litr = al.listIterator();
		while(litr.hasNext())
		{
			String element = litr.next();
			litr.set(element+"+");
		}
		
		System.out.print("Modified contents of al: ");
		itr = al.iterator();
		while(itr.hasNext())
		{
			String element = itr.next();
			System.out.print(element+" ");
		}
		System.out.println();
		
		//Now,display the list backwards.
		System.out.print("Modified list backwards: ");
		while(litr.hasPrevious())
		{
			String element = litr.previous();
			System.out.print(element+" ");
		}
		System.out.println();
	}
```
	
	输出：
	Original contents of al: C A E B D F 
	Modified contents of al: C+ A+ E+ B+ D+ F+
	Modified list backwards: F+ D+ B+ E+ A+ C+
	
**18.5.2 使用for-each循环替代迭代器**

	如果不修改集合的内容，也不用反向获取元素，那么使用for-each风格的for循环遍历集合通常比使用迭代器更方便。请记住，可以使用for循环遍历任何实现了Iterator接口的集合对象。因为所有集合类都实现了这个接口。
	
**18.6 Spliterator**  

	JDK 8新增了一种叫做spliterator的迭代器，这种迭代器有Spliterator接口定义。Spliterator用于循环遍历元素序列，在
	这一点上与刚才介绍过的迭代器类似。但是，使用spliterator的方法与使用迭代器的不同。另外，它提供的功能远比
	Iterator或ListIterator多。可能对于Spliterator来说，最重要的一点是它支持并行迭代序列的一部分。Spliterator支持
	并行编程。然后，即使用不到并行编程，也可以使用Spliterator。这么做的一个理由是它将hasNext()和next()操作合并到
	一个方法中，从而提高效率。
	
	Spliterator是一个泛型接口，其声明如下所示：
	
	interface Spliterator<T>
	
	Spliterator接口声明的几个下面用到的方法：
	
	default void forEachRemainning(Consumer<? super T> action):将action应用到数据源中未被处理的每一个元素。
	
	boolean tryAdvance(Consumer<? super T> action)：
	在迭代中的下一个元素上执行action。如果有下一个元素，就返回true；否则返回false；
	
	将Spliterator用于基本迭代任务十分简单：只需要调用tryAdvance()方法，直到其返回false。如果要为序列中的每个元素应用相同的动作，forEachRemainning()提供了一种高效的替代方法。对于这两个方法，在每次迭代中将发生的动作都有
	Consumer对象对每个元素执行的操作定义。Consumer是一个函数式接口，向对象应用了一个动作。他是java.util.function
	中声明的一个泛型接口。Consumer仅指定了一个抽象方法accept()，如下所示：
	
	void accept(R objRef)
	
	对于tryAdvance()，每次迭代会将序列中的下一个元素传递个objRef。通常，实现Consumer最简单的方法是lambda表达式。
	
	下面的程序给出了Spliterator的一个简单示例。注意，这个程序同时演示了tryAdvance()和forEachRemainning()。另外，还要注意这些方法如何把Iterator的next()和hasNext()方法操作到一个调用中：
	
```
	import java.util.*;
	
	class SpliteratorDemo
	{
		public static void main(String[] args)
		{
			//Create an array list for doubles.
			ArrayList<Double> vals = new ArrayList<>();
			
			vals.add(1.0);
			vals.add(2.0);
			vals.add(3.0);
			vals.add(4.0);
			vals.add(5.0);
			
			//Use tryAdvance() to display contents of vals.
			System.out.println("Contents of vals: ");
			Spliterator spltitr = vals.spliterator();
			while(spltitr.tryAdvance((n) -> System.out.println(n)));
			System.out.println();
			
			//Create new list that contains square roots.
			spltitr = vals.spliterator();
			ArrayList<Double> sqrs = new ArrayList<>();
			while(spltitr.tryAdvance((n) -> sqrs.add(Math.sqrt(n))));
			
			//Use forEachRemainning() to display contents of sqrs.
			System.out.println("Contents of sqrs: ");
			spltitr = sqrs.spliterator();
			spltitr.forEachRemainning((n) -> System.out.println(n));
			System.out.println();
		}
	}
```
	
	输出：
	Contents of vals:
	1.0
	2.0
	3.0
	4.0
	5.0
	
	Contents of sqrs:
	1.0
	1.414...
	1.732...
	2.0
	2.236...
	
	虽然这个程序演示了Spliterator的原理，却没有展现其强大的能力。如前所述，在涉及并行处理的地方，Spliterator的最大优势才能体现出来。
	
**18.9 使用映射**

	映射是存储键和值之间关联关系(键值对)的对象。给定一个键，就可以找到对应的值。键和值都是对象。键必须唯一，但是只可以重复。某些映射可以接受null键和null值，其他映射则不能。
	
	关于映射需要注意的关键一点是：它们没有实现Iterator接口。这意味着不能使用for-each风格的for循环遍历映射。此外，不能为映射获取迭代器。但是，正如即将看到的，可以获取映射的集合视图，集合视图允许使用for循环和迭代器。
	
**18.9.1 映射接口**

	支持映射的接口有Map、Map.Entry、NavigableMap和SortedMap。
	
**1.	Map接口**

	Map接口将唯一键映射到值。键是以后用于检索值的对象。给定键和值，可以在Map对象中存储值；存储值以后，可以使用相应的键检索值。Map是泛型接口，其声明如下：
	
	interface Map<K,V>
	
	其中，K指定了键的类型，V指定了值的类型。
	
	映射围绕两个基本方法：get()和put()。为了将值放入映射中，使用put()方法，指定键和值。为了获取值，调用get()方法，传递键作为参数，值会被返回。
	
	尽管映射是集合框架的一部分，但映射不是集合，因为没有实现Collections接口。但是，可以获取映射的集合视图。为此，可以使用entrySet()方法。该方法返回一个包含映射中元素的Set对象。为了获取键的集合视图，使用keySet()方法；为了得到值的集合视图，使用values()方法。对于这3个集合视图，集合都是基于映射的。修改其中的一个集合会影响其他集合。集合视图是将映射集成到更大集合框架中的手段。
	
**2.	SortedMap接口**

	SortedMap接口扩展了Map接口，确保条目以键的升序保存。SortedMap是泛型接口，其声明如下：
	
	interface SortedMap<K,V>
	
**3.	NavigableMap接口**

	NavigableMap接口扩展了SortedMap接口，支持基于最接近匹配原则的条目的检索行为，即支持检索与给定的一个或多个键最相匹配的条目。NavigableMap是泛型接口，其声明如下：
	
	interface NavigableMap<K,V>
	
**4.	Map.Entry接口**

	Map.Entry接口提供了操作映射条目的功能。请记住，Map接口声明的entrySet()方法返回一个包含映射条目的Set对象。组的所有元素都是Map.Entry对象。Map.Entry是泛型接口，起声明如下：
	
	interface Map.Entry<K,V>
	
**18.9.2 映射类**

	AbstractMap:实现了Map接口的大部分
	HashMap：扩展了AbstractMap，以使用哈希表
	TreeMap：扩展了AbstractMap，以使用树结构
	LinkedHashMap：扩展了HashMap，以允许按照插入顺序进行迭代
	EnumMap：扩展了AbstractMap，以使用enum键
	WeekHashMap:扩展了AbstractMap，以使用带有弱键的哈希表
	IdentityHashMap:扩展了AbstractMap，并且当比较文档时使用引用相等性
	
	注意AbstractMap是所有具体映射实现的超类。
	
**1.	HashMap类**

	HashMap扩展了AbstractMap类并实现了Map接口。它使用哈希表存储映射，这使得即使对于比较大的集合，get()和put()方法的执行时间也保持不变。
	
	应当注意哈希映射不保证元素的顺序。所以，向哈希映射添加元素的顺序不一定是通过迭代器读取它们的顺序。
	
	下面的程序演示了HashMap，将名字映射的账户金额。注意获取和使用组视图的方式：
	
```
	import java.util.*;
	
	class HashMapDemo
	{
		public static void main(String[] args)
		{
			HashMap<String,Double> hm = new HashMap<>();
			
			hm.put("John Doe",new Double(3434.34));
			hm.put("Tom Smith",new Double(123.22));
			hm.put("Jane Baker",new Double(1378.00));
			hm.put("Tod Hall",new Double(99.22));
			hm.put("Ralph Smith",new Double(-19.08));
			
			//Get a set of the entries.
			Set<Map.Entry<String,Double>> set = hm.entrySet();
			
			//Display the set.
			for(Map.Entry<String,Double> me:set)
			{
				System.out.print(me.getKey()+";")
				System.out.println(me.getValue());
			}
			
			System.out.println();
			
			//Deposit 1000 into John Doe's account.
			double balance = hm.get("John Doe");
			hm.put("John Doe",balance+1000);
			
			System.out.println("John Doe's new balance: "+hm.get("John Doe"));
		}
	}
```
	
	输出(精确的顺序可能会有所变化)：
	Ralph Smith: -19.08 
	Tom Smith: 123.22
	John Doe: 3434.34
	Tod Hall: 99.22
	Jane Baker: 1378.0 
	
	John Doe's new balance: 4434.34
	
**2. TreeMap类**

	TreeMap扩展了AbstractMap类并实现了NavigableMap接口，该类用于创建存储在树结构中的映射。TreeMap提供了有序存储键/值对的高效手段，并支持快速检索。应当注意，与哈希映射不同，树映射确保元素以键的升序存储。
	
	下面的程序对前面的程序进行了修改，以使用TreeMap类：
	
```
	import java.util.*;
	
	class TreeMapDemo
	{
		public static void main(String[] args)
		{
			TreeMap<String,Double> tm = new TreeMap<>();
			
			tm.put("John Doe",new Double(3434.34));
			tm.put("Tom Smith",new Double(123.22));
			tm.put("Jane Baker",new Double(1378.00));
			tm.put("Tod Hall",new Double(99.22));
			tm.put("Ralph Smith",new Double(-19.08));
			
			Set<Map.Entry<String,Double>> set = tm.entrySet();
			
			for(Map.Entry<String,Double> me : set)
			{
				System.out.print(me.getKey()+":");
				System.out.println(me.getValue());
			}
			
			System.out.println();
			
			//Deposit 1000 into John Doe's account.
			double balance = tm.get("John Doe");
			tm.put("John Doe",balance+1000);
			
			System.out.println("John Doe's new balance: "+tm.get("John Doe"));
		}
	}
```
	
	输出：
	Jane Baker: 1378.0 
	John Doe: 3434.34
	Ralph Smith: -19.08 
	Tod Hall: 99.22
	Tom Smith: 123.22
		
	John Doe's new balance: 4434.34
	
	注意TreeMap对键进行排序，然而这个例子中，它们根据名(first name)而不是姓(last name)进行排序。正如稍后描述的，在创建映射时，可以通过指定比较器来改变这种行为。
	
**3.	LinkedHashMap类**

	LinkedHashMap扩展了HashMap，在映射中以插入条目的顺序维护一个条目链表，从而可以按照插入顺序迭代整个映射。
	
	也就是说，当遍历LinkedHashMap的集合视图时，将以元素的插入顺序返回元素。也可以创建按照最后访问的顺序返回元素的LinkedHashMa。
	
**18.10	比较器** 

	TreeSet和TreeMap类以有序顺序存储元素。然而，是比较器精确定义了"有序顺序"的含义。默认情况下，这些类使用Java称为"自然顺序"的方式对元素进行排序，自然顺序通常是你所期望的顺序(A在B之前，1在2之前，等等)。如果希望以不同的方式排序元素，可以在构造函组或映射时指定比较器，这样就可以精确控制在有序集合和映射中存储元素的方式。
	
	Comparator是泛型接口，其声明如下：
	
	interface Comparator<T>
	
	在JDK 8之前，Comparator接口只定义了两个方法：compare()和equals()。
	compare()方法如下所示，用于比较两个元素以进行排序：
	
	int compare(T obj1,T obj2)
	
	obj1和obj2是要进行比较的对象。在正常情况下，如果对象相等，该方法返回0；如果obj1大于obj2，返回一个正值，反之，返回一个负值。如果要进行比较的对象的类型不兼容，该方法会抛出ClassCastException异常。通过实现compare()方法，可以改变对象的排序方式。例如，为了按照相反的顺序进行排序，可以创建比较器以反转比较的结果。
	
	equals()方法如下所示，用于测试某个对象是否等于比较器：
	
	boolean equals(object obj)
	
	其中，obj是将要进行相等测试的对象。如果obj和调用对象都是比较器，并且使用相同的排序规则，那么该方法返回true；
	否则，返回false。不必重写equals()方法，并且大多数简单的比较器都不重写该方法。
	
	在JDK 8后，通过使用默认接口方法和静态接口方法，JDK 8为Comparator添加了许多新功能。下面逐一进行介绍。
	
	通过reserved()方法，可以获得一个比较器，该比较器颠倒了调用reserved()的比较器的排序：
	
	default Comparator<T> reserved()
	
	该方法返回一个颠倒的比较器。
	
	reserseOrder()是与reserve()关联的一个方法，如下所示：
	
	static <T extends Comparable<? super T>> Comparator<T> reserseOrder()
	
	它返回一个颠倒元素的自然顺序的比较器。对应的，通过调用静态的naturalOrder()方法，可以获得一个使用自然顺序的比较器。该方法声明如下所示：
	
	static <T extends Comparable<? super T>> Comparator<T> natualOrder()
	
	如果希望比较器能够处理null值，需要使用下面的nullsFirst()和nullsLast()方法：
	
	static <T> Comparator<T> nullsFirst(Comparator<? super T> comp)
	static <T> Comparator<T> nullsLast(Comparator<? super T> comp)
	
	nullsFirst()方法返回的比较器认为null比其他值小，nullLast()方法返回的比较器认为null比其他值大。对于这两个方法，如果被比较的两个值都是非null值，则comp执行比较。如果为comp赋值null，则认为所有非null值都是相等的。
	
	JDK 8添加的另一个默认方法是thenComparing()。该方法返回一个比较器，当第一次比较的结果指出被比较的结果相等时，
	返回的这个比较器将执行第二次比较。因此，可以使用该方法创建一个"根据X比较，然后根据Y比较"的序列。例如，当比较城市时，第一次可能比较城市名，第二次可能比较州名。如果城市名相同，则在比较州名。
	
	thenComparing()方法有三种形式。
	第一种形式如下所示：它允许通过传入Comparator的实例来指定第二个比较器。
	
	default Comparator<T> thenComparing(Comparator<? super T> thenByComp)
	
	其中，thenByComp指定在第一次比较返回相等后调用的比较器。
	
	thenComparing()的另外两个版本允许指定标准函数式接口Function，如下所示：
	
	default <U extends Comparator<? super U>> Comparator<T> 
		thenComparing(Function<? super T,? extends U> getKey)
		
	default <U extends Comparator<? super U>> Comparator<T> 
		thenComparing(Function<? super T,? extends U> getKey,Comparator<? super U> keyComp)
		
	在这两个版本中，getKey引用的函数用于获得下一个比较键，当第一次比较返回相等后，将使用这个比较键。后一个版本中的keyComp指定了用于比较键的比较器(U指定了键的类型)。
	

使用比较器：

	下面演示了一个自定义比较器功能的例子。该例为字符串实现了compare()方法，以与正常顺序相反的顺序进行操作。因此，这将导致树组(tree set)以相反的顺序进行存储。
	
```
	import java.util.*;
	
	class MyComp implements Comparator<String>
	{
		public int compare(String aStr,String bStr)
		{
			//Reverse the comparison
			return bStr.compareTo(aStr);
		}
	}
	
	class CompDemo
	{
		public static void main(String[] args)
		{
			TreeSet<String> ts = new TreeSet<>(new MyComp());
			
			ts.add("C");
			ts.add("A");
			ts.add("B");
			ts.add("E");
			ts.add("F");
			ts.add("D");
			
			for(String element : ts)
			{
				System.out.print(element+" ");
			}
			
			System.out.println();
		}
	}
```
	
	输出：
	F E D C B A 
	
	
	尽管上面的程序实现逆序比较器的方法完全可以接受，但是从JDK8开始，还有另外一种方法可以获得解决方案。
	即，现在可以简单的对自然顺序比较器调用reversed()方法。该方法返回一个等价的比较器，不过比较器的顺序是相反的。
	例如，在前面的程序中，可以把MyComp重写为自然顺序的比较器，如下所示：
	
```
	class MyComp implements Comparator<String>
	{
		public int compare(String aStr,String bStr)
		{
			return aStr.compareTo(bStr);
		}
	}
```
	
	然后，可以使用下面的代码段创建一个反向排序字符串元素的TreeSet:
	
	MyComp mc = new MyComp();
	
	TreeSet<String> ts = new TreeSet<>(mc.reserved());
	
	如果把这段新代码放到前面的程序中，得到的结果与原来的一样。在这个例子中，使用reversed()方法没有什么优势。但是，当需要同时创建自然顺序和反向顺序的比较器时，使用reversed()方法能够方便的获得逆序比较器，而不需要显示对其编码。
	
	从JDK 8开始，在前面的例子中，实际上没有必要创建MyComp类，因为可以很方便的使用lambda表达式作为替代。
	例如，可以彻底删除MyComp类，使用下面的代码创建字符串比较器:
	
	Comparator<String> mc = (aStr,bStr)->aStr.compareTo(bStr);
	
	另外还有一点：在这个简单的示例中，通过使用lambda表达式，可以在对TreeSet()构造函数的调用中直接指定逆序比较器，如下所示：
	
	TreeSet<String> ts = new TreeSet<>((aStr,bStr) -> bStr.compareTo(aStr));
	
	做了上述修改后，程序得以显著缩短。下面显示了最终版本：
	
```
	import java.util.*;
	
	class CompDemo2
	{
		public static void main(String[] args)
		{
			TreeSet<String> ts = new TreeSet<>((aStr,bStr) -> bStr.compareTo(aStr));
			
			ts.add("C");
			ts.add("A");
			ts.add("B");
			ts.add("E");
			ts.add("F");
			ts.add("D");
			
			for(String element : ts)
			{
				System.out.print(element+" ");
			}
			
			System.out.println();
		}
	}
```
	
	
	作为使用自定义比较器的更实际的例子，下面的程序是前面显示的存储账户余额的TreeMap程序的升级版。在前面的版本中，账户根据名字进行存储，但排序是从名开始的。下面的程序根据姓对账户进行排序。为此，使用比较器比较每个账户的姓，这会使得映射根据姓进行排序。
	
```
	import java.util.*;
	
	class TComp implements Comparator<String>
	{
		public int compare(String aStr,String bStr)
		{
			int i,j,k;
			
			i = aStr.lastIndexOf(' ');
			j = bStr.lastIndexOf(' ');
			
			k = aStr.subString(i).compareToIgnoreCase(bStr.subString(j));
			if(k == 0)	//last name match,check entire name
			{
				return aStr.compareToIgnoreCase(bStr);
			}
			else
			{
				return k;
			}
			
		}
	}
	
	class TreeMapDemo2
	{
		public static void main(String[] args)
		{
			TreeMap<String,Double> tm = new TreeMap<>(new TComp());
			
			tm.put("John Doe",new Double(3434.34));
			tm.put("Tom Smith",new Double(123.22));
			tm.put("Jane Baker",new Double(1378.00));
			tm.put("Tod Hall",new Double(99.22));
			tm.put("Ralph Smith",new Double(-19.08));
			
			Set<Map.Entry<String,Double>> set = tm.entrySet();
			
			for(Map.Entry<String,Double> me : set)
			{
				System.out.print(me.getKey()+":");
				System.out.println(me.getValue());
			}
			
			System.out.println();
			
			//Deposit 1000 into John Doe's account.
			double balance = tm.get("John Doe");
			tm.put("John Doe",balance+1000);
			
			System.out.println("John Doe's new balance: "+tm.get("John Doe"));
		}
	}
```
	
	输出：
	Jane Baker: 1378.0 
	John Doe: 3434.34
	Tod Hall: 99.22
	Ralph Smith: -19.08 	
	Tom Smith: 123.22
		
	John Doe's new balance: 4434.34
	
	比较器类TComp比较包含名和姓的两个字符串。该类首先比较姓，如果姓相同的话，在根据名进行排序。
	
	如果使用的是JDK 8或更高版本，还有另一种方法来编码前面的程序，让映射首先根据姓进行排序，然后根据名进行排序。
	这会用到thenComparing()方法。回忆一下，thenComparing()允许指定第二个比较器，当调用比较器返回相等时，就会使用第二个比较器。下面的程序演示了这种方法，它修改了前面的程序，使之使用thenComparing()方法。 
	
```
	import java.util.*;
	
	//A comparator that compares last names.
	class CompLastNames implements Comparator<String>
	{
		public int compare(String aStr,String bStr)
		{
			int i,j;
		
			i = aStr.lastIndexOf(' ');
			j = bStr.lastIndexOf(' ');
		
			return aStr.subString(i).compareToIgnoreCase(bStr.subString(j));
		}
	}
	
	//Sort by entire name when last names are equals.
	class CompThenByFirstName implements Comparator<String>
	{
		public int compare(String aStr,String bStr)
		{			
			return aStr.compareToIgnoreCase(bStr);
		}
	}
	
	class TreeMapDemo2A
	{
		public static void main(String[] args)
		{
			CompLastNames compLN = new CompLastNames();
			Comparator<String> compLastThenFirst = compLN.thenComparing(new CompThenByFirstName());
			
			TreeMap<String,Double> tm = new TreeMap<>(compLastThenFirst);
			
			tm.put("John Doe",new Double(3434.34));
			tm.put("Tom Smith",new Double(123.22));
			tm.put("Jane Baker",new Double(1378.00));
			tm.put("Tod Hall",new Double(99.22));
			tm.put("Ralph Smith",new Double(-19.08));
			
			Set<Map.Entry<String,Double>> set = tm.entrySet();
			
			for(Map.Entry<String,Double> me : set)
			{
				System.out.print(me.getKey()+":");
				System.out.println(me.getValue());
			}
			
			System.out.println();
			
			//Deposit 1000 into John Doe's account.
			double balance = tm.get("John Doe");
			tm.put("John Doe",balance+1000);
			
			System.out.println("John Doe's new balance: "+tm.get("John Doe"));
		}
	}
```
	
	这个版本产生的输入与原来的版本相同，二者唯一的区别在于完成的工作方法。
	
	最后，注意为了清晰起见，本例显示创建了两个比较器类CompLastNames和ComThenByFirstName，但是其实也可以使用
	lambda表达式。例如：
	
```
	Comparator<String> compLastName = (aStr,bStr)->
    {
        int i,j;

        i = aStr.lastIndexOf(' ');
        j = bStr.lastIndexOf(' ');

        return aStr.substring(i).compareToIgnoreCase(bStr.substring(j));
    };

    Comparator<String> compThenFirstName = (aStr,bStr)->aStr.compareToIgnoreCase(bStr);

    Comparator<String> comp = compLastName.thenComparing(compThenFirstName);

    TreeMap<String,Double> tm = new TreeMap<>(comp);
```
	
	
	
**18.11 集合算法**

	集合框架定义了一些可以应用于集合和映射的算法，这些算法被定义为Collections类中的静态方法中，可以自行查看
	Collections类的源码查看这些函数。
	
	注意有些方法用于获取各种集合的同步(线程安全的)副本，例如synchronizedList()和synchronizedSet()。作为一般规则
	，标准集合实现不是同步的。必须使用同步算法提供同步。另外一点：同步集合的迭代器必须在synchronized代码块中使用
	
	以unmodifiable开始的一组方法集返回各种不能修改的集合视图。如果希望确保对集合进行某些读取操作——但是不允许写入操作，这些方法是有用的。
	
	下面的程序演示了一些算法，创建并初始化一个链表。reverseOrder()方法返回翻转Integer对象比较结果的比较器。列表元素根据这个比较器进行排序，然后显示。接下来，调用shuffle()方法以随机化链表，然后显示链表中的最大值和最小值
	
```
	import java.util.*;
	
	class AlgorithmsDemo
	{
		public static void main(String[] args)
		{
			LinkedList<Integer> ll = new LinkedList<Integer>();
			ll.add(-8);
			ll.add(20);
			ll.add(-20);
			ll.add(8);
			
			//Create a reserve order comparator.
			Comparator<Integer> r = Collections.reserseOrder();
			
			//Sort list by using the comparator.
			Collections.sort(ll,r);
			
			System.out.print("List sorted in reverse: ");
			for(int i : ll)
			{
				System.out.print(i+" ");
			}
			
			System.out.println();
			
			//Shuffle list.
			Collections.shuffle(ll);
			
			//Display randomized list.
			System.out.print(i+" ");
			
			System.out.println();
			System.out.println("Mininum: "+Collections.min(ll));
			System.out.println("Maxinum: "+Collections.max(ll));
		}
	}
```
	
	输出：
	List sorted in reserve: 20 8 -8 -20 
	List shuffled: 20 -20 8 -8 
	Mininum: -20 
	Maxinum: 20 
	
	注意是在随机化后，才对链表进行min()和max()操作的。这两个方法的操作都不要求列表是排序过的。
	

**18.12	Arrays类**

	Arrays类提供了对数组操作有用的方法，这些方法有助于连接集合和数组。
	
	asList()方法返回基于指定数组的列表。
	static <T> List asList(T ...array)
	其中，array是包含数据的数组。
	
	binarySeach()方法使用二分搜索法查找特定数值。
	
	copyOf()方法返回数组的副本。
	
	copyOfRange()方法返回数组中某个范围的副本。
	
	fill()方法可以将某个值赋给数组中的所有元素。
	
	sort()方法用于对数组进行排序。
	
	JDK 8为Arrays类添加了一些新的方法，其中最重要的可能是parallelSort()方法，因为该方法按升序对数组的各部分
	进行并行排序，然后合并结果。这种方法可以显著加速排序时间。
	
	通过包含spliterator()方法，JDK 8为Arrays类添加了对spliterator的支持。该方法有两种基本形似。
	第一种版本返回整个数组的spliterator,如下所示： 
	
	static<T> Spliterator spliterator(T array[])
	
	这里，array是spliterator将循环遍历的数组。第二种版本的spliterator()方法允许指定希望进行迭代的数组范围。
	
	从JDK 8开始，通过包含stream()方法，Arrays类支持新的Stream接口。stream()方法有两个版本，下面显示了第一个
	版本：
	
	static<T> Stream stream(T array[])
	
	这里，array是流将引用的数组。第二种版本的stream()方法允许指定数组内的一个范围。
	
	除了刚才讨论的方法，JDK 8还添加了另外3个新方法。其中两个彼此相关：setAll()和parallelSetAll()。这两个方法
	都是为所有元素赋值，但是parallelSetAll()以并行方式工作。
	
	最后JDK 8还为Arrays类添加了一个有意思的方法：parallelPrefix().该方法对数组进行修改，使每个元素都包含对其
	前面所有元素应用某个操作的累积结果。
	
	
	下面演示了如何使用Arra类的一些方法：
	
```
	import java.util.*;
	
	class ArraysDemo
	{
		public static void main(String[] args)
		{
			//Allocate and initialize array.
			int[] array = new int[10];
			for(int i=0; i<10; i++)
			{
				array[i] = -3*i;
			}
			
			//Display,sort,and display the array.
			System.out.print("Original contents: ");
			display(array);
			Arrays.sort(array);
			System.out.print("Sorted: ");
			display(array);
			
			//Fill and display the array.
			Arrays.fill(array,2,6,-1);
			System.out.print("After fill(): ");
			display(array);
			
			//Sorted and display the array.
			Arrays.sort(array);
			System.out.println("After sorting again: ");
			display(array);
			
			//Binary seach for -9;
			System.out.print("The value -9 is at location: ");
			int index = Arrays.binarySeach(array,-9);
			System.out.println(index);
		}
		
		static void display(int[] array)
		{
			for(int i : array)
			{
				System.out.print(i+" ");
			}
			
			System.out.println();
		}
	}
```

	输出：
	Original contents: 0 -3 -6 -9 -12 -15 -18 -21 -24 -27
	Sorted: -27 -24 -21 -18 -15 -12 -9 -6 -3 -0 
	After fill(): -27 -24 -1 -1 -1 -1 -9 -6 -3 0
	After sorting again: -27 -24 -9 -6 -3 -1 -1 -1 -1 0
	The value -9 is at location: 2

	
**18.13	遗留的类和接口**

	在本章开头解释过，早期版本的java.util包没有包含集合框架，而是定义了几个类和一个接口，用来提供存储对象的专业方法。
	
	本章介绍的所有现代集合类都不是同步的，但是所有遗留类都是同步的，有些情况这一差别很重要。当然，通过使用
	Collections提供的算法，可以很容易的同步集合。
	
	java.util定义的遗留类如下所示：
	Dictionary	Hashtable	Properies	Stack	Vector 
	还有遗留接口Enumeration.
	
	此处，就介绍遗留类和接口的介绍了。

出自：《Java 8编程参考官方教程(第9版)》
	
	