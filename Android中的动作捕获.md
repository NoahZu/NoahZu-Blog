title: Android中的动作捕获
date: 2016-04-12 16:18:01
tags: [ActionEvent,android]
categories: android
---
怎样捕获android的事件中常见的动作
<!--more-->

<h4>常见动作事件：</h4>

- 多点触控
- 拖动
- 快速滑动
- 按下屏幕
- 轻触屏幕瞬间
- 双击
- 单击

<h4>事件分发简述</h4>
要捕获我们所需要的动作，首要前提是能够得到分发过来的点击事件，这样我们才可以对事件进行一个判断和处理。我所认为的动作的判断主要有两种方式：

- 自己书写逻辑来判断
- 借助工具类

自己书写的话就是要获取到分发到的ActionEvent，然后来做判断。但是有一些常用的手势我们可以通过工具类完成，

<h4>利用GestureDetector来完成常用动作的捕获</h4>
- 创建GestureDetector类


		GestureDetector mGestureDetector;
		GestureDetector.SimpleOnGestureListener mSimpleGestureDetector;
注意创建的时候要根据自己的需要来指定实现的接口，主要有：
	- OnGestureListener （常用的监听事件）
	- OnDoubleTapListener （双击相关的监听事件）

如果你想都实现这两个接口中的方法，那么可以使用GestureDetector.SimpleOnGestureListener

- 接着，托管View的onTouchEvent

	    public boolean onTouchEvent(MotionEvent event) {
	        boolean b = mGestureDetector.onTouchEvent(event);
	        int fingers = event.getPointerCount();
	        if(fingers > 2){
	            Toast.makeText(this,fingers+"点触摸",Toast.LENGTH_SHORT).show();
	        }
	        return b;
	    }

- 做完这些以后，我们就可以有选择的实现其中的方法来实现对应的事件了，这里我将所有的方法都实现了一遍。

			mSimpleGestureDetector = new GestureDetector.SimpleOnGestureListener(){
	            @Override
	            public boolean onSingleTapUp(MotionEvent e) {
	                Log.d(TAG,"SingleTapUp");
	                return true;
	            }
	
	            /**
	             * 长按
	             * @param e
	             */
	            @Override
	            public void onLongPress(MotionEvent e) {
	                Log.d(TAG,"LongPress");
	                super.onLongPress(e);
	            }
	
	            /**
	             * 手指按下屏幕并拖动
	             * @param e1
	             * @param e2
	             * @param distanceX
	             * @param distanceY
	             * @return
	             */
	            @Override
	            public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
	                Log.d(TAG,"Scroll");
	                return super.onScroll(e1, e2, distanceX, distanceY);
	            }
	
	            /**
	             * 用户按下触摸屏，快速滑动后松开
	             * @param e1
	             * @param e2
	             * @param velocityX
	             * @param velocityY
	             * @return
	             */
	            @Override
	            public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
	                Log.d(TAG,"Fling");
	                return super.onFling(e1, e2, velocityX, velocityY);
	            }
	
	            /**
	             * 用户按下屏幕，有一个ACTION——DOWN触发
	             * @param e
	             * @return
	             */
	            @Override
	            public boolean onDown(MotionEvent e) {
	                Log.d(TAG,"Down");
	                return super.onDown(e);
	            }
	
	            /**
	             * 手指轻轻触摸屏幕，尚未松开或者拖动
	             * @param e
	             */
	            @Override
	            public void onShowPress(MotionEvent e) {
	                Log.d(TAG,"ShowPress");
	                super.onShowPress(e);
	            }
	
	            /**
	             * 双击，由两次单击触发
	             * @param e
	             * @return
	             */
	            @Override
	            public boolean onDoubleTap(MotionEvent e) {
	                Log.d(TAG,"DoubleTap");
	                return super.onDoubleTap(e);
	            }
	
	            /**
	             * 表示发生了双击行为
	             * @param e
	             * @return
	             */
	            @Override
	            public boolean onDoubleTapEvent(MotionEvent e) {
	                Log.d(TAG,"DoubleTapEvent");
	                return super.onDoubleTapEvent(e);
	            }
	
	            /**
	             * 严格的单击行为，表示仅仅是一个单击，而不是双击中的一个单击
	             * @param e
	             * @return
	             */
	            @Override
	            public boolean onSingleTapConfirmed(MotionEvent e) {
	                Log.d(TAG,"SingleTapConfirmed");
	                return super.onSingleTapConfirmed(e);
	            }
	
	            @Override
	            public boolean onContextClick(MotionEvent e) {
	                Log.d(TAG,"ContextClick");
	                return super.onContextClick(e);
	            }
        };

也分别解释了哪一个函数是监听哪一个事件。
<h4>实现多点触控</h4>
实现多点触控的话没有现成的帮助类，不过也简单，如果是多个手指的话可通过几个方法来获取每个手指的情况：

   		int fingers = event.getPointerCount();//获取手指个数	
        int index = 0;
        MotionEvent.PointerCoords p = new MotionEvent.PointerCoords();
        event.getPointerCoords(index,p);//获取每个手指的坐标
