---
title: Android之Handler详解（二）---关联到非UI线程
date: 2016-07-24 00:12:35
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## Handler、Loop、MessageQueue的工作原理 ##
先介绍一下与Handler一起工作的几个组件。

 - Message: Handler接受和处理的消息对象。
 - Looper: 每个线程只能拥有一个Looper。它的loop方法负责读取MessageQueue中的消息，读到消息之后就把消息交给发送该消息的Handler进行处理。
 - MessageQueue: 消息队列，它采用先进先出的方式来管理Message。程序创建Looper对象时，会在它的构造器中创建MessageQueue对象。
 - Handler: 它的作用有两个---发送消息和处理消息，程序使用Handler发送消息，由Handler发送的消息必须发送到指定的MessageQueue中，也就是说如果希望Handler正常工作，必须在handler所关联的线程中有一个MessageQueue,而MessageQueue是由Looper来管理的，故Handler关联的线程中必须有一个Looper对象。

为了保证Handler关联的线程有Looper对象，可分一下两种情况：

 1. 在主UI线程中，系统已经初始化了一个Looper对象，因此程序直接创建Handler即可，然后就可以通过Handler来发送消息、处理消息了。(即之前写的博客：Handler详解(一)---关联到UI线程，**非UI线程向UI线程中发送消息**)
 2. 程序员自己启动的子线程，必须自己创建一个Looper对象，并启动它。创建Looper对象调用它的prepare()方法即可。prepare()方法保证每个线程最多只有一个Looper对象。**UI线程向非UI线程发送消息**。
 
调用Looper的静态loop()方法可以启动Looper对象。loop()方法使用一个死循环不断的取出MessageQueue中的消息，并将取出的消息发给给消息对应的Handler进行处理。

归纳，Looper、MessageQueue、Handler各自的作用如下：

 1. Looper： 每个线程只有一个Looper,它负责管理MessageQueue，会不断的从MessageQueue中取出消息，并将消息分给对应的Handler处理。
 2. MessageQueue： 有Looper负责管理。它采用先进先出的方式来管理Message.
 3. Handler: 它能把消息发送给Looper管理的MessageQueue，并负责处理Looper分给它的消息。
 
在自己创建的线程中使用Handler的步骤:

	

 1. 调用Looper的prepare方法为当前线程创建Looper对象，创建Looper对象时，它的构造器会创建与之配套的MessageQueue。
 2. 有了Looper后创建Handler子类的实例，重写handleMessage()方法，该方法处理来自于其他线程的消息。
 3. 调用Looper的loop()方法启动Looper.
 
实例：

```
public class MainActivity extends Activity {
    private TextView textView;
    private Button showButton;
    CallThread callThread;
    Handler handler;

    class CallThread extends Thread
    {
        @Override
        public void run()
        {
            Looper.prepare();
            handler = new Handler()
            {
                @Override
                public void handleMessage(Message msg) {
                    super.handleMessage(msg);
                    if (msg.what == 1)
                    {
                        try {
                            Thread.sleep(5000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        Toast.makeText(MainActivity.this,textView.getText(),LENGTH_LONG).show();
                    }
                }
            };
            Looper.loop();
        }
    }
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        textView = (TextView) findViewById(R.id.textView);
        showButton = (Button) findViewById(R.id.showBtn);

        callThread = new CallThread();
        callThread.start();

        showButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                //callThread在此处初始化不行，必须放到上面初始化，
                // 否则导致程序无法运行，本人刚学Android，等以后知道了才解释吧。
//                callThread = new CallThread();
//                callThread.start();
                Message msg = Message.obtain();
                msg.what = 1;
                handler.sendMessage(msg);
            }
        });
    }
}
```

效果：
![这里写图片描述](http://img.blog.csdn.net/20150820135129361)
点击SHOW TOAST按钮,5s后按钮下面显示Toast的内容为Hello World.

如果将上述代码该为：

```
    showButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            Thread.sleep(5000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        Toast.makeText(MainActivity.this,textView.getText(),LENGTH_LONG).show();
                    }
                }).start();
            }
        });
```
点击SHOW TOAST按钮,5s后不会出现Toast,程序运行失败，怎么回事？
报的错误为：

> java.lang.RuntimeException: Can't create handler inside thread that has not called Looper.prepare()


当然，有人会问UI线程向非UI线程之间进行通信时，每次都得写Looper对象，那岂不是很麻烦。故Android已经帮我们实现好了自带Looper的线程，那就是HandlerThread类。
## HandlerThread ##
Android对HandlerThread的描述：

> Handy class for starting a new thread that has a looper. The looper can then be used to create handler classes. Note that start() must still be called.

使用HandlerThread的步骤：

 1. 创建一个HandlerThread，即创建了一个包含Looper的线程。
 `handlerThread = new HandlerThread("handlerThread");
        //创建handlerThread后一定要记得start.
        handlerThread.start();`
 2. 获取HandlerThread的Looper
```
Looper looper = handlerThread.getLooper();
```
 3   创建Handler，通过Looper初始化
 handler = new Handler(looper);
 
 通过以上三步我们就成功创建HandlerThread。通过handler发送消息，就会在子线程中执行。如果想让HandlerThread退出，则需要调用handlerThread.quit();
 
 注：如果hander传递的是Message,则需要重写handlerMessage(）函数。
 

```
 handler = new Handler(looper)
        {
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                if (msg.what == 1)
                {
                    //要处理的事情
                }
            }
        };
```
或者自己创建个线程类继承HandlerThread类并实现CallBack接口以重写CallBack里面的handlerMessage方法，在handlerMessage方法中来处理自己的逻辑。

如果hander传递的是Runnable对象，则不需要重写handlerMessage()函数。

使用HandlerThread重写上面的Demo:
public class MainActivity extends Activity {
    private TextView textView;
    private Button showBtn;
    private HandlerThread handlerThread;
    private Handler handler;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        handlerThread = new HandlerThread("handlerThread");
        //创建handlerThread后一定要记得start.
        handlerThread.start();

        Looper looper = handlerThread.getLooper();

        //handler与谁关联不是看在哪创建handler，而是看handler创建时与哪个线程的looper挂钩。
        handler = new Handler(looper)
        {
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                if (msg.what == 1)
                {
                    Toast.makeText(getApplicationContext(),textView.getText()
                    ,Toast.LENGTH_LONG).show();
                }
            }
        };

        textView = (TextView) findViewById(R.id.textView);
        showBtn = (Button) findViewById(R.id.showBtn);

        showBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                handler.sendEmptyMessageDelayed(1,5000);
            }
        });
    }
}

## HandlerThread和普通Thread的比较 ##

 - HandlerThread继承自Thread，当线程开启时，也就是它run方法运行起来后，线程同时创建了一个含有消息队列的Looper，并对外提供自己这个Looper对象的get方法。
 - 为什么要使用HandlerThread。

	1.开发中如果多次使用类似new Thread(){...}.start()。这种方式开启一个子线程，会创建多个匿名线程，使得程序运行起来越来越慢，
	而HandlerThread自带Looper使他可以通过消息来多次重复使用当前线程，节省开支；

	2.android系统提供的Handler类内部的Looper默认绑定的是UI线程的消息队列，对于非UI线程又想使用消息机制，那么HandlerThread内部的Looper是最合适的，它不会干扰或阻塞UI线程。

 

