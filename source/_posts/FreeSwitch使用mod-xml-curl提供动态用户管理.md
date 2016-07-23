---
title: FreeSwitch使用mod_xml_curl提供动态用户管理
date: 2016-07-24 00:44:26
tags: VoIP
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 一、实现思路　　
FreeSWITCH默认使用静态的XML文件配置用户，但如果要动态认证，就需要跟数据库相关联。FreeSWITCH通过使用mod_xml_curl模块完美解决了这个问题。它的实现思路是你自己提供一个HTTP服务器，当有用户有注册请求时(或INVITE或其它，总之需要XML的请求时)，FreeSWITCH向你的HTTP服务器发送请求，你查询数据库生成一个标准的XML文件，FreeSWITCH进而通过这一文件所描述的用户信息对用户进行认证。

　　The Logic is simple. Incoming SIP requests are received by Freeswitch which asks a web-service about the details of the user. The web-server just queries data and if any relevant details are found returns them in XML format. Freeswitch accepts or reject the SIP request depending upon the XML response.


  <center>![](http://i.imgur.com/0CMNXpN.png)</center>

Another way to represent the above flowchart.

<center>![](http://i.imgur.com/oCUkIfI.png)</center>


## 二、步骤

### 1.安装Apache+php5+mysql+phpmyadmin

### 2.FreeSwitch加载mod_xml_curl模块
(1)在源码目录下：

    cd /usr/src/freeswitch
    make mod_xml_curl && make mod_xml_curl-install

(2)Then enable the module in modules.conf

    cd /usr/local/freeswitch/conf/
    vim /usr/local/freeswitch/conf/autoload_configs/modules.conf.xml

Uncomment the following tag by removing the  "<!--"and "--\>"

    <!-- <load module="mod_xml_curl"/> -->

### 3.为mod\_xml\_curl绑定网关

Edit the settingfor the XML_CURL module.

    vim /usr/local/freeswitch/conf/autoload_configs/xml_curl.conf.xml

The most importthing is to set the web-service URL where FS will send all the queries.

	

	<configuration name="xml_curl.conf" description="cURL XML Gateway">
	  <bindings>
		<binding name="directory">
        	<param name="gateway-url" value="http://localhost/fs_curl/directory.php" bindings="directory"/>
 		</binding>
		</bindings>
	</configuration>


In my case the web-server is on localhost and I plan on using the sub directory fs_curl for the php pages.


### 4.编写(万能脚本)directory.php

	<?php
 	  $user =  $_POST['user'];
 	  $domain = $_POST['domain'];
	  $password = "1234";
	?>

	<document type="freeswitch/xml">
		<section name="directory">
			<domain name="<?php echo $domain;?>">
	  			<params>
					<param name="dial-string" value="{presence_id=${dialed_user}@${dialed_domain}}${sofia_contact(${dialed_user}@${dialed_domain})}"/>
 			    </params>
				<groups>
					<group name="default">
 					 	<users>
							<user id="<?php echo $user; ?>">
 								<params>
									<param name="password" value="<?php echo $password; ?>"/>
									<param name="vm-password" value="<?php echo $password; ?>"/>
								</params>
 	 							<variables>
									<variable name="toll_allow" value="domestic,international,local"/>
									<variable name="accountcode" value="<?php echo $user; ?>"/>
									<variable name="user_context" value="default"/>
									<variable name="effective_caller_id_name" value="FreeSWITCH-CN"/>
									<variable name="effective_caller_id_number" value="<?php echo 	$user;?>"/>
									<!-- <variable name="outbound_caller_id_name" value="$${outbound_caller_name}"/> -->
									<!-- <variable name="outbound_caller_id_number" value="$${outbound_caller_id}"/> -->
									<variable name="callgroup" value="default"/>
									<variable name="sip-force-contact" value="NDLB-connectile-dysfunction"/>
									<variable name="x-powered-by" value="http://www.freeswitch.org.cn"/>
  								</variables>
							</user>
  						</users>
					</group>
  				</groups>
			</domain>
  		</section>
	</document>
将上面的directory.php放到**/var/www/html//fs_curl/**目录下。

然后，

    reloadxml
    reload mod_xml_curl


通过上述脚本变可生成XML并返回给FreeSWITCH，然后FreeSWITCH在解析该XML并进行下一步的认证等操作。因为该脚本根本不查询数据库，也没有任何业务逻辑，任何注册或呼叫请求只要密码是1234就都能通过验证，因此，我们可以把它称做**万能脚本**.当然，熟悉PHP的读者可以自己尝试添加一些if...else类的条件判断以针对不同的用户返回不同的XML，或者通过查询数据库构造不同的XML。



### 5. directory.php中通过查询数据库来生成用户的XML验证

(1)创建数据库表，用户存储用户名和密码

创建数据库表：users,用来存储所有用户的用户名和密码

<center>![](http://i.imgur.com/c4PaBWu.png)</center>

(2) 重写上面的万能脚本directory.php,从数据库中查询用户的验证密码。

	<?php
  		$user =  $_POST['user'];
  		$domain = $_POST['domain'];

  		$host = "127.0.0.1";
  		$dbroot = "root";
  		$dbpwd = "root";
  		$dbname = "freeswitch";
  		$dbtable = "users";

  		$con = mysql_connect($host,$dbroot,$dbpwd);
  		if ($con == false)
  		{
   			 echo "connect databse failuer!";
  		}
  		mysql_select_db("$dbname", $con);
  		$result = mysql_query("SELECT * FROM $dbtable WHERE name = '$user'");

  		//echo $result，will error
 		 while($row = mysql_fetch_array($result))
  		{
    		$password = $row['passwd'];
 		 }
      
  		# echo $password;
   
   		mysql_close($con);
	?>
	//后面的与上面的directory.php


## 三、调试

如果在测试过程中遇到错误，试着看一看出错的内容，一般是HTTP服务器无法正常访问造成的。如果没有直接看到错误，可以在控制台上打开mod\_xml\_curl的调试选项，如：

    freeswitch> xml_curl debug_on

然后，FreeSWITCH会将每次请求得到的XML文件保存到文件名类似/tmp/xxx.xml的文件中，检查一下里面的内容是否跟你HTTP服务预期的输出是一致的。


