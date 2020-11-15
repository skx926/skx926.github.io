---
title: 移动端H5页面选择图片的各种姿势
date: 2018-12-28 17:05:50
tags: iOS
---

## 通过HTML的`<input>`标签调用系统api进行选择

通过下面的代码就可以调用系统的api选择图片

```html
<!-- 选择文件 -->
<input type="file">
<!-- 选择图片 -->
<input type="file" accept='image/*'>
<!-- 选择多张图片 -->
<input type="file" multiple accept='image/*'>
<!-- 拍照 -->
<input type="file" capture='camera' accept='image/*'>
```

这种方式的优点就是使用简单，缺点也很明显：

- 没有办法进行自定义的操作，比如说我们希望选择到的图片是压缩过的图片，而不是原图
- UI是系统默认的样式，不能自定义

综上所述，我们伟大的产品经理美眉肯定是不会满足于系统的样式的，所以才有了下面的姿势

## 通过Naitive端选择图片回传给H5

这个姿势就涉及到Hybrid混合开发的框架了，现在很多App里面都有一套自己的Hybrid框架，关于这个东西，这里就不多说了，简而言之：就是h5页面通过jsbridge来和Native端进行交互，调用原生的一些能力。

既然h5可以调用Native的能力了，那我们就可以在Native实现一个高度自定义的图片选择器，解决前一种姿势不候灵活的问题。

这个姿势乍一看很完美，但是也有个很棘手的问题：由于iOS应用程序沙盒的限制，我们把本地图片路径回传给h5之后，h5其实是没有权限来访问这个路径的，当然也就拿不到图片的数据。

要解决这个问题，有两套方案：

- 方案一：Native端拿到图片之后将它上传到图片服务器，然后把图片的地址回传给h5，h5通过这个地址获取图片

- 方案二：Native端拿到图片之后将图片数据转成base64的字符串，直接回传给h5

- 方案三：Native端拿到图片之后虚拟出一个url scheme或者http地址回传给h5，h5在加载这个地址的时候Native端拦截这个请求，然后把图片数据作为请求的response进行返回

  ![]({{site.url}}/assets/img{{page.id}}/flow.png)

对于方案一，我们应该都很熟悉。这个方案的优点就是用户的图片都会在服务端存储一份，对于做用户数据分析什么的有用；缺点也很明显：

1. 需要Native端把图片上传到服务端，图片比较大、多，或者网络环境差的时候耗时会很久
2. 需要服务端资源的支持：包括技术人员、服务器等。数据量比较大的话服务器和带宽的开支会很大
3. 如果第三方h5需要将图片存储到自己的服务器，就需要先通过网络请求把图片又下载到本地再上传

