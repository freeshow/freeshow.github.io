---
title: Android之数据存储--外部存储（SD卡）
date: 2016-07-23 23:54:38
tags: Android
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

使用外部存储实现数据持久化，这里的外部存储一般就是指的是sdcard。使用sdcard存储的数据，不限制只有本应用访问，任何可以有访问Sdcard权限的应用均可以访问，而Sdcard相对于设备的内部存储空间而言，会大很多，所以一般比较大的数据，均会存放在外部存储中。

使用SdCard存储数据的方式与内部存储的方式基本一致，但是有三点需要注意的：

 - 需要首先判断是否存在可用的Sdcard，调用Environment的getExternalStorageState()方法判断手机上是否插入了SD卡，并且应用程序具有读写SD卡的权限。使用如下代码：
 

```
//如果手机已插入SD卡，且应用程序具有读写SD卡的能力，下面语句返回true。
Environment.getExternalStorageState().
			equals(Environment.MEDIA_MOUNTED)
```

>使用的Environment.getExternalStorageState()方法，返回的是一个字符串数据，Environment封装好了一些final对象进行匹配，除了Environment.MEDIA_MOUNTED外，其他均为有问题，所以只需要判断是否是Environment.MEDIA_MOUNTED状态即可。

 - 既然转向了Sdcard，那么存储的文件路径就需要相对变更,
 可通过Environment.getExternalStorageDirectory()函数获得Sdcard的根目录。

	标准手机的SD卡根目录是:/mnt/sdcard/
	其他国产手机SD卡的根目录各不相同，如/storage/emulated/0/，下面写的程序已标注SD卡目录，即/mnt/sdcard/目录讲解为主。

 - 为了读写SD卡上的数据，不许在应用程序的清单文件(AndroidManifest.xml)中添加读写权限。例如如下配置：
 

```
//在SD卡中创建与删除文件权限
<uses-permission 
	android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS">
</uses-permission>

//向SD卡中写入数据权限
<uses-permission
	 android:name="android.permission.WRITE_EXTERNAL_STORAGE">
</uses-permission>
```


----------


外部存储中的文件是可以被用户或者其他应用程序修改的，有两种类型的文件（或者目录）：

***1.私有文件Private files：***

其实由于是外部存储的原因即是是这种类型的文件也能被其他程序访问，只不过一个应用私有的文件对其他应用其实是没有访问价值的（恶意程序除外）。外部存储上，应用私有文件的价值在于卸载之后，这些文件也会被删除。类似于内部存储。

所有应用程序的外部存储的私有文件都放在SD卡根目录的Android/data/下，目录形式为：**/mnt/sdcard/Android/data/< package_name>/files**,可以通过函数Context.getExternalFilesDir()函数获得该目录.

往外部存储内读写文件和判断文件是否存在（写入到
/mnt/sdcard/Android/data/com.xxx.xxx/files目录下）

```
//往外部存储器写文件，将String s 写入文件Extral.test中
private void write(String s)
{
        if(Environment.getExternalStorageState().
		        equals(Environment.MEDIA_MOUNTED))
        {
            File file = new File(Environment.getExternalStorageDirectory(),"Extral.test");
            OutputStream outputStream = null;
            try {
                outputStream = new FileOutputStream(file);
                outputStream.write(s.getBytes());
                outputStream.close();
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

//从外部存储器中的Extral.test文件中读取内容。
private String read() 
{
        File file = new File(Environment.getExternalStorageDirectory(),"Extral.test");
        InputStream inputStream = null;
        try {
            inputStream = new FileInputStream(file);
            byte[] data = new byte[1024];
            inputStream.read(data);
            return new String(data);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
```

运行上面代码会发现/mnt/sdcard/Android/data/< package_name>/files/目录下多了个Extral.test文件。

如果你的api 版本低于8，那么不能使用getExternalFilesDir()，而是使用Environment.getExternalStorageDirectory()获得根路径之后，自己再想办法操作/Android/data/< package_name>/files下的文件。

也就是说api 8以下的版本在操作文件的时候没有专门为私有文件和公共文件的操作提供api支持。你只能先获取根目录，然后自行想办法。

> ***在外部存储器上存储缓存文件：***

