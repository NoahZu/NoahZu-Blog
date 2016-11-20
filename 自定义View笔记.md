title: 自定义View笔记
date: 2016-08-30 22:10:04
tags: [android,Custom View]
categories: [android]
---
http://www.gcssloop.com/1970/01/CustomViewIndex/阅读笔记。
<!--more-->
###基础篇
- 坐标系
    - android中的坐标系，比较简单
- 角度弧度
    - 角度弧度的定义，换算关系，比较简单
- 颜色
    - 掌握Android中关于颜色的常用用法即可
###进阶篇
- 分类与流程
    - 流程
		![](http://i4.buimg.com/4851/0d8d35b373148ce2.png)
    - 分类：
        - 自定义ViewGroup
            - 一般是继承ViewGroup或者各种Layout
        - 自定义View
            - 大多数情况下有替代方案，但是有可能内存消耗过大，还是自定义一个比较好。
    - 几个重要的函数
        - 构造函数：
        
	            public void SloopView(Context context) {}
	            public void SloopView(Context context,AttributeSet attrs) {}
	            public void SloopView(Context context, AttributeSet attrs, int defStyleAttr) {}
	            public void SloopView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {}

            - 其中：
                - defStyle指的是在当前Application和Activity中Theme的默认Style，用处不大。
                - Context是当前的上下文环境
                - AttributeSet是在布局文件中使用的时候属性以及他们的值，包含自定义属性

        - 测量View大小（onMeasure）
            - 在onMeasure中可以取出控件的测量模式和大小的数值
            - 可以用setMeasuredDimension( widthsize, heightsize); 这个函数改变控件的大小
            - 疑问，在自定义一个控件的时候如何判断是否应该重写onMeasure来改变控件的大小
        - 确定View大小（onSizeChanged）
            - Q: 在测量完View并使用setMeasuredDimension函数之后View的大小基本上已经确定了，那么为什么还要再次确定View的大小呢？
            - A: 这是因为View的大小不仅由View本身控制，而且受父控件的影响，所以我们在确定View大小的时候最好使用系统提供的onSizeChanged回调函数。
            - 当View大小改变（有可能是View本身自己设置，也有可能是父控件设置）时的一个回调
            - 也就是说，在这个函数才是真正确定View大小的时机
        - 确定子View的布局位置（onLayout）
            - 一般在ViewGroup中用到
            - 在自定义ViewGroup中，onLayout一般是循环取出子View，然后经过计算得出各个子View位置的坐标值，然后用child.layout(l, t, r, b)设置子View位置;
            - 传的参数是上下左右距离父控件的大小
        - 绘制内容
            - 使用canvas画图
        - 对外提供操作方法和监听回调
- Canvas绘制图形
    - Canvas常用操作速查表：
![](http://i4.buimg.com/4851/b5dcf10e1f5b2978.png)
    - Canvas详解
        - 绘制的基本形状由Canvas确定，但绘制出来的颜色,具体效果则由Paint确定
        - 画笔的模式：
            - STROKE                //描边
            - FILL                  //填充
            - FILL_AND_STROKE       //描边加填充
- Canvas之画布操作：
    - 所有的画布操作都只影响后续的绘制，对之前已经绘制过的内容没有影响。
    - 位移translate
    - 缩放（scale）：
        - 0-1之间是缩小
        - 大于1是放大
        - 负数是先缩放，后翻转 围绕缩放中心
    - 旋转(rotate)
        - 搞清楚旋转中心，旋转角度
    - 错切(skew)
        - 含义是将画布倾斜一定角度
    - 快照（save）和回滚（restore）
        - Q: 为什存在快照与回滚
        - A：画布的操作是不可逆的，而且很多画布操作会影响后续的步骤，例如第一个例子，两个圆形都是在坐标原点绘制的，而因为坐标系的移动绘制出来的实际位置不同。所以会对画布的一些状态进行保存和回滚。
- Canvas之图片文字
    - 绘制图片：
        - drawPicture()
            - 使用Picture要关闭硬件加速：在AndroidMenifest文件中application节点下添上 android:hardwareAccelerated=”false”以关闭整个应用的硬件加速。
            - 你可以把Picture看作是一个录制Canvas操作的录像机。
            - Picture的关键方法：
![](http://i4.buimg.com/4851/9c05f015d631da89.png)
            - 将Picture中的内容绘制出来可以有以下几种方法：
            ![](http://i4.buimg.com/4851/5e14a58327e5e901.png)
        - drawBitmap()
            - 获取Bitmap的方式：
                - 通过Bitmap创建，复制一个已有的Bitmap或者空白的Bitmap
                - 【不推荐】通过BitmapDrawable获取    从资源文件 内存卡 网络等地方获取一张图片并转换为内容不可变的Bitmap
                - 通过BitmapFactory获取    从资源文件 内存卡 网络等地方获取一张图片并转换为内容不可变的Bitmap
    - 绘制文字
        - 方法：
			![](http://i4.buimg.com/4851/5a906d9b46a16a8f.png)
            - 第一类：只能指定文本基线位置
            - 第二类：可以指定每个文字的位置
            - 第三类：指定一个路径，根据路径绘制文字
- Path之基本操作
    - Path常用方法表
    - Path详解
        - 关闭硬件加速
        - Path作用：
            - 绘制复杂图形
            - 根据路径绘制文本
            - 裁剪画布
        - Path含义
        - Path使用方法详解
            - 第一组
                - lineTo 连线至
                - moveTo 跳至 （不连线）
                - setLastPoint 设置之前操作的最后一个点位置，跟跳至差不多，但是比跳至少一个点
            - 第二组 addXxx和arcTo（在Path中添加图形）
                - addXxx中的Path.Direction dir的理解：
                    - 是指在存储这个图形的各个点的时候是按照顺时针还是逆时针
                - addPath
                - addArc和arcTo：
                    - addArc 添加一个圆弧到path 直接添加一个圆弧到path中
                    - arcTo 添加一个圆弧到path 添加一个圆弧到path，如果圆弧的起点和上次最后一个坐标点不相同，就连接两个点
            - 第三组：isEmpty、 isRect、isConvex、 set 和 offset
- Path之贝塞尔曲线
    - 贝塞尔曲线的原理：
        - 贝塞尔曲线是用一系列点来控制曲线状态的，这些点分为两类：
            - 数据点：确定曲线的起始和结束位置
            - 控制点：确定曲线的弯曲程度
        - 一阶曲线原理：
            - 一阶曲线是没有控制点的，仅有两个数据点(A 和 B)
            - 对应的方法就是lineTo
        - 二阶曲线原理：
            - 二阶曲线由两个数据点(A 和 C)，一个控制点(B)来描述曲线状态
            - 生成过程：
                - 确定一个变化的比例p范围是（0，1]，这个比例的变化伴随的是曲线的生成过程
                - 连接AB，CB，在AB上取D，BC上取E，使得：p=AD/AB=BE/BC
                - 连接DE，在DE上取F，使得p=AD/AB=BE/BC=DF/DE
                - 当比例p变化时F点的坐标跟着变化，F点的变化轨迹就是二阶贝塞尔曲线
            - 原理动图：
                - https://upload.wikimedia.org/wikipedia/commons/3/3d/B%C3%A9zier_2_big.gif
            - 二阶曲线对应的方法是quadTo
        - 三阶曲线原理
            - 三阶曲线由两个数据点(A 和 D)，两个控制点(B 和 C)来描述曲线状态
            - 三阶的原理与二阶类似
            - 原理动图：
                - https://upload.wikimedia.org/wikipedia/commons/d/db/B%C3%A9zier_3_big.gif
            - 三阶曲线对应的方法是cubicTo
        - 练习贝塞尔曲线的网站：http://bezier.method.ac/
    - 了解贝塞尔曲线的使用：
        - 贝塞尔曲线在动态变化过程中有类似于橡皮筋一样的弹性效果，因此在制作一些弹性效果的时候很常用。
        - 使用场景：
		![](http://i1.piimg.com/4851/080f79de81a76a8e.png)
- Path之完结篇
    - rXxx方法方法，带有r的方法都是相对坐标，不带r的是绝对坐标
    - 填充模式：
        - 填充
        - 描边
        - 填充+描边
    - 布尔运算
        - path1.op(path2, Path.Op.DIFFERENCE);   // 对 path1 和 path2 执行布尔运算，运算方式由第二个参数指定，运算结果存入到path1中。
        - path3.op(path1, path2, Path.Op.DIFFERENCE)   // 对 path1 和 path2 执行布尔运算，运算方式由第三个参数指定，运算结果存入到path3中。
    - 边界运算：
        - void computeBounds (RectF bounds, boolean exact)
- PathMeasure
    - 方法一览：
	![](http://i1.piimg.com/4851/2afd169f5d0e40c3.png)
- Matrix原理
    - Matrix简介
        - Matrix是一个矩阵，主要功能是坐标映射，数值转换。