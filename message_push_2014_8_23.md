Title: 消息推送
Date: 2014-08-23
Category: python
Tag: 消息推送,ios,android

## 消息推送

### ios消息推送

简单来说，要实现ios消息推送，我们只需要根据APNS相关格式封装消息（[相关链接](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html)），发送给Apple的服务器即可，Apple会推送到我们指定的具体设备和应用。Apple的推送服务会为每台设备的每个应用生成token，在服务器端需要记录该token，在推送消息时指定token和应用证书即可。

在实际的实现中，为了高效率的推送，我们可以在一条链接上实现批量推送，而不需要为每次推送到一个token都建立一个链接。比如我们需要推送到3个用户，token分别为1， 2， 3，我们可以在与Apple的服务器建立一个链接C1，在C1上先后write token为1，2，3的相关消息，而不是为每个token建立链接。

Apple在推送成功时并不会返回数据告诉我们相关消息是否成功，只是在推送失败时会返回相关的失败信息，并主动关闭链接。在这种情况下可能会产生如下的问题。

#### 推送中可能遇到的问题

Apple的推送分为测试环境和正式的环境，在两种环境中生成的token是不同的。如果我们使用测试环境的token组装消息发送到正式环境，Apple就会认为该token为非法token，并返回状态码8（invalid token）及该token的identifier（该identifier是在我们推送中指定的），并关闭链接。因此在推送时我们需要知道具体时哪一个token非法并将它从数据库中剔除，以免在下次推送中产生干扰。这时候就需要我们的token能够处理如下的情况：
假设我们有需要推送的队列如下，其中5是非法token：

        1， 2， 3， 4， 5， 6， 7， 8

建立链接后开始依次推送，由于推送成功Apple不返回状态，因此我们需要遍历该数组，当5发送出去后，由于不可能实时返回消息，我们实际上已经将6，7也发送了出去，当发送到8时得到Apple返回的消息，5是非法token，这时候我们就需要能够重新建立链接，并将5后面的token重新发送。

[DEMO传送门](https://github.com/wbsking/ios_push_demo)

### android消息推送

目前google也开放了自己的推送服务GCM，但由于众所周知的原因，在国内基本处于不可用的状态。一般都是直接使用的第三方推送服务，比如百度云推送，AVOS等等。这些服务商一般都会提供REST API，使用起来都比较简单。

#### 使用socketio实现消息推送
使用socketio我们也可以实现基于长连接的消息推送，并结合redis的订阅发布来实现消息的向连接的批量发送。具体流程如下：
当手机客户端连接建立后，我们可以为该连接订阅redis相关的频道，当需要发送消息时仅仅向该频道上发布消息即可，通过使用[brukva](https://github.com/kmerenkov/brukva)可以实现在redis相关频道上异步监听，当收到消息后通过长连接发送给客户端。
#### 需要解决的问题

如果自己实现android消息推送，有几个部分需要处理，涉及到客户端及服务端：

1、心跳机制，包括心跳时长设置等。
2、移动网络长连接的稳定性。
3、在android客户端如何确保后台消息服务进程不会被杀掉。

基于以上几个原因，对于android消息推送在推送量不是很大的情况下可以使用第三方推送服务。