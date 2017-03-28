---
title: ubuntu下opensips安装配置
date: 2016-07-23 20:43:40
tags: SIP
categories: 网络通信
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>


### 1.opensips安装
opensips需要编译源码安装
官网：[http://opensips.org](http://opensips.org)
我下载的是:opensips-2.1.2_src.tar.gz

**安装依赖库：**

```
apt-get install gcc bison flex make openssl
libmysqlclient-dev perl libdbi-perl libdbd-mysql-perl
libdbd-pg-perl libfrontier-rpc-perl libterm-readline-gnu-perl
libberkeleydb-perl mysql-server ssh libxml2 libxml2-dev
libxmlrpc-core-c3-dev libpcre3 libpcre3-dev subversion
libncurses5-dev git ngrep libssl-dev
```

```
tar xcf opensips-2.1.2_src.tar.gz
cd opensips-2.1.2-tls/
sudo make menuconfig
```

在menuconfig中选择—>Configure Compile Options—> Configure Excluded Modules
方向键向下滚动，按空格选中[*] db_mysql

![这里写图片描述](http://img.blog.csdn.net/20160307191112407)

按q键返回上一级，选择—> Configure Install Prefix，输入/usr/local/opensips_proxy后按回车表示安装在/usr/local/opensips_proxy目录下。
![这里写图片描述](http://img.blog.csdn.net/20160307191359566)

选择 —> Save Changes 保存修改。
按q返回，选择 —> Compile And Install OpenSIPS，回车安装。

安装完成后会将配置文件放在/usr/local/opensips_proxy/etc/opensips目录下。运行文件在/usr/local/opensips_proxy/sbin目录下。
如果出现依赖错误，先通过apt-get安装依赖。
源代码安装软件要注意查看README，INSTALL等文件，这些文件里有很重要的说明和安装信息，里面有安装Opensips所需要的依赖包。

opensips安装之后的文件目录：

 - /sbin/中的可执行命令有如下：opensips 、opensipsctl  、  opensipsdbctl  、 opensipsunix
 -   /etc/opensips/中的配置文件有：opensips.cfg、opensipsctlrc和osipsconsolerc
 -  /lib/opensips/中的库文件有：modules和 opensipsctl两个目录。modules为当前opensips所支持的模块

### 2.opensips配置

opensips的配置文件都在/etc/opensips/中，分别为opensips.cfg、opensipsctlrc和osipsconsolerc。

opensips.cfg文件主要用于opensips启动的配置，所有应用功能的配置都在这个文件中说明。该配置文件主要由三个部分组成：

<font color=red>第一部分是全局变量，如：</font>

  listen=udp:127.0.0.1:5060

  disable_tcp=yes

  disable_tls=yes等。

<font color=red>第二部分主要用来加载模块，并设置相应参数，如：</font>

loadmodule "db_mysql.so"
loadmodule "auth.so"
loadmodule "auth_db.so"
modparam("auth", "calculate_ha1", yes)
modparam("auth_db", "password_column", "password")等。

<font color=red>第三部分主要是路由策略和功能应用，如：</font>

route[relay] {

 # for INVITEs enable some additional helper routes
if (is_method("INVITE")) {
	t_on_branch("per_branch_ops");
	t_on_reply("handle_nat");
	t_on_failure("missed_call");
}

opensipsctlrc文件中包含了数据库配置的信息。


(1)
进入/usr/local/opensips_proxy/etc/opensips目录，运行osipsconfig命令

```
cd /usr/local/opensips_proxy/sbin
sudo ./osipsconfig
```
依次选择—> Generate OpenSIPS Script —> Residential Script —> Configure Residential Script
选中如下几项
[*] ENABLE_TCP
[*] USE_ALIASES
[*] USE_AUTH
[*] USE_DBACC
[*] USE_DBUSRLOC
[*] USE_DIALOG
[*] USE_NAT
按q返回，选择 —> Generate Residential Script 回车，生成新的配置文件。按q(三次)退出命令
将新生成的opensips_residential_*.cfg文件重命名为opensips.cfg编辑

```
cd /usr/local/opensips_proxy/etc/opensips/
sudo mv opensips_residential_2016-3-7_16:19:22.cfg opensips.cfg
sudo vi opensips.cfg
```
总共改动了3处：
```
#
# $Id$
#
# OpenSIPS residential configuration script
#     by OpenSIPS Solutions <team@opensips-solutions.com>
#
# This script was generated via "make menuconfig", from
#   the "Residential" scenario.
# You can enable / disable more features / functionalities by
#   re-generating the scenario with different options.#
#
# Please refer to the Core CookBook at:
#      http://www.opensips.org/Resources/DocsCookbooks
# for a explanation of possible statements, functions and parameters.
#

####### Global Parameters #########

debug=3
log_stderror=no
log_facility=LOG_LOCAL0

fork=yes
children=4

/* uncomment the following lines to enable debugging */
#debug=6
#fork=no
#log_stderror=yes

/* uncomment the next line to enable the auto temporary blacklisting of 
   not available destinations (default disabled) */
#disable_dns_blacklist=no

/* uncomment the next line to enable IPv6 lookup after IPv4 dns 
   lookup failures (default disabled) */
#dns_try_ipv6=yes

/* comment the next line to enable the auto discovery of local aliases
   based on revers DNS on IPs */
auto_aliases=no

#改动第1处
#注意：xxx.xxx.xxx.xxx.xxx是你自己的IP地址，不可以写为127.0.0.1
listen=udp:xxx.xxx.xxx.xxx:5060   # CUSTOMIZE ME

listen=tcp:xxx.xxx.xxx.xxx:5060   # CUSTOMIZE ME 


####### Modules Section ########

#set module path
mpath="/usr/local/opensips_proxy//lib64/opensips/modules/"

#### SIGNALING module
loadmodule "signaling.so"

#### StateLess module
loadmodule "sl.so"

#### Transaction Module
loadmodule "tm.so"
modparam("tm", "fr_timeout", 5)
modparam("tm", "fr_inv_timeout", 30)
modparam("tm", "restart_fr_on_each_reply", 0)
modparam("tm", "onreply_avp_mode", 1)

#### Record Route Module
loadmodule "rr.so"
/* do not append from tag to the RR (no need for this script) */
modparam("rr", "append_fromtag", 0)

#### MAX ForWarD module
loadmodule "maxfwd.so"

#### SIP MSG OPerationS module
loadmodule "sipmsgops.so"

#### FIFO Management Interface
loadmodule "mi_fifo.so"
modparam("mi_fifo", "fifo_name", "/tmp/opensips_fifo")
modparam("mi_fifo", "fifo_mode", 0666)


#### URI module
loadmodule "uri.so"
modparam("uri", "use_uri_table", 0)
modparam("uri", "db_url",
	"mysql://opensips:opensipsrw@localhost/opensips") # CUSTOMIZE ME






#### MYSQL module
loadmodule "db_mysql.so"



#### USeR LOCation module
loadmodule "usrloc.so"
modparam("usrloc", "nat_bflag", "NAT")
modparam("usrloc", "db_mode",   2)
modparam("usrloc", "db_url",
	"mysql://opensips:opensipsrw@localhost/opensips") # CUSTOMIZE ME


#### REGISTRAR module
loadmodule "registrar.so"
modparam("registrar", "tcp_persistent_flag", "TCP_PERSISTENT")
modparam("registrar", "received_avp", "$avp(received_nh)")
/* uncomment the next line not to allow more than 10 contacts per AOR */
#modparam("registrar", "max_contacts", 10)

#### ACCounting module
loadmodule "acc.so"
/* what special events should be accounted ? */
modparam("acc", "early_media", 0)
modparam("acc", "report_cancels", 0)
/* by default we do not adjust the direct of the sequential requests.
   if you enable this parameter, be sure the enable "append_fromtag"
   in "rr" module */
modparam("acc", "detect_direction", 0)
modparam("acc", "failed_transaction_flag", "ACC_FAILED")
/* account triggers (flags) */
modparam("acc", "db_flag", "ACC_DO")
modparam("acc", "db_missed_flag", "ACC_MISSED")
modparam("acc", "db_url",
	"mysql://opensips:opensipsrw@localhost/opensips") # CUSTOMIZE ME


#### AUTHentication modules
loadmodule "auth.so"
loadmodule "auth_db.so"
modparam("auth_db", "calculate_ha1", yes)
modparam("auth_db", "password_column", "password")
modparam("auth_db|uri", "db_url",
	"mysql://opensips:opensipsrw@localhost/opensips") # CUSTOMIZE ME
modparam("auth_db", "load_credentials", "")


#### ALIAS module
loadmodule "alias_db.so"
modparam("alias_db", "db_url",
	"mysql://opensips:opensipsrw@localhost/opensips") # CUSTOMIZE ME






#### DIALOG module
loadmodule "dialog.so"
modparam("dialog", "dlg_match_mode", 1)
modparam("dialog", "default_timeout", 21600)  # 6 hours timeout
modparam("dialog", "db_mode", 2)
modparam("dialog", "db_url",
	"mysql://opensips:opensipsrw@localhost/opensips") # CUSTOMIZE ME


####  NAT modules
loadmodule "nathelper.so"
modparam("nathelper", "natping_interval", 10)
modparam("nathelper", "ping_nated_only", 1)
modparam("nathelper", "sipping_bflag", "SIP_PING_FLAG")
modparam("nathelper", "sipping_from", "sip:pinger@127.0.0.1") #CUSTOMIZE ME
modparam("nathelper", "received_avp", "$avp(received_nh)")

#改动第2处
#端口号22222可以随便修改，但是必须和后面安装的rtpproxy的端口号相同
loadmodule "rtpproxy.so"
modparam("rtpproxy", "rtpproxy_sock", "udp:localhost:22222") # CUSTOMIZE ME

#改动第3处
#增加MediaProxy模块
#增加下边这一段
#### MediaProxy module
loadmodule "mediaproxy.so"
modparam("mediaproxy", "disable", 0)
modparam("mediaproxy", "mediaproxy_socket", "/var/run/mediaproxy/dispatcher.sock")
modparam("mediaproxy", "mediaproxy_timeout", 1000)
modparam("mediaproxy", "signaling_ip_avp", "$avp(nat_ip)")
modparam("mediaproxy", "media_relay_avp", "$avp(media_relay)")
modparam("mediaproxy", "ice_candidate", "low-priority")

loadmodule "proto_udp.so"

loadmodule "proto_tcp.so" 


####### Routing Logic ########

# main request routing logic

route{
	force_rport();
	if (nat_uac_test("23")) {
		if (is_method("REGISTER")) {
			fix_nated_register();
			setbflag(NAT);
		} else {
			fix_nated_contact();
			setflag(NAT);
		}
	}
 	

	if (!mf_process_maxfwd_header("10")) {
		sl_send_reply("483","Too Many Hops");
		exit;
	}

	if (has_totag()) {
		# sequential request withing a dialog should
		# take the path determined by record-routing
		if (loose_route()) {
			
			# validate the sequential request against dialog
			if ( $DLG_status!=NULL && !validate_dialog() ) {
				xlog("In-Dialog $rm from $si (callid=$ci) is not valid according to dialog\n");
				## exit;
			}
			
			if (is_method("BYE")) {
				setflag(ACC_DO); # do accounting ...
				setflag(ACC_FAILED); # ... even if the transaction fails
			} else if (is_method("INVITE")) {
				# even if in most of the cases is useless, do RR for
				# re-INVITEs alos, as some buggy clients do change route set
				# during the dialog.
				record_route();
			}

			if (check_route_param("nat=yes")) 
				setflag(NAT);

			# route it out to whatever destination was set by loose_route()
			# in $du (destination URI).
			route(relay);
		} else {
			
			if ( is_method("ACK") ) {
				if ( t_check_trans() ) {
					# non loose-route, but stateful ACK; must be an ACK after 
					# a 487 or e.g. 404 from upstream server
					t_relay();
					exit;
				} else {
					# ACK without matching transaction ->
					# ignore and discard
					exit;
				}
			}
			sl_send_reply("404","Not here");
		}
		exit;
	}

	# CANCEL processing
	if (is_method("CANCEL"))
	{
		if (t_check_trans())
			t_relay();
		exit;
	}

	t_check_trans();

	if ( !(is_method("REGISTER")  ) ) {
		
		if (from_uri==myself)
		
		{
			
			# authenticate if from local subscriber
			# authenticate all initial non-REGISTER request that pretend to be
			# generated by local subscriber (domain from FROM URI is local)
			if (!proxy_authorize("", "subscriber")) {
				proxy_challenge("", "0");
				exit;
			}
			if (!db_check_from()) {
				sl_send_reply("403","Forbidden auth ID");
				exit;
			}
		
			consume_credentials();
			# caller authenticated
			
		} else {
			# if caller is not local, then called number must be local
			
			if (!uri==myself) {
				send_reply("403","Rely forbidden");
				exit;
			}
		}

	}

	# preloaded route checking
	if (loose_route()) {
		xlog("L_ERR",
		"Attempt to route with preloaded Route's [$fu/$tu/$ru/$ci]");
		if (!is_method("ACK"))
			sl_send_reply("403","Preload Route denied");
		exit;
	}

	# record routing
	if (!is_method("REGISTER|MESSAGE"))
		record_route();

	# account only INVITEs
	if (is_method("INVITE")) {
		
		# create dialog with timeout
		if ( !create_dialog("B") ) {
			send_reply("500","Internal Server Error");
			exit;
		}
		
		setflag(ACC_DO); # do accounting
	}

	
	if (!uri==myself) {
		append_hf("P-hint: outbound\r\n"); 
		
		route(relay);
	}

	# requests for my domain
	
	if (is_method("PUBLISH|SUBSCRIBE"))
	{
		sl_send_reply("503", "Service Unavailable");
		exit;
	}

	if (is_method("REGISTER"))
	{
		# authenticate the REGISTER requests
		if (!www_authorize("", "subscriber"))
		{
			www_challenge("", "0");
			exit;
		}
		
		if (!db_check_to()) 
		{
			sl_send_reply("403","Forbidden auth ID");
			exit;
		}

		if ( proto==TCP ||  0 ) setflag(TCP_PERSISTENT);

		if (isflagset(NAT)) {
			setbflag(SIP_PING_FLAG);
		}

		if (!save("location"))
			sl_reply_error();

		exit;
	}

	if ($rU==NULL) {
		# request with no Username in RURI
		sl_send_reply("484","Address Incomplete");
		exit;
	}

	
	# apply DB based aliases
	alias_db_lookup("dbaliases");

	

	 

	# do lookup with method filtering
	if (!lookup("location","m")) {
		if (!db_does_uri_exist()) {
			send_reply("420","Bad Extension");
			exit;
		}
		
		t_newtran();
		t_reply("404", "Not Found");
		exit;
	} 

	if (isbflagset(NAT)) setflag(NAT);

	# when routing via usrloc, log the missed calls also
	setflag(ACC_MISSED);
	route(relay);
}


route[relay] {
	# for INVITEs enable some additional helper routes
	if (is_method("INVITE")) {
		
		if (isflagset(NAT)) {
			rtpproxy_offer("ro");
		}

		t_on_branch("per_branch_ops");
		t_on_reply("handle_nat");
		t_on_failure("missed_call");
	}

	if (isflagset(NAT)) {
		add_rr_param(";nat=yes");
		}

	if (!t_relay()) {
		send_reply("500","Internal Error");
	};
	exit;
}




branch_route[per_branch_ops] {
	xlog("new branch at $ru\n");
}


onreply_route[handle_nat] {
	if (nat_uac_test("1"))
		fix_nated_contact();
	if ( isflagset(NAT) )
		rtpproxy_answer("ro");
	xlog("incoming reply\n");
}


failure_route[missed_call] {
	if (t_was_cancelled()) {
		exit;
	}

	# uncomment the following lines if you want to block client 
	# redirect based on 3xx replies.
	##if (t_check_status("3[0-9][0-9]")) {
	##t_reply("404","Not found");
	##	exit;
	##}

	
}



local_route {
	if (is_method("BYE") && $DLG_dir=="UPSTREAM") {
		
		acc_db_request("200 Dialog Timeout", "acc");
		
	}
}
```
(2)修改opensipsctlrc文件

```
cd /usr/local/opensips_proxy/etc/opensips
sudo vi opensipsctlrc
```
去掉所有DB相关的注释。
```
# $Id$
#
# The OpenSIPS configuration file for the control tools.
#
# Here you can set variables used in the opensipsctl and opensipsdbctl setup
# scripts. Per default all variables here are commented out, the control tools
# will use their internal default values.

## your SIP domain
 SIP_DOMAIN=xxx.xxx.xxx.xxx  #你自己的IP地址

## chrooted directory
# $CHROOT_DIR="/path/to/chrooted/directory"

## database type: MYSQL, PGSQL, ORACLE, DB_BERKELEY, or DBTEXT, 
## by default none is loaded
# If you want to setup a database with opensipsdbctl, you must at least specify
# this parameter.
 DBENGINE=MYSQL

## database host
 DBHOST=localhost

## database name (for ORACLE this is TNS name)
 DBNAME=opensips

# database path used by dbtext or db_berkeley
# DB_PATH="/usr/local/etc/opensips/dbtext"

## database read/write user
 DBRWUSER=opensips

## password for database read/write user
 DBRWPW="opensipsrw"

## database super user (for ORACLE this is 'scheme-creator' user)
 DBROOTUSER="root"

# user name column
# USERCOL="username"

```

为opensips新建数据库，增加域名及用户：

```
cd /usr/local/opensips_proxy/sbin/
sudo ./opensipsdbctl create
sudo ./opensipsctl domain add xdty.org #如果没有域名可不用添加
sudo ./opensipsctl add 10000 123456
sudo ./opensipsctl add 10001 123456
```

### 3.安装rtpproxy并配置

```
sudo apt-get install rtpproxy
sudo vi /etc/default/rtpproxy
```
修改为如下内容
```
# Defaults for rtpproxy

# The control socket.
#CONTROL_SOCK="unix:/var/run/rtpproxy/rtpproxy.sock"
# To listen on an UDP socket, uncomment this line:
CONTROL_SOCK=udp:127.0.0.1:22222

# Additional options that are passed to the daemon.
EXTRA_OPTS=""
LISTEN_ADDR=xxx.xxx.xxx.xxx #你自己的IP地址
EXTRA_OPTS="-l ${LISTEN_ADDR}"
```
启动rtpproxy

```
sudo killall rtpproxy
sudo /etc/init.d/rtpproxy start
```

注意：如果rtpproxy启动失败，请检查/etc/init.d/rtpproxy脚本DAEMON路径是否正确，默认为DAEMON=/usr/sbin/\$NAME，可能要改为DAEMON=/usr/bin/\$NAME

### 4.安装并配置mediaproxy
导入源密钥，增加mediaproxy的源到/etc/apt/sources.list

```
wget http://download.ag-projects.com/agp-debian-gpg.key
sudo apt-key add agp-debian-gpg.key
sudo vi /etc/apt/sources.list
```
最后位置添加

```
deb http://ag-projects.com/ubuntu precise main
deb-src http://ag-projects.com/ubuntu precise main
```
安装mediaproxy

```
sudo apt-get update
sudo apt-get install mediaproxy-dispatcher mediaproxy-relay mediaproxy-web-sessions
```
因media-relay需要内核支持ipv4 forwarding,所以需要执行(以root用户执行):

```
sudo su
echo 1 > /proc/sys/net/ipv4/ip_forward
```
<font color=red>注意：可以vi /proc/sys/net/ipv4/ip_forward查看里面是否为1,有时候还是为0，如果此时为0，继续执行一遍echo 1 > /proc/sys/net/ipv4/ip_forward，直到里面的内容为1</font>

在文件/etc/sysctl.config中打开net.ipv4.ip_forward=1 这样即便重启设备也可以运行mediaproxy了。

另外，media的dispatcher和relay之间需要通过tls通信，故需要在/etc/mediaproxy/tls/中有认证文件，进入/etc/mediaproxy/tls目录，拷本密钥文件，修改配置文件

```
cd /etc/mediaproxy/tls/
sudo cp /usr/share/doc/mediaproxy-common/tls/* .
cd ..
sudo vi config.ini
```
修改为类似如下内容

```
[Relay]
dispatchers = xxx.xxx.xxx.xxx #你自己的地址
passport = None
relay_ip = xxx.xxxx.xxx.xxx #你自己的IP地址
port_range = 50000:60000
log_level = DEBUG
stream_timeout = 90
on_hold_timeout = 7200
reconnect_delay = 10
traffic_sampling_period = 15
[Dispatcher]
socket_path = dispatcher.sock
listen = xxx.xxx.xxx.xxx
listen_management = xxx.xxx.xxx.xxx #你自己的IP地址
management_use_tls = yes
passport = None
management_passport = None
log_level = DEBUG
relay_timeout = 5
[TLS]
certs_path = tls
verify_interval = 300
[Database]
[Radius]
[OpenSIPS]
socket_path = /var/run/opensips/socket
max_connections = 10
```

启动mediaproxy服务

```
sudo media-dispatcher restart
sudo media-relay restart
```  
或者：

```
sudo service mediaproxy-dispatcher restart
sudo service mediaproxy-relay restart
```

查看mediaproxy是否正常运行：

```
ps -ef | grep media
```
如果出现：

```
root      6592  2110  0 14:25 ?        00:00:01 /usr/bin/python /usr/bin/media-dispatcher restart
root      6969  2110  0 14:35 ?        00:00:04 /usr/bin/python /usr/bin/media-relay restart

```
则说明已经正常启动。如果没有正常启动可以在/var/log/syslog中查看原因

### 5.启动服务并检验
修改日志文件配置，是opensips的日志保存在/var/log/opensips.log

```
sudo touch /var/log/opensips.log
sudo chmod 777 /var/log/opensips.log
sudo vi /etc/rsyslog.d/opensips.conf
```
增加如下内容

```
local0.*             /var/log/opensips.log
```
重启syslog服务，启动opensips

```
sudo service rsyslog restart
sudo /usr/local/opensips_proxy/sbin/opensipsctl start
```
如果启动失败，查看/var/log/opensips.log文件查找错误.

服务启动后，防火墙开启tcp及udp的端口

```
iptables -I INPUT -p tcp –dport 5060 -j ACCEPT
iptables -I INPUT -p udp –dport 5060 -j ACCEPT
iptables-save
```

好了，现在可以用SIP客户端登录上面创建的帐号，测试了。

<font color=red>注意：</font>
当Linphone客户端启用stun和ICE时，处于内网的客户端不能呼叫处用公网的客户端。

使用csipsimple则不会出现Linphone客户端出现的问题，但是csipcimple需要勾选 设置--->网络--->Use compact SIP选项。

转载自：[ubuntu12.04安装配置opensips，搭建voip服务器](https://www.xdty.org/1709)

