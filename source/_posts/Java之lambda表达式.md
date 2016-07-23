---
title: Java之lambda表达式
date: 2016-07-23 23:06:29
tags: Java
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

**15.1 lambda表达式简介**

	对于理解lambda表达式的Java实现，有两个结果十分关键。第一个就是lambda表达式自身，第二个是函数式接口。
	
	lambda表达式本质上就是一个匿名(即未命名的方法)。但是这个方法不能独立执行的，而是用于实现由函
	数式接口定义的一个方法。因此，lambda表达式会导致产生一个匿名类。lambda表达式也常被称为闭包。
	
	函数式接口是仅包含一个抽象方法的接口。一般来说，这个方法指明了接口的目标用途。因此，函数式接口通常表示单个动作。例如，标准接口Runnable是一个函数式接口，因为它只定义了一个方法run().因此，run()定义了Runnable的动作。此外，函数式接口定义了lambda表达式的目标类型。特别注意：lambda表达式只能用于其目标类型已经被指定的上下文中。另外，函数式接口有时候被称为SAM类型，意思是单抽象方法(Single Abstract Method)。 
	
	注意：
	函数式接口可以定义Object定义的任何公有方法，例如equals()，而不影响其作为"函数式接口"的状态。Object的公有方法被视为函数式接口的隐式成员，因为函数式接口的实例会默认自动实现它们。
	
**15.1.1 lambda表达式的基础知识**

	lambda表达式在Java语言中引入了一个新的语法元素和操作符。这个操作符是->,有时候被称为lambda操作符或箭头操作符。它将lambda表达式分成两部分。左侧指定了lambda表达式所需要的所有参数(如果不需要参数，则使用空的参数列表)。右侧指定了lambda体，即lambda表达式要执行的动作。在用语言描述时，可以把->表达成"成了"或"进入"。
	
	Java定义了两种lambda体。第一种包含单独一个表达式，另一种包含一个代码块。
	
	(1)lambda体包含单独一个表达式
	
	首先，看一个最简单的lambda表达式。它的计算结果是一个常量值，如下所示：
	
	()->123.45
	
	这个lambda表达式没有形参，所以参数列表为空。它返回常量值123.45.因此，这个表达式的作用类似于下面的方法：
	
	double myMeth(){return 123.45;}
	
	当然，lambda表达式定义的方法没有名称。
	
	下面给出一个更有意思的表达式：
	
	()->Math.random()*100
	
	这个lambda表达式使用Math.random()获得一个随机数，将其乘以100，然后返回结果。这个lambda表达式也不需要参数。
	
	当lambda表达式需要参数时，需要在操作符左侧的参数列表中加以指定。下面是一个简单的例子：
	
	(n)->(n%2)==0
	
	如果参数n的值是偶数，这个lambda表达式会返回true。尽管可以显示指定参数的类型，例如本例中的n，但是通常不需要这么做，因为很多时候，参数的类型是可以推断出来的。与命名方法一样，lambda表达式可以指定需要用到的任意参数的数量。
	
**15.1.2 函数式接口** 

	函数式接口是指仅定义了一个抽象方法的接口。从JDK8开始，可以为接口声明的方法指定默认行为，即所谓的默认方法。如今，只有当没有指定默认方法实现时，接口方法才是抽象方法。因为没有指定默认实现的接口方法隐式的是抽象方法，所以没有必要使用abstract修饰符(如果愿意的话，也可以指定该修饰符)。
	
	下面是函数式接口的例子：
	
