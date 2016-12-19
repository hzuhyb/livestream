## 用到的技术

1、webRTC：Web Real-Time Communication，是一个支持网页浏览器进行实时语音对话或视频对话的技术，用于*视频录制*，缺点是只在 PC 的 chrome 上支持较好，移动端支持不太理想。

2、HLS：HTTP Live Streaming，用于视频播放， ios 和 android 都天然支持这种协议，配置简单，直接使用 video 标签即可。

3、RTMP：Real Time Messaging Protocol，是 Macromedia 开发的一套视频直播协议，现在属于 Adobe。和 HLS 一样都可以应用于视频直播，区别是 RTMP 基于 flash 无法在 ios 的浏览器里播放，但是实时性比 HLS 要好。所以一般使用这种协议来上传视频流，也就是视频流推送到服务器。

## 服务器搭建

1、nginx+rtmp+module+ffmpeg

简简单单的推流服务器搭建，由于我们上传的视频流都是基于rtmp协议的，所以服务器也必须要支持rtmp才行，大概需要以下几个步骤：

(1)、安装一台nginx服务器;  
(2)、安装nginx的rtmp扩展 https://github.com/arut/nginx-rtmp-module;  
(3)、配置nginx的conf文件：
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
(4)、重启nginx，将rtmp的推流地址写为rtmp://ip:1935/hls/mystream，其中hls_path表示生成的.m3u8和ts文件所存放的地址，
hls_fragment表示切片时长，mysteam表示一个实例，即将来要生成的文件名可以先自己随便设置一个。

2、SRS
3、FMS
4、RED5
5、Crtmpserver等等

## 直播原理

把主播录制的视频，推送到服务器，再由服务器分发给观众看。

## 直播环节

- **视频录制端（采集、美颜处理、编码、推流）**：一般是电脑上的音视频输入设备或者手机端的摄像头或者麦克风，目前以移动端的手机视频为主。

- **视频服务器端（转码、录制、截图、鉴黄）**：可以是电脑上的播放器，手机端的 native 播放器，还有就是 h5 的 video 标签等，目前还是已手机端的 native 播放器为主。

- **视频播放端（拉流、解码、渲染）**：一般是一台 nginx 服务器，用来接受视频录制端提供的视频源，同时提供给视频播放端流服务。

## Ref

- [从无到有开发连麦直播技术点整理](https://github.com/DyncLang/DevLiveBook)
- [Bilibili H5 FLV视频开源](https://github.com/Bilibili/flv.js)
- [基于h5和videojs的视频直播插件](https://github.com/fzninja/hLive)
- [hls-decryptor](https://github.com/mafintosh/hls-decryptor)
- [videojs-contrib-hls](https://github.com/videojs/videojs-contrib-hls)
- [MSE-based HLS client](https://github.com/dailymotion/hls.js)
- [livestream](https://github.com/lcxfs1991/livestream)
