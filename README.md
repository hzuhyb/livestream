## 用到的技术

## 服务器搭建

简简单的推流服务器搭建，由于我们上传的视频流都是基于rtmp协议的，所以服务器也必须要支持rtmp才行，大概需要以下几个步骤：

1、安装一台nginx服务器
2、安装nginx的rtmp扩展 https://github.com/arut/nginx-rtmp-module
3、配置nginx的conf文件：
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
4、重启nginx，将rtmp的推流地址写为rtmp://ip:1935/hls/mystream，其中hls_path表示生成的.m3u8和ts文件所存放的地址，
hls_fragment表示切片时长，mysteam表示一个实例，即将来要生成的文件名可以先自己随便设置一个

## 直播原理

把主播录制的视频，推送到服务器，再由服务器分发给观众看。

## 直播环节

- 推流端（采集、美颜处理、编码、推流）
- 服务端处理（转码、录制、截图、鉴黄）
- 播放器（拉流、解码、渲染）
- 互动系统（聊天室、礼物系统、弹幕等等）

## Ref

- [从无到有开发连麦直播技术点整理](https://github.com/DyncLang/DevLiveBook)
- [Bilibili H5 FLV视频开源](https://github.com/Bilibili/flv.js)
- [基于h5和videojs的视频直播插件](https://github.com/fzninja/hLive)
- [hls-decryptor](https://github.com/mafintosh/hls-decryptor)
- [videojs-contrib-hls](https://github.com/videojs/videojs-contrib-hls)
- [MSE-based HLS client](https://github.com/dailymotion/hls.js)
- [livestream](https://github.com/lcxfs1991/livestream)
