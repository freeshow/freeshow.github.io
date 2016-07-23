---
title: FreeSwitch压力测试
date: 2016-07-24 00:45:54
tags: VoIP
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 一.安装sipp
这一步很简单，按照sipp的官网上的安装步骤安装即可。

## 二.通过mod\_xml\_curl为FreeSwitch添加动态账户验证。
### 1. 配置mod_xml_curl添加动态账号验证，前面博客已写。
### 2. 往数据库中添加注册账号。

创建存储过程addUsers(),向freeswitch数据库的users中添加100000条用户：
使用Navicate,点击 `Function --> New Function -->Proceduce` 创建addUsers存储过程。
<center>![](http://i.imgur.com/Lr0WoEi.png)</center>

addUser()函数如下：

	BEGIN
		#Routine body goes here...
		DECLARE i INT;
		SET i = 1;

		WHILE i <100000  do
			INSERT INTO users(name,passwd) VALUES(i,'1234'); 
			SET i=i+1;
		END WHILE;
	END

然后点击按钮run,就可在users表中添加100000条注册账户。


## 三. FreeSWITCH注册的压力测试。

### 1.编写注册脚本reg.xml

	<?xml version="1.0" encoding="ISO-8859-1" ?>
		<scenario name="Basic Sipstone UAC">
 			<send retrans="500">
				<![CDATA[
  					REGISTER sip:[remote_ip] SIP/2.0
  					Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
  					From: <sip:[field0]@[field1]>;tag=[call_number]
  					To: <sip:[field0]@[field1]>
  					Call-ID: [call_id]
  					CSeq: [cseq] REGISTER
  					Contact: sip:[field0]@[local_ip]:[local_port]
  					Max-Forwards: 10
  					Expires: 3600
  					User-Agent: SIPp/Win32
  					Content-Length: 0
				]]>
  		</send>

  		<!-- kamailio -->
  		<recv response="100" optional="true"></recv>

  		<recv response="401" auth="true"></recv>

  		<send retrans="500">
			<![CDATA[
				REGISTER sip:[remote_ip] SIP/2.0
  				Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
  				From: <sip:[field0]@[field1]>;tag=[call_number]
  				To: <sip:[field0]@[field1]>
  				Call-ID: [call_id]
  				CSeq: [cseq] REGISTER
  				Contact: sip:[field0]@[local_ip]:[local_port]
  				[field2]
  				Max-Forwards: 10
  				Expires: 3600
  				User-Agent: SIPp/Win32
  				Content-Length: 0
			]]>
		</send>


### 2. 编写freeswitchreg.csv

添加100000条账户。

    SEQUENTIAL
	1;freeswitch.org;[authentication username=1 password=1234]
	2;freeswitch.org;[authentication username=2 password=1234]
	3;freeswitch.org;[authentication username=3 password=1234]
	4;freeswitch.org;[authentication username=4 password=1234]
	5;freeswitch.org;[authentication username=5 password=1234]
	6;freeswitch.org;[authentication username=6 password=1234]
	7;freeswitch.org;[authentication username=7 password=1234]
	8;freeswitch.org;[authentication username=8 password=1234]
	9;freeswitch.org;[authentication username=9 password=1234]
	10;freeswitch.org;[authentication username=10 password=1234]
	...... //100000条用户

其中，freeswitch.org替换为你freeswitch服务器的ip地址(当然去掉也可以)。

### 3.使用sipp进行FreeSwitch注册的压力测试

**执行命令**：

	sudo ./sipp -sf reg.xml -inf freeswitchreg.csv -i sippIp -p 5555 -r 100 -d 100000 freeswitch.org

- sf：指定注册脚本
- -inf：指定注册用户信息
- -i：sipp所在主机地址
- -p：sipp端口号
- -r：每秒发送多少sip消息
- -d：sip消息持续时间(ms)
- freeswitch.org：freeswitch服务器所在地址，默认为5060端口号，如果修改了端口号，则格式为freeswitch.org:port


**测试结果**:

在freeswitch控制台中输入：
	
	sofia status profile internal reg

测试效果如下图：
<center>![](http://i.imgur.com/j65iJIk.png)</center>

## 四、FreeSwitch呼叫的压力测试

### 1.修改FreeSWITCH的dialplan

	/usr/local/freeswitch/conf/dialplan
	vi public.xml

修改如下：

	<extension name="public_extensions">
      <condition field="destination_number" expression="^(.*)$">
        <action application="transfer" data="$1 XML default"/>
      </condition>
    </extension>
 
即将原先的只能呼叫freeswitch中的1000-1019的默认20个用户，改为呼叫所有用户，即`^(.*)$`

同样，修改`default.xml`"，修改如下：

	<extension name="Local_Extension">
      <condition field="destination_number" expression="^(.*)$">
        <action application="export" data="dialed_extension=$1"/>
		......

### 2.使用sipp进行呼叫测试

**执行命令**：

	sipp -sn uac -r 30 -d 100000 -rtp_echo freeswitch.org:5080

- freeswitch.org为freeswitch所在主机的地址。

**测试效果如下**：
<center>![](http://i.imgur.com/QnkSKiG.png)</center>

