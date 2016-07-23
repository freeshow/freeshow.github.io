---
title: Android之Handler详解（一）---关联到UI线程
date: 2016-07-24 00:13:31
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

此文转载自[http://www.cnblogs.com/shirley-1019/p/3557800.html](http://www.cnblogs.com/shirley-1019/p/3557800.html)
[http://www.cnblogs.com/shirley-1019/p/3566730.html](http://www.cnblogs.com/shirley-1019/p/3566730.html)

## Handler ##

 - Handler， 它直接继承自Object，一个Handler允许发送和处理Message或者Runnable对象，并且会关联到所属线程的MessageQueue 中。每个Handler具有一个单独的线程，并且关联到一个消息队列的线程，就是说一个Handler有一个固有的消息队列。当实例化一个Handler 的时候，它就承载在一个线程和消息队列的线程，这个Handler可以把Message或Runnable压入到消息队列，并且从消息队列中取出 Message或Runnable，进而操作它们。
 - Handler主要有两个作用：
   1.从其他线程中发送消息。
   2.在handler所关联的线程中获取、处理消息。
	
注:
	***Handler与谁相关联不是看声明在什么地方，是看与哪个线程的looper挂钩。默认是主线程的looper.因为主线程中默认就有了looper,消息循环队列。***

Handler可以把一个Message对象或者Runnable对象压入到消息队列中，进而在Handler所关联线程中获取Message或者执行Runnable对象，所以Handler把压入消息队列有两大体系，Post和sendMessage：

**Post**：Post允许把一个Runnable对象入队到消息队列中。它的方法有：post(Runnable)、
postAtTime(Runnable,long)、
postDelayed(Runnable,long)。

**sendMessage**：sendMessage允许把一个包含消息数据的Message对象压入到消息队列中。它的方法 有：
sendEmptyMessage(int)、
sendMessage(Message)、
sendMessageAtTime(Message,long)、
sendMessageDelayed(Message,long)。

　　从上面的各种方法可以看出，不管是post还是sendMessage都具有多种方法，它们可以设定Runnable对象和Message对象被入队到消息队列中，是立即执行还是延迟执行。

Post
　　对于Handler的Post方式来说，它会传递一个Runnable对象到消息队列中，在这个Runnable对象中，重写run()方法。一般在这个run()方法中写入需要在UI线程上的操作。

在Handler中，关于Post方式的方法有：
1.	boolean post(Runnable r)：把一个Runnable入队到消息队列中，UI线程从消息队列中取出这个对象后，立即执行。
2.	boolean postAtTime(Runnable r,long uptimeMillis)：把一个Runnable入队到消息队列中，UI线程从消息队列中取出这个对象后，在特定的时间执行。
3.	boolean postDelayed(Runnable r,long delayMillis)：把一个Runnable入队到消息队列中，UI线程从消息队列中取出这个对象后，延迟delayMills秒执行
4.	void removeCallbacks(Runnable r)：从消息队列中移除一个Runnable对象。
	

	下面通过一个Demo，讲解如何通过Handler的post方式在新启动的线程中修改UI组件的属性：
	（在Android项目中经常有碰到这样的问题，在子线程中完成耗时操作之后要更新UI）
	

```
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        final TextView textView = (TextView) findViewById(R.id.textView);
        Button updateBtn = (Button) findViewById(R.id.updateBtn);
        Button cancleBtn = (Button) findViewById(R.id.cancleBtn);
        final Handler handler = new Handler();
        final Runnable run = new Runnable() {
            @Override
            public void run() {
                textView.setText("After Update");
            }
        };

        updateBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
            /*
                //这里可以处理耗时的操作
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                
                //把一个Runnable入队到消息队列中，handler关联的线程从消息队列中取出这个对象后，立即执行。
                handler.post(run);
			*/
                //将run放入handler关联的消息队列中，hander关联的线程从消息队列中取出5秒后执行。
                handler.postDelayed(run,5000);
            }
        });

        cancleBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                //从消息队列中移除一个Runnable对象。不能移除放入消息队列中立即执行的post(run),
                //可以移除postDelayed(run,5000)中的run对象。
                handler.removeCallbacks(run);
            }
        });
    }
}
```
效果：
![这里写图片描述](http://img.blog.csdn.net/20150819202530789)
1.如果UPDATE按钮执行的是注释/* handler.post(run) */里面的代码
(1)点击UPDATE按钮,5s之后 TextView中Before Update变为After Update。
(2)点击UPDATE按钮后，5s之前点击CANCLE UPDATE按钮，TextView中Before Update还是会变为After Update，移除不掉run对象，因为handler.post(run)是立即执行run对象。
2.如果UPDATE按钮执行的是handler.postDelayed(run,5000)方法，5s之前点击CANCLE UPDATA按钮，则可以将run对象从消息队列中移除，因为postDelayed(run,5000) 5s之后才会执行run对象，post(run)是立即执行run对象。


Message:

　　Handler如果使用sendMessage的方式把消息入队到消息队列中，需要传递一个Message对象，而在Handler中，需要重 写handleMessage()方法，用于获取工作线程传递过来的消息，此方法运行在handler关联的线程上。下面先介绍一下Message。

　　Message是一个final类，所以不可被继承。Message封装了线程中传递的消息，如果对于一般的数据，Message提供了getData()和setData()方法来获取与设置数据，其中操作的数据是一个Bundle对 象，这个Bundle对象提供一系列的getXxx()和setXxx()方法用于传递基本数据类型的键值对，对于基本数据类型，使用起来很简单，这里不 再详细讲解。而对于复杂的数据类型，如一个对象的传递就要相对复杂一些。在Bundle中提供了两个方法，专门用来传递对象的，但是这两个方法也有相应的 限制，需要实现特定的接口，当然，一些Android自带的类，其实已经实现了这两个接口中的某一个，可以直接使用。方法如下：
		
		putParcelable(String key,Parcelable value)：需要传递的对象类实现Parcelable接口。
	
	pubSerializable(String key,Serializable value)：需要传递的对象类实现Serializable接口。
　　还有另外一种方式在Message中传递对象，那就是使用Message自带的obj属性传值，它是一个Object类型，所以可以传递任意类型的对象，Message自带的有如下几个属性：

int arg1：参数一，用于传递不复杂的数据，复杂数据使用setData()传递。
int arg2：参数二，用于传递不复杂的数据，复杂数据使用setData()传递。
Object obj：传递一个任意的对象。

 　　对于Message对象，一般并不推荐直接使用它的构造方法得到，而是建议通过使用Message.obtain()这个静态的方法或者 Handler.obtainMessage()获取。Message.obtain()会从消息池中获取一个Message对象，如果消息池中是空的， 才会使用构造方法实例化一个新Message，这样有利于消息资源的利用。并不需要担心消息池中的消息过多，它是有上限的，上限为10个。 Handler.obtainMessage()具有多个重载方法，如果查看源码，会发现其实Handler.obtainMessage()在内部也是 调用的Message.obtain()。　　

　　Message是在线程间传递消息，将上面的post demo转化为sendMessage的形式。

```
public class MainActivity extends Activity {
    private TextView textView;
    private Button sendBtn;
    Handler handler;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        textView = (TextView) findViewById(R.id.textView);
        sendBtn = (Button) findViewById(R.id.sendBtn);
        handler = new Handler(){
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                if (msg.what == 1)
                {
                    textView.setText(msg.obj.toString());
//                    textView.setText("After Message");
                }
            }
        };

        sendBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        //此处可处理耗时操作
                        try {
                            Thread.sleep(5000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        //耗时操作完成之后发送Message
                        Message msg = Message.obtain();
                        msg.what = 1;
                        msg.obj = "After Message";
                        handler.sendMessage(msg);
                    }
                }).start();
            }
        });
    }
}
```
效果：
![这里写图片描述](http://img.blog.csdn.net/20150820085945025)

点击SENDMESSAGE按钮，5s后Before Message变为After Message


工作流程：
1. Handler默认关联到UI线程(主线程),在非UI线程中调用handler向自己发送message,将Message发送到handler所关联的线程的MessageQueue.
2. 通过handler.sendMessage()回调handler中的回调函数handleMessage(),在handleMessage()中更新UI.