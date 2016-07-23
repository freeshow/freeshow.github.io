---
title: WebRTC工作流程
date: 2016-07-23 22:20:28
tags: WebRTC
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

转载自：[WebRTC 工作流程](http://segmentfault.com/a/1190000000608413)

假设用户A想要和用户B进行视频聊天（使用 socket.io）

```
用户A                         服务器                         用户B
上线                                                        上线
emit 'online'  ->                                      <-   emit 'online'
                             on 'online'
                        <-   emit 'online'  ->
on 'online'                                                 on 'online'
获得在线用户列表                                               获得在线用户列表
选择用户B
确认要和用户B聊天
emit 'request chat'  ->
                             on 'request chat'
                             emit 'request chat'  ->
                                                             on 'request chat'
                                                             getUserMedia()
                                                         <-  emit 'stream ok'
                             on 'stream ok'
                         <-  emit 'stream ok'
on 'stream ok'
getUserMedia()
emit 'stream ok'  ->
                             on 'stream ok'
                             emit 'stream ok'  ->
                                                              on 'stream ok'
                                                              createPeerConnection()
                                                              pc.createOffer()
                                                              pc.setLocalDescription()
                                                          <-  emit 'offer'
                              on 'offer'
                          <-  emit 'offer'
on 'offer'
createPeerConnection()
pc.setRemoteDescription
pc.createAnswer()
pc.setLocalDescription()
emit 'answer'  ->
                              on 'answer'
                              emit 'answer'  ->
                                                              on 'answer'
                                                              pc.setRemoteDescription() 

```

这样就能进行视频聊天了。

其中 createPeerConnection 过程如下：

```
pc = new RTCPeerConnection(config)
// 向pc中加入需要发送的流
pc.addStream(localStream)
// onicecandidate 处理器会在网络候选可用的时候调用。
pc.onicecandidate = function (event) {
    if (event.candidate) {
        socket.emit('candidate', requestSocketId, {
            type: 'candidate',
            label: event.candidate.sdpMLineIndex,
            id: event.candidate.sdpMid,
            candidate: event.candidate.candidate
        });
    } else {
        console.log('End of candidates.');
    }
};
// 如果检测到流媒体流到本地，就把它显示出来，同时把流赋值给 remoteStream
pc.onaddstream = function (event) {
    attachMediaStream(remoteVideo, event.stream);
    remoteStream = event.stream;
};

```

参考资料：

[使用WebRTC搭建前端视频聊天室——入门篇](http://blog.segmentfault.com/skyinlayer/1190000000436544)

[Getting Started with WebRTC](http://www.html5rocks.com/en/tutorials/webrtc/basics/)
