---
title: 获取App Store下载的Xcode安装包
date: 2017-06-09 11:49:58
tags: [App Store, Xcode]
categories: [Mac]
---

通过 App Store 下载安装 Xcode 的时候可能会遇到各种奇葩的事情。

这不，我刚才就摊上事了：用公司的渣网络辛辛苦苦下载两个多小时眼看开始安装了结果弹框提示我说空间不足无法继续安装。既然空间不足咱就清理一下呗，清理完了之后想要继续安装发现没有办法重新开始安装，App Store 里的按钮显示 `发生错误`，`/Applications` 文件加下的 `Xcode.appdownload` 文件双击也没有任何反应，真是哔了狗了！

如果是下载还在进行，我们就可以根据[这篇文章](http://www.cnblogs.com/On1Key/p/6169878.html) 所说的方法通过在活动监视器中查看进程 `storedownloadd` 打开的文件和端口中找到安装包的路径。

![]({{site.url}}/assets/img{{page.id}}/activity1.png)

![]({{site.url}}/assets/img{{page.id}}/activity2.png)

但是我已经下载完了，文件应该已经关闭了，所以这个方法可能用不了了。但是我们可以照葫芦画瓢，根据文章中提供的路径 `/private/var/folders/xx/xxxxx/C/com.apple.appstore/xxx/xxxx.pkg` 直接到 `/private/var/folders` 文件夹下去寻找。

![]({{site.url}}/assets/img{{page.id}}/folders.png)

最终我们发现了上图所示的文件，重点是文件的大小 `4.54GB`，跟在 App Store 下载时看到的大小相同，双击打开进行安装即可。