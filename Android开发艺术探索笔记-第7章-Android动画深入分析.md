title: Android开发艺术探索笔记-第7章 Android动画深入分析
date: 2016-03-30 23:46:02
tags: [android,Animation,Animator,属性动画]
categories: 《android开发艺术探索》笔记
---
分析android中的动画，包括View动画、帧动画、属性动画
<!--more-->
<h2>动画简介：</h2>
<h5>View动画:</h5>
可以实现平移、缩放、旋转、透明度四种变换效果，但是不会改变控件的真实属性。只是控件的一个影印。
<h5>帧动画</h5>
用一幅幅图片来组成动画，稍有不慎，容易产生oom
<h5>属性动画</h5>
本质是根据控件的真实属性来达到动画效果
</h2>View动画</h2>
<h5>种类</h5>
四种变化效果对应Animation的四个子类：TranslateAnimation、ScaleAnimation、RotateAnimation和AlphaAnimation。
<h5>XML方式定义</h5>
![](http://www.2cto.com/uploadfile/Collfiles/20140412/20140412090606140.jpg)
set标签标示动画的集合，可以包含多个动画，关于set，以下两个属性比较重要：

- android:interpolator 表示动画集合所采用的差值器，默认为@android:anim/accelerate_decelarete_interpolator
- android：shareInterpolator 标示集合中的动画是否和集合共享一个差值器，如果是false，那么，每一个动画还要指定差值器

每种动画的标签可以查看官方文档
<h5>使用XML定义的动画</h5>
<h5>此外，还可以通过代码来实现View动画</h5>
不过比较麻烦，还是xml来的快一些
<h5>帧动画</h5>
1. 首先通过一个xml文件来定义一个AnimstionDrawable

		<animation-list xmlns:android="http://schemas.android.com/apk/res/android">
			<item android:drawable="@mipmap/ic_launcher" android:duration="1000"/>
		    <item android:drawable="@mipmap/ic_launcher" android:duration="1000"/>
		    <item android:drawable="@mipmap/ic_launcher" android:duration="1000"/>
		    <item android:drawable="@mipmap/ic_launcher" android:duration="1000"/>
		</animation-list>
2. 将上述的drawable作为View的背景，并通过drawable来设置动画

	 		TextView textView = (TextView) findViewById(R.id.text);
	        textView.setBackgroundResource(R.drawable.frame_animation);
	        AnimationDrawable animationDrawable = (AnimationDrawable) textView.getBackground();
	        animationDrawable.start();
<h2>View动画的特殊使用场景</h2>
自从有了属性动画以后，View动画显得有些鸡肋，但是他还是有存在的必要的，原因如下：

- 在api 11之前，还是要使用View动画
- View动画在一些特殊场景中有着很好地用途
	- LayoutAnimation可以为ViewGroup中的子元素指定动画
	- Activity的切换动画


下面总结下这两种动画的使用方法：
#####LayoutAnimation
LayoutAnimation可以作用于ViewGroup，指定以后当他的子元素出场的时候就会播放这个动画

- 定义LayoutAnimation，如下所示

		<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/	android"
        	android:delay="30%"
        	android:animationOrder="reverse"
       		android:animation="@anim/slide_right"
		/>
- 为子元素指定具体的入场动画

	    <set xmlns:android="http://schemas.android.com/apk/res/android"   
	        android:interpolator="@android:anim/accelerate_interpolator">  
	    <translate android:fromXDelta="-100%p" android:toXDelta="0"  
	            android:duration="@android:integer/config_shortAnimTime" />  
		</set>  
- 为ViewGroup指定android:layoutAnimation属性，对于ListView而言，其子元素就具有入场动画了
<h5>Activity切换动画</h5>
主要用到 `overridePendingTransition(enter_anim,exit_anim);`在他的第一个参数中指定进入的Activity的动画，第二个参数指定退出的Activity的动画。这样就可以实现Activity的切换动画了。
<h2>属性动画</h2>
<h5>分类</h5>
属性动画有ValueAnimation、ObjectAnimation、AnimationSet，其中ObjectAnimation是ValueAnimation的子类。
<h5>原理</h5>
属性动画的原理其实就是每个一定时间设置控件的属性，来实现动画效果，动画的默认时间间隔是300ms。要注意的以下两点：
我们队Object的属性abc做动画，如果想让动画生效，必须满足两个条件

- Object必须提供setAbc方法，如果动画没有初始值，还要提供getABC方法，
- Object的setAbc对属性所做的改变必须能够通过某个方法展示出来

<h5>两个重要属性：</h5>
- android:repeatCount 动画循环次数，默认值是0，-1是无数次循环
- android:repeadMode 动画的循环模式，repeat表示连续重复，reverse表示逆向重复 1 -1 1 -1...

<h5>差值器和估值器的理解：</h5>
- TimeInterpolator，时间插值器，他的作用是根据时间流逝的百分比来计算出当前属性改变的百分比
- TypeEvaluator，类型估值器，他的作用是根据属性变换的百分比来计算改变后的属性值

<h5>属性动画的监听器：</h5>
属性动画有两个监听器:
- AnimatorListener 监听动画的开始结束等等
- AnimatorUpdateListener 监听动画每一次的改变

 	ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(textView,"scale",0,12);
        objectAnimator.addListener(new Animator.AnimatorListener() {
            @Override
            public void onAnimationStart(Animator animation) {

            }

            @Override
            public void onAnimationEnd(Animator animation) {

            }

            @Override
            public void onAnimationCancel(Animator animation) {

            }

            @Override
            public void onAnimationRepeat(Animator animation) {

            }
        });
        objectAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {

            }
        });
<h5>有一些属性不生效的解决办法</h5>
有一些setXXX属性并不能改变View的外观，因为它的作用根本就不是这个，比如Button的setWidth的含义是设置最大宽度，并不是设置Button的宽度，那么这个时候应该如何就Button的宽度来做一个动画呢?

- 给你的对象加上get和set方法
- 用一个类来包装原始对象，使get和set达到想要的效果
- 采用ValueAnimator，监听动画过程，自己实现属性的改变
<h2>使用动画的注意事项</h2>
- OOM问题 帧动画的时候要注意OOM问题的预防及解决
- 内存泄露，无限动画在activity退出时要及时停止
- 兼容性问题 3.0之前不支持属性动画
- View动画的问题，入出现setVisibility失效的情况要记得调用clearAnimation清除动画
- 不要使用px
- 硬件加速