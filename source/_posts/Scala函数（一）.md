---
title: Scala函数（一）
date: 2016-07-23 22:43:56
tags: Scala
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## Scala函数
Scala有函数和方法，我们术语说的方法和函数有微小的差别。
函数，如果其被定义为某些对象的一个成员，被称为方法。

数定义可以出现在在源文件的任何地方，Scala允许嵌套函数的定义，那就是其他函数定义的内部函数定义。需要注意的最重要的一点是，Scala的函数名称可以类似+, ++, ~, &,-, -- , , /, : 等字符。

### 一、函数的声明、定义、调用
### 1.函数声明
Scala函数声明有如下形式：
```
def functionName ([list of parameters]) : [return type]
```
### 2.函数定义
Scala函数定义有如下形式：
```
def functionName ([list of parameters]) : [return type] = {
   function body
   return [expr]
}
```
在这里，返回类型可以是任何有效的scala数据类型，参数列表将是用逗号和参数，返回值类型列表分离变量是可选的。非常类似于Java，一个返回语句可以在函数表达式可用情况下返回一个值。以下是这将增加两个整数并返回的函数：
```
object add{
   def addInt( a:Int, b:Int ) : Int = {
      var sum:Int = 0
      sum = a + b

      return sum
   }
}
```
函数，它不返回任何东西，可以返回这相当于在Java中void(在Scala中用Unit表示不返回任何类型)，并表示该函数不返回任何单元。Scala中不返回任何东西函数被称为过程。以下是语法：
```
object Hello{
   def printMe( ) : Unit = {
      println("Hello, Scala!")
   }
}
```
### 3.函数调用
Scala提供了一些语法的变化来调用方法。以下是调用一个方法的标准方法：
```
functionName( list of parameters )
```
如果函数被对象的一个实例调用使用，那么使用类似于Java点标记如下：
```
[instance.]functionName( list of parameters )
```
下面是一个例子用来定义，然后调用函数：
```
object Test {
   def main(args: Array[String]) {
        println( "Returned Value : " + addInt(5,7) );
   }
   def addInt( a:Int, b:Int ) : Int = {
      var sum:Int = 0
      sum = a + b

      return sum
   }
}
```

### 二、函数定义的几种写法

**注意**：Scala里方法参数的一个重要特征是它们都是val，不是var。参数是val的理由是val更容易讲清楚。你不需要多看代码以确定是否val被重新赋值，而var则不然。 如果你想在方法里面给参数重新赋值，结果是编译失败：
```
def addOne(x:Int):Int = {
		//Error：编译不过，因为x是val
        x += 1
        return x
    }
```
### 1.始终带返回值
```
   def add(x:Int,y:Int):Int={
     x+y
   }

```
### 2.省略非Unit返回值；如果没有写返回值，则根据等号后面的东西进行类型推演
```
 def test(x:Int)={
      x
   }
```
### 3.省略等号，返回Unit
```
def returnVoid(){
     println("return void")
}
```
**注意**：当你去掉方法体前面的等号时，它的结果类型将注定是Unit。不论方法体里面包含什么都不例外，因为Scala编译器可以把任何类型转换为Unit。例如，如果方法的最后结果是String，但方法的结果类型被声明为Unit，那么String将被转变为Unit并失去它的值。下面是这个例子：
```
scala> def g() { "this String gets lost too" }
g: ()Unit
```
即只有当函数返回值类型为Unit时，才可以省略=，如果大意将=省略，则会导致将想要返回的函数返回值类型转化为Unit类型。
### 4.省略花括号，如果函数仅包含一条语句，那么连花括号都可以选择不写
```
def max2(x: Int, y: Int) = if (x > y) x else y 
```

_ _ _

## 三、本地函数
Scala中你可以把函数定义在别的函数之内，就好像本地变量那样，这种函数称为本地函数。这种本地函数仅在包含它的代码块中可见。


参考自：
[Scala函数的定义的几种写法](http://my.oschina.net/scipio/blog/277456?fromerr=8z7aB5YR)
[Scala函数](http://www.yiibai.com/scala/scala_functions.html)
