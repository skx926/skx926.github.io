---
title: 对extern的理解
date: 2014-12-04 01:46:00           
tags: [Extern]
categories: [iOS]
---

很少用到extern这个关键字，对它的了解也只停留在大学课堂那一点印象：外部变量，与局部变量相对。
最近项目中需要用到全局变量，使用全局变量大体上分为3种方式：

- 给AppDelegate增加property
- 使用单例模式，其实这个方法跟上一个方法比较类似，因为UIApplication的delegate也同样是一个单例，所以它的属性可以全局去访问
- 使用外部变量extern关键字

重点说说第三种方法，iOS里面使用extern关键字一般是用来声明全局的变量，在.h文件中声明：

```objc
extern NSString * const kLoginSuccessNotification;
```

在.m文件中对变量进行赋值：

```objc
NSString * const kLoginSuccessNotification = @"kLoginSuccessNotification";
```
使用的时候只需要包含.h文件就可以通过变量名进行调用了。我开始的时候很奇怪这种写法的意义何在，相比宏定义来说要麻烦许多，工作量是宏定义的两倍。声明和实现分开写虽然增加了工作量，但是可以很好的隐藏常量的实际值，不让其他人访问。而且这样常量的类型是明确的，相比宏定义简单的字符串替换，编译器可以在编译阶段就检查出代码中的错误，使得我们的代码更加安全。

但是如果我们用相同的方法声明一个全局变量：

```objc
extern NSString * IMAPIRequestURL;
```

因为我们不知道真实的值，所以不对其进行赋值，如果通过以下代码使用这个变量

```objc
extern NSString * IMAPIRequestURL;
IMAPIRequestURL = @“http://www.google.com”;
```

编译器就会报 ld: symbol(s) not found for architecture i386 错误，网上找到了半天在stackoverflow上找到了答案，原来是我对extern的理解还不够，回答着原文是这样的

```objc
extern means: "this thing is defined somewhere else"
```

看到这个答案我恍然大悟，extern关键字告诉编译器我要使用的这个变量已经在其他地方定义了，但是声明和使用的时候都是用了extern关键字，编译器都是去别的文件找，找不到就自然报错了。所以解决办法就是在.m文件中对其进行实现，实现的时候就不能加extern关键字了

```objc
NSString * IMRequestURL = @"http://www.google.com";
```
