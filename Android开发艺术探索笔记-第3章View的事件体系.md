title: Android开发艺术探索笔记-第3章View的事件体系
date: 2016-04-02 17:20:55
tags: [android,View事件体系]
categories: 《android开发艺术探索》笔记
---
<h2>View基础知识</h2>
<h4>View的位置参数</h4>
- View的位置主要由四个顶点来决定:top、left、right、bottom，当View移动的时候，这四个参数并不会发生改变
- android3.0以后新增的参数：
	- x、y表示View左上角的坐标
	- translationX、translationY表示左上角相对于父容器的偏移量
- x=left+translationX
- y=top + translationY

<h4>MotionEvent和TouchSlop</h4>
- MotionEvent对象可以使我们得到点击事件发生的x和y坐标
- 典型的事件类型有：
	- ACTION_DOWN：手指刚接触屏幕
	- ACTION_MOVE：手指在屏幕上移动
	- ACCTION_UP：手指从屏幕上松开的一瞬间
- TouchSlop是系统所能识别的被认为是滑动的最小距离
- 获取方法：`ViewCOnfiguration.get(getContext().getScaledTouchSlop())`
- 可以对一些滑动进行过滤

<h4>VelocityTracker、GestureDetector、Scroller</h4>
VelocityTracker用于追踪速度：

第一步：

`velocityTracker.addMovement(event);`

第二步：

	velocityTracker.computeCurrentVelocity(1000);
	p.x = (int)velocityTracker.getXVelocity();
	p.y = (int)velocityTracker.getYVelocity();

GestureDetector用于辅助检测用户的单击、滑动、长按、双击等行为，使用过程如下：

- 创建GestureDetector对象，顺便传入监听事件
- 接管onTouchEvent的event事件 

Scroller弹性滑动对象，用于实现View的弹性滑动
