---
title: 利用Pinch和Pan手势实现UIView的缩放和拖拽
date: 2017-06-21 14:25:01
tags: [手势]
categories: [iOS]
---

先来看下我们要实现的效果：

![Demo](https://raw.githubusercontent.com/skx926/ScalableViewDemo/master/Demo.gif)

其实这个效果UIScrollView已经自带了，但是某些情况下由于某些原因我们可能不能或者不想使用UIScrollView，那么我们就得自己实现这个效果。

如果要单独用Pinch手势实现缩放或者用Pan手势实现拖拽，这个比较简单，直接贴代码：

```swift
// 利用Pinch手势实现缩放
func didPinch(_ pinch: UIPinchGestureRecognizer) {
    if pinch.state != .changed {
        return
    }
    let scale = pinch.scale
    let scaleTransform = imageView.transform.scaledBy(x: scale, y: scale)
    imageView.transform = scaleTransform
    pinch.scale = 1
}
```

```swift
// 利用Pan手势实现拖拽
func didPan(_ pan: UIPanGestureRecognizer) {
    if pan.state != .changed {
        return
    }
    let scale = imageView.frame.size.width / initialFrame.size.width
    let translation = pan.translation(in: view)
    let transform = imageView.transform.translatedBy(x: translation.x / scale, y: translation.y / scale)
    imageView.transform = transform
    pan.setTranslation(.zero, in: view)
}
```

确实是很简单，但是从不同位置缩放一下图片你就会发现不论你的手指在什么位置，图片总是以中心为基准进行放大缩小，感觉很不自然。这是为什么呢？因为我们上面的代码只是单纯根据手势的scale对图片进行了缩放，根本没有考虑到手指的位置。要有良好的缩放体验，一般来说都需要以`两指的中心点`为基准来进行缩放。这样的话我们就需要在缩放的过程中对图片再增加一个偏移，那么这个偏移的值具体是多少呢？我们来看一张图：

![]({{site.url}}/assets/img{{page.id}}/position.png)

如上图所示，假设我们有一个初始状态的View为A，他的中心点为a，我们以点b为基准把A放大一倍，理想的结果就是C，缩放的基准点b点不会发生位移。

我们可以把这个过程拆分成两个步骤：
1. 把A放大一倍至B
2. 给B添加偏移 (x, y) 至C

那么偏移(x, y)怎么求得呢？

根据上图我们可以计算出基准点b相对于中心点a的偏移(dx, dy)：
```swift
// location为Pinch手势的中心点b
let dx = imageView.frame.midX - location.x
let dy = imageView.frame.midY - location.y
```
然后计算出放大之后的基准点b相对于中心点a的偏移(sdx, sdy)：
```swift
// scale为缩放的比例
let sdx = dx * scale
let sdy = dy * scale
```
由图可得(x, y)为上面的两项的差：
```swift
let x = sdx - dx
let y = sdy - dy
```

最终的代码如下：
```swift
func didPinch(_ pinch: UIPinchGestureRecognizer) {
    if pinch.state != .changed {
        return
    }
    let scale = pinch.scale
    let location = pinch.location(in: view)
    let scaleTransform = imageView.transform.scaledBy(x: scale, y: scale)
    imageView.transform = scaleTransform
 
    let dx = imageView.frame.midX - location.x
    let dy = imageView.frame.midY - location.y
    let x = dx * scale - dx
    let y = dy * scale - dy
 
    // 这样的话计算会有错误
    //imageView.transform = imageView.transform.translatedBy(x: x, y: y);
 
    let translationTransform = CGAffineTransform(translationX: x, y: y)
    imageView.transform = imageView.transform.concatenating(translationTransform)
 
    pinch.scale = 1
}
```

这里有个比较坑的地方就是给`imageView`应用完ScaleTransform之后直接使用`public func translatedBy(x tx: CGFloat, y ty: CGFloat) -> CGAffineTransform`方法增加偏移的时候的结果与我们预期的不一样。

比如说我们的`imageView`初始状态的frame是`(x: 100, y: 100, width: 100, height: 100)`, 缩放的时候以左上角点b为基准，使用ScaleTransform放大一倍之后变为`(x: 0, y: 0, width: 200, height: 200)`，然后我们通过`translatedBy`方法偏移之后得到的frame为`(x: 200, y: 200, width: 200, height: 200)`, 这个结果与我们所期望的`(x: 100, y: 100, width: 200, height: 200)`有所不同。而使用`concatenating`方法就不会有问题，按照我的理解，这两个方法应该产生同样的结果才对。如果你知道其中的原因，欢迎在下面留言。

文章里提到的内容我已经写了一个Demo欢迎[点击此处](https://github.com/skx926/ScalableViewDemo)进行下载查看。