---
title: 'NSClassFromString,NSSelectorFromString...'
date: 2017-07-06 16:58:02
tags:
   - 技术
---

###  NSClassFromString

这个方法判断类是否存在，如果存在就动态加载的，不存为就返回一个空对象;

id myObj = [[NSClassFromString(@"MyClass") alloc] init];

正常情况下等价于：id myObj = [[MyClass alloc] init];

<!-- more -->
优点：

1， 弱化连接，因此并不会把没有的Framework也link到程序中。

2，不需要使用import，因为类是动态加载的，只要存在就可以加载。因此如果你的toolchain中没有某个类的头文件定义，而你确信这个类是可以用的，那么也可以用这种方法。

### NSSelectorFromString

这个方法是上个方法的补充，也是动态加载实例方法。

SEL sel　=　NSSelectorFromString(@"doSomethingMethod:")//注意这个冒号,说明方法带有参数

if([object　respondsToSelector:sel]) {

[object　performSelector:sel　withObject:color]; //注意如果有两个参数,使用两个withObject:参数;

}

### isKindOfClass

isKindOfClass 我们也可以使用isKindOfClass来检查一个对象是否是一个类的成员

### isMemberOfClass

isMemberOfClass方法是来确定对象是否是某一个类的成员

### initWithCoder和initWithFrame的区别　

initWithCoder是一个类在IB中创建但在xocdde中被实例化时被调用的.比如,通过IB创建一个controller的nib文件,然后在xocde中通过initWithNibName来实例化这个controller,那么这个controller的initWithCoder会被调用.

initWithFrame是由用户创建的UIView子类，实例时被调用

### UIView autoresizingMask

如果视图的autoresizesSubviews属性声明被设置为YES，则其子视图会根据autoresizingMask属性的值自动进行尺寸调整。简单配置一下视图的自动尺寸调整掩码常常就能使应用程序得到合适的行为；否则，应用程序就必须通过重载layoutSubviews方法来提供自己的实现。

self.autoresizingMask = UIViewAutoresizingFlexibleWidth;//这个常量如果被设置，视图的宽度将和父视图的宽度一起成比例变化。否则，视图的宽度将保持不变。

UIViewAutoresizingNone

这个常量如果被设置，视图将不进行自动尺寸调整。

UIViewAutoresizingFlexibleHeight

这个常量如果被设置，视图的高度将和父视图的高度一起成比例变化。否则，视图的高度将保持不变。

UIViewAutoresizingFlexibleWidth

这个常量如果被设置，视图的宽度将和父视图的宽度一起成比例变化。否则，视图的宽度将保持不变。

UIViewAutoresizingFlexibleLeftMargin

这个常量如果被设置，视图的左边界将随着父视图宽度的变化而按比例进行调整。否则，视图和其父视图的左边界的相对位置将保持不变。

UIViewAutoresizingFlexibleRightMargin

这个常量如果被设置，视图的右边界将随着父视图宽度的变化而按比例进行调整。否则，视图和其父视图的右边界的相对位置将保持不变。

UIViewAutoresizingFlexibleBottomMargin

这个常量如果被设置，视图的底边界将随着父视图高度的变化而按比例进行调整。否则，视图和其父视图的底边界的相对位置将保持不变。

UIViewAutoresizingFlexibleTopMargin

这个常量如果被设置，视图的上边界将随着父视图高度的变化而按比例进行调整。否则，视图和其父视图的上边界的相对位置将保持不变。

