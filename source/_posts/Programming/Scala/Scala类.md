---
title: Scala类
date: 2016-07-23 22:46:50
tags: Scala
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## Scala类
### 要点
- 类里的属性必须赋初值。
- def函数时如果没参数可不带括号。
- 类中的字段自动带有getter方法和setter方法。
- 可以用定制的getter/setter方法替换掉字段的定义，而不必修改使用类的客户端----这就是“统一访问原则”。
- 用@BeanProperty注解来生成JavaBeans的getXxx/setXxx方法。
- 每个类都有一个主构造器，这个构造器和类定义“交织”在一起。它的参数直接成为类的字段，主构造器执行类体中所有的语句。
- 辅助构造器是可选的，它们叫做this。

在Scala源文件中，可以包含多个类，所有这些类都具有共有可见性。
Scala对每个字段都提供setter和getter方法，比如class Person{var age = 0}，Scala生成面向JVM的类，其中有一个私有的age字段以及相应的getter和setter方法，这两个方法是公有的，如果age是私有的，那么相应的getter和setter也是私有的，在Scala中，getter和setter分别叫做age和age_=。默认的getter和setter是由Scala自动生成，但是你也可以手动生成，例如:

```
class Person
{
  private var privateAge = 0
  //自定义getter方法
  def age = privateAge
  //自定义setter方法
  def age_=(newAge: Int)
  {
    if (newAge > privateAge) privateAge = newAge
  }
}

object HelloWorld
{
  def main(args: Array[String])
  {
    var p = new Person()
    //其实是执行的p.age_=(11)即自定义setter方法。
    p.age = 11
    p.age_=(1) //设置age的值，但是该调用其实相当于调用setter方法，在setter方法中有控制语句，所以该句话不起作用
    //p.age其实是执行的上面自定义getter方法。
    println(p.age) //取出age的值，相当于getter方法
  }
}
```

### 统一访问原则
由Eiffel语言的发明者Bertrand Meyer提出，内容是“某个模块提供的所有服务都应该能通过统一的表示法访问到，至于它们是通过存储还是通过计算来实现的，从访问方式上应无从获知”。

Scala对setter和getter的控制：
1. 如果字段是私有的，则getter和setter方法也是私有的
2. 如果字段是val，则只有getter方法被生成
3. 如果你不需要任何getter和setter，可以将字段声明为private[this]

总结，在实现属性时有如下四种选择：
1. var foo:Scala自动合成一个getter和setter
2. val foo:Scala自动合成getter
3. 由你来定义foo和foo_=方法
4. 由你来定义foo方法


### 对象私有字段
private[this] var value = 0//类似于某个对象.value这样的访问将不被允许这时候，value就是对象私有的，其他同一个类的对象也无法访问到，对象私有的字段，Scala不会生成getter或setter方法。

### Bean属性
如果你将Scala对象标注为@BeanProperty时，例如：@BeanProperty var name : String = _
将会生成四个方法：
1. name:String
2. name_=(newValue:String):Unit
3. getName():String
4. setName(newValue:String):Unit

### 举例
Scala中定义的类默认都是public的，在类中声明的属性默认是private的，并且Scala会生成默认的get和set的方法。

**1.var age = 0**

如下：
```
class Student
{
	//age默认是private级别的，并且会有默认的公有getter和setter方法
    //因为getter和setter方法是公有的，故生成的此类的对象可以引用getter和setter方法。
     var age = 0
}

object StudentDemo
{
	def main(args:Array[String]):Unit =
    {
    	var student = new Student
        student.age = 25
        println(student.age) //输出25
    }
}
```
上述代码中
student.age = 25 等价于student.age_ = (25)
即student.age = 25表示引用age属性的setter方法。
println(studeng.age)中student.age表示引用age属性的getter方法。因为属性age是私有的，故不可能是引用的age属性，而是引用的age属性的方法。

**2.private var age = 0**

