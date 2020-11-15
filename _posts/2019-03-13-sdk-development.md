---
title: SDK开发和打包静态库遇到的坑
date: 2019-03-13 14:27:34
tags: iOS
---

我们在使用第三方库的时候一般有三种接入方式：

1. 直接把第三方库的源码拖入工程
2. 通过CocoaPods等包管理工具进行引入
3. 通过.a或者.framework静态库引入

前两种情况一般是用于引入开源的项目，比如`AFNetworking`、`SDWebImage`；第三种情况一般是用于引入一些不方便开源的SDK，比如微信支付的SDK、百度地图的SDK。



## SDK开发注意点

如果我们要做一个静态库形式的SDK，有什么需要注意的呢？

1. 没有必要暴露的头文件就不要暴露给外部
2. 类名、分类方法名、全局变量、全局函数、枚举、宏定义要加前缀
3. SDK中引入的第三方库和接入项目中引入的第三方库冲突问题

这里的第三点是我们在SDK开发中需要重点注意的问题：比如说我们SDK中引入了`AFNetworking`，我们打包的静态库中就会包含AF的所有代码，如果接入我们SDK的项目中也引入了AF，编译的时候就会报`duplicate symbols`符号重复的错误。

怎么解决这个问题呢？有下面几种方法：

- 改名。顾名思义，改名就是把我们SDK中引入的AF库中的类名、分类方法名、全局变量、全局函数、枚举、宏定义都加上前缀。这个办法有点麻烦，后续如果要升级第三方库就又得改一遍，当然如果写一个脚本来帮我们做这个事情也挺不错。这个方法的好处就是如果接入方项目中和我们SDK中引入的AF的版本不一样时也不会出错。因为他们是完全独立的，不会互相影响。
- 使用CocoaPods引入第三方库并使用[CocoaPods Packager](https://github.com/CocoaPods/CocoaPods-packager)来打包静态库。CocoaPods Packager在打包静态库的过程中会自动将引入的第三方库的符号加上前缀（Name-Mangling）。如果我们引入的第三方库也是一个静态库，那这个自动改名就无法生效了。
- 打包的静态库中不要包含第三方库，让接入方去引入。这样就可以保证同一个第三方库只有一份了。但是如果我们SDK中使用的第三方库的和接入方项目中引入的第三方库的版本不同的时候也会有一些问题。需要双方修改成使用统计的版本，要么你改，要么我改。
- SDK中不要引入第三方库，自己写一份实现（如果你不差时间、也能写得出来的话）。

上面这三种方式各有各的优缺点，总结起来一下就是：如果SDK中引入的第三方库很少或者很简单的话，你可以考虑手动改名字或者自己实现一个；如果SDK中不会引入其他的静态库，使用CocoaPods Packager将会是你不二的选择；如果SDK中引入了静态库，那么在打包静态库的时候就不要把引入的静态库打包进去，直接把两个静态库文件（SDK的静态库和SDK引入的静态库）都提供给接入方使用。



## 使用CocoaPods Packager打包静态库

由于我们初期没有预估到SDK中会引入静态库，所以最先采用了CocoaPods Packager的方式。

### 创建私有pod库

要使用CocoaPods Packager打包静态库，我们首先得有一个pod库：

``` bash
$ pod lib create 'YourPodName'
```

使用上面的命令之后我们就可以创建一个名为`YourPodName`的本地pod库，你会注意到Pods项目中比平常多了一个Development Pods文件夹，这个文件夹下面的`YourPodName`文件夹就是我们的库的源文件的存放目录。打开Podfile文件你会看到一行`pod 'YourPodName', :path => '../'`，这个path指向的就是我们的pod库的podspec文件的路径，也就是Podfile文件的上一级目录。

打开`YourPodName.podspec`文件可以看到很多内容，我们逐个来解释一下：

``` ruby
Pod::Spec.new do|s|
  # 项目名称
  s.name             = 'YourPodName'
  # 版本
  s.version          = '0.1.0'
  # 简介
  s.summary          = 'A short description of YourPodName.'
  # 详细介绍
  s.description      = "详细介绍"
  # 项目主页
  s.homepage         = "https://github.com/skx926/YourPodName"
  # 截图                      
  s.screenshots      = "www.example.com/screenshots_1"
  # 支持的协议及文件
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  # 作者名字和邮箱
  s.author           = { 'skx926' => 'skx926@gmail.com' }
  # 仓库地址
  s.source           = { :git => 'https://github.com/skx926/YourPodName.git', :tag => s.version.to_s }
  # 社交媒体地址
  s.social_media_url = 'https://twitter.com/skx926'  
  # 最低要求的系统版本
  s.ios.deployment_target = '8.0'
  # 源文件路径
  s.source_files = 'YourPodName/Classes/**/*'
  # 资源文件存放位置
  s.resource_bundles = {
    'YourPodName' => ['YourPodName/Assets/*.png']
  } 
  # 暴露给外部的头文件
  s.public_header_files = 'YourPodName/Classes/**/*.h'
  # 项目中使用的系统framework
  s.frameworks = 'UIKit', 'MapKit'
  # 项目中使用的第三方framework
  s.vendored_frameworks = 'Thirdparty.framework'
  # 项目中使用的系统静态库
  s.libraries = 'iconv', 'xml2'
  # 项目中使用的第三方静态库
  s.vendored_libraries = 'Library/gmssl/*.a'
  # Xcode配置
  s.xcconfig = { "HEADER_SEARCH_PATHS" => "${PODS_ROOT}/../../Library/gmssl"}
  # 项目依赖的第三方库
  s.dependency 'AFNetworking', '~> 2.3'
end
```

比较麻烦的是我们要往pod库中添加源文件的时候不能只在Xcode中拖进去，如果要正常使用的话，每次添加文件都需要执行`pod install`命令。

项目中使用的资源文件需要添加到`s.resource_bundles`里面所设置的路径当中，执行`pod install`命令之后他们就会出现在Resources文件夹当中。

在项目运行的时候这些资源文件会被打包到`YourPodName.bundle`当中，这个bundle的名字就是`s.resource_bundle`中设置的。

由于这些资源被打包在bundle之中，使用的时候也需要有所变化。我们以图片为例：假如我们有一张名为`avatar@2x.png`的图片放在`YourPodName/Assets/images/`文件夹下，它在bundle中的路径就是`YourPodName.bundle/images/avatar@2x.png`，要使用这张图片的话我们首先得拿到这个bundle的实例，为了方便拿到这个实例，我们给NSBundle增加一个分类方法:

``` objc
+ (NSBundle *)ks_bundle {
    static NSBundle *resourceBundle = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        // 通过SDK中的一个类来获取SDK所在的Bundle，然后在在这个bundle中寻找存放我们资源文件的bundle
        NSString *resourceBundlePath = [[NSBundle bundleForClass:NSClassFromString(@"AnyClassInYourPod")] pathForResource:@"YourPodName" ofType:@"bundle"];
        resourceBundle = [self bundleWithPath:resourceBundlePath];
    });
    return resourceBundle;
}
```

然后再给UIImage增加一个分类方法：

``` objc
+ (UIImage *)ks_imageNamed:(NSString *)name {
    // 图片名称需要包含它的路径
    NSString *imageName = [NSString stringWithFormat:@"images/%@", name];
    return [UIImage imageNamed:imageName inBundle:NSBundle.spy_bundle compatibleWithTraitCollection:nil];
}
```

这样就可以通过`[UIImage ks_imageNamed:@"avatar"]`来获取图片了。

### 安装CocoaPods Packager

CocoaPods Packager是CocoaPods的一个插件，需要单独的命令来安装:

``` bash
$ sudo gem install cocoapods-packager
```

安装成功就可以使用`pod package YourPodName.podspec`命令来打包了，我们来看一下这个命令有哪些参数:

``` bash
// 强制覆盖之前生成的文件
--force
// 不使用name-mangling技术，也就是自动改类名等符号
--no-mangle
// 生成静态的framework
--embedded
// 生成静态.a
--library
// 生成动态framework
--dynamic
// 使用本地文件
--local
// 生成动态framework的时候需要这个BundleId来签名
--bundle-identifier
// 不包含依赖的符号表，也就是不把依赖的第三方库打包进去
--exclude-deps
// 生成debug还是release的库，默认是release
--configuration=Release 
// 如果你的pod库有subspec，那么加上这个命名表示只给某个或几个subspec生成二进制库
--subspecs=subspec1,subspec2
// 默认是CocoaPods的Specs仓库，如果你的项目依赖使用的是私有的source，就可以通过这个参数来设置
--spec-sources=private,https://github.com/CocoaPods/Specs.git

```

使用下面的命令就可以打包静态库：

```bash
$ pod package YourPodName.podspec --force --library
```

打包的流程是这样的：

1. 根据podspec里`s.source`指定的git仓库中克隆tag和`s.version`相同的版本到本地
2. 执行pod install安装依赖（这些依赖的来源就是`--spec-sources`参数所指定的）
3. 编译生成目标文件
4. 执行Name-Mangling进行改名
5. 生成最终的静态库文件

由于每次打包都是从git仓库上面克隆，所以我们在打包之前必须把本地的代码提交到git仓库并打上相应版本的tag。

这样操作确实挺麻烦，所以有人给CocoaPods Packager提交了一个[Pull Request:Add --local option to use local sources while packaging #195](https://github.com/CocoaPods/cocoapods-packager/pull/195) 来增加一个`--library`的命令来支持使用本地的环境直接打包，而不是每次都从git上面clone。

作者已经merge了这个pr，但是并没有发布新的版本，所以使用gem更新cocoapods-packager这个插件的话是更新不了的，你可以自己从[GitHub](https://github.com/CocoaPods/cocoapods-packager)上克隆源码编译安装最新版就支持这个功能了。

### Name-Mangling做了哪些事

到目前为止，如果你的SDK里面没有包含第三方的静态库，那么使用上面的命令就已经很舒服了。你可能会好奇Name-Mangling具体会改哪些东西，我们从CocoaPods Packager源码[mangle.rb](https://github.com/CocoaPods/CocoaPods-packager/blob/master/lib/CocoaPods-packager/mangle.rb)文件中可以窥探一二：

``` ruby
  #
  # performs symbol aliasing
  #
  # for each dependency:
  # 	- determine symbols for classes and global constants
  # 	- alias each symbol to Pod#{pod_name}_#{symbol}
  # 	- put defines into `GCC_PREPROCESSOR_DEFINITIONS` for passing to Xcode
  #
```

注释里写的很清楚了，Name-Mangling会把类名和全局常量改成`Pod#{pod_name}_#{symbol}`的形式，比如说我们的pod库YourPodName中有一个YourClass类，那它最终会被改成`PodYourPodName_YourClass`。

需要注意的是Name-Mangling并不会改方法名，也就是说如果我们引入的第三方库给系统类比如UIImage增加了分类方法，而恰巧接入方的源码中也引入了这个第三方库，给UIImage增加同样名称的分类方法，由于分类的特性我们可以知道他们不会冲突，而是会产生覆盖，一般来说也不会有问题。

但是如果两边引入的第三方库的版本不同，分类方法的实现也有所不同，那这就可能会产生问题。不过分类的实现一般比较固定，出现这种情况的概率比较小。



### SDK中引入静态库

我们私有pod库中要引入静态库很简单，只需要在podspec文件中配置好静态库的路径`s.vendored_libraries = 'YourLibrayPath/*.a'`，并把静态库的头文件加入到源码路径中`s.source_files = 'YourPodName/Classes/**/*'`，执行`pod install`就可以正常使用了。

但是如果这个静态库比较复杂，比如说gmssl，他里面包含两个静态库和几十上百个头文件，事情就变得比较棘手了，如下图所示：

![]({{site.url}}/assets/img{{page.id}}/gmssl.png)

主要问题在与这些个头文件之间是用尖括号`<>`以模块的形式引入的，比如：

``` c
#include <openssl/e_os2.h>
#include <openssl/opensslconf.h>
```

这里就涉及到一个头文件的搜索路径的问题，如果我们直接把这些头文件加入到项目当中去就会报找不到头文件的异常。我们可以手动把他们都改成双引号的形式:

``` c
#include "e_os2.h"
#include "opensslconf.h"
```

这种办法虽然可以解决问题，但是几十个文件改起来也不是件轻松的事情。更简单的办法是在`Pods.xcodeproj`文件的`Build Settings`中设置`Header Search Paths`，让它指向openssl所在的文件夹。`Pods.xcodeproj`中的设置在每次执行`pod install`之后都会被重置，这个时候我们上面提到的在podspec文件中设置`s.xcconfig = { "HEADER_SEARCH_PATHS" => "${PODS_ROOT}/../../Library/gmssl"}`就排上用场了。

通过这种方式引入的静态库在我们的Demo project下是可以正常运行的，但是一旦用CocoaPods Packager打包还是会报找不到静态库头文件的异常。这个问题我暂时知道的唯一的解决办法就是上面提到的直接把头文件中的互相引用改成双引号的形式。

假如你被逼的没办法手动或者写程序改完了这几十号头文件，那么你可以满怀期待的用我们上面提到的命令来打包了:

``` bash
$ pod package YourPodName.podspec --force --library
```

然而你又会收到另一条错误提示`[!] podspec has binary-only depedencies, mangling not possible.`，没错，Name-Mangling无法对静态库生效，我们只能暂时先选择不使用它:

``` bash
$ pod package YourPodName.podspec --force --library --no-mangle
```

这下终于可以打包成功了！

但是...你会发现最终生成的静态库文件会包含我们引入的第三方静态库，即便你使用了`--exclude-deps`参数，它也只会去除我们引入的第三方开源库，静态库还是会被打包进去。那如果接入方也引入了这个第三方静态库，不还是会冲突？

What the f**k!!!

看来这条路是走不通了...

最终我们还是得回到Xcode上来...



## 使用Xcode打包静态库

在我们的Demo project里面新增一个名为YourPodName的Library类型的Target，或者你重新建一个新的Library类型的项目，然后把我们pod库中的源文件都加进这个新的target，就像下面这样:

![]({{site.url}}/assets/img{{page.id}}/target.png)

修改一下`Build Settings`里的配置：

1. 设置Supported Platforms为iOS
2. 在Valid Architectures中添加`armv7`、`armv7s`、`arm64`、`arm64e`、`x86_64`等常用的架构

3. 手动设置`Header Search Paths`和`Library Search Paths`以引入第三方静态库。

不要忘记在Podfile当中新增下面的代码来让新的Target可以正常引入我们依赖的第三方开源库:

``` ruby
target 'SmartPay' do
  pod '第三方开源库'
end
```

然后执行`pod install`，你会发现CocoaPods在`Framework Search Paths`里面自动添加了类似`"/build/Debug/AFNetworking"`这样的路径，这个就是我们通过CocoaPods引入的第三方开源库最终编译成的framework的路径。

点击运行之后就会生成最终的静态库。这里生成的静态库是不会包含我们引入的第三方静态库和开源库的，所以在接入方接入我们的SDK的时候我们需要下面的这些东西:

1. SDK的静态库和头文件
2. 引入的第三方静态库和头文件
3. CocoaPods生成的第三方开源库的.framework静态库（如果接入方项目中已经包含就不需要再引入了）
4. 引入的系统framework名称列表

### 合并静态库文件

我们可以在项目左侧Products文件夹下面亩看到一个名为libYourPodName的静态库文件。点击右键`Show in Finder`就可以在Finder中看到它。使用下面的命令可以查看它所包含的架构:

```bash
$ lipo -info 静态库路径
```

如果你运行的时候选择的是模拟器，它就只包含x86_64架构，如果你选择的是Generic iOS Device，那么它会包含我们上面设置的arm架构。

对外我们可以把模拟器版和真机版的静态库分别输出或者使用下面的命令将他们合并成一个方便使用:

``` bash
$ lipo -create 静态库1路径 静态库2路径 -output 新静态库路径
```



### 接入SDK注意点

接入方接入我们的SDK就跟我们接入别人的静态库一样，除了需要在`Build Settings`中设置`Header Search Paths`和`Library Search Paths`以外还需要在`Build Phases`中的`Link Binary With Libraries`中添加我们SDK中引入的系统的framework。

然后把我们提供给他们的第三方开源库的静态framework拖入项目当中，这步操作就会把这些framework自动加入到`Build Phases`中的`Link Binary With Libraries`中去。这样还不够，因为这些第三方开源库是披着framework外衣的静态库，并不能像系统库那样动态链接。所以我们得在`General `中的`Embedded Binaries`中加入这些framework，这样他们就会像.a静态库一样内嵌到ipa当中去。

经过上面这些设置，项目已经可以正常运行了，但是在调用我们SDK中的分类方法时还是会报找不到Selector的错误，导致崩溃。

对于这个问题，只需要在`Build Settings`中的`Other Linker Flags`中添加`-ObjC`标志就可以了。

### -ObjC标志的作用

Objective-C没有为每个函数（或者方法）定义链接符号，它只为每个类创建链接符号。这样当在一个静态库中使用类别来扩展已有类的时候，链接器不知道如何把类原有的方法和类别中的方法整合起来，就会导致你调用类别中的方法时，出现”selector not recognized”，也就是找不到方法定义的错误。为了解决这个问题，引入了-ObjC标志，它的作用就是让链接器将静态库中所有的Objective-C的代码都链接进来。

在64位的Mac系统或者iOS系统下，链接器有一个bug，会导致只包含有类别的静态库无法使用-ObjC标志来链接Objective-C代码。解决方法是使用-all_load 或者-force_load标志，它们的作用都是链接静态库中所有代码，不过all_load作用于所有的库，而-force_load后面必须要指定具体的文件。