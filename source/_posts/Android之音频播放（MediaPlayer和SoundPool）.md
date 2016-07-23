---
title: Android之音频播放（MediaPlayer和SoundPool）
date: 2016-07-23 23:46:57
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

Android提供简单的API来播放音频。

> **使用MediaPlayer播放音频**

 - 此类适合播放较大文件，此类文件应该存储在SD卡上，而不是在资源文件里，还有此类每次只能播放一个音频文件。
 - 缺点：资源占用量较高，延迟时间较长；不支持多个音频同时播放。
 
使用MediaPlayer非常简单，当程序控制MediaPlayer对象装载音频完成后，程序可以调用MediaPlayer的如下三个方法进行播放控制。
 
 - start()：开始或恢复播放。
 - stop()：停止播放。
 - pause():暂停播放。

为了让MediaPlayer来装载指定的音频文件，MediaPlayer提供如下简单的静态方法。

 - static MediaPlayer create(Context context, Uri uri):从执定Uri来装载音频文件，并返回新创建的MediaPlayer对象。
 - static MediaPlayer create(Context context, int resid)：从resid资源的ID对应的资源文件中装载音频文件，并返回新创建的对象。
上面方法用起来非常方便，但这两个方法每次都会返回新创建的MediaPlayer对象，如果程序需要使用MediaPlayer循环播放多个音频文件，使用MediaPlayer的静态create()方法就不太合适了，此时可通过MediaPlayer的setDataSource()方法来装载指定的音频文件。MediaPlayer提供了如下方法来指定装载相应的音频文件。
 - void setDataSource(String path):指定装载path路径所代表的文件。
 - void setDataSource(FileDescriptor fd, long offset, long length):指定装载fd所代表的的文件中从offset开始、长度为length的文件内容。
 - void setDataSource(Context context, Uri uri):指定装载uri所代表的文件。
 - void setDataSource(FileDescriptor fd)：指定装载fd所代表的的文件。


执行上面所示的setDataSource()方法后，MediaPlayer并未真正的去装载那些音频文件，还需要调用MediaPlayer的prepare()方法去准备音频。所谓“准备”是让MediaPlayer真正去装载音频文件。

除此之外，MediaPlayer还提供了一些绑定事件监听器的方法，用于监听MediaPlayer播放过程中所发生的特定事件。绑定事件监听器的方法如下。

 - setOnCompletionListener():为MediaPlayer的播放完成事件绑定事件监听器。
 - setOnErrorListener():为MediaPlayer的播放错误事件绑定事件监听器
 - setOnPreparedListener()：为MediaPlayer调用Prepare()方法时触发该监听器。
 
使用MediaPlayer播放不同来源的音频文件的方式：

**1.播放应用中的资源文件**

```
 MediaPlayer 
 mediaPlayer = MediaPlayer.create(this,R.raw.incomingcall);
        mediaPlayer.start();
```

提示：音频资源文件一般放在Android应用的/res/raw目录下

**2.播放应用原始资源文件**
播放应用原始资源的步骤：

 - 调用Context的getAssets()方法获取应用的AssetManager。
 - 调用AssertManager对象的openFd(String name)方法代开指定的原始资源，该方法返回一个AssetFileDescriptor对象。
 - 调用AssetFileDescriptor的getFileDescriptor()、getStartOffset和getLength()方法来获取音频文件的FileDescriptor、开始位置、长度等。
 - 创建MediaPlayer对象(或利用已有的MediaPlayer对象)，并调用MediaPlayer对象的setDataSource(FileDescriptor fd, long offset, long length)方法来装载音频资源。
 - 调用MediaPlayer对象的prepare()方法准备音频。
 - 调用MediaPlayer的start()、pause()、stop()等方法控制播放即可。

```
 AssetManager am = getAssets();
 //打开指定音乐文件
 AssetFileDescriptor afd = am.openFd(music);
 MediaPlayer mediaPlayer = new MediaPlayer();
 //使用MediaPlayer装载指定的声音文件
 mediaPlayer.setDataSource(afd.getFileDescriptor(),
                afd.getStartOffset(),afd.getLength());
 //准备声音
 mediaPlayer.prepare();
 //播放
 mediaPlayer.start();
```

