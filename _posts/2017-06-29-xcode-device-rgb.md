---
title: 解决Storyboard中设置的颜色和代码设置的颜色不一样的问题
date: 2017-06-29 14:15:35
tags: iOS开发
---

最近发现在Storyboard中设置的颜色和用代码设置的颜色居然不一样，当时我就震惊了！

更让人震惊的是搞了这么多年iOS开发，居然现在才发现！惭愧啊。。。

网上查了一下才知道原来Xcode里的色彩选择器默认的Color Profile是`Generic RGB`，如下图所示：

![]({{site.url}}/assets/img{{page.id}}/color-picker.jpg)

把它改为`Device RGB`就可以解决这个问题了。