---
title: GCD中判断线程内容执行完毕
date: 2017-08-18 19:34:22
tags:
   - GCD
   - 信号量
   - 队列
   - Group
---

### 前言:

&nbsp;&nbsp;在使用GCD多线程做操作时，有些时候需要咱们多线程里面的内容全部完成以后，再去刷新页面内容，这种情况下就需要咱们知道什么时候多线程队列里面的内容已经执行完毕，这里列举三种方法，希望能对有需要的小伙伴有启发。

#### 一：信号量

信号量是一个整形值并且具有一个初始计数值，并且支持两个操作：信号通知和等待。当一个信号量被信号通知，其计数会被增加。当一个线程在一个信号量上等待时，线程会被阻塞（如果有必要的话），直至计数器大于零，然后线程会减少这个计数。

<!-- more -->

在GCD中有三个函数是semaphore的操作，分别是：
dispatch_semaphore_create　　　创建一个semaphore
dispatch_semaphore_signal　　　发送一个信号
dispatch_semaphore_wait　　　　等待信号

第一个函数是创建一个信号量，我们会给它一个初始值，第二个函数是发送信号量，当调用该函数的时候，信号量的值会加一，第三方函数是等待信号量，调用等待信号量以后信号量的值会减一，并且当信号量的值小于0时，线程就会一直等待，直到信号量的值大于等于0时才能继续执行。

咱们下面逐一介绍这三个函数的调用以及参数意义

1.dispatch_semaphore_create

    dispatch_samaphore_t dispatch_semaphore_create(long value);
     传入的参数为long，输出一个dispatch_semaphore_t类型且值为value的信号量。
     值得注意的是，这里的传入的参数value必须大于或等于0，否则dispatch_semaphore_create会返回NULL。
 
 2.dispatch_semaphore_signal
 
    long dispatch_semaphore_signal(dispatch_semaphore_tdsema)
    这里的参数是第一个函数创建的信号
    
3.dispatch_semaphore_wait

    long dispatch_semaphore_wait(dispatch_semaphore_t dsema,  dispatch_time_t timeout)；
    第一参数的是信号，第二个参数是超时时间的设定，系统给我们了两         种超时时间设定，一种是　DISPATCH_TIME_NOW　　表示当前；　DISPATCH_TIME_FOREVER 表示遥远的未来；你可以选择其中一种也可以自己创建一个超时时间函数
    
创建超时timeout有两种方法

第一种方法：

    dispatch_time_t dispatch_time(dispatch_time_t when, int64_t delta)；
    其参数when需传入一个dispatch_time_t类型的变量，和一个delta值。表示when加delta时间就是timeout的时间。
    
第二种方式:

    dispatch_time_t dispatch_walltime(<#const struct timespec * _Nullable when, int64_t delta);
     其参数when需传入一个dispatch_time_t类型的变量，和一个delta值。表示when加delta时间就是timeout的时间。
     
6.关于信号量，一般可以用停车来比喻。

网吧剩余6个座位，那么即使同时来了6个人也能同时接待。如果此时来了7个人，那么就有一个人需要等待。

信号量的值就相当于剩余座位的数目，dispatch_semaphore_wait函数就相当于来了一个人，dispatch_semaphore_signal

就相当于走了一个人。网络座位的剩余数目在初始化的时候就已经指明了（dispatch_semaphore_create（long value）），

调用一次dispatch_semaphore_signal，剩余的座位就增加一个；调用一次dispatch_semaphore_wait剩余座位就减少一个；

当剩余座位为0时，再来人（即调用dispatch_semaphore_wait）就只能等待。有可能同时有几个人等待一个座位。有些人

没有耐心，给自己设定了一段等待时间，这段时间内等不到座位就走了，如果等到了就去上网。而有些人就想站在这里，所以就一直等下去。

具体到代码中
假设咱们现在需要做完A、B、事件以后才能去做C事件那么咱们就可以仿照下面的做法

    #import "ViewController.h"
    #import <objc/runtime.h>



    @interface ViewController ()

    @end

    @implementation ViewController

    - (void)viewDidLoad {
    [super viewDidLoad];
    [self initlizeAppeareces];
    }


     - (void)initlizeAppeareces{
    //crate的value表示，最多几个资源可访问
    dispatch_semaphore_t semaphore =  dispatch_semaphore_create(2);
    dispatch_queue_t quene = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

    //任务1
    dispatch_async(quene, ^{
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    NSLog(@"办理A事件");
    dispatch_semaphore_signal(semaphore);
    });
    //任务2
    dispatch_async(quene, ^{
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    NSLog(@"办理B事件");
    dispatch_semaphore_signal(semaphore);
    });

    }
    
当咱们信号量为2时，证明之前的事件都办理完了，咱们就可以做其他的事情了。



#### 二：队列加group分组

    dispatch_group_t group =  dispatch_group_create();

    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

    // 执行1个耗时的异步操作

    });

     dispatch_group_async(group,     dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

    // 执行1个耗时的异步操作

    });

     dispatch_group_notify(group, dispatch_get_main_queue(), ^{

    // 等前面的异步操作都执行完毕后，回到主线程...

    });
    
    
当程序执行dispatch_group_notify()函数时，证明队列中的任务已经执行完了。

#### 第三种方式dispatch_barrier_sync和dispatch_barrier_async

共同点：

1、等待在它前面插入队列的任务先执行完

2、等待他们自己的任务执行完再执行后面的任务

不同点：

1、dispatch_barrier_sync将自己的任务插入到队列的时候，需要等待自己的任务结束之后才会继续插入被写在它后面的任务，然后执行它们

2、dispatch_barrier_async将自己的任务插入到队列之后，不会等待自己的任务结束，它会继续把后面的任务插入到队列，然后等待自己的任务结束后才执行后面任务。

所以调用这两种方法中的其中一个都会等前面的方法执行完以后再执行之后的方法，就可以确定队列中任务是否完成

注： 在信号量的demo编写过程中我发现当信号量等于0的时候都崩溃了,如果有知道原因的老铁欢迎前来分享。