**3.播放外部存储器上的音频文件**

```
MediaPlayer mediaPlayer = new MediaPlayer();
mediaPlayer.setDataSource("/mnt/sdcard/mysong.mp3");
mediaPlayer.prepare();
mediaPlayer.start();
```

4.播放来自网络的音频文件

有两种方式:
(1)直接使用MediaPlayer的静态方法create(Context context, Uri uri).
(2)调用MediaPlayer的setDataSource(Context context, Uri uri)方法装载Uri对对应的音频文件。
第(2)中方式的代码：

```
Uri uri = Uri.parse("http://www.xxx.cn/abc.mp3");
MediaPlayer mediaPlayer = new MediaPlayer();
mediaPlayer.setDataSource(this,uri);
mediaPlayer.prepare();
mediaPlayer.start();
```

另外，MediaPlayer除了调用prepare()方法来准备声音之外，还可以调用prepareAsync()来准备声音。prepareAsync()与prepare()方法的区别在于，prepareAsync()是异步的，它不会阻塞当前的UI线程。

以上MediaPlayer的只是转载自：《Android疯狂讲义》


----------

> **使用SoundPool播放音频**

SoundPool主要用与播放一些较短的声音片段，与MediaPlayer相比，SoundPool的优势在于CPU资源占用率低和反映延迟小。SoundPool使用音效池的概念来管理多个短促的音效，因此适合实时同时播放多个声音，反复播放的声音，如游戏中炸弹的爆炸音等小资源文件，此类音频比较适合放到资源文件夹 res/raw下和程序一起打成APK文件。另外SoundPool还支持自行设置声音的品质、音量、播放比率等参数。

**1.创建SoundPool对象：**

