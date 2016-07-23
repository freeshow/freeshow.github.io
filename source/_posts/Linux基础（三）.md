---
title: Linux基础（三）
date: 2016-07-24 00:17:11
tags: Linux
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

系统命令一般存在如下命令下：
/usr/bin, /usr/sbin, /usr/local/bin,... 

###id
Displays the user name corresponding to the effective user ID .

格式: id [option(s)] user_name
-a: 显示所有信息

eg:
id          显示当前用户ID
id root 		显示root用户的ID
id  -a root

###Finding People 
who 
w 
finger 

The who am i command displays information about your real user ID 
who am I 
whoami  

###find
Find files based on specific criteria, then execute a command on the matching files 

**格式：**
find path expression [action] 
path:路径，expression:表达式.

eg:
find / -name “perl*”          -name:命名    	查找/目录下以 perl*命名的文件      

find . -mtime 10 –print 		 -mtime:最后修改时间  -print:打印	查找.目录下最后修改时间超过10的文件并打印 	

find /etc -user 0 –size +400 –print  
-user:用户ID，-size:块大小
查看/etc下 用户ID为0的用户块大小大于400的文件并打印

find ~ -perm 777 > ~/holes   
-perm:权限   把权限为777的文件重定入到~/holes下

find /export/home -type f -atime +365 -exec rm {} \
-type:类型 ，-atime：最后访问时间，-exec:执行命令,rm {} \\:删除操作。  把/export/home下文件类型为f最后一次访问时间超过365天的文件，执行删除命令。
find /export/home -type f -atime +365 |	 xargs rm (另一种删除方法)

find /export/home/html -name "*.html" -print | xargs perl -p -i .bak -e “s/Copyright 2004/Copyright 2005/g;”  将print出来的内容中的Copyright 2004替换成Copyright 2005并将未替换是的内容备份到.bak中

###du&df
####du
du file1 	查看文件和目录（即目录下文件）的实际大小
du –sk(-sh) file1 	只查看本文件的实际大小
用ls -l 查看的目录大小全是4096

####df	
df 查看但前挂载情况
df -k	以K为单位显示
df -h	以人类单位显示

###ps
ps (Process):list the processes currently running on the system .

-e: List every process now running (列出全部进程)
-f :Generates a full listing 			（建成表）

ps –ef | grep telnet 

###kill
Terminate unwanted command processes that you cannot quit in the normal way 

格式：
kill [-signal] pid  给进程发信号
pkill 进程名

eg: 
如运行程序：sleep 100
pkill sleep
kill 12932 
kill -9 12418 
sleep 100 &		&:后台运行	

PID		进程号 
PPID	父进程号

 ###Job Control(任务管理)

jobs 	Display which jobs are currently running 

fg %n ：Place a job in the foreground （调度到前台）   n:任务号
bg %n 	：Place a job in the background （调度到后台）
kill -stop %n ：Suspend（悬挂，暂停） a background job （挂起后台）
kill %n ：Abort（中断） the specified background job 
Control+c ：Abort the foreground job 
Control+z ：Suspend the foreground job. 挂起前台任务，就会被调度到后台，但以停止.

###Port numbers （端口号）
所有端口信息都存在于/etc/services文件中
 
ping -s host2 	测试连通性
ifconfig 		查看IP信息等
netstat –rn 		查看路由表
traceroute www.sina.com.cn 查看路由信息

###telnet&ftp

####telnet: 远程登录   
与ftp不同之一是telnet不能从服务器上进行传输功能 即不能下载
telnet 172.16.0.50  远程登录后仍然可以使用shell命令

####ftp
FTP的文件传输共享等	ftp登录后不可以使用shell命令了,有自己的命名

ftp 172.16.0.50

Commands: 

- cd（sever）, lcd (本地)
- dir   	= ls
- bye    结束
- bin, asc 	传输方式  bin:以二进制形式传输 asc:asc码文本形式传输  
- get, put, mget, mput 	get:下载，put：上传，mget,mput(针对目录)
- hash 	打开和关闭hash图功能
- prompt 打开和关闭提示功能 

**User Communication Programs(=QQ)**
**wall （广播）**
格式：
1.wall 
标准输入
 ctrl+d 结束输入 

2.echo "hello" | wall  
管道 

3.wall < /etc/passwd 
重定向 

write:向某个用户发信息

格式：
write user_name

将wall换成user_name用法与wall相同

mesg y ：接受信息
mesg n:不接受信息


