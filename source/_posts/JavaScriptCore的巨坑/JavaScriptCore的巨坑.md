---
title: JavaScriptCore的巨坑（JSExportAs方式绑定的本地通信）
date: 2017-07-06 16:42:37
tags:
  - javascript 
---

# 前言

本篇分享的类型不是学习教程，并且要有一点JavaScriptCore基础，
         
         毕竟这一块网上一大堆的学习教程，博主就没必要班门弄斧了。

本篇的目的是分享JavaScriptCore中用JSExport协议和JSExportAs宏来进行js和oc通信的两个大坑。
<!-- more -->

1.内存泄露

2.调用-[JSValue callWithArguments]野指针问题

     注: 用block方式来进行js和oc的通信没这两个大坑。

# 第一个坑：内存泄露

一般绑定JSContext里的native的写法都是self.context[@"native"] = self。但是这样写会产生内存泄露（泄露原理就是互相持有了），这个坑随便百度Google一下也能找到很多解决方案。目前博主的解决方案是native指定一个新的对象，然后在指定对象里实现JSExport协议。
贴上博主在项目里用到的核心代码 :

### 和js通信的控制器页面核心代码
	{// 以 JSExport 协议关联 native 的方法
	self.context[@"native"] = [[NMFormFlowWapNativeManager alloc] initWithDelegate:self];}
	
	
### NMFormFlowWapNativeManager.h
	{@interface NMFormFlowWapNativeManager : NSObject

	- (instancetype)initWithDelegate:(id<NMFormFlowWapNativeManagerDelegate>)delegate;

	@property (nonatomic,weak) id<NMFormFlowWapNativeManagerDelegate> delegate;

	@end}
	
	
### NMFormFlowWapNativeManager.m

	{@import JavaScriptCore;

	@protocol TestJSExport <JSExport>

	JSExportAs(nativeCall, - (void)nativeCallHandleWithType:(NSString *)nativeType parameter:(NSString *)parameter jsType:(NSString *)jstype);

	@end

	@interface NMFormFlowWapNativeManager () <TestJSExport>

	@end

	@implementation NMFormFlowWapNativeManager

	- (instancetype)initWithDelegate:(id<NMFormFlowWapNativeManagerDelegate>)delegate {
    if (self = [super init]) {
        self.delegate = delegate;
    }
    return self;
	}

	- (void)nativeCallHandleWithType:(NSString *)nativeType parameter:(NSString *)parameter jsType:(NSString *)jsType {
    NSDictionary *dicParams = [NSJSONSerialization JSONObjectWithData:[parameter dataUsingEncoding:NSUTF8StringEncoding] options:NSJSONReadingMutableLeaves error:nil];
    [self.delegate nativeCallHandleWithThread:webThread type:nativeType parameter:dicParams jsType:jsType];
	}}
	
	
 PS:代码并不是完整的，但最核心的关键已经贴上来了。顺便简单解释一下。由于native管理的对象交给了另一个，所以在管理者对象里新开了一个代理回调。方便在控制器那边接收得到JS的事件。只要有点基础的，一看就懂了。毕竟本篇不是学习教程，而是分享坑的。
 
# 第二个坑：-[JSValue callWithArguments]野指针问题

这个问题有点奇葩，JSValue的callWithArguments就是oc调用js函数所执行的方法。那这简单的函数怎么发生野指针问题尼。
那就是oc进行网络请求，请求完回调的时候调用JSValue的callWithArguments的方法就是产生野指针，而且是间接性的，有时候会有时候不会。一旦崩溃基本都直接飞去main函数了。。。。

 这个问题百度Google都找了许久也没找到类似的问题和解决方案。只是崩溃的时候，左边的堆栈提示webThread（当时猜测可能是线程间通信影响的此问题），然后我蒙一下切换到webView的线程里去调用callWithArguments函数试试，结果就从未发生过崩溃了。
 
# 例子：
假设，h5上有一个图片显示和一个button，点击button的时候，调用本地摄像头并且上传图片到服务器，上传完之后在调用js一个函数，告诉js图片上传成功，让js去做对应的逻辑。这个时候网络请求完回调里的线程是主线程，调用callWithArguments的时候，就会间接性的崩溃。

# 解决方案

解决办法就是回到webView的线程去调用callWithArguments就不会崩溃（因为js和oc绑定的函数，在函数里执行的代码不是在主线程里执行的）。
模拟代码：

	{///假设这个函数是和js的test函数绑定的。如果监听到这个函数就进行网络请求或者上传图片等操作。
	- (void)test {
 	 //获取webView线程，因为js和oc绑定的函数里执行的代码不是在主线程里。
	NSThread *webThread = [NSThread currentThread];
	//网络请求
	@weakify(self);
	[HTTPRequest requestGetTokenWithFinished:^(void){
 	 @strongify(self);

  	//通知js请求完了。

 	//正常情况下是直接在这里调用，但是会间接性发生野指针问题，差不多每隔四五次发生一次野指针。
 	 //JSValue *jsCall = self.context[@"jsCall"];
 	 //[jsCall callWithArguments:nil];

 	 //线程安全的，用此方式，笔者再也没发生过野指针问题。
  	[self performSelector:@selector(jsCall) onThread:webThread withObject:nil waitUntilDone:NO];
	}];
	}

	- (void)jsCall {
 	 JSValue *jsCall = self.context[@"jsCall"];
 	 [jsCall callWithArguments:nil];
	}}
	
#	结语

总之笔者分享此文章的主要目的是第二个野指针问题，因为笔者在Google和stackoverflow里也找了很久也找不到问题原因，然后都是蒙对的，所以才来进行分享。可能对于不懂JavaScriptCore看起来有点困难，总之可以先了解一下。而对于js和oc的通信的业务不复杂的或者使用block进行通信的，应该很难遇到此问题。再者，网上很多学习教程基本都是推荐callWithArguments在主线程里调用，但目前笔者认为应该还是让它在webView的线程里去执行（那个野指针问题就是在主线程里执行所发生的）。

而callWithArguments野指针问题的底层实际发生原理也并不是很清楚。所以目前只能说博主是怎么解决的，但是为什么.....博主也不知其然了。有知道的方便的话也可告知一下。

而对于demo.....笔者也想写，但对于html、js并不是很熟悉（顶多看得懂几个标签）。所以....无能为力奉上demo了。