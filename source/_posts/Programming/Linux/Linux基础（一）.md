---
title: Linux基础（一）
date: 2016-07-24 00:15:26
tags: Linux
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

uname -a：查看内核版本
uname -n：查看主机名

lsb_release -a:查看发行版本

echo $SHELL:查看使用的是什么shell


----------


####挂载点：入口
df -h:查看当前挂载情况
sudo fdisk -l:查看当前系统下硬盘的分区情况
sudo tune2fs -l 分区名(或路径):查看单个分区状况


----------


####touch:
1. 创建文件 eg:touch a.txt
2. update access(访问) and modification(修改) times
  作用：用于数据库管理和系统文件的管理
  eg:
  7:00时  touch a.txt;则a.txt的修改时间为7：00
  8：00时在touch a.txt;则a.txt的修改时间为8:00
  touch 虽然改变文件修改时间但不会改变文件内容

###cat:
cat主要有三大功能：
1. 一次显示整个文件:cat filename
2. 从键盘创建一个文件:cat > filename
3. 将几个文件合并为一个文件:cat file1 file2 > file

cat命令参数：
-A, --show-all           等价于 -vET
-b, --number-nonblank    对非空输出行编号
-e                       等价于 -vE
-E, --show-ends          在每行结束处显示 $
-n, --number     对输出的所有行编号,由1开始对所有输出的行数编号
-s, --squeeze-blank  有连续两行以上的空白行，就代换为一行的空白行 
-t                       与 -vT 等价
-T, --show-tabs          将跳格字符显示为 ^I
-u                       (被忽略)
-v, --show-nonprinting   使用 ^ 和 M- 引用，除了 LFD 和 TAB 之外

实例一：把 log2012.log 的文件内容加上行号后输入 log2013.log 这个文件里
命令：cat -n log2012.log log2013.log

实例二：把 log2012.log 和 log2013.log 的文件内容加上行号（空白行不加）之后将内容附加到 log.log 里。
命令：cat -b log2012.log log2013.log log.log


----------


####user account(用户信息):

cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
briup:x:1000:1000:briup,,,:/home/briup:/bin/bash 

每项以：分割,系统用户ID是从0到999，非系统用户ID在1000以后;
root(用户名):x:0(用户ID):0(组ID):root(组名):/root(家目录):/bin/bash(什么SHELL即echo $SHELL)

UID:用户ID user
GID:组ID group
sudo cat /etc/shadow: 密码（密码放在阴影（shadow）的地方）

whoami:显示当前登录用户名
who am i:显示登录情况
which:查看程序在哪执行的

####文件类型：
\-	普通文件(normal)
d	目录文件(directory)
c	字符设备文件(character) 以字符为单位读取
b	块设备文件(block)	以块单位读取
l	链接文件(link)
s	套节字文件(socket)	用于网络通信
p	管道文件(pipe)

file filename:显示文件类型

####ls:查看显示
ls -a:显示隐藏文件（hidden file): 以.作为文件名开头的文件为隐藏文件。 图形界面下用ctrl+h显示隐藏文件
ls -R:显示带有递归的（recursive）列表 
ls -t:以文件最后一次修改时间的顺序显示
ls -l:显示详细信息 (以字节为单位显示文件大小)
ls -lh（以人类（human）习惯的方式显示文件大小）
ls -l -t =ls -lt
ls -l -h =ls -lh

-rw-rw-r-- 1 user group 15  3月 21 09:16 a.txt
-rw(拥有者权限)-rw(同组人权限)-r(其他人权限)-- 1(链接号) user(拥有者) group(拥有组) 1024(文件大小)  3月 21 09:16(最后一次修改时间) a.txt(文件名)
其中:
拥有者:生成文件时登录的人,权限最高,u表示
同组人:系统管理员分配的同组的一个或几个人,g表示
其它人:除拥有者,同组人外的人,o表示
所有人:包括拥有者、同组人及其它人,a表示

