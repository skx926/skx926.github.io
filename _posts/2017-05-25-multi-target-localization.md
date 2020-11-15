---
title: Xcode多Target下本地化App名称
date: 2017-05-25 12:43:00
tags: [本地化]
Categories: [iOS]
---

## Target

可能很多人都会有开发多个相似的App的需求，这些相似的App可能也就是名称、BundleID、证书配置不同，其他的功能都基本一样。对于这种情况的处理，一种比较笨的办法就是手动修改 `Info.plist` 文件中相应的内容，打包完了之后再次修改再打包。这种方法如果只是搞一次也不会麻烦很多，但是如果多搞几次，谁都会觉得烦，况且在修改的过程中还有可能会出错。那有没有什么更好的办法呢？答案是：使用 `Target`。

那么什么是 Target 呢？简单点说，一个 Target 描述了一个产品的属性并且包含了产品所需要的一系列文件。而一个 Project 中就包含多是一个或者多个产品所需要的文件、资源、信息等。所以一个Project可以包含多个 Target，你可以在一个 Target 中对 Project 中包含的文件、资源和信息进行不同的组合从而生成不同的产品。

## 多Target下本地化App名称

对于单个Target下App名称的本地化比较简单，我这里就不赘述了，我们来看一下多Target下怎么本地化App的名称：

首先创建一个名为 `LocalizationDemo` 的项目，创建完成之后如下图所示：

![]({{site.url}}/assets/img{{page.id}}/image0.png)

Xcode默认创建了一个名为 `LocalizationDemo` 的 Target，我们修改它的名字为 `Product1` 并在修改后的名字上点击右键选择 `Duplicate` 复制一个 Target（在弹出的对话框中选择Duplicate Only），这样就会创建一个名为 `Product1 copy`的 Target，同时会创建一个名为 `Product1 copy-Info.plist` 文件，我们将建的 Target 的名字修改为 `Product2`，结果如下图所示：

![]({{site.url}}/assets/img{{page.id}}/image1.png)

这个自动创建的 `Product1 copy-Info.plist` 文件在 `$(SRCROOT)`(工程文件.xcodeproj文件所在目录)目录下，而 `Info.plist` 文件则在 `$(SRCROOT)/LocalizationDemo` 目录下。为了统一起见，我们把 `Product1 copy-Info.plist` 文件移动到 `$(SRCROOT)/LocalizationDemo` 目录下并改名为 `Product2-Info.plist`，同时修改 `Info.plist` 文件名为 `Product1-Info.plist`。结果如下图所示：

![]({{site.url}}/assets/img{{page.id}}/image2.png)

由于我们修改了 `Info.plist` 文件的名称和路径，Xcode表示找不到它了，所以 `Identity` 那里会出现一个名叫 `Choose Info.plist file` 的按钮，我们分别点击两个Target中的按钮选择对应的文件，完成之后如下图所示：

![]({{site.url}}/assets/img{{page.id}}/image3.png)

现在我们添加需要本地化的语言，这里作为演示就只选择一个简体中文，添加完后效果如图所示：

![]({{site.url}}/assets/img{{page.id}}/image4.png)

对于单个Target的App名称的本地化我们只需要在项目中新建一个 `InfoPlist.strings` 文件对不同的语言设置不同的 `CFBundleDisplayName` 就可以了。但是这里有个问题就是 `InfoPlist.strings` 文件的名称是固定的，它不像 `Info.plist` 文件一样可以用不同的名称，然后在 `Build Setting` 中给不同的 Target 设置不同的 `Info.plist File` 路径和名称就可以了。

幸运的是我们可以通过建立多个不同路径下的 `InfoPlist.strings` 文件来实现对多个 Target 下的App名称进行本地化。我们在 `$(SRCROOT)/LocalizationDemo` 目录下分别建立 `Product1` 和 `Product2` 两个文件夹，分别包含一个 `InfoPlist.strings` 文件，效果如下图所示：

![]({{site.url}}/assets/img{{page.id}}/image5.png)

然后我们分别将两个文件夹添加到项目中来，添加的时候注意勾选对应的Target：

![]({{site.url}}/assets/img{{page.id}}/image6.png)

添加完成之后我们分别在两个InfoPlist.strings文件中添加 `CFBundleDisplayName = "Product Name";` 对应的名称，结果如下图所示：

![]({{site.url}}/assets/img{{page.id}}/image7.png)

然后我们点击右侧的 `Localize` 按钮分别对两个 `InfoPlist.strings` 文件进行本地化，勾选需要的语言并修改不同语言下对应的名称，这里我设置 Product1 的英文名为 `Product1`、中文名为`产品1`，Product2 的英文名为 `Product2`、中文名为`产品2`：

![]({{site.url}}/assets/img{{page.id}}/image8.png)

到这里我们的目的基本上已经达成了，但是我们运行之前发现虽然改了Target的名称，但是左上角的 Scheme 的名称并没有变化，看起来很不爽。单击 Scheme 名称然后选择 `Manage Schemes`, 然后修改 `LocalizationDemo` 为 `Product1`，`Product1 copy` 为 `Product2`。修改完效果如下图：

![]({{site.url}}/assets/img{{page.id}}/image9.png)

All done! 赶紧选择不同的 Scheme 运行一下然后把系统切换到不同的语言看看是不是成功了！

根据本文创建的示例项目我已经上传到了 Github 上，欢迎下载查看：

[Multi-TargetLocalizationDemo](https://github.com/skx926/Multi-TargetLocalizationDemo)