```
	interface MyNumber
	{
		double getValue();
	}
```
	
	在本例中，getValue()方法隐式的是抽象方法，并且是MyNumber定义的唯一方法。因此，MyNumber是一个函数式接口，其功能由getValue()定义。
	
	如前所述，lambda表达式不是独立执行的，而是构成了一个函数式接口定义的抽象方法的实现，该函数式接口定义了它的目标类型。结果，只有在定义了lambda表达式的目标类型的上下文中，才能使用该表达式。当把一个lambda表达式赋给一个函数式接口的引用时，就创建了这样的上下文。
	
	下面通过一个例子来说明如何在参数上下文中使用lambda表达式。首先，声明对函数式接口MyNumber的一个引用：
	
	MyNumber myNum;
	
	接下来，将一个lambda表达式赋值给该接口引用：
	
	myNum = ()->123.45
	
	当目标类型上下文中出现lambda表达式时，就会自动创建实现了函数式接口的一个类的实例(类似于匿名类)，函数式接口声明的抽象方法的行为由lambda表达式定义。当通过目标调用该方法时，就会执行lambda表达式。因此，lambda表达式成了getValue()方法的实现。因此，下面的代码将显示123.45：
	
	System.out.println(myNum.getValue());
	
	因为赋给myNum的lambda表达式返回值123.45，所以调用getValue()方法时返回的值也是123.45。
	
	为了在目标类型上下文中使用lambda表达式，抽象方法的类型和lambda表达式的类型必须兼容。例如，如果抽象方法指定了两个int类型的参数，那么lambda表达式也必须执行两个参数，其类型要么被显示指定为int类型，要么在上下文中可以被隐式的推断为int类型。总的来说，lambda表达式的参数的类型和数量必须与方法的参数兼容；返回类型必须兼容；并且lambda表达式可能抛出的异常必须能被方法接受。
	
**15.1.3 几个lambda表达式示例**

	(1)
	
```
	//函数式接口 
	interface MyNumber
	{
		double getValue();
	}
	
	class lambdaDemo
	{
		public static void main(String[] args)
		{
			MyNumber myNum;
			
			myNum = ()->123.45;
			System.out.println("A fixed value: "+myNum.getValue());
			
			myNum = ()->Math.random()*100;
			System.out.println("A random value: "+myNum.getValue());
			
			//lambda表达式的返回值类型与函数式接口中抽象函数的类型不匹配。
		//	myNum = ()->"123.03"	//error! 
		}
	}
```
	
	(2)带参数的lambda表达式：
	
```
	interface NumericTest
	{
		boolean test(int n);
	}
	
	class lambdaDemo2
	{
		public static void main(String[] args)
		{
			NumericTest isEven = (n)->(n%2)==0;
			if(isEven.test(10)) System.out.println("10 is even");
			if(!isEven.test(9)) System.out.println("9 is not even");
			
			NumericTest isNonNeg = (n)-> n>=0;
			if(isNonNeg.test(1)) System.out.println("1 is non-negative");
			if(!isNonNeg.test(-1)) System.out.println("-1 is negative");
		}
	}
```
	
	测试奇偶性的lambda表达式，如下所示：
	
	NumericTest isEven = (n)->(n%2)==0;
	
	注意，这里没有指定n的类型。相反，n的类型是从上下文中推断出来的。本例中，其类型是从NumericTest接口定义的test()方法的参数类型推断出来的，而该参数的类型是Int。在lambda表达式中，也可以显示指定参数的类型。例如，下面的写法也是合法的：
	
	NumericTest isEven = (int n)->(n%2)==0;

	如果lambda表达式只有一个参数，在操作符的左侧指定该参数时，没有必要使用括号扩住该参数的名称。例如下面的下法也是合法的：
	
	NumericTest isEven = n->(n%2)==0;

	(3)接受两个参数的lambda表达式：
	
```
	interface NumericTest2
	{
		boolean test(int n,int d);
	}
	
	class lambdaDemo3
	{
		public static void main(String[] args)
		{
			//测试一个数字是否是另一个数字的因子。
			NumericTest2 isFactor = (n,d) -> (n%d)==0;
			
			if(isFactor.test(10,2)) System.out.println("2 is a factor of 10");
			if(!isFactor.test(10,3)) System.out.println("3 is not a factor of 10");
		}
	}
```

	注意指定这种lambda表达式的方法：
	
	NumericTest2 isFactor = (n,d) -> (n%d)==0;
	
	两个参数n和d在参数列表中指定，并使用逗号分隔开。可以把这个例子推而广之。每当需要一个以上的参数时，就在lambda操作符的左侧，使用一个带括号的参数列表指定参数，参数之间使用逗号分隔开。
	
	对于lambda表达式中的多个参数，有一点十分重要：如果需要显示的声明一个参数的类型，那么必须为所有的参数声明类型。例如下面的代码是合法的：
	
	NumericTest2 isFactor = (int n,int d) -> (n%d)==0;
	
	但下面的不合法：
	
	NumericTest2 isFactor = (int n,d) -> (n%d)==0;
	
	
