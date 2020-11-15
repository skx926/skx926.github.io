---
title: 嵌套UIScrollView滑动手势冲突的解决
date: 2017-05-28 10:55:23
tags: iOS开发
---

## 问题

最近有有朋友在Github上提出issue说 [KSPhotoBrowser](https://github.com/skx926/KSPhotoBrowser/issues/14) 在图片长度超出屏幕大小多时候没有办法通过拖拽返回。其实这个问题从我开始开发的时候就存在，感觉影响不是很大就一直拖着没解决。既然有人提出来了，那看来是必须得解决一下了。。。

![]({{site.url}}/assets/img{{page.id}}/normal.gif)

![]({{site.url}}/assets/img{{page.id}}/abnormal.gif)

第一张图就是正常的拖拽返回，第二张有问题。

## 解决

要解决这个问题我们不得不说下 [KSPhotoBrowser](https://github.com/skx926/KSPhotoBrowser) 的布局结构，先来看一张图：

![]({{site.url}}/assets/img{{page.id}}/layout.jpg)

如上图所示，一个父 UIScrollView 里面添加了多个子 UIScrollView，每个子 UIScrollView 里包含一个 UIImageView 来显示图片，可以通过左右滑动父 UIScrollView 来查看不同的图片。单击、双击和拖动手势是添加在父 UIScrollView 的 SuperView 上，也就是 ViewController 的根 View 之上。

当图片的高宽比小于屏幕大高宽比的时候，拖动手势可以正常触发（如上图1所示）。但是当图片的高宽比大于屏幕的高宽比的时候（也就是图片比较长），拖动手势就无法正常触发（如上图2所示）。

经过查看 UIScrollView 的头文件发现 UIScrollView 内部也有一个拖动手势:

```objc
// Use these accessors to configure the scroll view's built-in gesture recognizers.
// Do not change the gestures' delegates or override the getters for these properties.

// Change `panGestureRecognizer.allowedTouchTypes` to limit scrolling to a particular set of touch types.
@property(nonatomic, readonly) UIPanGestureRecognizer *panGestureRecognizer NS_AVAILABLE_IOS(5_0);
```

所以当我们往下拖动的时候 UIScrollView 的拖动手势拦截了事件，使其无法传递给下方的 View 上的手势。那我们能不能根据滑动时 UIScrollView 的 `contentOffset` 和滑动的方向来决定触发哪一个手势呢？答案是肯定的，通过查看 UIGestureRecognizer 的头文件我们发现下面这个代理方法：

```objc
// called when a gesture recognizer attempts to transition out of UIGestureRecognizerStatePossible. returning NO causes it to transition to UIGestureRecognizerStateFailed
- (BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer;
```

利用这个方法我们就可以在手势识别阶段 `UIGestureRecognizerStatePossible` 根据 UIScrollView 的 `contentOffset` 和滑动的方向来决定要不要触发 UIScrollView 的手势，如果我们 `return NO`，事件就会正常传递给下面的 View 上的手势，从而解决问题。

所以现在我们使持有 UIImageView 的 KSPhotoView 继承自 UIScrollView 并重写 UIScrollView 的拖动手势的代理方法，代码如下：

```objc
// 判断scrollView是不是在最顶部往下滑或者在最底部往上滑，如果是这两种情况才需要把事件往下传递
- (BOOL)isScrollViewOnTopOrBottom {
    CGPoint translation = [self.panGestureRecognizer translationInView:self];
    if (translation.y > 0 && self.contentOffset.y <= 0) {
        return YES;
    }
    CGFloat maxOffsetY = floor(self.contentSize.height - self.bounds.size.height);
    if (translation.y < 0 && self.contentOffset.y >= maxOffsetY) {
        return YES;
    }
    return NO;
}

#pragma mark - GestureRecognizerDelegate

- (BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer {
    if (gestureRecognizer == self.panGestureRecognizer) {
        if (gestureRecognizer.state == UIGestureRecognizerStatePossible) {
            if ([self isScrollViewOnTopOrBottom]) {
                return NO;
            }
        }
    }
    return YES;
}
```

运行一下代码发现问题完美解决：

![]({{site.url}}/assets/img{{page.id}}/nice.gif)