## 用到的技术

1、*webRTC*：Web Real-Time Communication（网页即时通信），是一个支持网页浏览器进行实时语音对话或视频对话的 API，用于*视频录制*，缺点是只在 PC 的 chrome 上支持较好，移动端支持不太理想。目前主要应用于视频会议和连麦中。

优点：
- W3C 标准，主流浏览器支持程度高
- Google 在背后支撑，并在各平台有参考实现
- 底层基于 SRTP 和 UDP，弱网情况优化空间大
- 可以实现点对点通信，通信双方延时低

缺点：
- ICE,STUN,TURN 传统 CDN 没有类似的服务提供

2、*HLS*：HTTP Live Streaming，用于视频播放， ios 和 android 都天然支持这种协议，配置简单，直接使用 video 标签即可。

3、*RTMP*：Real Time Messaging Protocol（实时消息传输协议），该协议基于 TCP，是一个协议族，包括 RTMP 基本协议及 RTMPT/RTMPS/RTMPE 等多种变种。RTMP 是一种设计用来进行实时数据通信的网络协议，主要用来在 Flash/AIR 平台和支持 RTMP 协议的流媒体/交互服务器之间进行音视频和数据通信。是 Macromedia 开发的一套视频直播协议，现在属于 Adobe。和 HLS 一样都可以应用于视频直播，区别是 RTMP 基于 flash 无法在 ios 的浏览器里播放，但是实时性比 HLS 要好。所以一般使用这种协议来上传视频流，也就是视频流推送到服务器。

RTMP 是目前主流的流媒体传输协议，广泛用于直播领域，可以说市面上绝大多数的直播产品都采用了这个协议。

优点：
- CDN 支持良好，主流的 CDN 厂商都支持
- 协议简单，在各平台上实现容易

缺点：
- 基于 TCP ，传输成本高，在弱网环境丢包率高的情况下问题显著
- 不支持浏览器推送
- Adobe 私有协议，Adobe 已经不再更新

## 服务器搭建

1、nginx+rtmp+module+ffmpeg

简简单单的推流服务器搭建，由于我们上传的视频流都是基于rtmp协议的，所以服务器也必须要支持rtmp才行，大概需要以下几个步骤：

（1）安装一台nginx服务器;  
（2）安装nginx的rtmp扩展 https://github.com/arut/nginx-rtmp-module;  
（3）配置nginx的conf文件：
``` python
rtmp {  
    server {  
        listen 1935;  #监听的端口
        chunk_size 4000;  
        application hls {  #rtmp推流请求路径
            live on;  
            hls on;  
            hls_path /usr/local/var/www/hls;  
            hls_fragment 5s;  
        }  
    }  
}
```
（4）重启nginx，将rtmp的推流地址写为rtmp://ip:1935/hls/mystream，其中hls_path表示生成的.m3u8和ts文件所存放的地址，
hls_fragment表示切片时长，mysteam表示一个实例，即将来要生成的文件名可以先自己随便设置一个。

2、SRS
3、FMS
4、RED5
5、Crtmpserver等等

## 直播原理

把主播录制的视频，推送到服务器，再由服务器分发给观众看。

## 直播环节

1、**视频录制端（采集、处理、编码、封装、推流）**：一般是电脑上的音视频输入设备或者手机端的摄像头或者麦克风，目前以移动端的手机视频为主。

（1）*采集*：视频的采集涉及两方面数据的采集：**音频采集** 和 **图像采集**，它们分别对应两种完全不同的输入源和数据格式。采集源：摄像头采集、屏幕录制、从视频文件推流。

（2）*处理*：处理环节中分为 **音频处理** 和 **视频处理** ，音频处理中具体包含混音、降噪和声音特效等处理，视频处理中包含美颜、水印、以及各种自定义滤镜等处理。常见视频处理功能：美颜、视频水印、滤镜、连麦（基于 WebRTC 的实时通讯方案）。