**15.2 块lambda表达式**

	前面例子中显示的lambda体只包含单个表达式。这种类型的lambda体被称为表达式体，具有表达式体的lambda表达式有时被称为表达式lambda。在表达式体重，操作符右侧的代码必须包含单独一个表达式。尽管表达式lambda十分有用，但是有时候会要求使用一个以上的表达式。为了处理这种情况，Java支持另外一种类型的lambda表达式，其中操作符右侧的代码可以由一个代码块构成，其中可以包含多条语句。这种类型的lambda体被称为块体。具有块体的lambda表达式有时候被称为块lambda。
	
	块lambda表达式扩展了lambda表达式内部可以处理的操作类型，因为它允许lambda体包含多条语句。例如，在lambda块中，可以声明变量、使用循环、指定if和switch语句、创建嵌套代码块等。创建块lambda很容易，只需要使用花括号包围lambda体，就像创建其他语句一样。
	
	除了允许多条语句，块lambda的使用方法与刚才讨论过的表达式lambda十分类似。但是，也有一个重要区别：在块lambda中必须显示使用return语句来返回值。必须这么做，因为块lambda体代表的不是一个单独的表达式。
	
	下面的例子是使用块lambda来计算并返回一个int类型值的阶乘：
	
```
	interface NumericFunc
	{
		inf func(int n);
	}
	
	class BlockLambdaDemo
	{
		public static void main(String[] args)
		{
			NumericFunc factorial = (n)->
			{
				int result = 1;
				
				for(int i=1; i<=n; i++)
				{
					result = i*result;
				}
				
				return result;
			};
			
			System.out.println("The factorial of 3 is "+factorial.func(3));
			System.out.println("The factorial of 5 is "+factorial.func(5));
		}
	}
```
	
	注意：
	当lambda表达式中出现return语句时，只是从lambda体返回，而不会导致包围lambda体的方法返回。
	
**15.3 泛型函数式接口**

	lambda表达式自身不能指定类型参数。因此，lambda表达式不能是泛型(当然，由于存在类型推断，所有lambda表达式都展现出了一些类似于泛型的特征)。然而，与lambda表达式关联的函数式接口可以泛型。此时，lambda表达式的目标类型部分由声明函数式接口引用时指定的参数类型决定。
	
	例如：
	
```
	interface SomeFunc<T>
	{
		T func(T t);
	}
	
	class GenericFunctionalInterfaceDemo
	{
		public static void main(String[] args)
		{
			SomeFunc<String> reverse = (str)->
			{
				int i;
				String result = "";
				
				for(i = str.length()-1; i>=0; i--)
				{
					result += str.charAt(i);
				}
				
				return result;
			};
			
			System.out.println("lambda reserved is "+reverse.func("lambda"));
			
			SomeFunc<Integer> factorial = (n)->
			{
				int result = 1;
				
				for(int i=1; i<=n; i++)
				{
					result = result*i;
				}
				
				return result;
			};
			
			System.out.println("The factorial of 3 is "+factorial.func(3));
		}
	}
```
	
	输出：
	lambda reserved is adbmal
	The factorial of 3 is 6
	
	其中，T指定了func()函数的返回类型和参数类型。这意味着它与任何接收一个参数，并返回一个相同类型的值的lambda表达式兼容。
	
	SomeFunc接口用于提供对两种不同类型的lambda表达式的引用。第一种表达式使用String类型，第二种表达式使用Integer类型。因此，同一个函数式接口可以用于reserve lambda表达式和factorial lambda表达式。区别仅在于传递给SomeFunc的参数类型。
	
**15.4 作为参数传递lambda表达式**

	为了将lambda表达式作为参数传递，接收lambda表达式的参数的类型必须是与该lambda表达式兼容的函数式接口的类型。例如：
	
