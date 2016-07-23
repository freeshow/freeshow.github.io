---
title: Scala单例对象
date: 2016-07-23 22:40:12
tags: Scala
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

声明：作为自己学习Scala的笔记，只是为了以后复习Scala方便。

## Singleton对象

Scala比Java更为面向对象的特点之一是Scala不能定义静态成员，而是代之以定义单例对象(singleton object)。除了用object关键字替换了class关键字以外，单例对象的定义看上去与类定义一致。
```
// 私有构造方法
class Marker private(val color:String) {

    println("Creating " + this)

    override def toString(): String = "marker color "+ color

}

// 伴生对象，与类共享名字，可以访问类的私有属性和方法
object Marker{

    private val markers: Map[String, Marker] = Map(
        "red" -> new Marker("red"),
        "blue" -> new Marker("blue"),
        "green" -> new Marker("green")
    )

    def apply(color:String) = {
        if(markers.contains(color)) markers(color) else null
    }


    def getMarker(color:String) = {
        if(markers.contains(color)) markers(color) else null
    }
}

object ObjectDemo {
    def main(args: Array[String]) {
        // 方法糖 apply
        //单例对象在第一次被访问时才会被初始化。
        println(Marker("red"))

        println()

        // 单例函数调用，省略了.(点)符号
        println(Marker getMarker "blue")
    }
}
```
输出为：
```
Creating marker color red
Creating marker color blue
Creating marker color green
marker color red

marker color blue
```
上面例子中单例对象叫做Maker，与Maker类同名。当单例对象与某个类共享同一个名称时，它就被称为是这个类的伴生对象。类和它的伴生对象必须定义在一个源文件中。类被称为是这个单例对象的伴生类。类和它的伴生对象可以相互访问其私有成员。

对于Java程序员来说，可以把单例对象当做是Java中可能会用到的静态 方法工具类。也可以用类似的语法做方法调用：单例对象名，点，方法名。

类和单例对象间的差别是，单例对象不带参数，而类可以。因为单例对象不是用new关键字实例化的，所以没机会传递给它实例化参数。每个单例对象都被实现为虚构类的实例，并指向静态的变量，因此它们与Java静态类有着相同的初始化语义。

特别要指出的是，单例对象在第一次被访问的时候才会被初始化。

## 将伴生对象作为工厂使用
我们通常将伴生对象作为工厂使用。

下面是一个简单的例子，可以不需要使用’new’来创建一个实例了。
```
class Bar(foo: String)

object Bar {
	def apply(foo: String) = new Bar(foo)
}
```
创建对象是只需要使用工厂方法，而不许要使用new来创建：
val bar = Bar("foo") 等价与
var bar = Bar.apply("foo")

## 独立对象(standalone object)
不与伴生类共享名称的单例对象被称为独立对象。它可以用在很多地方。例如，作为相关功能方法的工具类，或者定义Scala应用的入口点。

## Demo
```
class ApplyDemo
{
    def apply() = "apply in class"
    def test
    {
        println("test")
    }
}

/**
  * 伴生对象，相当于类的静态方法
  */
object ApplyDemo {

    def stat
    {
        println("static method")
    }

    def apply() = new ApplyDemo

    var count = 0

    def incc =
    {
        count += 1
    }
}

object ClassTest {
    def main(args: Array[String]): Unit = {
        ApplyDemo.stat

        //类名后面加括号，相当于调用伴生对象的apply方法
        val a = ApplyDemo()
        a.test

		//对象加括号相当于调用对象的apply方法
        println(a())

		val b = ApplyDemo.apply()
        b.test

		println(a.apply())

        for(i <- 0 until 10){
            ApplyDemo.incc
        }
        println(ApplyDemo.count)
    }
}
```
输出为：
```
static method
test
apply in class
test
apply in class
10
```
