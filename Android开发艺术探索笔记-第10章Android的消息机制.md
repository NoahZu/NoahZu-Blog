title: Android开发艺术探索笔记 第10章Android的消息机制
date: 2016-03-29 23:53:54
tags: [android,Handler]
categories: 《android开发艺术探索》笔记
---
Android的消息机制，也就是Handler的使用以及实现原理。
<!--more-->

<h2>Android消息机制概述</h2>
Android消息机制的：  

- 作用：可以将一个任务切换到Handler所在的线程执行
- Handler的运行需要MessageQueue和Looper的支撑
- MessageQueue是消息队列
- Looper是轮询器
- ThreadLocal的作用是可以在不同线程中存储数据，各个线程中的数据是毫无关联的

概述

- 首先，在子线程中不能访问UI，是在ViewRootImpl中限定的。之所以这样做是为了保证UI的渲染速度，Android的UI控件都不是线程安全的
- Handler的出现就是为了解决子线程无法渲染UI的矛盾

Handler的工作原理浅析：

1. Handler创建的时候会根据当前线程的Looper来构建内部的消息循系统，如果当前线程没有Looper，则报错
2. Handler发送的消息会调用MessageQueue的enqueueMessage方法加入消息队列
3. Loop而不断的从MessageQueue中调用MessageQueue的next方法取出消息。
4. 取出消息后会调用相关联的Handler的handleMessage交给Handler处理


<h2>Android消息机制分析</h2>
<h4>ThreadLocal的工作原理</h4>
ThreadLocal的特点：

- ThreadLocal的作用是可以在不同线程中存储数据，各个线程中的数据是毫无关联的

ThreadLocal的使用场景：

- 某些数据以线程为作用域，而且，不同线程中存在不同的数据副本
- 复杂逻辑下对象的传递（没太明白这点）

ThreadLocal的实现原理：

- 其实每一个Thread对象内部存储着这么一个成员变量`ThreadLocal.Values localValues;` ,ThreadLocal会获取到当前所在线程，然后获取当前线程内部的这个成员变量，将数据从`ThreadLocal.Values localValues;`里边存取。同时，若为空，也负责创建它。


<h4>消息队列（MessageQueue）的工作原理</h4>
MessageQueue主要包含两个操作：

- 插入 enqueueMessage
- 读取 next
- 实际上，MessageQueue是通过一个单链表来维护这些消息的，而并非是队列
- enqueueMessage 方法

		boolean enqueueMessage(Message msg, long when) {
	        if (msg.target == null) {
	            throw new IllegalArgumentException("Message must have a target.");
	        }
	        if (msg.isInUse()) {
	            throw new IllegalStateException(msg + " This message is already in use.");
	        }
	
	        synchronized (this) {
	            if (mQuitting) {
	                IllegalStateException e = new IllegalStateException(
	                        msg.target + " sending message to a Handler on a dead thread");
	                Log.w(TAG, e.getMessage(), e);
	                msg.recycle();
	                return false;
	            }
	
	            msg.markInUse();
	            msg.when = when;
	            Message p = mMessages;
	            boolean needWake;
	            if (p == null || when == 0 || when < p.when) {
	                // New head, wake up the event queue if blocked.
	                msg.next = p;
	                mMessages = msg;
	                needWake = mBlocked;
	            } else {
	                // Inserted within the middle of the queue.  Usually we don't have to wake
	                // up the event queue unless there is a barrier at the head of the queue
	                // and the message is the earliest asynchronous message in the queue.
	                needWake = mBlocked && p.target == null && msg.isAsynchronous();
	                Message prev;
	                for (;;) {
	                    prev = p;
	                    p = p.next;
	                    if (p == null || when < p.when) {
	                        break;
	                    }
	                    if (needWake && p.isAsynchronous()) {
	                        needWake = false;
	                    }
	                }
	                msg.next = p; // invariant: p == prev.next
	                prev.next = msg;
	            }
	
	            // We can assume mPtr != 0 because mQuitting is false.
	            if (needWake) {
	                nativeWake(mPtr);
	            }
	        }
	        return true;
    	}
- next方法：


	    Message next() {
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }

            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message.
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
        	    nextPollTimeoutMillis = 0;
        	}
    	}

值得注意的是next是一个无线循环的方法，如果没有消息就一直等着，直到取到消息。
<h4>Looper的工作原理</h4>
Handler的工作需要Looper，没有Looper就会报错，那么怎么获取Looper呢？

- ActivityThread，如果在主线程中，可通过getMainLooper来获取Looper
- 如果在子线程中，可以调用Looper.prepare()方法来为当前线程创建一个Looper

Looper的loop方法：

- loop是一个死循环
- 唯一的退出方式是MessageQueue的next方法返回null
- 由于MessageQueue的next方法是一个阻塞方法，所以next阻塞的时候，loop也会阻塞在那里。
- 如果MessageQueue返回了新的Message，那么Looper就会处理这条消息：msg.target.dispatchMessage(msg),这里的msg.target是发送消息的Handler对象，这样Handler发送的消息最终就又交给他的dispatchMessage来处理了。
<h4>Handler的工作原理</h4>
Handler的工作原理主要包括发送和接受消息：

- 发送消息的本质就是将一条Message翻入MessageQueue队列中
- 接收消息则由dispatchMessage来处理

Handler还有一个特殊的构造方法
	
	public Handler(Looper looper){
		this(looper,null,false);
	}
<h2>主线程的消息循环</h2>
Android的主线程就是ActivityTHread，入口方法为main，在main方法中系统会通过Looper.prepareMainLooper()来创建主线程的Looper以及MessageQueue，并通过Looper.loop()来开启主线程的消息循环。