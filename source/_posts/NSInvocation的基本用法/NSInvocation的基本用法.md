---
title: NSInvocation的基本用法
date: 2017-08-18 19:22:11
tags:
   - NSInvocation
   - runtime
   - performaSelector
---


### 小知识:

在 iOS中可以直接调用某个对象的消息方式有两种：
一种是performSelector:withObject；
再一种就是NSInvocation。

为什么这么说呢，首先某个对象指的实例化后的对象，对象所对应的方法，则指的是实例方法。在一个类中调用另一个类的实例方法，只有上面两种方式。第一种方式比较简单，能完成简单的调用。但是对于>2个的参数或者有返回值的处理，那performSelector:withObject就显得有点有心无力了，那么在这种情况下，我们就可以使用NSInvocation来进行这些相对复杂的操作

NSInvocation的基本使用
<!-- more -->

一、直接调用当前类对象消息

方法签名类

    // 方法签名中保存了方法的名称/参数/返回值，协同  NSInvocation来进行消息的转发
    // 方法签名一般是用来设置参数和获取返回值的, 和方法的调用没有太大的关系
    //1、根据方法来初始化NSMethodSignature
    NSMethodSignature  *signature = [ViewController    instanceMethodSignatureForSelector:@selector(run:)];
    
根据方法签名来创建NSInvocation对象

    // NSInvocation中保存了方法所属的对象/方法名称/参数/返回值
    //其实NSInvocation就是将一个方法变成一个对象
    //2、创建NSInvocation对象
    NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:signature];
    //设置方法调用者
    invocation.target = self;
    //注意：这里的方法名一定要与方法签名类中的方法一致
    invocation.selector = @selector(run:);
    NSString *way = @"byCar";
    //这里的Index要从2开始，以为0跟1已经被占据了，分   别是   self（target）,selector(_cmd)
    [invocation setArgument:&way atIndex:2];
    //3、调用invoke方法
    [invocation invoke];
    //实现run:方法
    - (void)run:(NSString *)method{

    }

二、在当前VC调用其他类对象消息

这里咱们假设一种情况需要在VC控制器中调用继承自NSObject的ClassBVc类中的run方法,并且要向该方法传递参数。

ClassBVc.h文件实现

    #import <UIKit/UIKit.h>

    @interface ClassBVc : UIViewController



     - (void)run:(NSString *)parames;

    @end


ClassBVc.m文件实现


    #import "ClassBVc.h"

    @interface ClassBVc ()

    @end

    @implementation ClassBVc

    - (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    }

     - (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
    }

    - (void)run:(NSString *)parames{

     NSLog(@"你猜我猜不猜?%@",parames);
    }
    
    
VC.m文件实现

    #import "ViewController.h"
    #import "ClassBVc.h"


    @interface ViewController ()

    @end

    @implementation ViewController



    - (void)viewDidLoad {
    [super viewDidLoad];

    ClassBVc *B = [[ClassBVc alloc]init];
    // 方法签名中保存了方法的名称/参数/返回值，协同  NSInvocation来进行消息的转发
    // 方法签名一般是用来设置参数和获取返回值的, 和方法的调用没有太大的关系
    //1、根据方法来初始化NSMethodSignature
    NSMethodSignature  *signature = [ClassBVc    instanceMethodSignatureForSelector:@selector(run:)];

    // NSInvocation中保存了方法所属的对象/方法名称/参数/返回值
    //其实NSInvocation就是将一个方法变成一个对象
    //2、创建NSInvocation对象
    NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:signature];
    //设置方法调用者
    invocation.target = B;
    //注意：这里的方法名一定要与方法签名类中的方法一致
    invocation.selector = @selector(run:);
    NSString *way = @"byCar";
    //这里的Index要从2开始，以为0跟1已经被占据了，分   别是self（target）,selector(_cmd)
    [invocation setArgument:&way atIndex:2];
    //3、调用invoke方法
    [invocation invoke];
    }

最终运行结果:

![image](http://upload-images.jianshu.io/upload_images/1863813-7e1ac9958876573b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

NSInvocation在项目中最重要的使用场景在消息转发上面，如果在做消息转发时你没有实现resolveClassMethod:或者forwardingTargetForSelector:方法，那么系统就会来到最后拦截的第三道屏障，当你实现methodSignatureForSelector:方法以后，系统会触发forwardInvocation:方法；在forwardInvocation:方法中你可以修改目标的接受者等等。

下面咱们来复现该场景：

假设现在有一个VC控制器、两个继承自NSObject的Target类和boy类，现在咱们需要在VC里面调用Target的methodC方法，但是在Target类里面我只写了methodC方法的声明，并没有在.m中实现它，但是我在Boy.m类中实现了methodC方法，现在需要做到的就是在调用Target方法时，触发Boy类中methodC的实现。

VC.m文件中的实现:

    #import "ViewController.h"
    #import "Target.h"


    @interface ViewController ()

    @end

    @implementation ViewController



    - (void)viewDidLoad {
    [super viewDidLoad];
    Target *targets = [[Target alloc]init];
    [targets methodC];


    }
    

Target.h文件中中的实现

    #import <Foundation/Foundation.h>

    @interface Target : NSObject


    - (void)methodC;

    @end
    
Target.m文件中把消息转发给了Boy类来执行

    #import "Target.h"
    #import "Boy.h"
    #import <objc/runtime.h>


    @implementation Target


    //methodSignatureForSelector用来生成方法签名，这个签    名就是给forwardInvocation中的参数NSInvocation调用的。
    - (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector{

    NSMethodSignature *signature = [super methodSignatureForSelector:aSelector];
    if (!signature) {
    if ([Boy instancesRespondToSelector:aSelector]) {
        signature = [Boy instanceMethodSignatureForSelector:aSelector];
    }
    }
    return signature;
    

Boy.h文件中没有实现

    #import <Foundation/Foundation.h>

    @interface Boy : NSObject

    @end
    
    
Boy.m文件中实现了methodC方法

    #import "Boy.h"

    @implementation Boy

    - (void)methodC{

    NSLog(@"爱我你就抱抱我");
    }

    @end
    
    
从上面可以看到，当咱们去调用类中没有找到实现方法的时候，如果我们在消息转发的过程中重写了那些方法，就可以实现一些特定的需求。

具体的demo地址在我个人github上面，如果有需要的朋友欢迎下载[->传送门](https://github.com/LiuFuBo1991/Foward)