```
	//Use lambda expressions as an argument to method
	
	interface StringFunc
	{
		String func(String n);
	}
	
	class lambdasAsArgumentsDemo
	{
		static String stringOp(StringFunc sf,String s)
		{
			return sf.func(s);
		}
		
		public static void main(String[] args)
		{
			String inStr = "lambda add power to java";
			String outStr;
			
			System.out.println("Here is input string: "+inStr);
			
			outStr = stringOp((str)->str.toUpperCase(),inStr);
			System.out.println("The string in uppercase: "+outStr);
			
			outStr = stringOp((str)->
			{
				String result = "";
				int i;
				
				for(i=0; i<str.length(); i++)
				{
					if(str.charAt(i) != '')
					{
						result += str.charAt(i);
					}
				}
				
				return result;
			},inStr);
			System.out.println("The string with spaces removed: "+outStr);
			
			//当块lambda看上去特别长，不适合嵌入到方法的调用中时，很容易把块lambda赋给一个函数式接口变量.
			//然后，可以简单地把该引用传递给方法。
			StringFunc reverse = (str)->
			{
				String result = "";
				int i;
				
				for(i=str.length()-1; i>=0; i--)
				{
					result += str.charAt(i);
				}
				
				result result;
			};
			System.out.println("The string reserved: "+stringOp(reverse,inStr));
		}
	}
```
	
	输出：
	Here is input string: lambda add power to java
	The string in uppercase: LAMBDAS ADD POWER TO JAVA 
	The string with spaces removed: lambdaaddpowertojava
	The string reserved: avaJ ot rewop dda sadbmal
	
	在该程序中，首先注意stringOp()方法。它有两个参数，第一个参数的类型是StringFunc,而StringFunc是一个函数式接口。因此，这个参数可以接受对任何StringFunc实例的引用，包括由lambda表达式创建的实例。stringOp的第二个参数是String类型，也就是要操作的字符串。
	
	接下来，注意对stringOp()的第一次调用，如下所示：
	
	outStr = stringOp((str)->str.uppercase(),inStr);
	
	这里，传递了一个简单的表达式lambda作为参数。这会创建函数式接口StringFunc的一个实例，并把对该实例的一个引用传递给stringOp()方法的第一个参数这就把嵌入在一个类实例中的lambda代码传递给了方法。目标类型上下文由参数的类型决定。因此lambda表达式与该类型兼容，调用是合法的。
	
	当块lambda看上去特别长，不适合嵌入到方法的调用中时，很容易把块lambda赋给一个函数式接口变量，正如上面代码中那样。然后，可以简单地把该引用传递给方法。
	
**15.5 lambda表达式与异常**

	lambda表达式可以抛出异常。但是，如果抛出经检查的异常，该异常就必须与函数式接口的抽象方法的throws子句中列出的异常兼容。下面的例子演示了这个事实。它计算一个double数组的平均值。但是，如果传递了长度为0的数组，就会抛出自定义异常EmptyArrayException。从示例中可以看出，DoubleNumericArrayFunc函数式接口内声明的func()方法的throws子句中列出了此异常。
	
```
	interface DoubleNumericArrayFunc
	{
		double func(double[] n)throws EmptyArrayException;
	}
	
	class EmptyArrayException extends Exception
	{
		EmptyArrayException()
		{
			super("Array Empty");
		}
	}
	
	class lambdaExceptionDemo
	{
		public static void main(String[] args)
		{
			double[] values = {1.0, 2.0, 3.0, 4.0, 5.0};
			
			DoubleNumericArrayFunc average = (n)->
			{
				double sum = 0;
				
				if(n.length == 0)
				{
					throw new EmptyArrayException();
				}
				
				for(int i=0; i<n.length; i++)
				{
					sum += n[i];
				}
				
				return sum / n.length;
			};
			
			System.out.println("The average is "+average.func(values));
			
			//This causes an exception to be throw.
			System.out.println("The average is "+average.func(new double[0]));
		}
	}
```
	
	对average.func()的第一次调用返回了值2.5。在第二次调用中，由于传递了一个长度为0的数组，EmptyArryException异常被抛出。记住，在func()方法中包含throws子句是必要的。如果不这么做，那么由于lambda表达式不在于func()兼容，程序将无法通过编译。
	
	注意：
	函数式接口DoubleNumericArrayFunc的func()方法指定的参数是数组。然而，lambda表达式的参数是n，而不是n[]。回忆一下，lambda表达式的参数类型将从目标上下文中推断得出。在这里，目标上下文是double[]，所以n的类型将会是double[]。没有必要指定n[](这么做也不合法)。将参数显示的声明为double[] n是合法的，但是在本例中这么做不会有什么好处。
	
