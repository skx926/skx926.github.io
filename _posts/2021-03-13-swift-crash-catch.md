---
title: Swift Crash 捕获和分析
date: 2021-03-13 14:27:34
tags: [Swift, Crash, Signal]
categories: [iOS]
---

## iOS中的异常

要捕获Crash，我们先需要明确几个概念及其之间的关系：`硬件异常`、`软件异常`、`mach异常`、`Signal信号`，这四种异常概念，自底向上构建了iOS系统的异常处理模型。他们之间的关系如下图所示：

![]({{site.url}}/assets/img{{page.id}}/exception.png)

iOS中的Crash主要分为`Mach Exception`、`Sinal` 和`NSException`，每一种都处在不同的系统层级上，也有各自不同的捕获方式。



### Mach Exception

Mach是一个XNU的微内核核心，Mach异常是指最底层的内核级异常，被定义在 `<mach/exception_types.h>`下 。Mach异常由CPU陷阱引发，在异常发生后会被异常处理程序转换成Mach消息，接着依次投递到`thread`、`task`和`host`端口。

如果没有上述任何一个端口来处理这个异常并返回`KERN_SUCCESS`，那么应用将被终止。每个端口都拥有一个异常端口数组，系统暴露了后缀为`_set_exception_ports`的多个 API 让我们注册对应的异常处理到端口中，用以来捕获Mach异常，抓取Crash事件。

Mach异常如果不处理，默认会转化为Signal异常（所有Mach异常都默认在`host`层被`ux_exception`转换为相应的Signal，并通过`threadsignal`将信号投递到出错的线程）。如：`EXC_BAD_ACCESS(SIGSEGV)`表示的意思就是：Mach层的EXC_BAD_ACCESS异常，在host层被转换成SIGSEGV信号投递到出错的线程。

而软件异常也会被转化为Signal。因此软/硬件异常最终都会转化为Signal信号，但他们的处理流程是不同的。



### Signal

UNIX signals是一套基于`POSIX标准`开发的通信机制，POSIX API 就是通过 Mach 之上的 BSD 层实现的。

在`signal.h`中声明了32种异常信号，以下六种为iOS常见的信号，它们均会导致程序崩溃。下面列举几个常见的信号：

| Signal  | 说明                                             |
| ------- | ------------------------------------------------ |
| SIGILL  | 执行了非法指令，一般是可执行文件出现了错误       |
| SIGTRAP | 断点指令或者其他trap指令产生                     |
| SIGABRT | 调用abort产生                                    |
| SIGBUS  | 非法地址。比如错误的内存类型访问、内存地址对齐等 |
| SIGSEGV | 非法地址。访问未分配内存、写入没有写权限的内存等 |

Signal异常我们可以通过注册信号处理函数来捕获：

```swift
func registerSignal() {
    signal(SIGABRT, crashHandler)
    signal(SIGSEGV, crashHandler)
    signal(SIGBUS, crashHandler)
    signal(SIGTRAP, crashHandler)
    signal(SIGILL, crashHandler)
}

func crashHandler(_ signal: Int32) {
    // 处理异常
    let callStacks = Thread.callStackSymbols
}
```



### NSException

NSException就属于我们前面所说的软件异常。它是应用级异常，发生在`CoreFoundation`以及更高抽象层级，会通过`__cxa_throw`函数抛出异常。如果没有人为进行捕获或者在捕获回调函数中没有进行操作终止应用，那么最终会通过`abort()`函数来向进程抛出一个`SIGABRT`的信号。

`NSException`可以通过`@try-@catch`机制捕获，以此来避免应用Crash。同样地，如果没有catch处理，那么会被系统自带的错误处理所捕获，这个时候可以通过注册`NSUncaughtExceptionHandler`来捕获NSException异常。

```swift
NSSetUncaughtExceptionHandler { exception in
    // 处理异常NSException
    let name = exception.name.rawValue
    let reason = exception.reason
    let callStacks = exception.callStackSymbols
}
```



### 避免覆盖

一般一个工程里面可能会存在一个或者多个第三方的统计SDK，他们大多都会通过`NSUncaughtExceptionHandler`进行crash捕获，那么问题来了，NSSetUncaughtExceptionHandler只能设置一个回调函数，后设置的总是可以覆盖先设置的，所以先设置的处理函数在crash发生时并不会被调用。

因此正确的做法是在注册之前先通过`NSGetUncaughtExceptionHandler`获取到先前别人注册的处理函数，并对其进行备份，在我们处理完成之后再将先前的处理函数进行调用。

```swift
var previoursHandler: NSUncaughtExceptionHandler? = nil

func registerUncaughtExceptionHandler() {
    previoursHandler = NSGetUncaughtExceptionHandler()
    NSSetUncaughtExceptionHandler { exception in
        // 写log文件保存crash信息等操作
        // 调用先前的handler
        previoursHandler?(exception)
        // 杀掉程序，这样可以防止同时抛出的SIGABRT被Signal异常捕获
        kill(getpid(), SIGKILL);
    }
}
```

