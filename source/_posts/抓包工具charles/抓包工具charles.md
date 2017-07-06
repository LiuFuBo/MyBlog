---
title: 抓包工具charles
date: 2017-07-06 15:12:03
tags:
   - 工具
---
前言:在任何时候，利用好工具总会给我们生活带来意想不到的惊喜，所谓磨刀不误砍柴工就是这个道理，平时开发的过程中，我们很多时候都是通过NSLog打印来获得结果费时又费力，最近刚好发现一款抓包软件还是挺好用的，只需要简单的设置以后就可以抓取所有的网络请求信息了。
<!-- more -->
#### 一:抓取电脑上的包
1.需要设置一下charles
![image](http://upload-images.jianshu.io/upload_images/1863813-273a0ddb9bee0f41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.进入设置页面勾选相应选项就可以使用了。
![image](http://upload-images.jianshu.io/upload_images/1863813-4d87faef41c7f2d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 二:抓取手机上的包
1.首相将charles的设置里面端口号设置为8888
![image](http://upload-images.jianshu.io/upload_images/1863813-15c79660913eba76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.在 iPhone 的 “ 设置 ”–>“ 无线局域网 ” 中，可以看到当前连接的 wifi 名，通过点击右边的详情键，可以看到当前连接上的 wifi 的详细信息，包括 IP 地址，子网掩码等信息。在其最底部有「HTTP 代理」一项，我们将其切换成手动，然后填上 Charles 运行所在的电脑的 IP，以及端口号 8888
![image](http://upload-images.jianshu.io/upload_images/1863813-d67a0891a19a1306.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3.设置好以后会弹出一个请求连接的弹出框，选择allow就可以愉快的抓取你想要的数据了。
