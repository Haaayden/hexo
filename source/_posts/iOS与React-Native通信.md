---
title: iOS与React-Native通信
date: 2017-03-17 09:46:48
tags:
  - React Native
  - iOS
---
# iOS 与 React Native 通信踩坑小结
公司目前的项目主要使用的 React Native，但是 RN 这么久了还只是在 0.x 徘徊，不可避免有很多缺失的功能需要原生开发，然后嵌入到 RN 中，尤其是一些有我天朝特色的，比如支付宝支付，微信 QQ 等的第三方分享。
集成支付宝支付的时候，需要在原生里面监听支付结束的动作，然后收到支付宝返回的结果参数，再与自家的后端交互确定最终的支付结果，这里不可避免的涉及到 iOS 监听到支付结束，然后向 RN 发消息。

<!--more-->

## 支付宝带给我的伤害
iOS 相关目录大致如下：
![](https://ww2.sinaimg.cn/large/006tNbRwly1fdpn3d7nuej30770fsmx7.jpg)
`Alipay` 文件夹下即为支付宝相关文件，这里就是比较常用的使 `AlipayXss` 类遵守 `RCTBridgeModule` 协议，`#import "RCTEventDispatcher.h"` 这个是 OC 向 RN 发事件需要用到的。
```Objective-C
// AlipayXss.h
#import <Foundation/Foundation.h>
#import "RCTBridgeModule.h"
#import "RCTEventDispatcher.h"

@interface AlipayXss : NSObject <RCTBridgeModule>

@end
```
```Objective-C
// AlipayXss.m
@synthesize bridge = _bridge;

// 括号中的参数用来制定在 js 中访问这个模块的名字，不指定的情况下使用类的名字
RCT_EXPORT_MODULE(AlipayXss);
// 为声明的模块添加方法，对外暴露的方法
RCT_EXPORT_METHOD(pay:(NSString *)orderString fromApp:(NSString *)appScheme) {

  // 调用支付宝支付
   [self doAlipayPay :orderString :appScheme];

}
```
这里`RCT_EXPORT_MODULE``RCT_EXPORT_METHOD`这两个宏是将 AlipayXss 模块暴露给 RN，如此 RN 才能调用这里的支付方法。
```Objective-C
[[AlipaySDK defaultService] payOrder:orderString fromScheme:appScheme callback:^(NSDictionary *resultDic) {
    // 向商家服务端发送 resultDic 在服务端验证是否正确完成支付
    [self.bridge.eventDispatcher sendAppEventWithName:@"payResult" body:resultDic];
}];
```
支付宝文档写明支付完成时会调用这个回掉，然后我就这么写了，在这里发事件将参数传给 RN，在模拟器上表现也很正常，但是后来拿到真机上就傻了，无论如何都 RN 都接收不到这个事件，后来仔细看看文档才知道这个回掉只在用户调起的是网页版支付宝支付结束才会启用，真机上调起的是支付宝 app ，所以不会走这个回调。
支付宝 app 支付结束后启用的回掉在 AppDelegate 里，如下
```Objective-C
// AppDelegate.m
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString*, id> *)options
{
  if ([url.host isEqualToString:@"safepay"]) {
    // 支付跳转支付宝钱包进行支付，处理支付结果
    [[AlipaySDK defaultService] processOrderWithPaymentResult:url standbyCallback:^(NSDictionary *resultDic) {
      [self.bridge.eventDispatcher sendAppEventWithName:@"payResult" body:resultDic];
    }];
  }
  return YES;
}
```
这是我最开始的代码，但是相信我，如果你也这么写了那么你还是只能在 RN 那里干瞪眼，因为你同样接收不到事件，如果你再打断点一步步跟着走你还会发现上面这个方法每一步都完美地执行了，但你就是收不到消息。原因大致是AppDelegate 那里我们用处理之前的 AlipayXss 一样的手法使它遵守了 RCTBridgeModule，这会在模块启动的时候创建一个对象，但是 iOS 应用启动的时候也会创建一个 AppDelegate 的对象，这两个对象是不一样的。
StackOverFlow 上有相关问题，[链接在这里](http://stackoverflow.com/questions/35192324/react-native-sending-events-from-native-to-javascript-in-appdelegate-ios)。
但是悲剧的是没有实例呀，对于我这种 OC 都看不太懂的人真是要了老命了，后来陆陆续续看了点 OC 与 RN 通信原理解析又尝试了多种方法弄出来了一个相对简单的方法，如下：
```Objective-C
// AppDelegate.m

RCTRootView *rootView;

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  [self configureAPIKey];
  NSURL *jsCodeLocation;


  jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index.ios" fallbackResource:nil];

  rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation
                                         moduleName:@"xss"
                                  initialProperties:nil
                                      launchOptions:launchOptions];
  rootView.backgroundColor = [[UIColor alloc] initWithRed:1.0f green:1.0f blue:1.0f alpha:1];

  // 其他操作
}
```
iOS 应用启动时会走 `- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions` 方法，在这里实例化了 RCTRootView 对象，并且在这里指定了 js 的引用位置，它作为一个容器包裹着我们的 RN 应用，而在`RootView`创建之前，RN 先创建了一个 `Bridge` 对象，它是 OC 与 RN 交互的桥梁，这个东西就与我们碰到的问题息息相关，顺手一提，这里还有一个 `setUp` 方法，任务是创建 `BatchBridge`，据说大部分的工作都在这个东西里面。
```Objective-C
// AppDelegate.m
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString*, id> *)options
{
  if ([url.host isEqualToString:@"safepay"]) {
    // 支付跳转支付宝钱包进行支付，处理支付结果
    [[AlipaySDK defaultService] processOrderWithPaymentResult:url standbyCallback:^(NSDictionary *resultDic) {
      [rootView.bridge.eventDispatcher sendAppEventWithName:@"payResult" body:resultDic];
    }];
  }
  return YES;
}
```
将原本的 self 变为 rootView，这次你可以再 RN 正常地接收事件了。

## 其他的坑
有的时候发消息 RN 无法接收到，一个可能的原因是此时 RN 端的监听事件还没有来得及建立，只要写个延时就 OK。
```Objective-C
[self performSelector:@selector(FounctionName) withObject:nil afterDelay:1.0f];
```

## 参考链接
- [React Native 从入门到原理](http://www.jianshu.com/p/978c4bd3a759)
- [一篇较为详细的 iOS react-native 创建 View 过程 源码解析](http://www.jianshu.com/p/c2a458555de9)
- [ReactNative iOS源码解析](http://awhisper.github.io/2016/06/24/ReactNative%E6%B5%81%E7%A8%8B%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/)
- [React Native 与原生之间的通信(iOS)](http://www.jianshu.com/p/9d7dbf17daa5#)