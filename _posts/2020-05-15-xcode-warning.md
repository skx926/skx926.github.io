---
title: Xcode常见Warning解决
date: 2020-05-15 14:27:34
tags: [Xcode, Warning]
categories: [iOS]
---

### Analysis Warning（依赖分析⚠️）

**例**：Warning: The Copy Bundle Resources build phase contains this target's Info.plist file `/Users/kylexsun/APP_NAME/Resources/project/Notification_Info.plist`.

Info.plist文件已经在Build Settings中的Info.plist File指定了路径，无需再添加到Build Phases中的Copy Bundle Resources中去

 

### Documentation Issue（文档问题⚠️）

**例**：Parameter `bytes` not found in the function declaration

修改了方法名，但是没有修改文档

 

### Semantic Issue（语义问题⚠️）

**例1**：Incompatible pointer types assigning to `XXView ` from `UIView `

由于将父类实例赋值给子类实例，很容易产生由于找不到子类方法产生崩溃

 

**例2**：Declaration shadows a local variable

```objc
[[XXPermissionsManager shareInstance] aimedToUsePermission:XXPermissionTypeCamera completionHandler:^(BOOL success, NSString *errorMsg) {
    [[XXPermissionsManager shareInstance] aimedToUsePermission:XXPermissionTypeMicroPhone completionHandler:^(BOOL success, NSString *errorMsg) {
    }];
}];
```

给同名的变量取一个更详细的变量名，比如外部的success变量可以改成cameraSuccess

 

**例3**：Assigning to `id<CAAnimationDelegate> _Nullable` from incompatible type `UILabel *__strong`

指定为delegate的变量没有声名代理协议

 

**例4**：Class `XXViewController` does not conform to protocol `XXDelegate`

声名了代理协议，但是没有实现所有方法。设计协议的人需要注意表名哪些方法是可选的

 

**例5**：Property attribute in class extension does not match the primary class

```objc
// .h文件
@interface AudioPlayManager : NSObject
@property (atomic, retain, readonly) UIImage *currentCoverImage;
@end
  
// .m文件
@interface AudioPlayManager()
@property (atomic, strong, readwrite) UIImage *currentCoverImage;
@end
```

两边声名的同一个property的内存管理属性不同

 

**例6**：Control may reach end of non-void function

```objc
+ (NSString *)stringWithoutDotThousandNumFomatter:(NSNumber *)num
{
    if (num.integerValue <= 999) {
        return [NSString stringWithFormat:@"%ld", num.integerValue];
    } else if (num.integerValue <= 999999)
    {
        float result = num.integerValue/1000.0;
        return [NSString stringWithFormat:@"%dK", (int)result];
    } else if (num.integerValue > 999999)
    {
        float result = num.integerValue/1000000.0;
        return [NSString stringWithFormat:@"%dM", (int)result];
    }
}
```

有返回值的方法没有兜底的返回值

 

**例7**：Method definition for `resetRequestState` not found

只有方法声明，没有实现

 

**例8**：Auto property synthesis will not synthesize property `viewID`; it will be implemented by its superclass, use @dynamic to acknowledge intention

父类已经声明过的属性，子类就不需要再声明一遍了

 

**例9**：Undeclared selector `xxAction`

对于没有声明的selector警告，如果只是忽略一个代码块就可以用下面的方式：

```objc
# pragma clang diagnostic push
# pragma clang diagnostic ignored "-Wundeclared-selector"    
[xxBtn addTarget:self action:@selector(xxAction) forControlEvents:UIControlEventTouchUpInside];
# pragma clang diagnostic pop
```

 

如果是要忽略整个文件这种类型的警告可以在Build Phases中的Compile Sources里面找到要忽略的文件，然后在文件名后面的Compliler Flags中添加`-Wundeclared-selector`就可以了。

 

如果要忽略整个项目中这种类型的警告可以在Build Settings中的Other Warning Flags中添加`-Wundeclared-selector`就可以了。

 

**例10**：PerformSelector may cause a leak because its selector is unknown

```objc
# pragma clang diagnostic push
# pragma clang diagnostic ignored "-Warc-performSelector-leaks"
    XXUpdaterTarget* t = self.objPoolDict[key];
    if (t.target != nil)
    {
        if (t.needUpdate && t.sel != NULL)
        {
            [t.target performSelector:t.sel withObject:key];
            t.needUpdate = NO;
        }
    }
# pragma clang diagnostic pop
```

同例9的解决方式，只是换个flag。

 

