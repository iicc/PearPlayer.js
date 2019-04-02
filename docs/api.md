# PearPlayer 文档

**PearPlayer（梨享播放器）** 实现了融合HTTP（包含HTTPS、HTTP2）、WebRTC的多协议、多源、低延迟、
高带宽利用率的无插件Web端流媒体加速能力。基于H5的MSE技术(Media Source Extension)将来自多个源节点的Buffer通过调度算法分块喂给播放器，
在保证用户流畅播放体验的前提下最大化P2P率。

使用PearPlayer，需确保浏览器支持[WebRTC](https://en.wikipedia.org/wiki/WebRTC)(Chrome, Firefox, Opera)和[MSE](https://en.wikipedia.org/wiki/Media_Source_Extensions)。

## 导入
PearPlayer有两种导入方式：通过script标签导入和npm安装

### script标签导入
首先通过script标签导入pear-player.min.js：
```html
<script src="./dist/pear-player.min.js"></script>
```
或者使用CDN：
```html
<script src="https://cdn.jsdelivr.net/npm/pearplayer@latest/dist/pear-player.min.js"></script>
```

### npm安装
在项目目录中通过npm安装pearplayer：
```bash
npm install pearplayer --save
```
然后就可以用require方式引入PearPlayer：
```js
var PearPlayer = require('PearPlayer');
```

# PearPlayer API

## PearPlayer.isMSESupported()

静态方法，检测当前浏览器环境是否支持MSE。
```js
if (PearPlayer.isMSESupported()) {
  // MSE is supported
} else {
  // Use a fallback
}
```

## PearPlayer.isWebRTCSupported()

静态方法，检测当前浏览器环境是否支持WebRTC。
```js
if (PearPlayer.isWebRTCSupported()) {
  // WebRTC is supported
} else {
  // Use a fallback
}
```

## `var player = new PearPlayer('#pearvideo', token, opts)`

创建一个新的PearPlayer实例，`#pearvideo`是video标签的id。

token是CP登陆后获取的token，具体获取方式可以参考https://github.com/PearInc/FogVDN/blob/master/For_CPs.md

其中`opts`可以指定PearPlayer的具体配置，相关字段的说明如下：

```js
{
  src: 'https://example.mp4',  //视频的url，目前仅支持MP4格式
  scheduler: 'IdleFirst',      //节点调度算法，默认IdleFirst，其它内置调度算法有“WebRTCFirst“和”CloudFirst”
  autoplay: true,              //是否自动播发视频,默认true
  interval: 5000,              //滑动窗口的时间间隔,单位毫秒,默认10s
  useDataChannel: true,        //是否开启data channel,默认true
  dataChannels: 20,            //创建data channel的最大数量,默认20
  useTorrent: true,            //是否开启Browser P2P(基于Webtorrent)，默认true
  magnetURI: 'magnet:?xt=...', //可手动传入magnetURI，需先将useTorrent设为true
  trackers:["wss://tracker.openwebtorrent.com"],    //可手动传入tracker服务器，需先将useTorrent设为true
  sources: [],                 //指定下载源，增加这个字段后pearplayer不会再向后台请求节点，建议下载源多于5个以保证流畅播放
  useMonitor: true,            //是否开启monitor,会稍微影响性能,默认false
  BTMode: false,               //是否开启纯BT下载模式（基于webtorrent），默认false，如果设为true，需要手动传入magnetURI（参考https://github.com/webtorrent/webtorrent）
  algorithm: 'pull',           //下载算法，有‘push’和‘pull’两种，默认‘pull’
  debug: true                  //是否开启debug模式，开启后可以在console中查看log，默认false
}
```

## `player.on('fallback', function () {})`

播放器出现异常时的回调函数，建议在此处作降级处理，如直接调用video.play()。

## `player.on('begin', function (fileLength, chunks) {})`

当PearPlayer完成初始化后会触发该事件，通过回调函数中的fileLength获取文件总大小，chunks获取文件被分成的块数（每块1M）。

## `player.on('done', function () {})`

当PearPlayer完成下载会触发该事件。

## `player.on('progress', function (downloaded) {})`

通过该事件可以监听PearPlayer的下载进度（下载的字节数除以总的字节数）。(useMonitor需设为true)

## `player.on('cloudspeed', function (speed) {})`

通过该事件可以监听cloud（即server节点）的平均下载速度（单位KB/s）。(useMonitor需设为true)

## `player.on('fogspeed', fuction (speed) {})`

通过该事件可以监听fog节点（包括WebRTC和HTTP）的平均下载速度（单位KB/s）。(useMonitor需设为true)

## `player.on('fogratio', function (fogRatio) {})`

通过该事件可以监听fog节点（包括WebRTC和HTTP）总的下载比率（fog下载的字节数除以目前总的下载字节数）。(useMonitor需设为true)

## `player.on('sourcemap', function (sourceType, index) {})`

每下载一个buffer都会触发该事件，sourceType是一个string，代表该buffer是从哪个源下载的，有以下几种取值：(useMonitor需设为true)<br/>
null: 该处的buffer还未下载<br/>
s: server，从服务器端下载（HTTP协议）<br/>
n: node，从节点下载（HTTP协议）<br/>
d: data channel，从节点下载（WebRTC协议）<br/>
b: browser，从其它浏览器下载（WebRTC协议）<br/>

index是对应的索引。

## `player.on('traffic', function (mac, size, type) {})`
通过该事件可以监听每个节点的实时流量，其中mac是节点的mac地址，size是对应节点的瞬时下载流量（字节），type是
节点的类型（http、datachannel等）。(useMonitor需设为true)


## `player.on('metadata', function(metadata) {})`
通过该事件监听视频元信息，回调的metadata是一个对象，包含视频平均码率和播放时长两个字段。

## `player.on('canplay', function(delay) {})`
该事件类似video的“canplay”事件，但回调出来的参数是从PearPLayer实例化到开始播放的延迟时间。

- 请参考[`/examples/player-test.html`](/examples/player-test.html)来了解API使用方法。
- CP接入方法：[Pear FogVDN](https://github.com/PearInc/FogVDN/blob/master/For_CPs.md)



