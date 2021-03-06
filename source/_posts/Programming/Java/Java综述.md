---
title: Java综述
date: 2016-07-23 23:20:49
tags: Java
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

**1.Java的魔力：字节码**

Java编辑器的输出不是可执行代码，而是字节码(bytecode)。字节码是高度优化的指令集合，这些指令由Java运行时系统执行，Java运行时系统也称为Java虚拟机(Java Virtual Machine,JVM)。这一点可能有点让人吃惊，因为处于性能的考虑，许多现代语言被设计为将源代码编译成可执行代码。然而，Java程序是由JVM执行这一事实，有助于解决基于Web的程序相关的主要问题。

将Java程序翻译成字节码，可以使其更容易的在各种环境中运行，因为只需要针对每种平台实现Java虚拟机即可。如果Java程序被编译成本机代码，就必须为相同的程序针对连接到Internet的不同类型的CUP提供不同版本，这当然是不行的。因此，通过JVM执行字节码是创建真正可移植程序最容易的方法。

Java程序由JVM执行这一事实，还有助于提供安全性。因为JVM控制执行，所以能够包含程序并防止它在系统之外产生负效应。

**2.Java applet(以过时)**

Java applet 时一种特殊类型的Java程序，是为了能够在Internet上传送而设计的，可以在兼容Java的Web浏览器中自动运行。此外，applet可以根据需要自动下载，不需要用户的自动交互。如果用户点击包含applet的链接，就会自动下载applet,并在浏览器中运行。Applet一般是小程序，它们通常用于显示服务器提供的数据、处理用户输入或者提供在本地执行而不是在服务器上执行的简单功能，例如贷款计算器。本质上，applet使得可以将某些功能从服务器移到客户端。

**3.servlet:服务器端的Java**

Servlet是在服务器上执行的小程序。就像applet动态扩展了Web浏览器的功能一样，servlet动态扩展了服务器的功能。

Servlet用于动态的创建发送到客户端的内容。例如在线商店可以使用servlet在数据库中查找某件商品的价格，然后使用价格信息动态生成发送到浏览器的Web页面。

**4.包含源代码的文件名称**

在Java中，源文件的正式称为是编译单元(compilation unit),是包含一个或多个类定义(以及其他内容)的文本文件。Java编译器要求源文件使用.java作为扩展名。

在Java中，所有代码必须位于类中。按照约定，主类的名称应当与包含程序的文件的名称相匹配，另外还应当确保文件名的大小写与类名相匹配。

将源代码编译会生成.class的字节码文件，其中包含了Java虚拟机将要执行的指令。编译过Java源代码后，每个单独的类被放置到它自己的输出文件中，输出文件以类名加上扩展名”.class”作为名称。这就是为什么将Java源文件的名称指定为它所包含的类名是一个好主意的原因---源文件的名称将于.class文件的名称相同。

**5.public static void main(String[ ] args){}**

main()是当Java程序开始时调用的方法。必须将main()方法声明为public,因为当程序启动时，必须从声明main()方法的类的外部调用它。关键字static表示不必先实例化类的特定实例就可以调用main()方法。这是必须的，因为Java要在创建任何对象之前调用main()方法。

Main()方法只不过是程序开始执行的地方。复杂的程序可能具有几十个类，但这些类中只有一个类需要具有main()方法，以提供程序的开始点。

另外，有些情况下，根本不需要main()方法。例如，对于创建applet——嵌入到web浏览器中的Java程序——不需要使用main()方法，因为Web浏览器使用一种不同的方法启动applet的执行。

出自：《Java 8编程参考官方教程(第9版)》