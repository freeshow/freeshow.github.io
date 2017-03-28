---
title: Android之使用全局变量的两种方法
date: 2016-07-23 23:57:44
tags: Android
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

 转载自：[Android之项目全局变量的定义](http://blog.csdn.net/sir_zeng/article/details/8198249)


 1. 使用静态类：
 

```
public class Data{
	private static String a ="Hello Android";
	
	public static String getA() {
		return a;
	}
	
	public static void setA(String a) {
		Data.a = a;
	}
}

```
调用就不再写了，直接使用类名.变量名就可以调用！

static修饰的静态变量，使用很方便，在不同的类和包中都可以使用，在虚拟机中单独占用内存，没错，这些都是它们的优点，不过在项目上线后，才发现static有一些不太好的地方。
        
在查看项目的崩溃信息时，发现很多地方莫明的出现空指针异常的错误，经过排查，发现可能就是static的问题。我们在项目中，将用户的信息也就是User对象保存成了一个静态变量，而在报错的地方，也都发现有使用过这种变量，因此，可以大致推断出与这种保存的方式有一定的联系。同时，有不少用户反映在打开应用的情况下，接个电话或者长时间待机后，再回到应用也会出现崩溃的现象，而这些崩溃都与静态变量的空指针有关系。

如此来说的话，static静态修饰在Android的开发中是不是很危险？或许我们可以说如果是static User u = new User();这样定义的话，那么应该不会有太大问题，而如果是static User u;这样定义的话，那么很可以会出现NULL的现象。当然，前面的方法里面的属性也可能会现空的情况，但是这个可以用封装来避免空指针。另外静态常量还是很好用的。

  那么应该如何保存登录或者全局的信息呢？根据Google官方的推荐以及百度到的各位大神的推荐，我们应该尽量使用继承自Application的自定义类，在我们继承的类中定义需要全局使用的变量，并通过getApplicationContext()来获取和保存相关的变量即可。


----------
2.使用Application

```
/**
* 自定义的MyApplication继承Application
*
* @author way
*
*/ 
public class MyApplication extends Application { 
    /**
     * 引发异常：在一些不规范的代码中经常看到Activity或者是Service当中定义许多静态成员属性。这样做可能会造成许多莫名其妙的 null pointer异常。
     */ 
 
    /**
     * 异常分析：Java虚拟机的垃圾回收机制会主动回收没有被引用的对象或属性。在内存不足时，虚拟机会主动回收处于后台的Activity或
     * Service所占用的内存。当应用再次去调用静态属性或对象的时候，就会造成null pointer异常
     */ 
 
    /**
     * 解决异常：Application在整个应用中，只要进程存在，Application的静态成员变量就不会被回收，不会造成null pointer异常
     */ 
    private int number; 
 
    @Override 
    public void onCreate() { 
        // TODO Auto-generated method stub 
        super.onCreate(); 
    } 
 
    public int getNumber() { 
        return number; 
    } 
 
    public void setNumber(int number) { 
        this.number = number; 
    } 
}
```

不过，为了让我们的MyApplication取代android.app.Application的地位，在我们的代码中生效，我们需要修改AndroidManifest.xml：

```
<application android:name=".MyApplication" ...> 
</application>
```

下面我们就可以在Activity或Service中灵活使用了：

```
MyApplication application = (MyApplication)this.getApplicationContext();  
//保存变量 
application.setNumber(5); 
//取出变量 
application.getNumber();
```


- 而且按照Java及C#的种编辑思想的话，还是建议使用第二种试，这样对于系统的安全是好的！而且我查了一些资料显示，这样也是符合Android这种思想的，因此建议使用第二种方式，设置公共变量！
- Application是与应用同时存在的，也就是应用在它就在，并不会被GC给莫名其妙的回收掉，因此，使用此方法更加安全。