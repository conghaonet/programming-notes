# Android-WebSocket实现即时通讯功能

[参考链接](https://www.jianshu.com/p/7b919910c892)

最近做这个功能，分享一下。即时通讯（Instant Messaging）最重要的毫无疑问就是即时，不能有明显的延迟，要实现IM的功能其实并不难，目前有很多第三方，比如极光的JMessage，都比较容易实现。但是如果项目有特殊要求（如不能使用外网），那就得自己做了，所以我们需要使用WebSocket。

## WebSocket
WebSocket协议就不细讲了，感兴趣的可以具体查阅资料，简而言之，它就是一个可以建立长连接的全双工(full-duplex)通信协议，允许服务器端主动发送信息给客户端。

## Java-WebSocket框架
对于使用websocket协议，Android端已经有些成熟的框架了，在经过对比之后，我选择了Java-WebSocket这个开源框架，GitHub地址：https://github.com/TooTallNate/Java-WebSocket，目前已经有五千以上star，并且还在更新维护中，所以本文将介绍如何利用此开源库实现一个稳定的即时通讯功能。
* 效果图  
<img src="https://i.ibb.co/bLMLFvr/image.png" alt="image" border="0">  <img src="https://i.ibb.co/1q845sc/image.png" alt="image" border="0">
<img src="https://i.ibb.co/hBhQdpZ/image.png" alt="image" border="0">  <img src="https://i.ibb.co/6R4ZvkF/image.png" alt="image" border="0">

* 文章重点
  * 与websocket建立长连接
  * 与websocket进行即时通讯
  * Service和Activity之间通讯和UI更新
  * 弹出消息通知（包括锁屏通知）
  * 心跳检测和重连（保证websocket连接稳定性）
  * 服务（Service）保活
