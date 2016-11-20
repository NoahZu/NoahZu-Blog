title: TextView Source Read
date: 2016-03-24 15:10:05
tags: [android,TextView,source]
categories: android
---
TextView 源码阅读
<!--more-->
####TextView概览
默认TextView是不可编辑的，如果想让他可以编辑，可以使用EditText。如果想允许TextView能被复制，要在xml文件中将android:textIsSelectable属性设置为true或者调用setTextIsSelectable(true)
####摘要
#####TextView内部涉及的类：  
- enum TextView.BufferType
- interface TextView.OnEditorActionListener--当Editor上有任何行为的时候的回调事件
- class TextView.SavedState onSaveInstanceState()--保存的用户接口状态
#####XML属性
- android:autoLink &nbsp;&nbsp;setAutoLinkMask(int) ：是否自动给链接加下划线
- android:autoText &nbsp;&nbsp;setKeyListener(KeyListener) 是否自动纠正语法错误
- android:breakStrategy &nbsp;&nbsp;setBreakStrategy(int):换行策略
- android:bufferType setText(CharSequence,TextView.BufferType):决定getText()要返回的最小类型
- android:capitalize&nbsp;&nbsp;setKeyListener(KeyListener) //TODO
- android:cursorVisible	setCursorVisible(boolean)：游标是否可见
- android:digits	setKeyListener(KeyListener)：是否数字
- android:drawableBottom 底部的drawable
- android:drawableEnd 结束处的drawable
- android:drawablePadding drawables和text之间的间距
- android:drawableRight 右边的drawable
- android:drawableStart 开始处的drawable
- 