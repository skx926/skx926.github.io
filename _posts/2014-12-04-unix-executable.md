---
title: Mac如何生成Unix可执行文件
date: 2014-12-04 02:00:00           
tags: Mac技巧
---

Yosemite升级10.10.2 prerelease 版本之后蛋疼的事情发生了，safari打不开github不说，就连google chrome也一打开就崩溃，升级到40的dev版本依旧崩溃，网上找了找，遇到同样问题的还挺多，有人已经提供了临时的解决办法

将以下代码保存为`patch.m`文件

```objc
#import <AppKit/AppKit.h>
__attribute((constructor)) void Patch_10_10_2_entry()
{
    NSLog(@"10.10.2 patch loaded");
}

@interface NSTouch ()

- (id)_initWithPreviousTouch:(NSTouch *)touch newPhase:(NSTouchPhase)phase position:(CGPoint)position isResting:(BOOL)isResting force:(double)force;

@end

@implementation NSTouch (Patch_10_10_2)

- (id)_initWithPreviousTouch:(NSTouch *)touch newPhase:(NSTouchPhase)phase position:(CGPoint)position isResting:(BOOL)isResting
{
    return [self _initWithPreviousTouch:touch newPhase:phase position:position isResting:isResting force:0];
}

@end
```

下面需要将源文件生成一个动态库, 前提是要装Xcode,在终端中执行下面的代码,完成后桌面的`patch.m`文件会变成`patch.dylib`动态链接库文件.

```bash
$ clang -dynamiclib -framework AppKit ~/Desktop/patch.m -o ~/Desktop/patch.dylib
```

然后在终端中执行下面的命令给`Chrome`打补丁,同时Chrome浏览器也会打开

```bash
$ env DYLD_INSERT_LIBRARIES=~/Desktop/patch.dylib "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
```

这样就大功告成了,Chrome不会崩溃了,但是这个终端窗口不能关,关了之后Chrome就无法正常运行了.但是这样新的问题就来了,重新打开都需要在终端执行第3步的命令,如果每次都这样太麻烦了,一点也不优雅,有没有优雅一点的办法呢?当然有,下面就来说说如何今天的重点

##### 如何生成Mac上的可执行文件

我们需要用到类Unix操作系统的一个命令`chmod`, `chmod`是用来改变文件文件的存取模式的, 我猜就是Change Mode的意思.

首先我们新建一个文本文件, 输入上面第3步的终端命令,另存为没有格式的文件, 比如文件名为Chrome.

```bash
$ clang -dynamiclib -framework AppKit ~/Desktop/patch.m -o ~/Desktop/patch.dylib
```

然后打开终端, 输入下面的命令

```bash
$ sudo chmod u+x script
```

注意`script`就是刚才创建的`Chrome`文件的绝对路径,比如此处将`script`替换为`/Users/Sun/Desktop/Chrome`.

执行完命令之后我们就会发现`Chrome`文件的图标变成了类似终端的图标的样子,这样就已经生成了可执行文件了, 双击这个文件就会自动打开终端执行文件里的命令启动Chrome浏览器了.

这条`chmod`命令中的u就是User用户, `+`是增加权限, `x`是执行, 总的来说就是给这个文件增加用户和执行的模式.

对于`chmod`命令更加详细的解释可以参看这里:[Linux chmod命令详解](http://www.cnblogs.com/younes/archive/2009/11/20/1607174.html)
