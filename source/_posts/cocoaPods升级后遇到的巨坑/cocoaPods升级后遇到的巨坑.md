---
title: cocoaPods升级后遇到的巨坑
date: 2017-07-06 16:03:08
tags:
  - cocoaPods
  - 技术
  - 终端
---
  之前我的CocoaPods一直是在0.0.39这个版本中，一直都处于相安无事的状态，后来看到同事将CocoaPods升级到1.1.1正式版本以后，我也跟着装逼，然后就一不小心装逼失败，把CocoaPods升级到最新的bata版本了，最后又搞了半天把版本降为了1.1.1版本。感觉自己棒棒哒，过了年来，完蛋了，出问题了。现在我就完全复现出错的整个流程以及解决办法，帮助有需要的人。
  <!-- more -->
  
#### 一：假设我们现在需要安装最新版本AFNetworking,第一步肯定是在终端导入地址
![image](http://upload-images.jianshu.io/upload_images/1863813-8a52de6c83d0169e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 二:在导入项目地址以后，我们开始查询AFNetworking有哪些版本，选中我们需要的版本
![image](http://upload-images.jianshu.io/upload_images/1863813-532fb3a37575205a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 三：选中我们需要的那个版本，并复制版本号
![image](http://upload-images.jianshu.io/upload_images/1863813-6a26413e86a6d89d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 四：配置Podfile文件
![image](http://upload-images.jianshu.io/upload_images/1863813-9ea3e68c4f0f0219.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 五：配置后是这样，然后保存Podfile文件
![image](http://upload-images.jianshu.io/upload_images/1863813-9f48dd88e367465a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 六:开始安装AFNetworking,这下你会发现报错了，完蛋了，好紧张，怎么办？
![image](http://upload-images.jianshu.io/upload_images/1863813-b839f74aad7d9c1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
最后，我们在通过查询资料以后发现。CocoaPods升级后，Podfile文件的内容格式要求发生了变化，必须指出指出所用第三方库的target。没有办法，只能指定对应的target,我查询了一下网上的办法是这样的。
![image](http://upload-images.jianshu.io/upload_images/1863813-30b1d2c6ad01dedf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
他这样也能实现，但是我觉得还是挺麻烦的，其实我们只需要在配置podfile文件之前创建一个podfile文件以后，他就会自动的生成指定了target的podfile文件。
![image](http://upload-images.jianshu.io/upload_images/1863813-620b1cb370b3151f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
先创建podfile文件以后，打开pod file文件，你看到的就是这个样子
![image](http://upload-images.jianshu.io/upload_images/1863813-8e1fe61984957ad7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这个就是需要指定的target的标准格式，系统自己就可以帮你生成，完全不用你去动手敲，最后一步，安装AFNetworking成功
![image](http://upload-images.jianshu.io/upload_images/1863813-8ca00166c9a13d8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
备注：虽然写的繁琐，但是重点是为了把问题讲清楚，希望能给被这个问题带来困惑的人给予帮助，大神勿喷，谢谢！