---
title: sourcetree的使用
date: 2017-12-12 12:26:13
tag:
- 分支
- 合并
- 冲突
---

### 前言：
&nbsp;&nbsp;在平时开发过程中，我们难免会跟sourcetree打交道，很多人都会用sourcetree来合并代码，所以掌握好sourcetree的使用，有时候显得格外重要，这里我列出平时会用到的一些操作，希望能对各位有所帮助。
<!-- more -->
#### 一:创建新的分支

方法一:</br>
1.通过点击分支右键选择新建分支
![](http://upload-images.jianshu.io/upload_images/1863813-f6d07bd47128d9e0.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
2.当弹出新建分支和删除分支的框来时，此时只需要输入新的分支名就可以创建好了
![](http://upload-images.jianshu.io/upload_images/1863813-e7bf97c2a5436fe7.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
方法二:</br>
1.直接点击最上面的分支图标，弹出新建和删除分支页面
![](http://upload-images.jianshu.io/upload_images/1863813-7ea1740ff33285f6.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
2.在弹出的新建分支框中输入新的分支名就可以创建好了
![](http://upload-images.jianshu.io/upload_images/1863813-c191bbecd2b2f649.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
3.创建好以后需要将新的分支和原来旧的分支上的代码进行合并，此时先选中当前新建分支，然后找到下面远端你需要合并的分支名右键点击它，拉取检出该分支代码到当前分支上
![](http://upload-images.jianshu.io/upload_images/1863813-655810aba39c3dcb.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
4.当前新分支跟旧的分支代码合并以后，此时新创建的分支还只存在于本地，我们此时需要将该分支推送到远程服务器存储起来
![](http://upload-images.jianshu.io/upload_images/1863813-4aa1248a9840e58c.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
此时咱们的新分支就在远程服务器上展示出来了
#### 二:拉取别人的分支
1.在平时的开发过程中，不一定新分支是自己创建的，所以在别人已经把新分支已经创建好并合并了你当前分支代码的情况下，你需要切换到跟对方同一个分支上去开发代码，我们只需要选择下面远端某一个别人创建的分支，右键点击检出
![](http://upload-images.jianshu.io/upload_images/1863813-da73c19e5bdd8ded.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
2.此时,你就可以看到当前分支上多了一个分支，并且该分支被选中为当前分支
![](http://upload-images.jianshu.io/upload_images/1863813-b1a7847bd06eb833.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
完成以后就可以轻松的跟别人在同一个分支上提交代码了。
#### 三:合并别人的分支
项目开发过程中，可能因为一些原因需要几个人在不同的分支上开发，最后再统一合并代码，这个时候就需要用到合并，此时假如我想把4.7.0的代码合并到4.8.1上面。

1.确保当前分支在4.8.1的情况下，单机选中4.7.1的分支，右键选中合并4.7.0至4.8.1
![](http://upload-images.jianshu.io/upload_images/1863813-4a39591759065f16.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
合并完以后，我们可以看到自己项目中OC代码多了4.7.0的代码，同理想要将4.8.1合并到其他分支上也是这样操作，当前最后不要忘记将最新合并分支的代码推送到远程服务器端

#### 四:解决合并冲突问题
咱们在小的分支上完成开发以后，需要将代码最终合并到master分支上，例如现在现在除了master还有两个分支A、B，当master跟A合并以后没有出现问题，但是在跟B合并以后，出现了冲突，这是因为A和B同时对同一块代码进行了修改。有冲突就要解决，右键单击冲突文件，选择解决冲突，这里有两个选项：</br>
1、使用 我的版本 解决冲突</br>
2、使用 他人版本 解决冲突</br>
这里首先是将A的分支合并到主分支master，那么“我的版本”就是对应的A的，“他人版本”对应的就是B的。如果首先合并B的分支，那么对应关系就要对调一下。总的来说，“我的版本”对应的是首先合并到主分支master的。</br>
采用一个人的版本，那么在冲突文件中就只会保留该人修改的代码，例如我这里就选择”使用 我的版本 解决冲突“，那么在master中就只会保留A分支添加的代码。（针对冲突部分）</br>

这里采用网络图片做一个示意图:
![](http://upload-images.jianshu.io/upload_images/1863813-75ba924bc2f0600b.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
到此为止，分支问题基本搞定,当然，如果你在运用过程中遇到任何问题，也欢迎你前来issue 我,我会为你做更详细的解答。


