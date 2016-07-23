---
title: Android之话筒、听筒、扬声器
date: 2016-07-23 23:45:26
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

话筒是输入（麦克风），听筒、扬声器是输出（喇叭）

**听筒/扬声器：**

- 听筒是用来听对方传送过来的声音，手机放的MP3和开机铃声是从喇叭发出的。
- 听筒模式,就是手机上一般听电话的内置小耳机,声音较小。 
- 扬声器模式,就是声音外放,声音较大

**话筒：**

- 传声器是一个声-电转换器件（也可以称为换能器或传感器），是和喇叭正好相反的一个器件（电→声）。是声音设备的两个终端，传声器是输入，喇叭是输出。
- 麦克风，学名为传声器，由Microphone翻译而来。传声器是将声音信号转换为电信号的能量转换器件，也称话筒，麦克风，微音器


----------
AudioManger对象通过getSystemService(Service.AUDIO_SERVICE)获取
AudioManger常用的几个方法void android.media.AudioManager.adjustStreamVolume(int streamType, int direction, int flags)：第二个表示调整音乐的大小，第三个参数表示显示调整是的标志 AudioManager.FLAG_SHOW_UI；调整手机类型的声音；第一个参数的几个值
    STREAM_ALARM：手机闹铃的声音
    STREAM_MUSIC：手机音乐的声音
     STREAM_DTMF：DTMF音调的声音
     STREAM_RING：电话铃声的声音
     STREAM_NOTFICATION：系统提示的声音
      STREAM_SYSTEM：系统的声音
      STREAM_VOICE_CALL：语音电话声音
      
void android.media.AudioManager.setMicrophoneMute(boolean on)设置是否让麦克风设置静音
// 打开扬声器  
audioManager.setSpeakerphoneOn(true);

void android.media.AudioManager.setRingerMode(int ringerMode)：设置手机电话铃声的模式；支持的几个属性值
   RINGER_MODE_NORMAL：正常的手机铃声
   RINGER_MODE_SILENT：手机铃声静音
   RING_MODE_VIBATE：手机震动
void android.media.AudioManager.setStreamMute(int streamType, boolean state)将指定的音量类型调整为静音

Android中打开扬声器关闭麦克风的代码实现：

```
//获取音频服务  
AudioManager audioManager = (AudioManager) this.getSystemService(Context.AUDIO_SERVICE);  
//设置声音模式  
audioManager.setMode(AudioManager.STREAM_MUSIC);  
//关闭麦克风  
audioManager.setMicrophoneMute(false);  
// 打开扬声器  
audioManager.setSpeakerphoneOn(true);  
//实例化一个SoundPool对象  
SoundPool soundPool =new SoundPool(10, AudioManager.STREAM_SYSTEM, 5);  
//加载声音  
int  id = soundPool.load(this,R.raw.beep,5);  
//播放声音  
 soundPool.play(id, 1, 1, 0, 0, 1);  
另外必须加上权限：<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS"/>
```


Android 手机听筒Earpiece和扬声器speaker切换：

```
AudioManager audioManager = (AudioManager)getSystemService(Context.AUDIO_SERVICE);
 private void setSpeakerphoneOn(boolean on) 
 {
        if(on)
        {
            audioManager.setSpeakerphoneOn(true);       
        } else 
        {
	        audioManager.setSpeakerphoneOn(false);//关闭扬声器                                
            //把声音设定成Earpiece（听筒）出来，设定为正在通话中
			audioManager.setMode(AudioManager.MODE_IN_CALL);																																				                                                                        															                                                                                
        }
 }
```


