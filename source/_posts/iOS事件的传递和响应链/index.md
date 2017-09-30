---
title: iOS事件的传递和响应链
date: 2017-09-30 16:50:00
tag:
  - 事件
  - 传递
  - 响应者链
  - UIResponder
---

### 前言:
&nbsp;&nbsp;在咱们平时的开发过程中，每天都在敲着带有各种响应事件的代码，那咱们iOS的响应事件是怎样传递的呢？系统是怎样找到发出该事件的控件并执行响应方法的？为此，我写下这篇文章算是自己对事件响应链的学习的理解和总结吧！希望对在这方面有疑惑的你有帮助。

在iOS中有好几类，咱们今天只分析一下平时用的最多的触摸事件，学习触摸事件首先要了解一个比较重要的概念，响应者对象(**UIResponder**)

概念:在iOS中并不是任何对象都能处理事件，只有继承自**UIResponder**的对象才能接受并处理事件，我们称之为"响应者对象"。**UIApplication**、**UIViewController**、**UIView**这些都是继承自UIResponder的，所以他们都能接受并处理事件。

<!-- more -->

### 一:为什么继承自UIResponder的类能接收和处理事件呢？

那是因为**UIResponder**提供了以下方法来处理事件：

    - (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;
    - (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;
    - (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event;
    - (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event;
     加速计事件
    - (void)motionBegan:(UIEventSubtype)motion withEvent:(UIEvent *)event;
    - (void)motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event;
    - (void)motionCancelled:(UIEventSubtype)motion withEvent:(UIEvent *)event;
    远程控制事件
     - (void)remoteControlReceivedWithEvent:(UIEvent *)event;
通过上面的阐述在了解哪些控件能接收和处理事件以及它们为什么能处理事件以后，咱们需要了解到事件的产生和传递的过程是怎样的

### 二:事件的产生和传递

2.1 **事件的产生**

* 发生触摸事件后，系统会将该事件加入到一个由**UIApplication**管理的事件队列中,为什么是队列而不是栈？因为队列的特点是先进先出，先产生的事件先处理才符合常理，所以把事件添加到队列。<br>
* **UIApplication**会从事件队列中取出最前面的事件，并将事件分发下去以便处理，通常，先发送事件给应用程序的主窗口（**keyWindow**）。<br>
* 主窗口会在视图层次结构中找到一个最合适的视图来处理触摸事件，这也是整个事件处理过程的第一步。
找到合适的视图控件后，就会调用视图控件的touches方法来作具体的事件处理。

2.2**事件的传递**

* 触摸事件的传递是从父控件传递到子控件<br>
* 也就是UIApplication->window->寻找处理事件最合适的view
注 意: 如果父控件不能接受触摸事件，那么子控件就不可能接收到触摸事件

### 三:App是如何找到最合适的控件来处理事件呢？

1.首先咱们的App在刚进入应用的时候会首先创建**UIApplication**对象，它是一个单例，在任何页面都能拿到它，当咱们点击一个视图UIView的时候，产生一个触摸事件A，这个触摸事件A会被添加到**UIApplication**的队列中，然后**UIApplication**会把事件A传递给应用程序的主窗口(**keyWindow**)<br>
2.判断触摸点是否在自己身上<br>
3.如果触摸点在自己身上，那就遍历自己的子控件数组，记得是从后往前遍历，因为最后加载上去的视图放在最上面，从后往前遍历会使遍历次数更少，重复前面的两个步骤（所谓从后往前遍历子控件，就是首先查找子控件数组中最后一个元素，然后执行1、2步骤）<br>
4。如果子控件没有任何一个符合条件的，那么系统就会认为它自己是最合适处理这个事件的控件，这是最适合的View。

为了演示具体的传递过程是怎么样的，这里盗用网上的一张示意图来展示:
![image](http://upload-images.jianshu.io/upload_images/1863813-8b3b4e766768692a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面演示的意思就是假如现在有一个**fitView**，点击**fitView**在调用系统方法

    - (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event
    {
    }
发现触摸点在**fitView**身上，那么系统就会将**fitView**的子控件倒序遍历一遍，如果找到能够符合该触摸事件的控件，就交给该控件执行响应的触摸方法，如果没有子控件符合要求，那就默认**fitView**为符合该条件的**View**

**UIView**不能接收触摸事件的三种情况：

1.不允许交互：userInteractionEnabled = NO;<br>
2.隐藏：如果把父控件隐藏，那么子控件也会隐藏，隐藏的控件不能接受事件<br>
3.透明度：如果设置一个控件的透明度<0.01，会直接影响子控件的透明度。alpha：0.0~0.01为透明。

### 四:寻找最适合的View的详细分析

在详细讲解之前，需要了解两个重要的方法:<br>

    1.- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event{}

    2.- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event{}


第一个方法事件一传递给一个控件,这个控件就会调用他自己的hitTest：withEvent：方法

hitTest:withEvent:方法作用:

寻找并返回最合适的view(能够响应事件的那个最合适的view)
注 意：不管这个控件能不能处理事件，也不管触摸点在不在这个控件上，事件都会先传递给这个控件，随后再调用hitTest:withEvent:方法

拦截事件的处理

正因为hitTest：withEvent：方法可以返回最合适的view，所以可以通过重写hitTest：withEvent：方法，返回指定的view作为最合适的view。
不管点击哪里，最合适的view都是hitTest：withEvent：方法中返回的那个view。
通过重写hitTest：withEvent：，就可以拦截事件的传递过程，想让谁处理事件谁就处理事件。
事件传递给谁，就会调用谁的hitTest:withEvent:方法。

事件传递给窗口或控件的后，就调用hitTest:withEvent:方法寻找更合适的view。所以是，先传递事件，再根据事件在自己身上找更合适的view。
不管子控件是不是最合适的view，系统默认都要先把事件传递给子控件，经过子控件调用自己的hitTest:withEvent:方法验证后才知道有没有更合适的view。即便父控件是最合适的view了，子控件的hitTest:withEvent:方法还是会调用，不然怎么知道有没有更合适的！即，如果确定最终父控件是最合适的view，那么该父控件的子控件的hitTest:withEvent:方法也是会被调用的。
技巧：想让谁成为最合适的view就重写谁自己的父控件的hitTest:withEvent:方法返回指定的子控件，或者重写自己的hitTest:withEvent:方法 return self。但是，建议在父控件的hitTest:withEvent:中返回子控件作为最合适的view！

原因在于在自己的hitTest:withEvent:方法中返回自己有时候会出现问题。因为会存在这么一种情况：当遍历子控件时，如果触摸点不在子控件A自己身上而是在子控件B身上，还要要求返回子控件A作为最合适的view，采用返回自己的方法可能会导致还没有来得及遍历A自己，就有可能已经遍历了点真正所在的view，也就是B。这就导致了返回的不是自己而是触摸点真正所在的view。所以还是建议在父控件的hitTest:withEvent:中返回子控件作为最合适的view！

注意:hitTest：withEvent:方法会在底层调用pointInside:withEvent:方法判断点在不在方法调用者的坐标系上。

### 五:响应者链条的工作原理

当用户点击屏幕产生一个触摸事件以后，经过一系列的事件传递，会找到最合适的视图控件来处理这个事件，找到最合适的视图控件以后，就会调用控件的touches方法来做具体的事件处理，而这些touches方法的默认做法是将事件顺着响应者链条向上传递，在代码上的最直接的展示如下:

    //只要点击控件,就会调用touchBegin,如果没有重写这个方法,自己处理不了触摸事件
    // 上一个响应者可能是父控件
    - (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{ 
    // 默认会把事件传递给上一个响应者,上一个响应者是父控件,交给父控件处理
    [super touchesBegan:touches withEvent:event]; 
    // 注意不是调用父控件的touches方法，而是调用父类的touches方法
    // super是父类 superview是父控件 
    }

在事件的响应中，如果某个控件实现了touches...方法，则这个事件将由该控件来接受，如果调用了[supertouches….];就会将事件顺着响应者链条往上传递，传递给上一个响应者；接着就会调用上一个响应者的touches….方法，所以我们可以利用这个原理，做到一个事件多个对象处理，我们只需要在重写touches方法的时候，除了写自己需要处理的代码，同时调用[super touchesBegan:touches withEvent:event],将事件响应再交给父类处理。

*经过上面的分析，咱们可以轻易的得出结论：事件的传递方式是从上往下(父控件到子控件),事件的响应方式是从下往上(顺着响应者链条，向上传递，从子控件到父控件)。*

*注:以上观点是本人参考别人文章后所作的总结和感悟，如有雷同，存属正常！

    