Signal的处理函数也会有类似的问题，也需要做类似的处理。

## Swift异常捕获

既然Mach异常、Signal、NSException都可以捕获到Crash事件，那么选用哪种方式更好呢？

理论上优先选择Mach异常，因为Mach异常会先于Signal发生，如果Mach异常的handler让程序exit了，那么Signal就永远不会到达这个进程了。

尽管Mach异常处理更有优势，但是我们还是需要注册Signal来处理软件异常，因此我们优先选择捕获Signal异常。

NSException也属于软件异常，它的优势是NSExcetion对象会包含大部分我们所需要的崩溃信息，比如crash原因、crash堆栈等等。但是它有个缺点就是只能捕获OC的异常，对于Swift无能为力。

`NSException`对象中包含了大部分我们所需要的崩溃信息，比如crash原因、crash堆栈等等，但是这种方式无法捕获到Swift的异常。

如果你的项目存在OC和Swift混编的情况，那就需要同时捕获Signal和NSException两种异常。



### 断点调试

因为Xcode屏蔽了Signal的回调，我们需要在`lldb`中输入以下命令，Signal的回调才可以进来：

```bash
pro hand -p true -s false SIGABRT 
```

> 注意：实际使用中发现，这种方式对于OC的crash有效，对于Swift的crash比如SIGTRAP还是无法进入handler中的断点。因此我们只能通过写log的方式来进行调试。



### SlideAddress

要定位crash我们需要两个东西，错误信息内存地址和SlideAddress。错误信息内存地址在捕获到的队战信息中即可获得，SlideAddress就需要我们自己来获取，通过调用下面的c函数来获取:

```objc
long getSlideAddress(void){
    long slide = 0;
    for (uint32_t i = 0; i < _dyld_image_count(); i++) {
        if (_dyld_get_image_header(i)->filetype == MH_EXECUTE) {
            slide = _dyld_get_image_vmaddr_slide(i);
            break;
        }
    }
    return slide;
}
```



### 符号解析

一般来说我们获取到的crash信息是这个样子的：

```bash
CRASH: SIGTRAP 
slideAdress:0x27f4000 
callStacks: 
0   WidgetExtension                     0x000000010283cd1c WidgetExtension + 298268
1   WidgetExtension                     0x000000010283cb40 WidgetExtension + 297792
2   libsystem_platform.dylib            0x00000001f409829c 8ED8149F-FD75-3C22-BB9C-55A34DEA5EEB + 21148
3   ???                                 0xffffff81ac3c66e4 0x0 + 18446743531138344676
4   libswiftCore.dylib                  0x00000001ac3c5d5c $ss17_assertionFailure__4file4line5flagss5NeverOs12StaticStringV_A2HSus6UInt32VtF + 380
5   WidgetExtension                     0x0000000102864c2c WidgetExtension + 461868
6   WidgetExtension                     0x0000000102865e3c WidgetExtension + 466492
7   WidgetExtension                     0x0000000102826afc WidgetExtension + 207612
8   WidgetExtension                     0x0000000102815fbc WidgetExtension + 139196
9   WidgetExtension                     0x0000000102804fcc WidgetExtension + 69580
10  WidgetExtension                     0x00000001028053fc WidgetExtension + 70652
11  libdispatch.dylib                   0x00000001a842da54 2E7BD844-2CA3-3B21-B71E-E0F771E7C54C + 10836
12  libdispatch.dylib                   0x00000001a842f7ec 2E7BD844-2CA3-3B21-B71E-E0F771E7C54C + 18412
13  libdispatch.dylib                   0x00000001a843dc40 _dispatch_main_queue_callback_4CF + 884
14  CoreFoundation                      0x00000001a87bc248 B4FCD0AF-8F9F-3BAE-9813-952E09AFEAFC + 668232
15  CoreFoundation                      0x00000001a87b6120 B4FCD0AF-8F9F-3BAE-9813-952E09AFEAFC + 643360
16  CoreFoundation                      0x00000001a87b5210 CFRunLoopRunSpecific + 600
17  Foundation                          0x00000001a9a96ffc 1813CCFA-E104-332F-8360-11A2AFC8926E + 32764
18  Foundation                          0x00000001a9aca6ac 1813CCFA-E104-332F-8360-11A2AFC8926E + 243372
19  libxpc.dylib                        0x00000001f40cdb7c 6B119FE1-7383-3007-9D3C-CD3A625AFE1D + 97148
20  libxpc.dylib                        0x00000001f40cfe9c xpc_main + 180
21  Foundation                          0x00000001a9acca38 1813CCFA-E104-332F-8360-11A2AFC8926E + 252472
22  PlugInKit                           0x00000001d8fd371c __PLUGINKIT_CALLING_OUT_TO_CLIENT_SUBSYSTEM_FOR_BEGINUSING__ + 38880
23  PlugInKit                           0x00000001d8fd3344 __PLUGINKIT_CALLING_OUT_TO_CLIENT_SUBSYSTEM_FOR_BEGINUSING__ + 37896
24  PlugInKit                           0x00000001d8fd3b34 __PLUGINKIT_CALLING_OUT_TO_CLIENT_SUBSYSTEM_FOR_BEGINUSING__ + 39928
25  ExtensionKit                        0x00000001acd16fe8 EXExtensionMain + 84
26  Foundation                          0x00000001a9c1b72c NSExtensionMain + 200
27  libdyld.dylib                       0x00000001a8471cf8 9BCEFD51-E03A-36F0-A15C-6421AF610549 + 7416 
```

