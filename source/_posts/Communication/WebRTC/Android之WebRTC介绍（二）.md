---
title: Android之WebRTC介绍（二）
date: 2016-07-23 22:16:13
tags: 
- WebRTC 
- Android
categories: 网络通信
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

WebRTC提供了点对点之间的通信，但并不意味着WebRTC不需要服务器。暂且不说基于服务器的一些扩展业务，WebRTC至少有两件事必须要用到服务器： 
1. 浏览器之间交换建立通信的元数据（信令）必须通过服务器 
2. 为了穿越NAT和防火墙

此处，我们使用XMPP协议实现信令，采用openfire当做服务器，通过openfire服务器+Smack API实现信令的传递。

因此，在建立PeerConnection实例之后，想要使用其建立一个点对点的信道，我们需要做两件事： 
1. 确定本机上的媒体流的特性，比如分辨率、编解码能力啥的（SDP描述符） 
2. 连接两端的主机的网络地址（ICE Candidate）


----------


### 通过offer和answer交换SDP描述符

大致上在两个用户（甲和乙）之间建立点对点连接流程应该是这个样子（这里不考虑错误的情况，PeerConnection简称PC）：

甲和乙各自建立一个PC实例
甲通过PC所提供的createOffer()方法建立一个包含甲的SDP描述符的offer信令
甲通过PC所提供的setLocalDescription()方法，将甲的SDP描述符交给甲的PC实例
甲将offer信令通过服务器发送给乙
乙将甲的offer信令中所包含的的SDP描述符提取出来，通过PC所提供的setRemoteDescription()方法交给乙的PC实例
乙通过PC所提供的createAnswer()方法建立一个包含乙的SDP描述符answer信令
乙通过PC所提供的setLocalDescription()方法，将乙的SDP描述符交给乙的PC实例
乙将answer信令通过服务器发送给甲
甲接收到乙的answer信令后，将其中乙的SDP描述符提取出来，调用setRemoteDescripttion()方法交给甲自己的PC实例

通过在这一系列的信令交换之后，甲和乙所创建的PC实例都包含甲和乙的SDP描述符了，完成了两件事的第一件。我们还需要完成第二件事——获取连接两端主机的网络地址


----------


### 通过ICE框架建立NAT/防火墙穿越的连接
这个网络地址应该是能从外界直接访问，WebRTC使用ICE框架来获得这个地址。PeerConnection在创立的时候可以将ICE服务器的地址传递进去，如：

```
 private List<PeerConnection.IceServer> getIceServers(String url,String user,String credential)
{
    PeerConnection.IceServer turn = new PeerConnection.IceServer(
                url,user,credential);
    LinkedList<PeerConnection.IceServer> iceServers = new LinkedList<PeerConnection.IceServer>();
    iceServers.add(turn);
    return iceServers;
}

//iceServer List对象获取
List<PeerConnection.IceServer> iceServers = getIceServers(CoturnData.url,
            CoturnData.userName,CoturnData.credential);
pcConstraints = new MediaConstraints();
pcConstraints.optional.add(new MediaConstraints.KeyValuePair(
            "DtlsSrtpKeyAgreement", "true"));
pcConstraints.mandatory.add(new
            MediaConstraints.KeyValuePair("VoiceActivityDetection", "false"));
                
pc = factory.createPeerConnection(iceServers,pcConstraints,pcObserver);
```

当然这个地址也需要交换，还是以甲乙两位为例，交换的流程如下（PeerConnection简称PC）：

1. 甲、乙各创建配置了ICE服务器的PC实例，并为其添加onicecandidate事件回调
2. 当网络候选可用时，将会调用onicecandidate函数
3. 在回调函数内部，甲或乙将网络候选的消息封装在ICE Candidate信令中，通过服务器中转，传递给对方
4. 甲或乙接收到对方通过服务器中转所发送过来ICE Candidate信令时，将其解析并获得网络候选，将其通过PC实例的addIceCandidate()方法加入到PC实例中

这样连接就创立完成了，可以向RTCPeerConnection中通过addStream()加入流来传输媒体流数据。将流加入到RTCPeerConnection实例中后，对方就可以通过onaddstream所绑定的回调函数监听到了。调用addStream()可以在连接完成之前，在连接建立之后，对方一样能监听到媒体流


