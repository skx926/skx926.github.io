---
title: Assign对象自动检测
date: 2020-03-29 14:27:34
tags: [assign, runtime]
categories: [iOS]
---

### 背景

在编码过程中，有时候会手误给对象类型的property设置assign修饰符，如果这个指针所指向的对象被系统释放，它不会被自动置nil而是变成野指针，产生crash的风险。

因此我们就需要一个工具能够在开发阶段将这些误用的地方检测出来及时进行修复。



### 原理

在App运行的时候我们可以通过rumtime来读取每个property的attribute（例如`T@"NSString",N,V_descriptionText`），通过attribute可以判断这个属性是否是对象、是否使用了assign修饰符。

对于这些attribute的含义，可以在Apple官方文档 [Declared Properties](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtPropertyIntrospection.html) 和 [Type Encodings](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1) 中查看具体的解释。



### 实现

1. 我们先获取到可执行文件中的所有类

```objc
#import <objc/runtime.h>
#import <dlfcn.h>
#import <mach-o/ldsyms.h>

@implementation NSBundle (JX)
  
+ (NSArray<Class> *)jx_bundleOwnClassesInfoWithExcludeClassNames:(NSArray<NSString *> *)exclueClassNames {
    
    NSMutableArray *resultArray = [NSMutableArray array];
    
    unsigned int classCount;
    const char **classes;
    Dl_info info;
    
    dladdr(&_mh_execute_header, &info);
    classes = objc_copyClassNamesForImage(info.dli_fname, &classCount);
    
    for (int index = 0; index < classCount; index++) {
        NSString *className = [NSString stringWithCString:classes[index] encoding:NSUTF8StringEncoding];
        if ([exclueClassNames containsObject:className]) {
            continue;
        }
        Class class = NSClassFromString(className);
        [resultArray addObject:class];
    }
    
    free(classes);
    
    return resultArray.copy;
}
```

2. 然后遍历每个类的每个属性

```objc
#import "JXAssignDetector.h"
#import "ASRuntimeUtil.h"
#import "NSBundle+JX.h"

@implementation JXAssignDetector

+ (NSArray<NSDictionary<NSString *, NSString *> *> *)detectAndPrint {
    NSArray *excludes = @[@"TXPicture"];
    NSArray *classes = [NSBundle jx_bundleOwnClassesInfoWithExcludeClassNames:excludes];
    NSMutableArray *infos = [NSMutableArray new];
    for (Class class in classes) {
        NSMutableArray *propertyNames = [NSMutableArray new];
        NSMutableArray *propertyAttributes = [NSMutableArray new];
        [ASRuntimeUtil getPropertyNames:propertyNames propertyAttributes:propertyAttributes class:class];
        
        for (int i = 0; i < propertyNames.count; i++) {
            NSString *name = propertyNames[i];
            NSString *attribute = propertyAttributes[i];
            
            // 由于assign是默认的标志，因此排除其他类型内存管理的标志W（weak）、&（strong或retain）、C（copy）、R（readonly）剩下的就是assign和unsafe_unretained
            if ([attribute hasPrefix:@"T@"]) {
                NSArray *components = [attribute componentsSeparatedByString:@","];
                if (![components containsObject:@"W"] &&
                    ![components containsObject:@"&"] &&
                    ![components containsObject:@"C"] &&
                    ![components containsObject:@"R"]) {
                    NSLog(@"%@ - %@ - %@", class, name, attribute);
                    [infos addObject:@{@"class": NSStringFromClass(class),
                                       @"property": name,
                                       @"attribute": attribute
                    }];
                }
            }
        }
    }
    return infos;
}
```



### 注意点

1. 由于`assign`和`unsafe_unretained`在attribute里面的表示相同（都是空），两者的意义也是一样的，区别在于`unsafe_unretained`是开发者显示指定为这种类型的，开发者会自己管理它的内存，检测结果中会同时包含这两种类型，在代码排查时注意区分。
2. 由于是检测整个App二进制文件中的类名，所以结果会包含第三方库中的内容，可以酌情忽略或者提供给第三方进行解决。