（3）*编码*：原始视频数据存储空间大，一个 1080P 的 7 s 视频需要 817 MB，原始视频数据传输占用带宽大，10 Mbps 的带宽传输上述 7 s 视频需要 11 分钟，而经过 H.264 编码压缩之后，视频大小只有 708 k ,10 Mbps 的带宽仅仅需要 500 ms ，可以满足实时传输的需求，所以从视频采集传感器采集来的原始视频势必要经过视频编码。编码主要有 **帧内编码** 和 **帧间编码**，我们通常说的 I 帧和 P 帧就是分别采用了帧内编码和帧间编码。编码器经历了数十年的发展，已经从开始的只支持帧内编码演进到现如今的 **H.265** 和 **VP9** 为代表的新一代编码器。谈到视频编码相关内容就不得不提一个伟大的软件包 -- **FFmpeg**，FFmpeg 是一个自由软件，可以运行音频和视频多种格式的录影、转换、流功能，包含了 libavcodec ——这是一个用于多个项目中音频和视频的解码器库，以及 libavformat ——一个音频与视频格式转换库。

（4）*封装*：如果把编比喻为把货装上货车，那么封装可以理解为采用哪种货车去运输，也就是媒体的容器。所谓容器，就是把编码器生成的多媒体内容（视频，音频，字幕，章节信息等）混合封装在一起的标准。常见的封装格式：AVI 格式（后缀为 .AVI）、DV-AVI 格式（后缀为 .AVI）、QuickTime File Format 格式（后缀为 .MOV）、MPEG 格式（文件后缀可以是 .MPG .MPEG .MPE .DAT .VOB .ASF .3GP .MP4等)、WMV 格式（后缀为.WMV .ASF）、Real Video 格式（后缀为 .RM .RMVB）、Flash Video 格式（后缀为 .FLV）、Matroska 格式（后缀为 .MKV）、MPEG2-TS 格式 (后缀为 .ts)。目前，我们在流媒体传输，尤其是直播中主要采用的就是 FLV 和 MPEG2-TS 格式，分别用于 RTMP/HTTP-FLV 和 HLS 协议。

（5）*推流和传输*：推送协议有：**RTMP**、**WebRTC**、**基于 UDP 的私有协议**。

- [《视频直播技术详解》系列之二：采集](http://blog.qiniu.com/archives/6713)
- [《视频直播技术详解》系列之三：处理](http://blog.qiniu.com/archives/6795)
- [《视频直播技术详解》系列之四：编码和封装](http://blog.qiniu.com/archives/6816)
- [《视频直播技术详解》系列之五：推流和传输](http://blog.qiniu.com/archives/6914)

2、**视频服务器端（转码、录制、截图、鉴黄）**：一般是一台 nginx 服务器，用来接受视频录制端提供的视频源，同时提供给视频播放端流服务。

（1）*转码*：

3、**视频播放端（拉流、解码、渲染）**：可以是电脑上的播放器，手机端的 native 播放器，还有就是 h5 的 video 标签等，目前还是已手机端的 native 播放器为主。

（1）*拉流*：
（2）*解码*：
（3）*渲染*：

## Ref

- [WebRTC+](http://l0gs.xf0rk.space/webrtc-plus/)
- [从无到有开发连麦直播技术点整理](https://github.com/DyncLang/DevLiveBook)
- [Bilibili H5 FLV视频开源](https://github.com/Bilibili/flv.js)
- [基于h5和videojs的视频直播插件](https://github.com/fzninja/hLive)
- [hls-decryptor](https://github.com/mafintosh/hls-decryptor)
- [videojs-contrib-hls](https://github.com/videojs/videojs-contrib-hls)
- [MSE-based HLS client](https://github.com/dailymotion/hls.js)
- [WebRTC 直播时代](https://www.villainhr.com/page/2017/02/20/WebRTC%20%E7%9B%B4%E6%92%AD%E6%97%B6%E4%BB%A3)