####r:读权限
w:写权限,对目录来说,可生成文件与子目录或删除文件与子目录
x:执行权限,对目录来说,可查找该目录下内容
-:没有权限
例如:-rw-rw-r--

####chmod:修改权限（change permissions）
eg:
chmod g-r file1
chmod u+x,go+r file1
chmod a=rw file1

=:Set permissions
-:Remove access
+:Give access

rwx rwx rwx
000 000 000    chmod 000 a.txt

111 111 111
  7	 7	 7    
chmod 777 a.txt

r-x rw- --x
101 110 001
 5	 6	 1     
 chmod 561 a.txt

eg:
某文件的组外成员的权限为只读；所有者有全部权限；组内的权限为读与写，则该文件的权限为(D) 。
A. 467  B. 674   C. 476   D. 764


----------


####cp(复制)：
操作需要对源文件有读权限，拷贝后的文件权限和源文件无关，和新文件的创建权限有关
**1.copy files**
cp [-i]:提示信息 source_file destination_file 
cp [-i]:提示信息source_file(s) destination_directory 
eg:cp -i b.txt a.txt 如果a.txt存在且有内容则会提示是否覆盖a.txt的内容，如果不加-i 则不会提示，直接覆盖;
**2.copy directories**  需要加-r:不跳过目录
eg:cp -r src a.txt bin 如果不加-r只会把a.txt拷贝到bin目录下 src目录不会被拷贝

####rm(删除)
**1.remove files**
rm [-i] filename(s) : rm a.txt b.txt
**2.Removing Directories**
rmdir directory_name(s)  只能用来删目录且目录必须为空
rm -r[i] directory_name(s) -r不跳过目录且不管目录是否为空都删除


----------


####compress & uncompress

**Compress file（压缩文件）**

gzip filename 压缩后格式：filename.gz
解压： gunzip filename.gz

bzip2 filename  压缩后格式：filename.bz2
解压：bunzip2 filename.bz2

zip filename.zip filename(s) 压缩后格式:.zip

eg:
gzip file1  对单个文件进行压缩且压缩后原文件不再了
bzip2 file1 对单个文件进行压缩且压缩后原文件不再了
zip my.zip file1 file2 可以把多个文件进行打包，打包后原文件还存在；

压缩：对单个文件进行压缩成单个文件，压缩后文件变小
打包：对多个文件进行打包成单个文件，只打包不压缩文件大小不变

####Uncompress file（解压文件）
gunzip filename
bunzip2 filename
unzip filename
eg:
gunzip file1.gz
bunzip2 file1.bz2
unzip my.zip

**pack & unpack**
**Pack file （打包文件）**
tar [cvfz] package_name filename(s)
[cvf]中 c表示打包 ,vf表示提示信息，z表示压缩
例: tar cvf all.tar file1 file2 file3 (打包，带提示信息，不压缩)
tar cvfz all.tar.gz file1 file2 file3 (打包，带提示信息，压缩)

**Unpack file （解压文件）**
tar [xvfz] package_name
[xvfz]中x表示解压，vf表示带提示信息，z表示压缩文件。
例: tar xvf all.tar
tar xvfz all.tar.gz 默认解压到当前目录

**若解压到其他地方（路径），格式为:tar xvfz all.tar.gz -C路径**


----------


####链接文件：
**硬链接**:文件的另一个入口，和原文件一样指向硬盘上的数据块
**软链接:**保存了路径信息，大小和路径长度相同
ln [-s] source_file destination_file
[-s] 加-s创建软链接，不加默认创建硬链接。


touch 可不可以去创建已丢失的链接目标?
eg:
slink --> a.txt
a.txt的软链接为slink，但a.txt以删除；
touch slink 则会创建a.txt（里面没有内容了）,不会更新时间，但会创建丢失的链接目标。


----------


####man :查看命令的详细信息即用法
eg:mam touch即可查看touch的用法
或者用：touch --help 也可查看touch的用法