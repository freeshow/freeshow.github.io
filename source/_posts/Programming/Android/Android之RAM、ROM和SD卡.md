---
title: Android之RAM、ROM和SD卡
date: 2016-07-23 23:59:04
tags: Android
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

 - RAM(内存)：物理位置是位于手机内部的随机存储器上，断电后资料丢失。相当于电脑的内存。
 - ROM：物理位置是位于手机内部的非易失性存储器上，断电后依然能够保存资料。主要包括：系统存储、系统缓存、内部存储。
 
 而android系统是基于linux系统建立的，它的分区结构跟windows不同，没有C盘D盘的，系统存储、系统缓存、内部存储分别都是不同的分区，每个分区的大小是在系统建立的时候就分配好了的，一般人是无法更改的。

系统存储相当于是windows的C盘，系统缓存相当于windows的临时文件夹
内部存储就相当于windows的其他盘。

 - android系统中，“/”以及“/system”等目录是用于系统存储的，（“/”是系统的根目录），比如“/system/app”是指系统软件的存放目录。
 - 系统缓存是存放在"/cache"下的
 - 内部存储一般是指用户可以使用的空间位于"/data"。
 
SD卡：手机的外部存储空间，这个我们可以理解成电脑的外接移动硬盘，U盘也行。

所有的Android设备都有两个文件存储区域：“内部”和“外部”存储器。这两个名称来自早期的Android，当时大多数设备都提供内置的固定的内存（内置存储器）即上面介绍的ROM中的内部存储，外加一个可移动的存储介质，如micro SD卡（外部存储器）。但也有些设备把固定不变的存储空间分成“内部”和“外部”两部分，这样即使没有可移动的存储介质，也总会有两个存储空间，并且不管外部存储器是可移动的，还是固定的，API的行为是相同的。


----------


内部存储器与外部存储器的区别：

所有的安卓设备都有外部存储和内部存储，这两个名称来源于安卓的早期设备，那个时候的设备内部存储确实是固定的，而外部存储确实是可以像U盘一样移动的。但是在后来的设备中，很多中高端机器都将自己的机身存储扩展到了8G以上，他们将存储在概念上分成了"内部internal" 和"外部external" 两部分，但其实都在手机内部。所以不管安卓手机是否有可移动的sdcard，他们总是有外部存储和内部存储。最关键的是，我们都是通过相同的api来访问可移动的sdcard或者手机自带的存储（外部存储）。

**外部存储虽然概念上有点复杂，但也很好区分，你把手机连接电脑，能被电脑识别的部分就一定是外部存储。**

内部存储器：

 - Android系统能够直接把文件存在设备的内部存储内。
 - 默认情况下，保存在内部存储内的文件是应用程序私有的，其他应用程序（或用户）是无法访问的。
 - 当用户卸载此应用程序时，内部存储的数据会一并清除。
 - Shared Preferences和SQLite数据库都是存储在内部存储空间上的。内部存储一般用Context来获取和操作。
 - 内部存储一般保存在“/data/”目录下

外部存储器：

 - 使用sdcard存储的数据，不限制只有本应用访问，任何可以有访问Sdcard权限的应用均可以访问，而Sdcard相对于设备的内部存储空间而言，会大很多，所以一般比较大的数据，均会存放在外部存储中。
 - 要向外部存储器写入数据，你必须在清单文件中申请WRITE_EXTERNAL_STORAGE权限
 - 外部存储中的文件是可以被用户或者其他应用程序修改的，有两种类型的文件（或者目录）：
  - .公共文件Public files：文件是可以被自由访问，且文件的数据对其他应用或者用户来说都是由意义的，当应用被卸载之后，其卸载前创建的文件仍然保留。比如camera应用，生成的照片大家都能访问，而且camera不在了，照片仍然在。
  - 私有文件Private files：其实由于是外部存储的原因即是是这种类型的文件也能被其他程序访问，只不过一个应用私有的文件对其他应用其实是没有访问价值的（恶意程序除外）。外部存储上，应用私有文件的价值在于卸载之后，这些文件也会被删除。类似于内部存储。
 - 外部存储一般保存在“/mnt/sdcard/Android/data/”下。
 

 