----------


代码实现：

```
public class VideoCallActivity extends ParentActivity{
    public static final String VIDEO_TRACK_ID = "video_track_id";
    public static final String AUDIO_TRACK_ID = "audio_track_id";
    public static final String LOCAL_MEDIA_STREAM_ID = "local_media_stream_id";
    private String mServiceName;    //XMPP服务器名称

    private GLSurfaceView mGLSurfaceView;
    private GLSurfaceView mGLSurfaceViewRemote;

    private PeerConnection pc;
    private final PCObserver pcObserver = new PCObserver();
    private final SDPObserver sdpObserver = new SDPObserver();

    private MediaConstraints sdpMediaConstraints;
    private MediaConstraints pcConstraints;
    private String remoteName;
    IceCandidate remoteIceCandidate;

    private boolean mIsInited;
    private boolean mIsCalled;
    PeerConnectionFactory factory;
    VideoCapturer videoCapturer;
    VideoSource videoSource;

    VideoRenderer localVideoRenderer;
    VideoRenderer remoteVideoRenderer;

    AudioManager audioManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_video_call);

        //打开扬声器
        audioManager = (AudioManager) getSystemService(AUDIO_SERVICE);
        audioManager.setSpeakerphoneOn(true);

        mServiceName = connection.getServiceName();
        mGLSurfaceView = (GLSurfaceView) findViewById(R.id.glsurfaceview);
//        mGLSurfaceViewRemote = (GLSurfaceView) findViewById(R.id.glsurfaceview_remote);
        //检查初始化音视频设备是否成功
        if (!PeerConnectionFactory.initializeAndroidGlobals(this,true,true,true,null))
        {
            Log.e("init","PeerConnectionFactory init fail!");
            return;
        }

//        Intent intent = getIntent().getBundleExtra()

        //Media条件信息SDP接口
        sdpMediaConstraints = new MediaConstraints();
        //接受远程音频
        sdpMediaConstraints.mandatory.add(new MediaConstraints.KeyValuePair(
                "OfferToReceiveAudio", "true"));
        //接受远程视频
        sdpMediaConstraints.mandatory.add(new MediaConstraints.KeyValuePair(
                "OfferToReceiveVideo", "true"));

        factory = new PeerConnectionFactory();

        //iceServer List对象获取
        List<PeerConnection.IceServer> iceServers = getIceServers(CoturnData.url,
                CoturnData.userName,CoturnData.credential);
        pcConstraints = new MediaConstraints();
        pcConstraints.optional.add(new MediaConstraints.KeyValuePair(
                "DtlsSrtpKeyAgreement", "true"));
        pcConstraints.mandatory.add(new
                MediaConstraints.KeyValuePair("VoiceActivityDetection", "false"));
        pc = factory.createPeerConnection(iceServers,pcConstraints,pcObserver);

        mIsInited = false;
        mIsCalled=false;

        boolean offer=getIntent().getBooleanExtra("createOffer",false);
        //offer：如果offer为true表示主叫方初始化，如果为false表示被叫方初始化。
        remoteName = getIntent().getStringExtra("remoteName");
        if(!offer)
        {
            initialSystem();
        }
        else {
            callRemote(remoteName);
        }
        //当VideoActivity已经打开时，处理后续的intent传过来的数据。
        processExtraData();
    }

    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        setIntent(intent);
        processExtraData();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        //通话结束，发送通话结束消息
        videoCallEnded();
        //释放资源
        videoCapturer.dispose();
        videoSource.stop();
        if (pc != null) {
            pc.dispose();
            pc = null;
        }

        audioManager.setSpeakerphoneOn(false);
    }

    private void videoCallEnded() {
        String chatJid = remoteName+"@"+mServiceName;
        Message message = new Message();
        VideoInvitation videoInvitation = new VideoInvitation();
        videoInvitation.setTypeText("video-ended");
        message.addExtension(videoInvitation);
        Chat chat = createChat(chatJid);
        try {
            chat.sendMessage(message);
        } catch (SmackException.NotConnectedException e) {
            e.printStackTrace();
        }
    }

    private void processExtraData() {
        Intent intent = getIntent();
        //获取SDP数据
        String sdpType = intent.getStringExtra("type");
        String sdpDescription = intent.getStringExtra("description");
        if (sdpType != null)
        {

            SessionDescription.Type type = SessionDescription.Type.fromCanonicalForm(sdpType);
            SessionDescription sdp = new SessionDescription(type,sdpDescription);
            if (pc == null)
            {
                Log.e("pc","pc == null");
            }
            pc.setRemoteDescription(sdpObserver,sdp);

            //如果是offer,则被叫方createAnswer
            if (sdpType.equals("offer"))
            {
                mIsCalled = true;
                pc.createAnswer(sdpObserver,sdpMediaConstraints);
            }

        }

        //获取ICE Candidate数据
        String iceSdpMid = intent.getStringExtra("sdpMid");
        int iceSdpMLineIndex = intent.getIntExtra("sdpMLineIndex",-1);
        String iceSdp = intent.getStringExtra("sdp");
        if (iceSdpMid != null)
        {
            IceCandidate iceCandidate = new IceCandidate(iceSdpMid,iceSdpMLineIndex,iceSdp);

            if (remoteIceCandidate == null)
            {
                remoteIceCandidate = iceCandidate;
            }

            //下面这步放到函数drainRemoteCandidates()中
            /*//添加远端的IceCandidate到pc
            pc.addIceCandidate(iceCandidate);*/
        }


        //结束activity
        boolean videoEnded = intent.getBooleanExtra("videoEnded",false);
        if (videoEnded)
        {
            finish();
        }
    }

    private void callRemote(String remoteName) {
        initialSystem();
        //createOffer
        pc.createOffer(sdpObserver,sdpMediaConstraints);
    }

    private void initialSystem() {
        if (mIsInited)
        {
            return;
        }
        //获取前置摄像头本地视频流
        String frontDeviceName = VideoCapturerAndroid.getNameOfFrontFacingDevice();
//        String frontDeviceName = "Camera 1, Facing front, Orientation 0";
        Log.e("CameraName","CameraName: "+frontDeviceName);
        videoCapturer = VideoCapturerAndroid.create(frontDeviceName);
        if (videoCapturer == null)
        {
            Log.e("open","fail to open camera");
            return;
        }
        //视频
        MediaConstraints mediaConstraints = new MediaConstraints();
        videoSource = factory.createVideoSource(videoCapturer, mediaConstraints);
        VideoTrack localVideoTrack = factory.createVideoTrack(VIDEO_TRACK_ID, videoSource);

        //音频
        MediaConstraints audioConstraints = new MediaConstraints();
        AudioSource audioSource = factory.createAudioSource(audioConstraints);
        AudioTrack localAudioTrack = factory.createAudioTrack(AUDIO_TRACK_ID,audioSource);

        Runnable runnable = new Runnable() {
            @Override
            public void run() {

            }
        };
        VideoRendererGui.setView(mGLSurfaceView,runnable);
        try {
            //改成ScalingType.SCALE_ASPECT_FILL可以显示双方视频，但是显示比例不美观，并且不知道最后一个参数true和false的含义。
            localVideoRenderer = VideoRendererGui.createGui(0,0,30,30, VideoRendererGui.ScalingType.SCALE_ASPECT_FILL,true);
            remoteVideoRenderer = VideoRendererGui.createGui(0,0,100,100, VideoRendererGui.ScalingType.SCALE_FILL,true);
            localVideoTrack.addRenderer(localVideoRenderer);
        } catch (Exception e) {
            e.printStackTrace();
        }

        MediaStream localMediaStream = factory.createLocalMediaStream(LOCAL_MEDIA_STREAM_ID);
        localMediaStream.addTrack(localAudioTrack);
        localMediaStream.addTrack(localVideoTrack);

        pc.addStream(localMediaStream);
    }

    private List<PeerConnection.IceServer> getIceServers(String url,String user,String credential)
    {
        PeerConnection.IceServer turn = new PeerConnection.IceServer(
                url,user,credential);
        LinkedList<PeerConnection.IceServer> iceServers = new LinkedList<PeerConnection.IceServer>();
        iceServers.add(turn);
        return iceServers;
    }

    private class PCObserver implements PeerConnection.Observer
    {

        @Override
        public void onSignalingChange(PeerConnection.SignalingState signalingState) {

        }

        @Override
        public void onIceConnectionChange(PeerConnection.IceConnectionState iceConnectionState) {

        }

        @Override
        public void onIceGatheringChange(PeerConnection.IceGatheringState iceGatheringState) {

        }

        //发送ICE候选到其他客户端
        @Override
        public void onIceCandidate(final IceCandidate iceCandidate) {
            //利用XMPP发送iceCandidate到其他客户端
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    String chatJid = remoteName+"@"+mServiceName;
                    Message message = new Message();
                    IceCandidateExtensionElement iceCandidateExtensionElement=
                            new IceCandidateExtensionElement();
                    iceCandidateExtensionElement.setSdpMidText(iceCandidate.sdpMid);
                    iceCandidateExtensionElement.setSdpMLineIndexText(iceCandidate.sdpMLineIndex);
                    iceCandidateExtensionElement.setSdpText(iceCandidate.sdp);
                    message.addExtension(iceCandidateExtensionElement);
                    Chat chat = createChat(chatJid);
                    try {
                        chat.sendMessage(message);
                    } catch (SmackException.NotConnectedException e) {
                        e.printStackTrace();
                    }

                }
            });
        }

        //Display a media stream from remote
        @Override
        public void onAddStream(final MediaStream mediaStream) {
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    if (pc == null)
                    {
                        Log.e("onAddStream","pc == null");
                        return;
                    }
                    if (mediaStream.videoTracks.size()>1 || mediaStream.audioTracks.size()>1)
                    {
                        Log.e("onAddStream","size > 1");
                        return;
                    }
                    if (mediaStream.videoTracks.size() == 1)
                    {
                        VideoTrack videoTrack = mediaStream.videoTracks.get(0);
                        videoTrack.addRenderer(remoteVideoRenderer);
                    }
                }
            });
        }

        @Override
        public void onRemoveStream(final MediaStream mediaStream) {
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    mediaStream.videoTracks.get(0).dispose();
                }
            });
        }

        @Override
        public void onDataChannel(DataChannel dataChannel) {

        }

        @Override
        public void onRenegotiationNeeded() {

        }
    }

    private class SDPObserver implements SdpObserver
    {

        @Override
        public void onCreateSuccess(final SessionDescription sessionDescription) {
            //sendMessage(offer);
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    String chatJid = remoteName+"@"+mServiceName;
                    Message message = new Message();
                    SDPExtensionElement sdpExtensionElement = new SDPExtensionElement();
                    sdpExtensionElement.setTypeText(sessionDescription.type.canonicalForm());
                    sdpExtensionElement.setDescriptionText(sessionDescription.description);
                    message.addExtension(sdpExtensionElement);
                    Chat chat = createChat(chatJid);
                    try {
                        chat.sendMessage(message);
                    } catch (SmackException.NotConnectedException e) {
                        e.printStackTrace();
                    }

                    pc.setLocalDescription(sdpObserver,sessionDescription);
                }
            });
        }

        @Override
        public void onSetSuccess() {
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    //主叫方
                    if (!mIsCalled)
                    {
                        if (pc.getRemoteDescription() != null)
                        {
                            drainRemoteCandidates();
                        }
                    }
                    //被叫方
                    else
                    {
                        //如果被叫方还没有createAnswer
                        if (pc.getLocalDescription() == null)
                        {
                            Log.e("SDPObserver", "SDPObserver create answer");
                        }
                        else
                        {
                            drainRemoteCandidates();
                        }
                    }
                }
            });
        }

        private void drainRemoteCandidates() {
            if (remoteIceCandidate == null)
            {
                Log.e("SDPObserver","remoteIceCandidate == null");
                return;
            }
            pc.addIceCandidate(remoteIceCandidate);
            Log.e("IceCanditate","添加IceCandidate成功");
            remoteIceCandidate = null;
        }

        @Override
        public void onCreateFailure(String s) {
            Log.e("SDPObserver","onCreateFailure");
        }

        @Override
        public void onSetFailure(String s) {
            Log.e("SDPObserver","onSetFailure");
        }
    }
}
```


