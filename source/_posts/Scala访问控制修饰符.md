---
title: Scala访问控制修饰符
date: 2016-07-23 22:48:43
tags: Scala
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

参考自《Programming in Scala》

## 一、访问控制修饰符
包的成员,类或对象可以使用访问控制修饰符,比如 private 和 protected 来修饰,通过这些修饰符可以控制其他部分对这些类,对象的访问。Scala 和访问控制大体上和 Java 类似,但也有些重要的不同,本篇将介绍这些。

### 私有成员
Scala的私有成员和Java类似,一个使用 private修饰过的类或对象成员,只能在该类或对象中访问,在Scala中,也可以在嵌套的类或对象中使用。比如:
```
class Outer
{
    class Inner
    {
        private def f() {println("f")}

        class InnerMost
        {
            f() //OK
        }
    }

    (new Inner).f() //错误，f不可访问
}
```
在Scala中,(new Inner).f()是不合法的,因为它是在Inner中定义的私有类型,而在InnerMost中访问f却是合法的,这是因为 InnerMost 是包含在Inner的定义中(子嵌套类型)。
在Java语言中,两种访问都是可以的。Java允许外部类型访问其包含的嵌套类型的私有成员。

### 保护成员
和私有成员类似,Scala的访问控制比Java来说也是稍显严格些。在 Scala中,由protected定义的成员只能由定义该成员和其派生类型访问。而在 Java中,由protected定义的成员可以由同一个包中的其它类型访问。在Scala中,可以通过其它方式来实现这种功能。

下面为protected的一个例子:
```
package p
{
    class Super
    {
        protected def f(){println("f")}
    }

    class Sub extends Super
    {
        f() //OK
    }

    class Other
    {
        (new Super).f() //Error:f不可访问
    }
}
```
### 公共成员
public访问控制为Scala定义的缺省方式,所有没有使用private和 protected修饰的成员(定义的类和方法)都是“公开的”,这样的成员可以在任何地方被访问。Scala不需要使用public来指定“公开访问”修饰符。

注意：Scala中定义的类和方法默认都是public的，但在类中声明的属性默认是private的。

## 二、为访问控制修饰符添加作用域
Scala 的访问修饰符可以添加作用域参数。作用域的语法如下:
private[x]或protected[x]
其中x代表某个包,类或者单例对象,表示可以访问这个private或 protected的范围直到X。

通过为访问修饰符添加作用域参数,可以非常精确的控制所定义的类型能够被其它类型访问的范围。尤其是可以支持Java语言支持的 package private,package protected 等效果。

下面的例子为这种用法的一个示例:
```
package bobsrockets
{
    package navigation
    {
    	//如果为private class Navigator,则类Navigator只会对当前包navigation中所有类型可见。
        //即private默认省略了[X],X为当前包或者当前类或者当前单例对象。
        //private[bobsrockets]则表示将类Navigator从当前包扩展到对bobsrockets包中的所有类型可见。
        private[bobsrockets] class Navigator
        {
            protected[navigation] def useStarChart() {}
            class LegOfJourney
            {
                private[Navigator] val distance = 100
            }

            private[this] var speed = 200
        }
    }
    package launch
    {
        import navigation._
        object Vehicle
        {
        	//private val guide：表示guide默认被当前单例对象可见。
            //private[launch] val guide：表示guide由默认对当前单例对象可见扩展到对launch包中的所有类型可见。
            private[launch] val guide = new Navigator
        }
    }
}
```
在这个例子中,类Navigator使用 private[bobsrockets] 来修饰,这表示这个类可以被bobsrockets包中所有类型访问,比如通常情况下 Vehicle无法访问私有类型Navigator,但使用包作用域之后,Vechile 中可以访问Navigator。

private的限定词还可以被执行所属类和对象。例如代码中类LegOfJourney
中的distance变量被标记为private[Navigator],表示它在Navigator类的任何地方都可见。这种访问能力与Java里的内部类的私有成员一致。

所有的限定词也可以用于protected，与private意思相同。也就是说，C类里的protected[X]修饰符允许C的所有子类及修饰符所属的包、类或者对象X访问带有此标记的定义。例如代码中的useStartChart方法能被类Navigator的所有子类以及包含在navigator中的所有代码访问。



这种技巧在分散在多个 Package 的大型项目时非常有用,它允许你定义一些在多个子包中可以访问,但对使用这些API的外部客户代码隐藏,而这种效果在Java中是无法实现的。

此外,Scala还支持一种比private还要严格的访问控制,本例中的 private[this],只允许在定义该成员的类型中访问,它表示该成员不仅仅只能在定义该成员的类型中访问,而且只能是由该类型本身访问，这种定义被称为对象私有。比如:本例中speed,使用protected[this]修饰,因此在Navigator类内部访问speed和this.speed是合法的。然而一下的访问，即使发生在Navigator类内部也是不允许的。

```
var other = new Navigator
other.speed	//error:此行不能编译
```
把成员标记为private[this]可以保证它不能被同一个类中其它对象访问。

注意：如果speed被private修饰，则上面的代码可以编译。


现在如果将这些组合应用到LegOfJourney.distance上，会有什么样的效果：

| 修饰符 | 公开访问 |
|--------|--------|
|private[bobsrockets]|在外部包中访问|
|private[navigation]|与Java的包可见度相同(包内可见)|
|private[Navigator]|与Java的private相同(类内可见)|
|private[LegOfJourney]|与Scala中的private相同(本类中可见，父类中不可见)|
|private[this]|只有同一个对象中可见|