**15.6 lambda表达式和变量捕获**

	在lambda表达式中，可以访问其外层作用域定义的变量。例如，lambda表达式可以使用其外层定义的实例或静态变量。lambda表达式也可以显示或隐式地访问this变量，该变量引用lambda表达式的外层类的调用的实例。因此，lambda表达式可以获取或设置其外层类的实例或静态变量的值，以及调用其外层类定义的方法。
	
	但是，当lambda表达式使用其外层作用域定义的局部变量时，会产生一种特殊的情况，称为变量捕获。在这种情况下，lambda表达式只能使用实质上final的局部变量。实质上final的变量是指在第一次赋值以后，值不在发生变化的变量(即在后面的程序中没有改变变量的值)。没有必要显示的将这种变量声明为final，不过那样做也不是错误(外层作用域的this参数自动是实质上final变量，lambda表达式没有自己的this参数)。
	
	lambda表达式不能修改外层作用域内的局部变量，理解这一点很重要。修改局部变量会移除其实质上的final状态，从而使捕获该变量变得不合法。
	
	下面程序演示了实质上的final的局部变量和可变局部变量的区别。
	
```
	interface MyFunc
	{
		int func(int n);
	}
	
	class VarCapture
	{
		int num = 10;
		
		MyFunc mylambda = (n)->
		{
			//This use of num is ok.It does not modify num.
			int v = num + n;
			
			//Hower,the following is illegal because it attempts to modify the value of num.	
	//		num++;
	
			return v;
		};
		
		//The following line would also cause an error,because it would remove the effectively 
		//final status from num.
	//	num = 9;
	}
```
	
	正如注释所指出的，num实质是final变量，所有可以在mylambda内使用。但是，如果修改了num，不管是在lambda表达式内还是表达式外，num就会丢失其实质上final的状态。这会导致发生错误，程序将无法通过编译。
	
	需要重点强调，lambda表达式可以使用和修改调用其调用类的实例变量，只是不能使用其外层作用域内的局部变量，除非该变量实质上是final变量。
	
**15.7 方法引用**

	有一个重要的特性与lambda表达式相关，叫做方法引用。方法引用提供了一种引用而不执行方法的方式。这种特性与lambda表达式相关，因为它也需要由兼容的函数式接口构成的目标类型上下文。计算时，方法引用会创建函数式接口的一个实例。
	
**15.7.1 静态方法的方法引用**

	要创建静态方法引用，需要使用下面的一般语法：
	
	className:methodName
	
	注意： 
	类名与方法名之间使用双冒号分隔开。::是JDK 8新增的一个分隔符，专门用于此目的。在于目标类型兼容的任何地方，都
	可以使用这个方法的引用。
	
	下面的程序演示了一个静态方法的引用：
	
```
	interface StringFunc
	{
		String func(String n);
	}
	
	class MyStringOps
	{
		static String strReverse(String str)
		{
			String result = "";
			int i;
			
			for(i=str.length()-1; i>=0; i--)
			{
				result += str.charAt(i);
			}
			
			return result;
		}
	}
	
	class MethodRefDemo
	{
		static String stringOp(StringFunc sf,String s)
		{
			return sf.func(s);
		}
		
		public static void main(String[] args)
		{
			String inStr = "lambda add power to Java";
			String outStr;
			
			outStr = stringOp(MyStringOps::strReverse,inStr);
			
			System.out.println("Original string: "+inStr);
			System.out.println("String reserved: "+outStr);
		}
	}
```
	
	输出：
	Original string: lambda add power to Java
	String reserved: avaJ ot rewop dda adbmal
	
	在程序中，特别注意下面这行代码：
	
	outStr = stringOp(MyStringOps::strReverse,inStr);
	
	这里，将对MyStringOps内声明的静态方法strReverse()的引用传递给stringOp()方法的第一个参数。可以这么做，因为
	strReverse与StringFunc函数式接口兼容。因此，表达式MyStringOps::strReverse的计算结果为对象引用，其中，
	strReverse提供了StringFunc的func()方法的实现。 
	
**15.7.2 实例方法的方法引用**

	要传递对某个对象的实例方法的引用，需要使用下面的基本语法：
	
	objRef::methodName
	
	可以看出，这种语法与用于静态方法的语法类似，只不过这里使用对象引用而不是类名。下面使用实例方法引用重写了前面的程序：
	
