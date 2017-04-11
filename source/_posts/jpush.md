---
title: 极光推送开发环境可以收到,生产环境收不到推送
tags: jpush
categories: iOS 极光推送
date: 2017-03-30
---

首先阐述一下我遇到的问题：

我们项目连通了极光推送，以前写过的项目也是这样，在开发环境下测试，好使了，但是打包ADHoc时候，就不好使了，<!--more-->当时也没在意，因为网上好多人说，只要测试好使了，证书显示配置成功了（绿灯），那就没问题了。


　　so，上线，然后上线以后，发现推送功能完全不好使！这就尴尬了，赶紧下架。

　　因为我们项目使用的是别名推送，使用极光网站推送时候，我使用了广播，别名推送，regID推送，结果 都能收到（开发环境）；然后我在生产环境－－－－－＞再次发送－－－－－＞广播，别名，regID，结果，很显然，别名收不到，这就纠结了，开始查看问题吧。

　　经过两天的爬坑，终于找到了问题所在，也正在积极解决。在这里再次感谢极光官方两位大牛，如果没有你们的帮忙，我想我还会纠结好久。    [Lris12](https://community.jiguang.cn/users/Lris/activity)            [Helperhaps](https://community.jiguang.cn/users/helperhaps/activity) 

　　好了，废话少说，阐述问题跟代码

-------

　　初次发现这个问题的时候，首先排查的就是证书配置，这里再次建议大家好好仔细的看看开发文档，因为很多错误都是细节处不注意造成的，[附上开发文档地址](https://docs.jiguang.cn/jpush/client/iOS/ios_sdk/)。因为这个项目是接手别人已经做得差不多的，我就负责收尾，所以，证书这方面我会优先查看，是否错误。

　　在developer.apple.com 中，我已经看到

![developer.apple.com](http://upload-images.jianshu.io/upload_images/3902605-0a2777d161ddf364.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

　　很明显，开发环境的证书跟生产环境的证书，都已经配置完成，那么证书是没有问题的，下面我们看看极光官网的配置：

![极光官网的配置](http://upload-images.jianshu.io/upload_images/3902605-11a65f1d008826fb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

　　也是已验证，这就奇怪了，到这，我表示，可能是代码出错了，好吧，我们来看代码。

![代码](http://upload-images.jianshu.io/upload_images/3902605-8d52a21aec08c89d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

　　别名设置，相信在官方文档中大家都明白怎么写。反复查看文档，我发现并没有任何问题这么写。经过跟  Lris12 大神的交流，觉得很可能是因为网络原因，在注册极光还没有返回成功的时候，就绑定别名，导致regID跟别名没有绑定成功。

　　解决方法：

　　添加5个监听，在监听到extern NSString * const kJPFNetworkDidLoginNotification; // 登录成功，之后再设置别名。

　　[极光集成指南](https://docs.jiguang.cn/jpush/client/iOS/ios_guide_new/#jpush-sdk1)

![代码](http://upload-images.jianshu.io/upload_images/3902605-9f2192a8e364ecc9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Ps：这里强烈建议这么写，安全第一 安全第一！！！**

　　这里附上代码：

```
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken

{

NSNotificationCenter *defaultCenter = [NSNotificationCenter defaultCenter];

[defaultCenter addObserver:self selector:@selector(networkDidReceiveMessage:) name:kJPFNetworkDidLoginNotification object:nil];

[JPUSHService registerDeviceToken:deviceToken];

}
```

//通知方法

```
- (void)networkDidReceiveMessage:(NSNotification *)notification {

[JPUSHService setTags:nil aliasInbackground:[OpenUDID value]];

dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0f * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{

[JPUSHService setTags:nil alias:[OpenUDID value] fetchCompletionHandle:^(int iResCode, NSSet *iTags, NSString *iAlias)

{

}];

});
```

//销毁通知：

```
[[NSNotificationCenter defaultCenter] removeObserver:self name:kJPFNetworkDidLoginNotification object:nil];

}
```
　　然而，问题依旧没有解决，但是这个时候，我们登录极光官网推送。我在生产环境，再次发送，广播，别名，regID，结果，都收到了！！正当我高兴的时候，发现 api推送依旧收不到！！收不到！！绝望！！

　　继续排查！！！按照开发文档，设置xcode配置，嗯 ，依旧没用。

　　这里我重点说一下 ：iOS9 之后 卸载重装后会改变token，所以registrationID会改变，如果你没有用到idfa。如果你的项目使用的是regID推送，那么你要注意，每次更新app，新用户下载app，重新下载app等一系列状况下，regID改变的问题。还有如果注册成功后，会返回，设置成功，有callback为0。这个也要注意下。

　　好，回归正题。这时候时间已经过去一天半了， 最后我觉得，我所有的代码，配置，证书，环境，都没有问题！再去极光官网看看，到底咋回事。

　　好吗 这一看 终于发现了问题所在！

![极光官网](http://upload-images.jianshu.io/upload_images/3902605-313a8945d475fa99.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![极光官网](http://upload-images.jianshu.io/upload_images/3902605-1f8d59529c776d8d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

　　大家看没看到，这里写的是推送平台iOS-dev？卧槽！！后台给我推送的居然是开发环境！！果断找后台理论！！！

　　Lris大神告诉我：「发布版本后无法有效推送」 or 「生产环境下收不到消息」 按以下步骤排查问题： 客户端－－－－＞客户端是否打包证书－－－－＞检查当前环境是否正确－－－－－＞设备里面的手机应用有没有添加/配置这个tag/ 别名－－－＞服务端注意改变环境参数，option的apns_production的值（true：生产）（false开发）。根据客户端环境改变服务端推送环境。环境要一致才能收到推送。

　　然后我们后台给我发送了一段代码 堵住了我的嘴：

```
$platform = 'android,ios' ;

$msg_content = json_encode(array('n_builder_id'=>0, 'n_title'=>$n_title, 'n_content'=>$n_content,'content-available'=>1,'apns_production'=>1));

$obj = new jpush($masterSecret,$appkeys);
```
　　我标红的位置，人家已经设置了1，为什么还不好用？经过Helperhaps大神的解释，好吧，我懂了！我们后台使用的过期的V2 api。

　　特别提示：建议不要在客户端里写代码直接调用此 API。因为 Android apk 比较容易破解，别人很容易从客户端代码里找出来调用 JPush Remote API 所需要的保密信息，从而可以模拟到你的身份来发起恶意的推送。

　　建议的使用方式是：调用 JPush Remote API 的代码放在你自己的应用服务器上。你自己的应用服务器对自己的客户端提供接口来推送消息。具体请参考推聊的作法：示例与代码。

　　升级到 v3 Push API：建议开发者升级到 v3 版本。此版本会继续支持到 2015 年。至此，这个问题才算是解决（至少对前端来说）。第一次发这样的帖子感觉有点乱 嗯。。。

　　下次大家在推送上有问题的话。。先看开发文档走一遍流程，然后看看极光个人推送，广播能否收到，然后就可以考虑跟服务端干一仗了！！

　　最后祝大家 永无BUG！！！！！！！！！！！！！

　　转自：http://www.jianshu.com/p/0d382c4d98ff


