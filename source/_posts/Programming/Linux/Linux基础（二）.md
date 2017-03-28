---
title: Linux基础（二）
date: 2016-07-24 00:16:27
tags: Linux
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

###通配符
1. \*：represents zero or more characters
eg:找出/etc下包含init关键字的文件  ls -l *init*
2. ？：represents any single character
eg：ls ?.txt
3. []：match a set or range of characters to a single character position
eg:ls [b-f]b*  [b-f]代表bcdef中的任意一个字符
ls [fF]f*a   [fF]代表fF中的任意一个字符
4. ;：enter multiple commands on a command line
eg: cd;ls
date(显示时间);cal（日历）;pwd
5. |：(piping(管道))：Takes the output of one command and passes it
as input into a following command （把一个命令的输出结果作为输入传递给下一个命令）
eg: ls -l /etc | more
more代表分页显示，即把ls -l /etc的结果，作为more的输入，将结果分页显示在屏幕上

_ _ _

###重定向
1. <	输入重定向 
2. \>	输出重定向 
3. \>>  输出重定向(追加)
4. 2>	错误重定向(输出)

cat 把标准输入（键盘）的内容输出到标准输出（屏幕或终端）上
标准输入（键盘） --> 标准输出（屏幕或终端）

cat > a.txt
标准输入 --> a.txt（输出重定向为a.txt） 

cat < /etc/passwd > a.txt 将/etc/passwd内容写入a.txt
/etc/passwd --> a.txt （输出重定向为a.txt）
(输入重定向为/etc/passwd)

cat >> a.txt(a.txt的内容不会被覆盖，而是追加)
(cat /etc/passwd;ls -l aaa.txt)>a.txt 2>err.log (假设没有aaa.txt文件)

ls -l aaa.txt 输出为 ls:无法访问aaa.txt:没有那个文件和目录
将/etc/passwd的内容输入到a.txt，

将ls -l aaa.txt的错误信息输入到err.log

/dev/null 黑洞,即将无用的文件等仍在里面。


_ _ _

###awk(分割)

cat /etc/passwd | awk -F: '{print \$1 "\t" $6}' \ 
| sort > ~/userinfo 

awk -F: '{print \$1 "\t" \$6}'
awk为分割命令，-F是以什么分割，print是打印 ，\是续行符
$1表示分割后的第一个字符，\$6分割后的第六个字符
即将cat /etc/passwd的内容以:分割，并以\$1 “\t” \$6的形式打印
sort是排序，把排序后的内容重定向到~/userinfo下

eg：将当前系统中所有的用户组及GID过滤出来并排序输出到一个文件grp.log中
cat /etc/passwd |awk -F:'{print \$6 “\t” \$4}' \
| sort > grp.log


_ _ _
###more(分页)

more(分页)：Displays the contents of a text file one screen at a time

格式 : more filename(s）

分页后的操作:
Spacebar  Scroll to the next screen
Return    Scroll one line at a time
b         Move back one screen
f         Move forward one screen
h         Display a Help menu of more features
q         Quit and return to the shell prompt
/string   Search forward for string
n         Find next occurrence of string

_ _ _

###head&tail


Displays the first or last n lines of one or more 
files.
Displays first or last 10 lines by default (不加[-n]即默认情况下去10行)

格式：head [-n] filename(s)
tail [-n] filename(s)

eg: head -30 a.txt 取a.txt内容的前30行

_ _ _

###sort
sort(分类) but does not change the original file.
格式： sort [-option] filename

-u:Generates a sorted list in which each line is  unique (no duplicates). 即取出重复行 unique


_ _ _
###uniq

uniq(取出重复行) but does not change the original file.

格式：uniq filename

_ _ _
###diff
Compares two files and displays a list of thedifferences between them.It is useful when you want to compare two versions of a letter or a report or two versions of the source code for a program.

diff [-u] filename1 filename2
diff –u file1 file2

_ _ _
### file
Learn about the contents of any file on a file system without having to open and examine the file yourself.

格式：file filename

_ _ _
### echo
Copies anything you put on the command line, after echo, to the screen.Copies A good tool for learning about the shell

echo [-option] message:
-e:Enables the interpretation of backslash escape
sequences such as \n. (使转义字符可用)

-n:Suppresses the NEWLINE terminating the message. （不换行）
echo “hello world.”
echo –e “hello\tworld.”
echo –n “hello world.”

_ _ _
### sricpt
Records(记录) all or part of a login session, including your input and the system's responses（回应）.

格式：
script [-a] filename
例如：
script file1  (script记录开始，将屏幕上的命令记录到file1)
...commands
exit  （退出script,注意只有退出后，才能查看到file1中的记录内容）
cat file1

_ _ _
### grep
Searches a file for a specified text string and prints all lines that contain that pattern to the screen .

格式：grep [option(s)] string filename

case sensitive (大小写敏感)
-i Ignore case of string when searching （忽略大写小敏感）
-v Search for all lines that do not match string (反选)

eg: grep root /etc/passwd  查找/etc/passwd中包含root的内容
ls -la | grep -i ’sep 1’ 查找ls -la下最后修改时间为9月1号的内容

_ _ _
### wc(一般用于统计)
Displays a line, word, or character count（数目） of a file.

格式：wc [options] filename(s)
-l: Counts lines
-w: Counts words
-c: Counts characters

eg: grep wang /etc/passwd | wc -l

统计出当前系统中所有使用bash登录的用户数
grep bash /etc/passwd | wc -l

_ _ _