```
	interface StringFunc
	{
		String func(String n);
	}
	
	class MyStringOps
	{
		String strReverse(String str)
		{
			String result = "";
			int i;
			
			for(i=str.length()-1; i>=0; i--)
			{
				result += str.charAt(i);
			}
			
			return result;
		}
	}
	
	class MethodRefDemo2
	{
		static String stringOp(StringFunc sf,String s)
		{
			return sf.func(s);
		}
		
		public static void main(String[] args)
		{
			String inStr ="lambda add power to Java";
			String outStr;
			
			MyStringOps strOps = new MyStringOps();
			
			outStr = stringOp(strOps::strReverse,inStr);
			
			System.out.println("Original string: "+inStr);
			System.out.println("String reserved: "+outStr);
		}
	}
```
	
	这个程序产生的输出与上一个版本相同。
	
	也可以指定一个实例方法，使其能够用于给定类的任何对象而不仅指定对象。此时需要像下面这样创建方法引用：
	
	ClassName::instanceMethodName
	
	这里使用了类的名称，而不是具体的对象，尽管指定的是实例方法。使用这种形式时，函数式接口的第一个参数匹配调用对象，第二个参数匹配方法指定的参数。
	
	下面是一个例子。它定义了一个方法counter()，用于统计某个数组中，满足函数式接口MyFunc的fun()方法定义的条件的对象个数。本例中，它统计HighTemp类的实例个数。
	
```
	interface MyFunc<T>
	{
		boolean func(T v1,T v2);
	}
	
	class HighTemp
	{
		private int hTemp;
		
		HighTemp(int ht)
		{
			hTemp = ht;
		}
		
		boolean sameTemp(HighTemp ht2)
		{
			return hTemp = ht2.hTemp;
		}
		
		boolean lessThanTemp(HighTemp ht2)
		{
			return hTemp < ht2.hTemp;
		}
	}
	
	class InstanceMethWithObjectRefDemo
	{
		static <T> int counter(T[] vals,MyFunc<T> f,T v)
		{
			int count = 0;
			
			for(int i=0; i<vals.length; i++)
			{
				if(f.func(vals[i],v)) count++;
			}
			
			return count;
		}
		
		public static void main(String[] args)
		{
			int count;
			
			HighTemp[] weekDayHighs = {
					new HighTemp(89),new HighTemp(82),
					new HighTemp(90),new HighTemp(89),
					new HighTemp(89),new HighTemp(91),
					new HighTemp(84),new HighTemp(83)};
					
			cout = counter(weekDayHighs,HighTemp::sameTemp,new HighTemp(89));
			System.out.println(cout+" days had a high of 89");
			
			HighTemp[] weekDayHighs2 = {
					new HighTemp(31),new HighTemp(12),
					new HighTemp(24),new HighTemp(19),
					new HighTemp(18),new HighTemp(12),
					new HighTemp(-1),new HighTemp(13)};
			
			cout = counter(weekDayHighs2,HighTemp::sameTemp,new HighTemp(12));
			System.out.println(count+" days had a high of 12");
			
			cout = counter(weekDayHighs,HighTemp::lessThanTemp,new HighTemp(89));
			System.out.println(count+" days had a high less than 89");
			
			count = counter(weekDayHighs2,HighTemp::lessThanTemp,new HighTemp(19));
			System.out.println(count+" days had a high of less than 19");
		}
	}
```
	
	输出：
	3 days had a high of 89 
	2 days had a high of 12 
	3 days had a high less than 89 
	5 days had a hign less than 19 
	
	在这个程序中，注意HighTemp有两个实例方法：someTemp()和lessThanTemp()。如果两个HighTemp对象包含相同的温度，sameTemp()方法返回true。如果调用对象的温度小于被传递的对象的温度，lessThanTemp()方法返回true。这两个方法都有一个HighTemp类型的参数，并且都返回布尔结果。因此，这两个方法都与MyFunc函数式接口兼容，因为调用对象类型可以映射到func()的第一个参数，传递的实参可以映射到func()的第二个参数。因此，当下面的表达式：
	
	HighTemp::sameTemp
	
	被传递给counter()方法时，会创建函数式接口的一个实例，其中第一个参数的参数类型就是实例方法的调用对象的类型，也就是HighTemp。第二个参数的类型也是HighTemp，因为这是sameTemp()方法的参数。对于lessThanTemp()，这也是成立的。
	
	另外一点，通过使用super，可以引用方法的超类版本。如下所示：
	
	super::name
	
	方法的名称由name指定。
	
	备注：
	上面程序中函数式接口中的函数boolean func(T v1,T v2)中含有两个参数，而HighTemp中函数sameTemp(HighTemp ht2)含有一个参数，但是能够兼容的原因是：
	其实HighTemp类中的sameTemp(HighTemp ht2)其实包含两个参数，默认隐藏调用这个函数的引用this。
	故，当使用类的实例方法作为方法引用时，函数式接口的第一个参数匹配类的实例方法的调用对象，第二个参数才匹配方法指定的参数。
	
