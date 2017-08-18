---
title: Objective-c方法调用流程
date: 2017-08-18 19:11:04
tags:

 - runtime
 - Invocation
 - isa
---

### 前言:

&nbsp;&nbsp;前段时间去面试，每个面试官都问到了OC中方法是如何调用的；感觉自己答到点子上了，但是回答的并不是很完善，所以我特地抽时间将这部分知识点重新看了一遍，算是最自己所学的一个总结吧！

Objective-c是一门动态语言，动态两个字主要就体现在我们调用方法的时候，运行时回动态的查找方法，然后调用相应的函数地址。运行时是整个Objective-c程序的基石，有了它我们的程序才能正常运行起来。

对于OC的方法调用，有两个重点需要掌握：
<!-- more -->
1.对于OC中一切的方法调用，最终都会转换为类似下面的C语言函数

    id objc_msgSend(id self, SEL op, . . .);
2.对象的方法调用传递过程，Objective-c中每个类对象最开始的位置都会有一个isa指针，该指针指向一个结点，其实就是一个链表，该链表主要包含两部分信息：

(1).指向父类的指针。<br>
(2).自身的函数分发表。

有了这两部分，Objective-c的方法的调用流程就可以跑起来了。当我们调用一个对象的某一个方法的时候，首先会在当前类的分发表中寻找该方法，如果找不到对应的方法，然后再去其父类中寻找该方法，依次类推直到找到对应的方法为止，流程图如下：
![image](http://upload-images.jianshu.io/upload_images/1863813-81e5fa1831aa8f71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当OC要调用方法时，会沿着这个链表一路查找，直到在函数分发表中找到要调用的函数为止。当然runtime会对这个过程进行优化，来缓存已经调用过得函数地址，不至于让我们每次都又沿着链表查找一遍,当程序运行一次以后再次调用就会更加高效。

通过学习官方文档[Objective-C Runtime Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtHowMessagingWorks.html#//apple_ref/doc/uid/TP40008048-CH104-SW1)可以发现所有的方法调用最后都转化为了C语言函数调用，例如：我们创建了一个ClassA类型对象person，然后调用try方法，[person try],在编译的时候，编译器就会将该方法转化为objc_msgSend(person,selector)的形式，runtime会调用try方法实现所对应的函数地址，该函数所对应的两个参数，其中第一个参数指向调用该方法的对象，第二个参数则代表要调用的方法。


上面说到objc_msgSend函数首先会根据消息接受者对象的isa指针找到它的真实类型，然后在该类的方法列表中查找是否有与当前选择器相对应的方法，如果没有就跳转到父类中去查找，当查找到元类NSObject还是没有找到的时候，就会执行消息转发流程，我们接下来就继续讨论运行时消息转发是怎么回事。


#### 消息转发

消息转发流程比较复杂，主要分为三个步骤，首先我们需要先来看一张消息转发的流程图。
![image](http://upload-images.jianshu.io/upload_images/1863813-aeed8ae17e94e6de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第一步

当消息派发流程最终在对象的元类中都没有找到对应的方法选择器时，就会开启消息转发流程，如果是实例方法的话，第一步会先调用消息接受者所在类的resolveInstanceMethod:方法，如果是类方法的话则会调用resolveClassMethod:，这两个方法的返回值都是BOOL值，表示是否动态添加一个方法来响应当前消息选择器。

    + (BOOL)resolveInstanceMethod:(SEL)sel;
    + (BOOL)resolveClassMethod:(SEL)sel;
    
以上两个方法均声明在NSObject类中，如果消息接收者所在类重写了这两个方法中的其中一个方法并返回YES，也就意味着想要动态添加一个方法来响应当前的消息选择器，可以在重写的方法内使用class_addMethod函数来为当前类添加方法。

    BOOL class_addMethod(Class cls, SEL name, IMP imp, const char *types)
该函数第一个参数代表为哪个类添加方法，第二个参数是方法所对应的选择器，第三个参数是C语言的函数指针，用来指向待添加的方法，最后一个参数表示待添加方法的类型编码（详情可查看苹果官方文档：[Objective-C Runtime Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1)）。

第二步

如果上一步过程中，并没有新方法能响应消息选择器，则会进入消息转发流程的第二步。在第二步中系统会调用当前消息接收者所在类的forwardingTargetForSelector:方法，用以询问能否将该条消息发送给其他接收者来处理，方法的返回值就代表这个新的接收者，如果不允许将消息转发给其他接收者则返回nil。

    - (id)forwardingTargetForSelector:(SEL)aSelector;
利用这个方法，我们再使用组合可以模拟多重继承，在其他类中重写forwardingTargetForSelector:将其他类所能响应的消息选择器的接收者设置成当前类，这样当前类就可以拥有其他类的方法或者属性。

第三步

如果forwardingTargetForSelector:方法的返回值为nil，那么消息转发机制还要继续进行最后一步。在这一步中，系统会将尚未处理的消息包装成一个NSInvocation对象，其内部包含与该消息相关的所有信息，比如消息的选择器、目标接收者、参数等。之后系统会调用消息接收者所在类的forwardInvocation:方法，并将生成的NSInvocation对象作为参数传入。

    - (void)forwardInvocation:(NSInvocation *)anInvocation;
forwardInvocation:方法同样声明在NSObject类中，我们可以重写该方法的实现。比如将NSInvocation对象的target属性设置为其他接收者，此操作可以实现与上一步操作同样的效果，但明显在效率上没有第二步的操作高，所以很少有人在这一步中仅仅只是改变消息的接收者。NSInvocation类中还提供了许多属性和方法用于修改其对应方法的信息，比如可以修改方法的参数和返回值，或者直接更改消息选择器转而调用其他方法。

最后如果消息接收者在这一步中仍然无法响应消息选择器，那么系统会自动调用doesNotRecognizeSelector:方法，该方法默认实现为抛出异常，也就是我们在开发中经常见到的unrecognized selector sent to instance。

    -[ViewController count]: unrecognized selector sent to instance