如果在声明的属性前加上 private修饰，那么要想访问这个属性必须定义get方法，要想给这个属性赋值必须定义set方法
如下：
```
class Student
{
	//该属性必须定义它的getter和setter方法，否则生成的对象不能引用该属性
    //因为privateAge属性生成的getter和setter是私有的
    private var privateAge = 0

    //下面两个函数实现了age属性
    //age属性的getter方法
    def age = privateAge
    //age属性的setter方法
    def age_=(newAge:Int)
    {
    	if(newAge > privateAge)privateAge = newAge
    }
}

object StudentDemo
{
	val student = new Student
    //age的setter方法，即student.age_=(25)
    student.age = 25
    //age的getter方法
    println(student.age) //输出：25
    student.age_=(1)
    println(student.age) //输出：25
}
```

**3.private[this] var privateAge = 0**

如果在上面的基础上再进行修改成如下代码，那么情况就又会有所不同，下面的代码在private的后面有加了[this]，那么说明该属性只允许当前对象中的方法访问，其它对象中的方法不能访问。

```
class Student {
	//该属性必须定义它的get和set方法，否则生成的对象不能引用该属性
    private[this] var privateAge = 0
    def age = privateAge
    def age_=(newAge:Int)
    {
    	if(newAge > privateAge) privateAge = newAge
    }

    //该对象中的isYounger方法不能访问other对象中的privateAge属性
    //Error
    def isYounger(other:Student)  = privateAge < other.privateAge

}
```

### 构造器
Scala类有一个构造器比其他所有构造器都更为重要，它就是主构造器，除了主构造器之外，类还有任意多的辅助构造器，辅助构造器的特点如下：
1. 辅助构造器的名称为this。
2. 每一个辅助构造器都必须以一个对先前已定义的其他辅助构造器或主构造器的调用开始。

在Scala中，如果没有显式定义主构造器，则自动拥有一个无参的主构造器，主构造器并不以this方法定义，而是与类定义交织在一起。
主构造器的参数直接放置在类名之后，称为类参数。
```
class Person(val name: String, val age: Int)
{
  //类参数(...)中的内容就是主构造器中的内容
}
```

如果类名之后没有参数，则该类具备一个无参主构造器。这样一个构造器仅仅是简单地执行类体中的所有语句而已。

| 主构造器参数 | 生成的字段/方法 |
|--------|--------|
| name:String | 对象私有字段 |
| private val/var name:String | 私有字段，私有的getter/setter方法 |
| val/var name:String | 私有字段，公有的getter/setter方法 |
| @BeanProperty val/var name:String | 私有字段，公有的Scala版和JavaBean版getter/setter方法 |

如果想让主构造器变成私有的，可以这样放置private关键字：
class Person private (val id : Int){...}；这样用户就必须通过辅助构造器来构造Person对象


### 内部类
在Scala中，你几乎可以在任何语法结构中内嵌任何语法结构，你可以在函数中定义函数，在类中定义类。要构造一个新的内部对象，只需要简单的new这个类名（new 外部类.内部类）。在内嵌类中，你可以通过外部类.this的方式来访问外部类的this引用，还可以用如下语法建立一个指向该引用的别名：
```
class NetWork(val name: String)
{
	//语法使得outer变量指向NetWork.this。
    //对这个变量，可以使用任何合法的名称
	outer =>

	class Member(val name: String)
    {
    	def description = name + " inside " + outer.name
    }
}

```
**注意：**在Java中内部类属于外部类，而在Scala中内部类属于外部类的实例。例如：

```
class Outer(val name:String)
{
    outer =>

    class Inner(val name:String)
    {
        def foo(b:Inner) =
            println("Outer: "+outer.name+" Inner: "+b.name)
    }
}

object OPPInScala
{
    def main(args: Array[String])
    {
        val outer1 = new Outer("Hadoop")
        val outer2 = new Outer("Spark")

        val inner1 = new outer1.Inner("Java")
        val inner2 = new outer2.Inner("Scala")

        inner1.foo(inner1)
        inner2.foo(inner2)

        //inner1是根据outer1创建的，Scala的内部类是面向外部类对象的。
        //所以inner1的外部类的name必须是outer1的Hadoop,而不能是outer2的Spark.
        inner1.foo(inner2)  //Error
    }
}
```

参考自：
[Scala类](http://blog.csdn.net/shijiebei2009/article/details/38666201)
[ 第7讲 Scala类的属性和对象私有字段实战详解](http://blog.csdn.net/wwz573398723/article/details/47427679)