Crash堆栈中的信息依次为`行号`、`image名称`、`错误信息内存地址`、`imageUDID或者image名称`、`偏移量`

获取到SlideAddress和错误信息内存地址之后我们就可以通过一个开源工具[dSYMTools](https://github.com/answer-huang/dSYMTools)还原出出错位置到具体信息。该项目的主页有关于dSYM文件的相关介绍，这里就不多说了。

我们从上往下找我们的项目target相关的行，比如我的target叫WidgetExtension，将相关信息填入dSYMTools工具中即可。

由于`错误信息内存地址 = Slide Address + 偏移量`，因此填入的数值有两种方式：

1. SlideAddress + SlideAddress + 偏移量
2. SlideAddress + 错误信息内存地址

我们以第2种为例：

![]({{site.url}}/assets/img{{page.id}}/dsymtools.png)

在选择cpu类型为arm64的时候，dSYMTools上面显示的默认的SlideAddress为`0x0000000100000000`，因此这里填入的值需要加这个值。

经过解析我们发现第一行和第二行对应的是我们的注册的crashHandler，对于定位crash没有什么用，我们再往下解析第五行就能定位到异常发生的位置了。

> 注意：如果你使用工具解析后得到的信息是类似于这样的`_hidden#30_ (in libswiftObjectiveC.dylib) (__hidden#70_:0)`,那么极有可能是因为你提交版本的时候勾选了BitCode，勾选了Bitcode之后,用户安装的二进制文件是苹果服务器经过优化后生成的,其对应的调试符号信息丢失了,所以你看到的全部是类似于`__hidden#70_:0`这样的信息，所以也无法通过还原奔溃现场找原因了。



### Xcode Build Settings

在测试的过程中发现不论是debug还是release包的crash堆栈中都包含符号信息，而不是地址，都不需要解析，这是为什么呢？这个与新建target时Xcode的几个设置相关。

#### **Strip Linked Product**

> If enabled, the linked product of the build will be stripped of symbols when performing deployment postprocessing.

开启之后Xcode就会去除不需要的符号信息（具体是哪些不需要，由Strip Style决定），去除了符号信息之后我们就只能使用 dSYM 来进行符号化了，所以需要将 Debug Information Format 修改为 DWARF with dSYM file。

Strip Linked Product 选项只在 Deployment Postprocessing 设置为 YES 的时候才生效，而在 Archive 的时候 Xcode 总是会把 Deployment Postprocessing 设置为 YES 。所以我们可以打开 Strip Linked Product 并且把 Deployment Postprocessing 设置为 NO，而不用担心调试的时候会影响断点和符号化，同时打包的时候又会自动去除符号信息。



#### **Strip Debug Symbols During Copy**

> Specifies whether binary files that are copied during the build, such as in a Copy Bundle Resources or Copy Files build phase, should be stripped of debugging symbols. It does not cause the linked product of a target to be stripped—use Strip Linked Product for that.

与 Strip Linked Product 类似，但是这个是将那些拷贝进项目包的三方库、资源或者 Extension 的  Debug Symbol 去除掉，同样也是使用的 strip 命令。这个选项没有前置条件，所以我们只需要在 Release 模式下开启，不然就不能对三方库进行断点调试和符号化了。

如果依赖的 Target 是独立签名的（比如 App Extension），strip 操作就会失效，并伴随着 Warning：warning: skipping copy phase strip, binary is code signed: xxxx。此情况将依赖的 Target 中的 Strip Linked Product 修改为 YES，保证依赖的 Target 是已经去除了符号即可，Waning 忽略掉就可以了。



#### **Strip Style**

表示的是我们需要去除的符号的类型的选项，其分为三个选择项：

- All Symbols: 去除所有符号，一般是在主工程中开启。
- Non-Global Symbols: 去除一些非全局的 Symbol（保留全局符号，Debug Symbols 同样会被去除），链接时会被重定向的那些符号不会被去除，此选项是静态库/动态库的建议选项。
- Debug Symbols: 去除调试符号，去除之后将无法断点调试。



## 参考

[浅谈IOS中的Crash捕获与防护](http://shevakuilin.com/ios-crashprotection/)

[iOS Swift Crash的捕获](https://www.jianshu.com/p/d2b7a2eb36ba)

[iOS 安装包瘦身 （上篇）](https://www.cnblogs.com/zyfd/p/9668379.html)