**15.7.3 泛型中的方法引用**

	在泛型类或泛型方法中，也可以使用方法引用。例如，分析下面程序：
	
```
	interface MyFunc<T>
	{
		int func(T[] vals,T v);
	}
	
	class MyArrayOps
	{
		static <T> int countMatching(T[] vals,T v)
		{
			int count = 0;
		
			for(int i=0; i<vals.length; i++)
			{
				if(vals[i] == v) count++;
			}
		
			return count;
		}	
	}
	
	class GenericMethodRefDemo
	{
		static<T> int myOp(MyFunc<T> f,T[] vals,T v)
		{
			return f.func(vals,v);
		}
		
		Integer[] vals = {1,2,3,4,2,3,4,4,5};
		String[] strs = {"One","Two","Three","Two"};
		int count;
		
		count = myOp(MyArrayOps::<Integer>countMatching,vals,4);
		System.out.println("vals contains "+count+" 4s");
		
		count = myOp(MyArrayOps::<String>countMatching,strs,"Two");
		System.out.println("strs contains "+count+" Twos");
	}
```
	
	输出：
	vals contains 3 4s
	strs contains 2 Twos
	
	在程序中，MyArrayOps是非泛型类，包含泛型方法countMatching()。该方法返回数组中与指定值匹配的元素的个数。注意这里如何指定泛型类型参数。例如，在main()方法中，对countMatching()方法的第一次调用如下所示：
	
	count = myOp(MyArrayOps::<Integer>countMatching,vals,4);
	
	这里传递了类型参数Integer。注意，参数传递发生在::的后面。这种语法可以推广。当把泛型方法指定为方法引用时，类型参数出现在::之后、方法名之前。但是，需要指出的是，在这种情况(和其它许多情况)下，并非必须显示指定类型参数，因为类型参数会被自动推断得出。对于指定泛型类的情况，类型参数位于类名的后面、::的前面。
	
	前面的例子显示了使用方法引用的机制，但是没有展现它们真正的优势。方法引用能够一展拳脚的一处地方是在于集合框架一起使用时。在第18章详细介绍。

	例如：找到集合中最大元素的一种方法是使用Collections类定义的max()方法。对于这里使用的max()版本，必须传递一个集合引用，以及一个实现了Comparator<T>接口的对象的实例。Comparator<T>接口指定如何比较两个对象，它只定义了抽象方法compare()，该方法接受两个参数，其类型均为要比较的对象的类型。如果第一个参数大于第二个参数，该方法返回一个正数；如果两个参数相等，返回0；如果第一个参数小于第二个参数，返回一个负数。
	
	过去，要在max()方法中使用用户定义的对象，必须首先通过一个类显式实现Comparator<T>接口，然后创建该类的一个实例，通过这种方法获得Comparator<T>接口的一个实例，然后，把这个实例作为比较器传递给max()方法。在JDK 8中，现在可以简单地将比较方法的引用传递给max()方法，因为这将自动实现比较器。
	
	下面的简单的示例显示了这个过程。该例创建MyCLass对象的一个ArrayList，然后找到列表中具有最大值的对象(这是由比较方法定义的)。
	
