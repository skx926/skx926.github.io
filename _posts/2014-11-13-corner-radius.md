---
title: iOS图片圆角实现和性能优化
date: 2014-11-13 12:31:19
tags: iOS开发
---

iOS的系统之中用了大量的圆角和圆形的图案，为了是我们的应用更加“接地气”，在iOS开发中我们也会不可避免的需要为自己的View添加圆角。

##### 我在开发中使用圆角有两种情况:

- 圆形的用户头像
- 给View或Button添加圆角

这两种情况都可以设置View的CALayer属性的cornerRadius属性来产生圆角，使用方法很简单

```objc
view.layer.cornerRadius = 5.0
```

如果只需要一个带圆角的Button或者Label，这个方法就已经可以满足你的需求了，但是对于头像，只是给UIImageView设置cornerRadius属性只是让这个View产生了圆角，真正的Image在View的上层，因此图像不会被裁剪，为了对Image进行裁剪，我们需要设置layer的另一个属性

```objc
view.layer.masksToBounds = YES
```

这样貌似很好的解决了圆形头像的问题，但是问题来了，当我真机运行程序的时候，滑动TableView会有明显的卡顿，我的界面并不复杂，TableView的Cell当中只有一个圆形的头像和几个Label，一屏也就七八个Cell，Cell的重用也没有问题，那为什么会有卡顿呢？经过多次的尝试终于发现这个罪魁祸首就是设置masksToBounds属性为YES，只要把这个属性设置为NO，界面立马就流畅了。但是设置为NO的话，圆形的头像效果就没有了，我感觉这是Cocoa框架的Bug，看来这条路是走不通了。那有没有别的办法呢？

办法是有的，既然给UIImageView添加圆角行不通我们可以选择利用CoreGraphic框架将图片直接裁剪成圆形，方法如下

```objc
// Begin a new image that will be the new image with the rounded corners
// (here with the size of an UIImageView)
UIGraphicsBeginImageContextWithOptions(self.bounds.size, NO, 0.0);

// Add a clip before drawing anything, in the shape of an rounded rect
[[UIBezierPath bezierPathWithRoundedRect:self.bounds cornerRadius:self.bounds.size.width/2] addClip];

// Draw your image
[image drawInRect:self.bounds];

// Get the image, here setting the UIImageView image
self.image = UIGraphicsGetImageFromCurrentImageContext();

// Lets forget about that we were drawing
UIGraphicsEndImageContext();
```

经过测试，这种方式在图片较多的情况下依然会有轻微的卡顿，但比起第一种方法已经改善了很多，基本可以满足项目的需求。
