---
title: openfire之SSL认证
date: 2016-07-23 22:04:20
tags: openfire
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

原先Android版 asmack连接服务器时默认已经实现了SSL认证，但是最新版Smack 4.1 以及以上版本没有实现SSL认证。

Smack 4.1 以及以上版本实现SSL认证方法：

无意中发现github上有个开源项目，可以进行SSL认证。

项目地址：[A "plugin" for Android Java to allow asking the user about SSL certificates](https://github.com/ge0rg/MemorizingTrustManager)


实现代码：

```java
SmackConfiguration.DEBUG = true;
XMPPTCPConnectionConfiguration.Builder configBuilder = XMPPTCPConnectionConfiguration.builder();
//设置服务器IP地址
configBuilder.setHost(host);
//设置服务器端口
configBuilder.setPort(port);
//设置服务器名称
configBuilder.setServiceName(serviceName);
//设置开启调试
configBuilder.setDebuggerEnabled(true);
//设置开启压缩，可以节省流量
configBuilder.setCompressionEnabled(true);

//SSL认证
try {
	SSLContext sc = SSLContext.getInstance("TLS");
    MemorizingTrustManager mtm = new MemorizingTrustManager(ctx);
    sc.init(null, new X509TrustManager[]{mtm}, new java.security.SecureRandom());
    configBuilder.setCustomSSLContext(sc);
    configBuilder.setHostnameVerifier(
		mtm.wrapHostnameVerifier(new org.apache.http.conn.ssl.StrictHostnameVerifier()));
} catch (NoSuchAlgorithmException|KeyManagementException e) {
    e.printStackTrace();
}

XMPPTCPConnection connection = new XMPPTCPConnection(configBuilder.build());
```