```
	import java.util.*;
	
	class MyClass
	{
		private int val;
		
		MyClass(int v){val = v;}
		
		int getValue(){return val;}
	}
	
	class UseMethodRef
	{
		static int compareMC(MyClass a,MyClass b)
		{
			return a.getValue()-b.getValue();
		}
		
		public static void main(String[] args)
		{
			ArrayList<MyClass> a1 = new ArrayList<MyClass>();
			
			a1.add(new MyClass(1));
			a1.add(new MyClass(4));
			a1.add(new MyClass(2));
			a1.add(new MyClass(9));
			a1.add(new MyClass(3));
			a1.add(new MyClass(7));
			
			//UseMethodRef::compareMC生成了抽象接口Comparator定义的compare()方法的实例。
			MyClass maxValObj = Collections.max(a1,UseMethodRef::compareMC);
			
			System.out.println("Maximum value is: "+maxValObj.getValue());
		}
	}
```
	
	输出：
	Maximum value is: 9 
	
	在程序中，注意MyClass即没有定义自己的比较方法，也没有实现Comparator接口。但是，通过调用max()方法，仍然可以获得MyClass对象列表中的最大值，这是因为UseMethodRef定义了静态方法compareMC()，它与Comparator定义的compare()方法兼容。因此，没哟必要显式的实现Comparator接口并创建其实例。
	
**15.8 构造函数引用**

	与创建方法引用相似，可以创建构造函数的引用。所需语法的一般形式如下所示：
	
	className::new 
	
	可以把这个引用赋值给定义的方法与构造函数兼容的任何函数式接口的引用。下面是一个例子：
	
```
	interface MyFunc
	{
		MyClass func(int n);
	}
	
	class MyClass
	{
		private int val;
		
		MyClass(int v){val = v;}
		
		MyClass(){val = 0;}
		
		int getValue(){return val;}
	}
	
	class ConstructorRefDemo
	{
		MyFunc myClassCons = MyClass::new;
		
		MyClass mc = myClassCons.func(100);
		System.out.println("val in mc is: "+mc.getValue());
	}
```
	
	输出：
	val in mc is: 100 
	
	在程序中，注意MyFunc的func()方法返回MyClass类的引用，并且有一个int类型的参数。接下来，注意MyClass定义了两个构造函数。第一个构造函数指定了一个int类型的参数，第二个构造函数是无参构造函数。现在，分析下面这行代码：
	
	MyFunc myClassCons = MyClass::new;
	
	这里，表达式MyClass::new创建了对MyClass构造函数的引用。在本例中，因为为MyFunc的func()方法接受了一个int类型的参数，所以被引用的构造函数是MyClass(int v),它是正确匹配的构造函数。还要注意，对这个构造函数的引用被赋值给
	了名为myClassCons的MyFunc引用。这条语句执行后，可以使用myClassCons来创建MyClass的一个实例，如下面这行代码所示：
	
	MyClass mc = myClassCons.func(100);
	
	实质上，myClassCons成了调用MyClass(int v)的另一种方式。
	
**15.9 预定义的函数式接口**

	直到现在，本章中的实例都定义了自己的函数式接口，以便清晰地演示lambda表达式和函数式接口背后的基本概念。但是很多时候，并不需要自己定义函数式接口，因为JDK 8中包含了新包java.util.function，其中提供了一些预定义的函数式
	接口。在第2部分详细讨论，这里只做简单简介。
	
	java.util.function包中提供的一些预定义函数式接口：UnaryOperator<T>,BinaryOperator<T>,Consumer<T>,Supplier<T>,Function<T,R>,Predicate<T>。
	
	Function<T,R>:对类型为T的对象应用操作，并返回结果。结果是类型为R的对象。包含的方法名为apply()。
	
	下面的程序通过使用Function接口重写前面的BlocklambdaDemo示例，演示了Function接口的实际应用。BlocklambdaDemo示例通过实现了一个阶乘，演示了块lambda。该示例创建了自己的函数式接口NumericFunc，但其实也可以使用内置的Function接口中定义的抽象函数apply()，如程序的下面这个版本所示：
	
```
	import java.util.function.Function;
	
	class UseFunctionInterfaceDemo
	{
		public static void main(String[] args)
		{
			Function<Integer,Integer> factorial = (n)->
			{
				int result = 1;
				
				for(int i=1; i<=n; i++)
				{
					result = result*i;
				}
				
				return result;
			};
			
			System.out.println("The factorial of 3 is "+factorial.apply(3));
		}
	}
```
	
	这个版本产生的输出与前一个版本相同。
	Function<T,R>是预定义的函数式接口，抽象函数为apply()，故不需要自己定义函数式接口。

出自：《Java 8编程参考官方教程(第9版)》
	
	