对于clang的所有warning的flag可以参考这里：[fucking-clang-warnings](https://github.com/JasonWorking/Articles/blob/master/Articles/fucking-clang-warnings.md)

 

**例11**：Variable `imagePartRef` is used uninitialized whenever `if` condition is false

```objc
CGImageRef imagePartRef;
if (imagePart == XXImagePartAll)
{
    imagePartRef = CGImageCreateWithImageInRect([image CGImage], CGRectMake(0, 0, image.size.width, image.size.height));
}
else if(imagePart == XXImagePartTop)
{
    imagePartRef = CGImageCreateWithImageInRect([image CGImage], CGRectMake(0, 0, image.size.width, image.size.height * partPercent));
}
else if(imagePart == XXImagePartLeft)
{
    imagePartRef = CGImageCreateWithImageInRect([image CGImage], CGRectMake(0, 0, image.size.width * partPercent, image.size.height));
}
else if(imagePart == XXImagePartBottom)
{
    imagePartRef = CGImageCreateWithImageInRect([image CGImage], CGRectMake(0, image.size.height * (1 - partPercent), image.size.width, image.size.height * partPercent));
}
else if(imagePart == XXImagePartRight)
{
    imagePartRef = CGImageCreateWithImageInRect([image CGImage], CGRectMake(image.size.width * (1 - partPercent), 0, image.size.width * partPercent, image.size.height));
}
```

没有给imagePartRef赋初值，条件控制没有包含所有情况

 

**例12**：Multiple methods named `length` found

```objc
NSArray *authenIcons = @[@"a", @"b"];
for (int i = 0; i < autheIcons.count; i++) {
    if (authenIcons[i].length > 0) {
        // do something...
    }
}
```

在遍历数组中的元素时不知道元素的实际类型，因此不确定会掉哪个类的length方法。建议声名集合类型时添加泛型：

```objc
NSArray<NSString *> *authenIcons = @[@"a", @"b"];
```

或者在遍历是指明元素的类型:

```objc
NSArray *authenIcons = @[@"a", @"b"];
for (int i = 0; i < autheIcons.count; i++) {
    NSString *authenIcon = authenIcons[i];
    if (authenIcon.length > 0) {
        // do something...
    }
}
```

 

**例13**：Autosynthesized property `key` will use synthesized instance variable `_key`, not existing instance variable `key`

```objc
@interface XXView : UIView {
    NSString* key;
}
@property(nonatomic,strong) NSString *key;
@end
```

两个变量的名称太相似，导致编译器以为可能是同一个变量，给出了提醒。

 

### Format String Issue（格式化字符串问题⚠️）

**例1**：Format string is empty

```objc
NSLog(@"");
```

**例2**：Values of type `NSUInteger` should not be used as format arguments; add an explicit cast to `unsigned long` instead

```objc
NSLog(@"add %d songs to first chache manager.", [songs count]);
```

NSUInteger和NSInteger在32位和64位机器上的长度不同：

```objc
# if __LP64__ || 0 || NS_BUILD_32_LIKE_64
typedef long NSInteger;
typedef unsigned long NSUInteger;
# else
typedef int NSInteger;
typedef unsigned int NSUInteger;
# endif
// You usually want to use NSInteger when you don`t know what kind of processor architecture your code might run on, so you may for some reason want the largest possible int type, which on 32 bit systems is just an int, while on a 64-bit system it`s a long.
/*
  在32位系统中：
  int 占4个字节
  long 占4个字节
  NSInteger 是int的别名，占4个字节
  long long 占8个字节
  int32_t 是int的别名，占4个字节
  int64_t 是long long的别名，占8个字节
  在64位系统中：
  int 占4个字节
  long 占8个字节
  NSInteger 是long的别名，占8个字节
  long long 占8个字节
  int32_t 是int的别名，占4个字节
  int64_t 是long long的别名，占8个字节
*/
```

为了不丢失精度，在格式化的时候需要显式的将他们转换成unsigned long 和 long。

```objc
NSLog(@"add %lu songs to first chache manager.", (unsigned long)[songs count]);
```

 

**例3**：Format specifies type `double` but the argument has type `CGSize` (aka `struct CGSize`)

```objc
NSLog(@"ContentSize:%f", scrollView.contentSize);
```

对于CG前缀的类型行比如CGPoint、CGSize可以转换成NSValue再进行打印：

```objc
NSLog(@"ContentSize:%@", [NSValue valueWithCGSize:scrollView.contentSize]);
```

 

**例4**：Null passed to a callee that requires a non-null argument

```objc
NS_ASSUME_NONNULL_BEGIN
@interface XXAudioPreviewMgr : NSObject
@property (nonatomic, strong) XXAudioEditer *audioEditer;
@end
NS_ASSUME_NONNULL_END
  
audioPreviewMgr.audioEditer = nil;
```

被`NS_ASSUME_NONNULL_BEGIN`和`NS_ASSUME_NONNULL_END`包裹的区域是默认nonnull的，如果要置空就需要添加nullable修饰符

```objc
@property (nonatomic, strong, nullable) XXAudioEditer *audioEditer;
```

 

#### Value Conversion Issue（值转换问题⚠️）

**例**：Implicit conversion loses integer precision: `XXViewFromType` (aka `enum XXViewFromType`) to `unsigned int`

```objc
[[XXManager sharedManager] addDataWithBuilder:[[XXBuilder alloc] initWithactionType:fromType singerId:singerId categoryId: (unsigned int)categoryId]];
```

由于XXBuilder里面的数字类型都是unsigned int，需要进行强转，如果是从更长的数据类型（如NSUInteger）转换过来需要注意精度损失会不会数据有影响

 

### Lexical or Preprocessor Issue（词汇或预处理问题⚠️）

**例1**： `NSLog` macro redefined

重复定义了NSLog这个宏

 

**例2**：Non-portable path to file `"XxBuilder.h"`; specified path differs in case from file name on disk

```objc
#import "XXBuilder.h"
```

引入的文件名和实际的文件名大小写不同，实际的文件名为`XxBuilder.h`，g是小写

 

### Apple Mach-O Linker (ld) Warning（链接警告⚠️）

**例**：Directory not found for option `-F/Users/kylexsun/APP_NAME/Library/AFNetworking`

再将AFNetworking从Framework引入改为pod引入之后没有在Build Settings中删掉相应的Framework Search Paths