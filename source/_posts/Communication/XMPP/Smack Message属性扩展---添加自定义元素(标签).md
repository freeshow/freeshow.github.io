---
title: Smack Message属性扩展--添加自定义元素（标签）
date: 2016-07-23 20:40:50
tags: XMPP
categories: 网络通信
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

Smack框架对XMPP协议进行了封装，从而方便与Openfire即时通信服务器做交互。说白了，Smack框架可以通过对象构造符合XMPP协议的XML字符串，避免手动拼接字符串。

> XMPP协议基本XML结构如下：

```
<message from='发送方jid' to='接收方jid' type='消息类型(普通消息/群聊)'>
   <body>消息内容</body>
</message>
```

大多数情况下，这么简单的结构是满足不了需求的，我们可能会尝试向message元素下增加子元素，用来描述更多信息。

     比如，除了发送方的jid，我们想直接带上发送方的昵称和头像URL，这样可以避免反复从数据库中查询这些基本信息。但这个看似简单的过程，在Smack中实现的却相当隐晦，接下来直接通过代码说明。

例如，实现如下XML结构：

```
<message id='76Ws9-11'>
    <body>hello 你好</body>
    <userinfo xmlns="com.xml.extension">
        <name>菜鸟</name>
        <url>http://www.liaoku.org/</url>
    </userinfo>
</message>
```

Message扩展类UserInfo4XMPP 定义：

```
public class SDPExtensionElement implements ExtensionElement {
    public static final String NAME_SPACE = "com.xml.extension";
    //用户信息元素名称
    public static final String ELEMENT_NAME = "userinfo";

    //用户昵称元素名称
    private String nameElement = "name";
    //用户昵称元素文本(对外开放)
    private String nameText = "";

    //用户头像地址元素名称
    private String urlElement = "url";    
    //用户头像地址元素文本(对外开放)
    private String urlText = "";

	public String getNameText() {
       return nameText;  
    } 
    public void setNameText(String nameText) {
        this.nameText = nameText;
    }
 
    public String getUrlText() {
        return urlText;
    }
    public void setUrlText(String urlText) {
        this.urlText = urlText;
    }

    @Override
    public String getNamespace() {
        return NAME_SPACE;
    }

    @Override
    public String getElementName() {
        return ELEMENT_NAME;
    }

    @Override
    public CharSequence toXML() {
        StringBuilder sb = new StringBuilder();

        sb.append("<").append(ELEMENT_NAME).append(" xmlns=\"").append(NAME_SPACE).append("\">");
        sb.append("<" + nameElement + ">").append(nameText).append("</"+nameElement+">");
        sb.append("<" + urlElement + ">").append(urlText).append("</"+urlElement+">");
        sb.append("</"+ELEMENT_NAME+">");

        return sb.toString();
    }
}

```

简单说明下，关键是实现ExtensionElement接口，然后实现自己的toXML方法，将要扩展的XML字符串返回即可，此字符串将作为message元素的子元素。

> 发送消息基本流程

```
//build chat
Chat chat = chatManager.createChat("对方jid");

//build extension
UserInfo4XMPP userInfo4XMPP = new UserInfo4XMPP();
userInfo4XMPP.setNameText("菜鸟");
userInfo4XMPP.setUrlText("http://www.liaoku.org/");

//build message
Message message = new Message();
message.setBody("hello 你好");  //消息内容
message.addExtension(userInfo4XMPP);  //添加扩展内容

//send
chat.sendMessage(message);
```

> 接收消息流程

方法一：直接判断接收到的Chat Message
```
//设置聊天对象管理处理监听。
        getChatManager().addChatListener(chatManagerListenerMain);
//创建聊天对象管理监听器,监听从远端发送过来的message。
    private ChatManagerListener chatManagerListenerMain = new ChatManagerListener() {
        @Override
        public void chatCreated(Chat chat, boolean b) {
            chat.addMessageListener(new ChatMessageListener() {
                @Override
                public void processMessage(Chat chat, Message message) {
                    android.os.Message msg = android.os.Message.obtain();
                    msg.obj = message;
                    myHandler.sendMessage(msg);
                }
            });
        }
    };


//......
//当接收到Chat Message时，进行判断

 if (message.hasExtension(UserInfo4XMPP.ELEMENT_NAME,UserInfo4XMPP.NAME_SPACE))
 {
    //<userinfo xmlns="com.xml.extension">
    //    <name>菜鸟</name>
    //    <url>http://www.liaoku.org/</url>
   // </userinfo>


   DefaultExtensionElement defaultExtensionElement = message.getExtension(VideoInvitation.ELEMENT_NAME,VideoInvitation.NAME_SPACE);
   String name = defaultExtensionElement.getValue("name");  //菜鸟
   String url = defaultExtensionElement.getValue("url");   //http://www.liaoku.org/
}

```

方法二：

[Smack Message 扩展属性](http://www.xuebuyuan.com/532432.html)


参考自：[Smack Message扩展，添加自定义元素(标签)经验分享](http://www.cnblogs.com/iyangyuan/p/4496015.html?utm_source=tuicool)