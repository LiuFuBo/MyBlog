---
title: 如果通过cocoaPods创建私有库
date: 2017-07-06 15:27:16
tags:
  - 私有库 
  - 技术
---

在写代码的时候，有时候封装了一个很好的工具类，系统在以后的项目中也能够被使用，但是每次使用的时候，都需要拷贝代码，感觉很麻烦，所以，我希望能够将这些代码通过cocoaPods来进行托管，这样只需要通过cocoaPods来导入就行了。
<!-- more -->
#### 一：咱们第一步需要在github上面创建一个仓库来存储代码
![image](http://upload-images.jianshu.io/upload_images/1863813-2b1bd2823c867886.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 二:点击创建项目以后，咱们需要将代码上传到仓库中，现在在本地创建一个用于装demo和工具类的文件夹
<img src = 'http://upload-images.jianshu.io/upload_images/1863813-3924f9c806565ed9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240' align = 'center' />

#### 三：创建文件夹以后，将文件夹拖入sourcetree并将其添加远程仓库
![image](http://upload-images.jianshu.io/upload_images/1863813-4e310d4a44fed494.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 四：然后在本地创建的文件夹中创建Demo以及工具类文件夹
![image](http://upload-images.jianshu.io/upload_images/1863813-07e8efb6cf3f0cda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 五：将本地代码上传到github上面，上传完以后是这个样子
![image](http://upload-images.jianshu.io/upload_images/1863813-b9f8d496d3890abd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 六:配置podspec文件,这个文件你可以通过命令创建，也可以通过拷贝过来再进行修改，但是请注意，不要用文本对该文件进行编辑，否则最后该文件将不能正常使用。这里我是用sublime Text 2这款软件打开的
![image](http://upload-images.jianshu.io/upload_images/1863813-3ce545e532c4f8c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 七：配置好以后，就把该配置文件上传到github上面，成功后多了一个podspec文件
![image](http://upload-images.jianshu.io/upload_images/1863813-15aeae84bb44547f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 八:新建一个项目，并将其通过cocoaPods导入项目
![image](http://upload-images.jianshu.io/upload_images/1863813-cf9f0af2bf0d7539.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 九：执行过后，结果就是这样,此时该工具类已经通过cocoaPods导入我们项目中了
![image](http://upload-images.jianshu.io/upload_images/1863813-3e1d402f4f50f140.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 十：打开Xcode就可以使用该工具类了
![image](http://upload-images.jianshu.io/upload_images/1863813-41154c6ca340934d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 续：如果你的工具类还关联其他第三方公开库的话，只需要将podspec改成如下形式
![image](http://upload-images.jianshu.io/upload_images/1863813-3e0379f2dfd52bb9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
