---
title: linux基础（四）
date: 2016-07-24 00:17:56
tags: Linux
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

#Setup Initialization Files

##Features of Initialization Files 
Initialization files contain commands and variable settings that are executed when a shell is started .

Two levels: 

- System-wide 	– Reside in the /etc directory 			(系统范围的放在/etc下)
- User-specific 	– Reside in a user’s home directory 	(用户范围的放在～/下) 

| Shell | System-wide(read first) | User-specific(read second) |
| :-------------: |:-------------:| :-----:|
| Bash | /etc/profile | ~/.bashrc |

###source
source 加载某个配置文件,使配置文件生效 
source ~/.bashrc 

注意：编辑配置文件后，只有source配置文件后，修改的配置文件才会生效。

###alias(别名)
alias   l='ls -l'  (将ls -l设置成l)
alias   c='clear'

unalias (取消设置)

###Variable
variable=value  (设置环境变量的值)

echo $PATH (显示PATH环境变量的值)

eg:
1.WTO='uname -n'
	echo $WTO (则显示 uname -a)
2.WTO=`uname -n`  (`:数字键1左边的键，它会把uname -n的执行结果赋给环境变量）
echo &WTO		（则显示uname -n的执行结果）	

###which&whereis
 which：查看命令在那个路径下

eg:
which vi	(则显示/user/bin/vi)

whereis 比which查找的更加详细，which只查找当前使用的命令的路径，whereis查找系统中所有与命令有关的文件中的路径。

###history
eg:
history (显示最近敲过的命令)
history 30 (显示最近敲过的30条命令)
！n  (执行最近敲过的以n开头的命令)
！！	(执行上次敲过的命令)

###umask(系统掩码)
umask 系统掩码 --> 0002 

与文件的默认创建权限有关 
文件创建权限=默认权限-umask掩码 
文件 --> 664 = 666 - 002 
目录 --> 775 = 777 - 002 

umask 022 (将系统掩码设置成022)

##用户组管理	
/etc/group 系统中用户组的信息 

###创建组 
参数: -g GID : 指定 GID 的值 

eg: 
groupadd g1 (创建组名为g1的组)
groupadd g1 -g 2000 (创建组名为g1，组号为2000的组)
	
###删除组 

格式: groupdel 组名 

eg:
 groupdel g1 

###修改组信息 
格式: groupmod [ 参数 ] 组名 

参数:
 -n 新组名:修改组的名字 
-g GID :修改组的 GID 

eg:
groupmod g1 -n g2 -g 2001 (将组名为g1的组修改组名为g2,组号为2001)

###添加/删除组成员 
格式: gpasswd [ 参数 ] 组名 

参数:
 -a 用户名 : 向指定组添加用户 
-d 用户名 : 从指定组中删除用户 

eg:
gpasswd -a u1 root (向root组中添加用户u1)
gpasswd -d u1 root (向root组中删除用户u1)

###查看用户所属组 
格式: groups [ 用户名 ] 

eg:
groups root (显示 root 用户的所属组) 

##用户管理
###创建用户 
格式: useradd [ 参数 ] 用户名 

参数:
 -u UID :指定用户的 UID 值 (默认为最小的可用ID) 
-g 组名:指定用户的所属组 (不指定则默认创建与用户名相同的组) 
 -d 路径:指定用户主目录 (不指定则默认为/home/下与用户名同名的目录 ) 
 -s SHELL :指定 SHELL 的类型 (默认为/bin/sh) 
 -m :建立用户主目录 

eg: 
useradd -m -d /home/u1 -g other -s /bin/csh u1 

###删除用户 
格式: userdel [ 参数 ] 用户名 

参数: -r : 删除用户主目录 

eg: userdel -r u1 

###修改用户信息 
格式: usermod [ 参数 ] 用户名 

参数:
 -l 新的用户名:修改用户名 
 -d 路径:修改用户主目录 

例:
 usermod -d /home/u2 u1 
 usermod -l u2 u1 

##文件所属
###chown 
功能:改变文件拥有者 
格式: chown [ 参数 ]< 用户名 >< 文件名 > 

参数:
-R :递归改变目录的拥有者 
-f :不显示拥有者的详细信息 

例:
 chown u1 file1 
 chown -R u1 /dir1 
chown lisi:root a.txt (将文件a.txt的拥有者改为lisi,拥有组改为root)

###chgrp 
功能:改变文件所属组 

格式: chgrp [ 参数 ]< 组名 >< 文件名 > 

参数: 
-R :递归改变目录的拥有者 
-f :不显示拥有者的详细信息 

例:
chgrp g1 file1 
chgrp -R g1 /dir1 

修改文件拥有者和所属组的限制 :
1.root全部都能改 
2.文件的拥有者可以把文件递交给其他的人或组 