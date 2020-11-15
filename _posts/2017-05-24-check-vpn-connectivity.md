---
title: 检测iOS设备是否连接VPN
date: 2017-05-24 13:42:39
tags: [VPN]
categories: [iOS]
---

## 前言（废话）

最近接到一个限制我司国内版的产品在国外的使用App的需求，其实也就是让这种“水货”的产品在国外无法连接我们的App。

为了达到这个目的，首先需要从产品中获取它的序列号来判断它是否属于国内版，然后根据手机的定位判断是否在国外，如果两个条件同时满足就断开连接。

为了判断手机的定位是否在国外有以下两种方法：
1. 根据手机的GPS定位坐标反地理编码获取国家代码判断是否是CN。
2. 后台接口根据手机的IP来判断是否在国外。

这两种方式都有缺陷，第一种可以通过模拟定位到国内的方式来避开，第二种可以通过连接服务器在国内的VPN来避开。上面提这需求的时候也没说要100%封死，况且一般人也不一定会想到这个办法，所以第一版就以这两种方式结合的形式发布了。

事实证明在利益面前，国外的人民群众也能充分发挥他们的“聪明才智”。没过多久市场部的同事就反馈说在国外的用户之间已经流行开使用VPN的方式了，导致我们的限制形同虚设。

既然使用IP验证的方式已经被成功避开，那么GPS定位的方式也没啥难度，虽说现在iOS越狱的人是越来越少了，但是安卓的root还是很容易的。所以看来以正常的方式是没办法了，所以我们想到通过检测是否连接VPN的方式来预判断，如果连接了VPN就不让它连接，没有连接的情况下再通过IP的方式来进行判断。这样做虽然牺牲了一些经常挂VPN的人的体验，但是却可以很方便的实现需求。

## 检测VPN是否连接

检测的方式就是利用 `getifaddrs()` 函数来获取当前的所有的网络接口，寻找有没有包含 `tap`、`tun` 或 `ppp`，如果包含了其中的任何一个，就说明VPN已经连接了，代码如下：

```objc
#include <ifaddrs.h>

- (BOOL)isVPNConnected {
    struct ifaddrs *interfaces = NULL;
    struct ifaddrs *temp_addr = NULL;
    int success = 0;

    // retrieve the current interfaces - returns 0 on success
    success = getifaddrs(&interfaces);
    if (success == 0) {
        // Loop through linked list of interfaces
        temp_addr = interfaces;
        while (temp_addr != NULL) {
            NSString *string = [NSString stringWithFormat:@"%s" , temp_addr->ifa_name];
            if ([string rangeOfString:@"tap"].location != NSNotFound ||
                [string rangeOfString:@"tun"].location != NSNotFound ||
                [string rangeOfString:@"ppp"].location != NSNotFound){
                return YES;
            }
            temp_addr = temp_addr->ifa_next;
        }
    }

    // Free memory
    freeifaddrs(interfaces);
    return NO;
}
```

上面的代码在iOS9之后不管用了，不过我们可以通过下面的方式来检测：

```objc
- (BOOL)isVPNConnected
{
    NSDictionary *dict = CFBridgingRelease(CFNetworkCopySystemProxySettings());
        NSArray *keys = [dict[@"__SCOPED__"]allKeys];
        for (NSString *key in keys) {
            if ([key rangeOfString:@"tap"].location != NSNotFound ||
                [key rangeOfString:@"tun"].location != NSNotFound ||
                [key rangeOfString:@"ppp"].location != NSNotFound){
                return YES;
            }
        }
        return NO;
}
```