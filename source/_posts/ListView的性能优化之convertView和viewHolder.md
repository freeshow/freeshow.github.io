---
title: ListView的性能优化之convertView和viewHolder
date: 2016-07-23 23:29:39
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

原文出自：方杰| http://fangjie.info/?p=344 转载请注明出处

最近碰到的面试题中经常会碰到问”ListView的优化”问题。所以就拿自己之前写的微博客户端的程序做下优化。

自己查了些资料，看了别人写的博客，得出结论，ListView优化大致从以下几个角度:

 1. 复用已经生成的convertView；
 2. 添加viewHolder类；
 3. 缓存数据（图片缓存）；
 4. 分页加载。

***一、复用convertView***

首先讲下ListView的原理：ListView中的每一个Item显示都需要Adapter调用一次getView的方法，这个方法会传入一个convertView的参数，返回的View就是这个Item显示的View。如果当Item的数量足够大，再为每一个Item都创建一个View对象，必将占用很多内存，创建View对象（mInflater.inflate(R.layout.lv_item, null);从xml中生成View，这是属于IO操作）也是耗时操作，所以必将影响性能。Android提供了一个叫做Recycler(反复循环器)的构件，就是当ListView的Item从上方滚出屏幕视角之外，对应Item的View会被缓存到Recycler中，相应的会从下方生成一个Item，而此时调用的getView中的convertView参数就是滚出屏幕的Item的View，所以说如果能重用这个convertView，就会大大改善性能。

![这里写图片描述](http://img.blog.csdn.net/20151201191737514)

图解：一个屏幕最多显示7个Item，如果当Item1滑出屏幕，此时Item1 的View被添加进Recycler中，相应的在下部要产生一个Item8，这时调用getView方法，convertView参数就是Item1 的View。

（1）Item固定高度

测试Demo：

```
public View getView(int position, View convertView, ViewGroup parent) {
  System.out.println("getView " + position + " " + convertView);
  ViewHolder holder = null;
  if (convertView == null) {
    convertView = mInflater.inflate(R.layout.lv_item, null);
    holder = new ViewHolder();
    holder.textView = (TextView)convertView.findViewById(R.id.tv_text);
    convertView.setTag(holder);
  } else {
    holder = (ViewHolder)convertView.getTag();
  }
  holder.textView.setText(mData.get(position));
  return convertView;
}
```

![这里写图片描述](http://img.blog.csdn.net/20151201191944066)

首先程序启动，9个Item显示调用getView方法，这时的convertView都是null，因为没有任何Item被滑出屏幕缓存到Recycler中。所以打印的Log如图：

![这里写图片描述](http://img.blog.csdn.net/20151201192029825)

继续向上滑动屏幕，当Item1，Item2，Item3…..陆续滑出屏幕之后，他们之前的View得到重用。所以最终屏幕上就只是这10个View反复循环重用了（因为屏幕最多显示10个Item）。

![这里写图片描述](http://img.blog.csdn.net/20151201192119210)

然后我尝试把屏幕往回滚，重新滚到前面。

![这里写图片描述](http://img.blog.csdn.net/20151201192210433)

图解：屏幕从显示Item 17-Item26 滚到显示Item13-Item22。分析这个过程，当滚动一个Item时，Item26消失，Item16出现，所以Item16又将复用Item26的View，观察Log，的确如此，其他依次类推。仔细分析，有这样一个规律：

无论怎么重用convertView，一个position的Item永远使用一个view。
（比如此例中Item13 永远显示的是 5272fb00的View）。


（2）Item高度不固定

以上的例子是针对于每个Item的高度是固定的，所以很容易从定量中计算出每一个屏幕能够显示的最多Item数量。如果当一个Item的高度很根据不同情况的数据发生变化时，这样就无法得知最多显示的Item数量。

看一个例子，我的微博客户端程序，都知道微博有的有转载，有的有图片，所以说微博的每一个Item高度是不固定的。同样在getView中打出日志，显示如下图：

![这里写图片描述](http://img.blog.csdn.net/20151201192340888)

图解：

因为没有固定的Item高度，无法计算一个屏幕中能够显示的最大高度，系统会会先创建一个View，第一轮是用这个View来试探能放多少个item，试探出结果可以放3个Item，所以第二轮的0-2才是真正创建的View，屏幕上显示了3个Item。当往下滚时，Item0没有完全出去，下面有来了个Item3，所以这时的Item有创建了一个View，屏幕上此时显示4个Item。之后4个Item就是做多显示的数量，再往上滚动，convertView就开始重用了，Item4和Item0的View是一个对象。

***二、使用viewHolder类***

我们都知道在getView方法中的操作是这样的：先从xml中创建view对象（inflate操作，我们采用了重用convertView方法优化），然后在这个view去findViewById，找到每一个子View，如：一个TextView等。这里的findViewById操作是一个树查找过程，也是一个耗时的操作，所以这里也需要优化，就是使用viewHolder，把每一个子View都放在Holder中，当第一次创建convertView对象时，把这些子view找出来。然后用convertView的setTag将viewHolder设置到Tag中，以便系统第二次绘制ListView时从Tag中取出。当第二次重用convertView时，只需从convertView中getTag取出来就可以。

测试Demo：

```
public View getView(int position, View convertView, ViewGroup parent) {
     System.out.println("getView " + position + " " + convertView);
     ViewHolder holder = null;
     if (convertView == null) {
       convertView = mInflater.inflate(R.layout.lv_item, null);
       holder = new ViewHolder();
       holder.textView = (TextView)convertView.findViewById(R.id.tv_text);
       convertView.setTag(holder);
     } else {
       holder = (ViewHolder)convertView.getTag();
     }
     holder.textView.setText(mData.get(position));
     return convertView;
   }
}
 public static class ViewHolder {
   public TextView textView;
 }
```

参考：[[Android] ListView中getView的原理＋如何在ListView中放置多个item](http://www.cnblogs.com/xiaowenji/archive/2010/12/08/1900579.html)

[[转]listview加载性能优化ViewHolder](http://www.cnblogs.com/meizixiong/p/4555786.html)