如果将应用程序的外部存储的私有文件作为缓存文件的话，可放在目录/mnt/sdcard/Android/data/< package_name>/cache下，可以通过函数：Context.getExternalCacheDir()

如果你的api 版本低于8，那么不能使用getExternalCacheDir()，而是使用Environment.getExternalStorageDirectory()获得根路径之后，自己再想办法操作/Android/data/< package_name>/files下的文件。

也就是说api 8以下的版本在操作文件的时候没有专门为私有文件和公共文件的操作提供api支持。你只能先获取根目录，然后自行想办法,例如下面代码，获取外部存储缓存的目录：

```
public static String getExternalCacheDir(Context context) {

      if (!isMounted())
           return null;

      StringBuilder sb = new StringBuilder();

      File file = context.getExternalCacheDir();

      // In some case, even the sd card is mounted,
      // getExternalCacheDir will return null
      // may be it is nearly full.

      if (file != null) {
           sb.append(file.getAbsolutePath()).append(File.separator);
      } 
      else {//如果api低于8,通过如下方法获取外部存储缓存目录
           sb.append(Environment.getExternalStorageDirectory().getPath()).
           append("/Android/data/").
           append(context.getPackageName()).
           append("/cache/").
           append(File.separator).toString();
      }

      return sb.toString();
 }
```


----------


***2.公共文件Public files：***

文件是可以被自由访问，且文件的数据对其他应用或者用户来说都是由意义的，当应用被卸载之后，其卸载前创建的文件仍然保留。比如camera应用，生成的照片大家都能访问，而且camera不在了，照片仍然在。

如果保存的文件不是应用程序所专有的，并且在应用程序被卸载时，不删除这些文件，那么就要把它们保存到外部存储器上的一个公共的目录上。这些目录位于外部存储器的根目录下，如：

/mnt/sdcard/Music/---媒体扫描器把在这个目录中找到所有媒体文件作为用户音乐。
/mnt/sdcard/Podcasts/---媒体扫描器把在这个目录中找到的所有媒体文件作为音/视频的剪辑片段。
/mnt/sdcard/Ringtones/---媒体扫描器把在这个目录中找到的所有媒体文件作为铃声。
/mnt/sdcard/Alarms/---媒体扫描器把在这个目录中找到的所有媒体文件作为闹钟的声音。
/mnt/sdcard/Pictures/---所有的图片（不包括那些用照相机拍摄的照片）。
/mnt/sdcard/Movies/---所有的电影（不包括那些用摄像机拍摄的视频）。
/mnt/sdcard/Download/---其他下载的内容。

如果你想在外存储上放公共文件你可以使用
Environment.getExternalStoragePublicDirectory(Sting type)

例子的话只需要把上面例子中的getExternalFilesDir(null)替换为Environment.getExternalStoragePublicDirectory()，此方法需要一个参数来指定公共目录类型（如Environment.DIRECTORY_MUSIC、Environment.DIRECTORY_PICTURES、Environment.DIRECTORY_RINGTONES或其他的类型。如果需要，这个方法会创建适当的目录）

例如：

```
File file = new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_MUSIC),"Extral.test")
```
在/mnt/sdcard/Music/目录下创建Extral.test文件。

![这里写图片描述](http://img.blog.csdn.net/20150828110343803)
----------
补充：对于现在市面上很多Android设备，自带了一个大的存储空间，一般是8GB或16GB，并且又支持了Sdcard扩展，对于这样的设备，使用Enviroment.
getExternalStorageDirectory()方法只能获取到设备自带的存储空间，对于另外扩展的Sdcard而言，需要修改路径。

注：
在没有安装SDcard的 华为荣耀6手机上(3RAM，16GROM）：
通过以下函数获得的路径:
Environment.getExternalStorageDirectory()：
/storage/emulated/0

getExternalFilesDir(null)：
/storage/emulated/0/Android/data/com.example.filetest/files

getExternalCacheDir()：
/storage/emulated/0/Android/data/com.example.filetest/cache

Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_MUSIC):
 /storage/emulated/0/Music
 
 但用以上函数存储文件时，获得的根目录为/storage/emulated/0，实际存储在/mnt/shell/emulated/0/目录下，
 如：/mnt/shell/emulated/0//Android/data/com.example.filetest/files
 不知道怎么回事？

参考：http://blog.csdn.net/tianjf0514/article/details/8271114#