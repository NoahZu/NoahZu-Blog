title: android studio导入.so文件的方法
date: 2015-12-19 16:12:18
tags: [android]
categories: [android,IDE]
---
.so文件导入到android studio总是出现这样或那样的问题，不过，今天我get到一个新的方法，以新浪微博的库文件为例。他的.so文件如下：

![这里写图片描述](http://img.blog.csdn.net/20151219162334915)

每一个目录下边有一个.so文件。

第一步：首先建立一个文件夹名为lib，然后把上面的含有.so文件的文件夹放入lib文件夹下边。  
第二步：压缩成.zip文件  

![这里写图片描述](http://img.blog.csdn.net/20151219162923447)  

第三步：将.zip后缀改成.jar也可以将lib改为其他的名字  
第四步：按照导入.jar文件的方式导入Android Studio就好了