![这里写图片描述](http://img.blog.csdn.net/20150831081604024)

SoundPool提供一个Builder内部类，该内部类专门用于创建SoundPool。
注：从Android5.0开始，SoundPool的构造器被设为过时了，因此推荐使用SoundPool.Builder来创建SoundPool对象。

Android 5.0之前：

```
soundPool = new SoundPool(10, AudioManager.STREAM_RING,100);
```
Android 5.0之后：
```
 //设置音效池的属性
 AudioAttributes audioAttributes = new AudioAttributes.Builder()
				 //设置音效使用场景
                .setUsage(AudioAttributes.USAGE_NOTIFICATION)
//设置音效类型					                 .setContentType(AudioAttributes.CONTENT_TYPE_MUSIC)
                .build();
                
 //创建SoundPool对象               
 soundPool = new SoundPool.Builder()
				 //设置音效池属性
                .setAudioAttributes(audioAttributes)
                //设置音效类型
                .setMaxStreams(10)
                .build();
```


**2.加载声音**

一旦得到SoundPool对象后，接下来就可调用SoundPool的多个重载的load()方法来加载声音了。SoundPool提供了如下load()方法。

 - public int load (Context context, int resId, int priority)：从resId加载声音。
 - public int load (FileDescriptor fd, long offset, long length, int priority)：从fd所对应的文件加载声音。
 - public int load (AssetFileDescriptor afd, int priority)
 - public int load (String path, int priority)

上面的方法中都有一个priority参数，该参数目前还没有任何作用，Android建议将该参数设置为1，保持和未来的兼容性。

Returns：
a sound ID. This value can be used to play or unload the sound.
即load()函数返回该声音的ID。

为了更好的管理SoundPool所加载的每个声音的ID，程序一般会使用HashMap< Interger,Interger>对象来管理声音。

```
HashMap<Integer,Integer> soundMap = new HashMap<>();
soundMap.put(1,soundPool.load(this,R.raw.one,1));
soundMap.put(2,soundPool.load(this,R.raw.two,1));
```
通过soundMap.get(1)就可以拿到，soundPool.load(this,R.raw.one,1)的返回值得声音ID。
同理，soundMap.get(2)就可以拿到，soundPool.load(this,R.raw.two,1)的返回值得声音ID。

**3.播放声音**

同过load()方法加载声音后，都会返回该声音的ID，以后程序就可以通过该声音的ID来播放声音了。SoundPool提供的播放指定声音的方法如下：
public final int play (int soundID, float leftVolume, float rightVolume, int priority, int loop, float rate)

```
public final int play (int soundID, float leftVolume, float rightVolume, int priority, int loop, float rate)

Parameters
soundID	a soundID returned by the load() function
leftVolume	left volume value (range = 0.0 to 1.0)
rightVolume	right volume value (range = 0.0 to 1.0)
priority	stream priority (0 = lowest priority)
loop	loop mode (0 = no loop, -1 = loop forever)
rate	playback rate (1.0 = normal playback, range 0.5 to 2.0)

Returns
non-zero streamID if successful, zero if failed
```
注：有时候play()函数的返回值为0,DDMS报的错是sample not ready的问题，也就是说是在load加载音乐文件出错，导致在play播放音乐时显示not ready； 在SoundPool中有setOnLoadCompleteListener方法用来判断音乐加载是否完成，因此解决方法如下：
(1)在音乐加载完成时间完成后播放

```
soundPool.setOnLoadCompleteListener(new SoundPool.OnLoadCompleteListener() 
{
            @Override
            public void onLoadComplete(SoundPool soundPool,
			{
                streamIdOne 
                = soundPool.play(soundMap.get(1),1,1,0,-1,1);
            }
  });

```
(2)可直接在load后面加sleep（1000），具体时间根据加载的文件的多少大小而定，给程序足够的时间去加载初始化音频文件。

```
// 将加载的声音资源id放进此Map
soundPoolMap.put(1, soundPool.load(this, R.raw.gamestart, 1));
try 
{
	Thread.sleep(1000);// 给予初始化音乐文件足够时间
} 
catch (InterruptedException e)
{
	e.printStackTrace();
}
```

**4.暂停**

public final void stop (int streamID)

Parameters：
streamID	a streamID returned by the play() function

Demo：

![这里写图片描述](http://img.blog.csdn.net/20150831110735909)

界面布局很简单我就不上传了，点击PlayOne按钮播放第一个音乐，点击StopOne按钮第一个音乐播放结束。点击PlayTwo按钮播放第二个音乐，点击StopTwo按钮第二个音乐暂停。

```
public class MainActivity extends Activity implements View.OnClickListener {
    private Button playOneBtn;
    private Button stopOneBtn;
    private Button playTwoBtn;
    private Button stopTwoBtn;

    SoundPool soundPool;
    HashMap<Integer,Integer> soundMap = new HashMap<>();

    int streamIdOne;
    int streamIdTwo;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        playOneBtn = (Button) findViewById(R.id.playOneBtn);
        stopOneBtn = (Button) findViewById(R.id.stopOneBtn);
        playTwoBtn = (Button) findViewById(R.id.playTwoBtn);
        stopTwoBtn = (Button) findViewById(R.id.stopTwoBtn);
        playOneBtn.setOnClickListener(this);
        stopOneBtn.setOnClickListener(this);
        playTwoBtn.setOnClickListener(this);
        stopTwoBtn.setOnClickListener(this);
        //创建SoundPool对象并设置属性
        /*AudioAttributes audioAttributes = new AudioAttributes.Builder()
                .setUsage(AudioAttributes.USAGE_NOTIFICATION)
                .setContentType(AudioAttributes.CONTENT_TYPE_MUSIC)
                .build();
        soundPool = new SoundPool.Builder()
                .setAudioAttributes(audioAttributes)
                .setMaxStreams(10)
                .build();*/

        soundPool = new SoundPool(10, AudioManager.STREAM_RING,100);
        //加载声音
        soundMap.put(1,soundPool.load(this,R.raw.one,1));
        soundMap.put(2,soundPool.load(this,R.raw.two,1));

    }

    @Override
    public void onClick(View v) {
        switch (v.getId())
        {
            case R.id.playOneBtn:
                streamIdOne = soundPool.play(soundMap.get(1),1,1,0,-1,1);
                break;
            case R.id.playTwoBtn:
                streamIdTwo = soundPool.play(soundMap.get(2),1,1,0,-1,1);
                break;
            case R.id.stopOneBtn:
                soundPool.stop(streamIdOne);
                break;
            case R.id.stopTwoBtn:
                soundPool.stop(streamIdTwo);
                break;
        }
    }
}

```

