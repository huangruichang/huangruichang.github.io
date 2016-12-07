# 浅谈直播技术

## 为何研究直播？
上年通过朋友的安利，我入坑了《硅谷》这部互联网版BBOF。《硅谷》第二季最后，全栈工程师 Gilfoyle 用临时架设的服务器集群扛住了30万用户同时在线观看直播的场景让我觉得，非常向往。流量瞬间爆炸的情况下，他们绞尽脑汁，又是凿墙拉线，又是现场撸缓存直接上线，连肥版乔布斯也上阵了,最后服务器还着火。但是，最后成功撑住了直播，在线人数定格在30万+。剧情感动了我，所以，我也想了解下现在非常火的直播技术。

![在这里输入图片描述][1]


## Prerequisites

假设你已经：

 1. [会 nginx 模块扩展][2]
 2. 安装 ffmpeg
 3. 看得懂前端代码
 4. 会 Android、iOS 或 React-Native 的 Hello World

## 我的想法
在接受别人的想法前，我想尝试用我的技术栈实现一个 demo。我的想法，用 web 技术实现一个视频获取程序，然后使用 ffmpeg 推流到 nginx 上，使用 html5 或者移动平台(自动识别 hls)进行播放。
nginx 只需要安装 [nginx-rtmp-module][3] 扩展就可以了，我是参照[《ffmpeg+nginx+nginx-rtmp-module 搭建 rtmp hls http 流媒体服务器成功经验分享》][4]成功运行 nginx-rmpt-module 里 hls 的 demo。
因为是 [hls][5]，所以需要手机端辅助测试，用安卓或者 iOS 都可以，只需要写个 webview 加载上面博客的 hls 链接，就会自动播放，最后我选择使用 ReactNative 。
在这个想法里，唯一需要自己实现的是视频获取程序的部分，在 web 上实现不是什么难事，只需要简单调用 getUserMedia 接口就可以了，网上的 demo 基本上都是把获取的 stream 直接赋值到 video 的 src 上。在这里我产生了疑问，获取的视频数据:

 1. 应该在 stream 对象上找办法获取到 buffer 上传到服务器
 2. 还是使用 canvas 把视频以帧为单位传到服务器上

实际上，两种方法都是可以的。
第一种方法我借助了 [RecordRTC][6] 这个库，把视频转成 buffer，服务器直接 pipe 文件落地后，就可以推流了。
第二种方法需要在服务器端使用 ffmpeg 辅助，把图片合成视频流，再推流。
在接触别人的做法后，个人认为第二种才是正确的做法，因为帧是视频基本元素，符合原理。另外，可以在图片之上进行各种滤镜，压缩的处理。但是我选择使用第一种方法做了一个 [demo][7]。

demo 运行成功后，去到项目根目录，然后在 stream 文件夹里运行以下命令进行推流:
````
ffmpeg -re -i "test.mp4" -vcodec libx264 -vprofile baseline -acodec aac -ar 44100 -strict -2 -ac 1 -f flv -s 1280x720 -q 10 rtmp://192.168.99.100:1935/hls/test
````
这句推流是往 hls 上推的，所以要使用移动设备去播放。

## 主流直播技术架构

![直播架构][8]

![直播架构][9]

在知乎上有一个[回答][10]，我觉得非常详细。涵盖了架构，学术，协议的全面解释。有空一定要以这个回答为索引，深入了解直播技术。

## ffmpeg 直推
在看了几篇博文之后，才发现，ffmpeg 非常强大，除了可以处理各种视频音频以外，还可以在机器上调起摄像头，麦克风和录屏，然后直接推流！命令如下:

````
ffmpeg -f avfoundation -framerate 30 -i "1:0" -f avfoundation -framerate 30 -video_size 640x480 -i "0" -c:v libx264 -preset ultrafast -filter_complex 'overlay=main_w-overlay_w-10:main_h-overlay_h-10' -acodec libmp3lame -ar 44100 -ac 1  -f flv rtmp://192.168.99.100:1935/myapp/test
````

在主流直播解决方案中，ffmpeg 占一席之地，除了自身强大的功能外，还有完善的 api 、插件机制和生态，是个值得深入学习的工具。

## 遇到的坑

 1. nginx 里，以常用的 http 模块为例

````
http {
    server {
        listen 80;
        server_name localhost;
        location / {
            root html;
            index index.html index.htm;
        }
    }
}
````
从外到里，其实可以理解成，http 协议下，端口80的服务器节点。
同理，在 nginx-rtmp-module 里，

````
rtmp { 
    server { 
        listen 1935; 
            application myapp { 
                live on; 
            } 
            application hls { 
                live on; 
                hls on; 
                hls_path /tmp/hls; 
            } 
        } 
    } 

````
从外到里，表达的是 rmpt 协议下，端口1935 的服务器节点，而且 myapp 是以用 rtmp 协议播放的，hls 是用移动设备播放的。
在测试 rtmp 播放的时候，我在推流命令了，错把应该像 rtmp://192.168.99.100:1935/myapp/test 的 rtmp 地址，写成了 rtmp://192.168.99.100:1935/hls/test 的 hls 地址。没有读懂代码就乱搞，是我的锅。希望能记下这个坑。

## 总结
研究了好几天，翻阅了很多博文和文档，虽然最后只是跑起来了一个 demo，但我也觉得心满意足了。对于诸如视频数据的编码转码、采集、传输，直播要素等等还是一窍不通，有待加强学习。虽然不敢跟大佬们班门弄斧，不过，这篇博文算是对《硅谷》的一次致敬吧，也算了了自己一直想试试直播的心愿。

## 参考

 - [《ffmpeg+nginx+nginx-rtmp-module 搭建 rtmp hls http 流媒体服务器成功经验分享》][11]
 - [《taobao/nginx-book》][12]
 - [《OSX下面用ffmpeg抓取桌面以及摄像头推流进行直播》][13]
 - [《网络直播需要哪些设备和技术？》][14]
 - [muaz-khan/RecordRTC][15]
 - [arut/nginx-rtmp-module][16]


  [1]: https://dn-coding-net-production-pp.qbox.me/89ce2b31-0fbb-4f0c-9bce-608e48b03f5f.png
  [2]: https://github.com/taobao/nginx-book/blob/master/source/chapter_03.rst
  [3]: https://github.com/arut/nginx-rtmp-module
  [4]: http://www.codeclip.com/3724.html
  [5]: http://baike.baidu.com/link?url=rPpbGvKjboAXlANtl8GMVvSPGFqSwGA30Z56LomAVoiynqPsxs1---Is8QHPzDOvxlzv-EzzLLV11XcrJZBJu_
  [6]: https://github.com/muaz-khan/RecordRTC
  [7]: https://github.com/huangruichang/live-tv
  [8]: https://dn-coding-net-production-pp.qbox.me/102e70a9-cb68-450a-93aa-98b67f7d85db.png
  [9]: https://dn-coding-net-production-pp.qbox.me/f3fda5ad-d2db-4028-8242-505221725fbc.png
  [10]: https://www.zhihu.com/question/22421708
  [11]: http://www.codeclip.com/3724.html
  [12]: https://github.com/taobao/nginx-book
  [13]: http://www.cnblogs.com/damiao/p/5233431.html
  [14]: https://www.zhihu.com/question/22421708
  [15]: https://github.com/muaz-khan/RecordRTC
  [16]: https://github.com/arut/nginx-rtmp-module