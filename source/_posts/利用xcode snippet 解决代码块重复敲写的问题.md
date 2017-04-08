---
title: 利用xcode snippet 解决代码块重复敲写的问题
date: 2016-03-20 19:28:15
tags:
---

   **当你在写一个tableview时,一般都需要实现它的delegate和datasource时,你是为否有为如何快捷的将那几个常用的方法快速敲打出来而苦恼过,或者你会在别的代码块将那几个常用的方法复制粘贴过来,但是效率上来说其实也不高。是否有一种方法可以像代码提示那样只需几个关键字就能将这好几个的代理方法直接Tab键一按就洋洋洒洒洒落在你眼前呢?
   通过xcode snippet便可解决上面提到的那个问题。xcode snippet是一个代码块重复使用管理的功能，通过创建一些可重用的代码块，并且在任何需要的地方很容易的就可以使用这些代码块。这样就可以节省很多输入操作的时间, 从而提高开发的效率。接下来我们就利用snippet 解决上述的问题。 **

  **首先, 创建需要重复使用的代码块。然后单击并按住代码块，等文本光标变为箭头光标。接着将代码块拖放到code snippet library中，然后松开鼠标。如下图所示**

![1.jpeg](http://upload-images.jianshu.io/upload_images/2185894-ee8743f5a8439f22.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  **此时会弹出一个编辑框，通过编辑框对代码块进行相关的设置，如下图所示。**
  
![2.jpeg](http://upload-images.jianshu.io/upload_images/2185894-13e7a901489bc586.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

** 设置相关说明:**
* Title：Code Snippets的标题；
* Summary：Code Snippets的描述文字；
* Platform：可以使用Code Snippets的平台
* Language：可以在哪些语言中使用该Code Snippets
* Completion Shortcut：Code Snippets的快捷方式，比如这里设置为 "fast tableview", 当敲写这个关键字符串时就会出现该代码块的提示。
* Completion Scopes:可以在哪些文件中使用当前Code Snippets，比如全部位置，头文件中等，当然可以添加多个支持的位置。
* 最后的一个大得空白区域是对Code Snippets的效果预览。

  **现在来试用一下刚刚创建的snippet！有两种方法。
第一种是在code snippet library中找到刚刚创建的snippet，然后用鼠标将其拖拽到代码编辑器中,然后松开鼠标就可以了。**

![3.jpeg](http://upload-images.jianshu.io/upload_images/2185894-ffd3b2898d720abb.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  **第二种方法是在代码编辑器里输入completion shortcut中设置的内容,这里是“fast tableview”。**
  
![4.jpeg](http://upload-images.jianshu.io/upload_images/2185894-98fe5a0bbbe1afe9.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  **通过xcode snippet对经常使用的代码块进行管理，是不是发现顿时开发效率提高了很多呢。**