对于方案二，相对于其他两种方案，优点就是步骤简单，但是缺点很致命：由于我们h5和Native交互都采用的是json格式的字符串，安卓端使用的Google自家的[Gson库](https://github.com/google/gson)对于解析json文件的大小有所限制。如果要同时传几张原图的话，那数据的大小就得有几十MB，这个大小超过限制。再者，将图片的二进制数据转为base64的string，也有一定的性能损耗。

对于方案三，缺点几乎没有，我们来说说它的优点：

1. Native拿到图片之后存储在沙盒里，速度快，不会涉及到服务端的任何东西
2. 第三方h5获取图片数据也是在应用内完成，不涉及网络，速度很快
3. 对于数据的大小没有限制，后期要扩展支持视频或者其他文件也很方便

结果显而易见，我们最终采用了方案三，下面我们来说说方案三具体的实现。

#### 保存图片到沙盒

我们使用自定义的图片选择器选择完成之后会拿到一个UIImage的数组，我们把这些图片进行压缩之后保存在`/tmp`文件夹里，这里需要注意给每张图片一个唯一的名称，而且为了和下次选择的图片名称不冲突，有个简单的办法就是用当前时间戳（iOS的时间戳是精确到毫秒的，没有人可以在1毫秒内选择两次图片），然后再拼一个图片的序列号，比如`1548918117.421-6.jpg`。

#### 生成虚拟地址

生成虚拟地址有两种方式，这两种方式的请求我们客户端都是可以拦截到并返回正确的数据的，但是h5使用起来的感受却是完全不同的。

1. 自定义scheme：比如`shfile://1548918117.421-6.jpg`

2. 自定义host：比如`https://custom-host/1548918117.421-6.jpg`

#### H5加载图片

如果h5只是使用`<img>`标签来展示选择到的图片的话，上面这两种方式没有任何区别。如果h5要获取到图片的数据上传自己的后台的话，第二种方式就可以通过`XMLHttpRequest`发起一个http请求就能获取到图片数据：

```javascript
let request = new XMLHttpRequest();
request.open('GET', file.path);
request.responseType = 'blob';
request.onreadystatechange = function () {
    if (request.readyState !== 4) return;
    if (request.status == 200) {
        // 拿到二进制图片数据
        let blob = request.response;
    } else {
        console.log('下载失败');}
    }
};
request.send();
```

而第一种方式就无法通过http请求来获取了，但是也是有办法获取的，我所知道的一种办法就是在`<img>`的`onload`方法中通过canvas重绘图片从而拿到图片数据:

```javascript
let image = new Image();
image.onload = function() {
    let canvas = document.createElement('canvas');
    canvas.width = this.width;
    canvas.height = this.height;
    let ctx = canvas.getContext('2d');
    
    // 重绘
    ctx.drawImage(this, 0, 0);
    
    // 拿到base64形式图片数据
    let dataURL = canvas.toDataURL('image/png');
}
image.src = 'shfile://1548918117.421-6.jpg';
```

这种方式在请求加载获取图片之外还需要创建Image对象、创建canvas对象、重绘图片、base64转换等步骤，毫无疑问是很低效的，所以我们采用第二种方式。

#### 通过NSURLProtocol拦截请求并返回数据

关于NSURLProtocol，我在另一篇[文章](http://ky1e.me/2019/01/24/32.nsurlprotocol)中做了介绍，这里假设你已经知道它怎么用了。

我们首先注册一个https的scheme，然后创建一个继承于`NSURLProtocol`的类，这个类的头文件和实现文件如下所示：

CustomURLProtocol.h

```objc
#import <Foundation/Foundation.h>

static NSString * const CustomInterceptHost = @"custom-host";

@interface CustomURLProtocol : NSURLProtocol

@end
```

CustomURLProtocol.m

```objc
#import "CustomURLProtocol.h"

@implementation CustomURLProtocol

+ (BOOL)canInitWithRequest:(NSURLRequest *)request {
    // 只拦截我们自定义host的请求
    if ([request.URL.host isEqualToString:CustomInterceptHost]) {
        return YES;
    }
    return NO;
}

+ (NSURLRequest *)canonicalRequestForRequest:(NSURLRequest *)request {
    return request;
}

- (void)startLoading {
    // 获取链接中的图片名称
    NSString *name = self.request.URL.lastPathComponent;
    NSString *filePath = [[self directory] stringByAppendingString:name];
    
    // 设置请求头，解决跨域问题
    NSMutableDictionary<NSString *, NSString *> *headerFields = [NSMutableDictionary<NSString *, NSString *> dictionary];
    headerFields[@"Access-Control-Allow-Origin"] = @"*";
    headerFields[@"Access-Control-Allow-Headers"] = @"Origin, X-Requested-With, Content-Type";
    headerFields[@"Content-Type"] = self.request.allHTTPHeaderFields[@"Accept"];
    
    // 创建http响应请求
    NSHTTPURLResponse *httpResponse;
    if ([[NSFileManager defaultManager] fileExistsAtPath:filePath]) {
        httpResponse = [[NSHTTPURLResponse alloc] initWithURL:self.request.URL statusCode:200 HTTPVersion:nil headerFields:headerFields];
    }
    else {
        httpResponse = [[NSHTTPURLResponse alloc] initWithURL:self.request.URL statusCode:404 HTTPVersion:nil headerFields:headerFields];
    }
    
    NSData *imageData = [NSData dataWithContentsOfFile:filePath];
    
    // 响应请求
    [self.client URLProtocol:self didReceiveResponse:httpResponse cacheStoragePolicy:NSURLCacheStorageNotAllowed];
    
    // 返回图片数据
    [self.client URLProtocol:self didLoadData:imageData];
    
    // 结束响应
    [self.client URLProtocolDidFinishLoading:self];
}

- (void)stopLoading {
    // Do nothing.
}

// 图片存储路径
- (NSString *)directory {
    return [NSString stringWithFormat:@"%@/tmp/", NSHomeDirectory()];
}

@end

```

实现的逻辑比较简单，上面的代码里面我也做了注释，这里就不多讲了。

以上就是移动端H5页面选择图片的各种姿势，不知道还有没有其他更好的姿势，如果有的话，还请在下面留言交流。