title: Android开发艺术探索笔记-第11章Android的线程和线程池
date: 2016-04-02 11:59:28
tags: [android,Thread]
categories: 《android开发艺术探索》笔记
---
<h2>主线程和子线程</h2>
主线程的作用是运行四大组件以及处理用户的交互，而子线程的作用则是执行耗时任务，比如网络请求、IO操作等。从Android3.0开始，不能在主线程中执行网络操作。
<h2>Android中的线程形态</h2>
在Android中，除了传统的Thread以外，还包括AsyncTask、HandlerThread以及IntentService，他们的底层实现都是线程。
<h3>AsyncTask</h3>
<h5>AsyncTask的使用</h5>
AsyncTask的使用比较简单，但是有以下几点要注意：

- AsyncTask的类必须在主线程中加载
- AsyncTask的对象必须在主线程中创建
- execute方法必须在UI线程中调用
- onPreExecute()、onPostExecute()、doInBackground()和onProgressUpdate()方法不能直接调用
- 一个AsyncTask对象非只能执行一次
- 在android1.6之前，AsyncTask是串行执行任务，1.6-3.0是并行，3.0以后又改成了串行，但是可以调用executeOnExecutor来并行的执行任务
<h5>AsyncTask的原理</h5>
execute()->executeOnExecutor()
其中有一个sDefaultExecutor串行的线程池来执行所有排队的AsyncTask，在这个线程池中的执行过程如下：首先inPreExecutor先执行，然后线程池执行，此时执行的是，系统会把AsyncTask的Params的参数封装为FutureTask对象，然后交给sDefaultExecutor来执行，sDefaultExecutor首先会把FutureTask插入到任务队列中，然后调用scheduleNext来一个个的执行。过程如下所示：

	private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;

        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();
                    }
                }
            });//将要执行的任务插入到任务栈中
            if (mActive == null) {
                scheduleNext();//调用scheduleNext一个个的来执行
            }
        }

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }

AsyncTask中有两个线程池:SerialExecutor、THREAD_POOL_EXECUTOR和一个Handler。其中，SerialExecutor用于任务的排队，而THREAD_POOL_EXECUTOR用于真正的执行任务。Handler用于从线程池切换回主线程

AsyncTask的执行流程：

由于FutureTask的run方法会调用mWorker的call方法，因此mWorker的call方法会在线程池中执行

	 mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);

                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                //noinspection unchecked
                Result result = doInBackground(mParams);
                Binder.flushPendingCommands();
                return postResult(result);
            }
        };

将doInBackground的结果作为postResult的参数：

    private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }

将结果进行了一层封装后发送Handler消息，Handler是这么定义的。

	private static class InternalHandler extends Handler {
        public InternalHandler() {
            super(Looper.getMainLooper());
        }

        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
    }

当任务执行结束后会执行finish()方法：

	  private void finish(Result result) {
	        if (isCancelled()) {
	            onCancelled(result);
	        } else {
	            onPostExecute(result);
	        }
	        mStatus = Status.FINISHED;
	    }
在finish()里边判断是取消了任务还是任务执行结束，做不同的操作：

	  private void finish(Result result) {
	        if (isCancelled()) {
	            onCancelled(result);
	        } else {
	            onPostExecute(result);
	        }
	        mStatus = Status.FINISHED;
	    }

<h3>HandlerThread</h3>
Handler是一种可以使用Handler的Thread，原理是在run方法中通过Looper.prepare()来创建消息队列，并通过Looper.loop开启消息循环。其实就是在Thread的基础上做了一层封装，使之可以接受消息，外部可以通过发送消息来控制Thread中任务的执行。下面是他的run方法:

	    public void run() {
	        mTid = Process.myTid();
	        Looper.prepare();
	        synchronized (this) {
	            mLooper = Looper.myLooper();
	            notifyAll();
	        }
	        Process.setThreadPriority(mPriority);
	        onLooperPrepared();
	        Looper.loop();
	        mTid = -1;
	    }

HandlerThread的run方法是一个无限循环，当明确不需要使用的时候应该调用quit或者quitSatety来退出。IntentService就用到了HandlerThread。
<h3>IntentService</h3>
从IntentService的onCreate可以看出，其执行后台任务是利用HandlerThread来实现的。

	 public void onCreate() {
	        // TODO: It would be nice to have an option to hold a partial wakelock
	        // during processing, and to have a static startService(Context, Intent)
	        // method that would launch the service & hand off a wakelock.
	
	        super.onCreate();
	        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
	        thread.start();
	
	        mServiceLooper = thread.getLooper();
	        mServiceHandler = new ServiceHandler(mServiceLooper);
	    }
onStartCommand是如何处理外界的Intent的？

	public int onStartCommand(Intent intent, int flags, int startId) {
        onStart(intent, startId);
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }
onStartCommand调用了onStart方法。

    public void onStart(Intent intent, int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }
在onStart里边仅仅是想mServiceHandler发送了一个消息，并将Intent传过去了。当mServiceHandler解析出来以后是将Intent交给onHandleIntent方法来执行的。他是一个抽象方法需要我们在子类中去实现它。
<h2>Android中的线程池</h2>
线程池的优点：
巴拉巴拉
<h3>ThreadPoolExecutor</h3>
ThreadPoolExecutor是线程池的真正实现：
下面是其构造方法：

	public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }

各个参数的含义：

- corePoolSize 核心线程数
- maximumPoolSize, 最大线程数
- keepAliveTime,非核心线程限制的超时时长
- unit,时间单位
- workQueue线程池中的任务队列

除此之外，他还有一个RejectedExecutionHandler当线程无法执行新的任务的时候，便会通过他通知调用者。

ThreadPoolExecutor执行任务的规则大致遵循如下规则：

1. 如果线程池中的线程数量未达到核心线程数就启动核心线程来执行任务
2. 如果达到核心线程数但未达到最大线程数，那么就启动一个非核心线程来执行任务
3. 如果达到最大线程数但队列未满，那么就等待
4. 如果任务队列已经满了，那么就拒绝执行此任务

<h3>线程池的分类</h3>
<h5>1.FixedThreadPool</h5>
- 只有核心线程，且核心线程没有超时
- 任务队列大小无限制
<h5>2.CachedThreadPool</h5>
- 只有非核心线程，且最大数为Integer.MAX_VALUE
- 超时是60s
- 比较适合大量的耗时较少的任务
<h5>3.ScheduledThreadPool</h5>
- 核心线程数固定,非核心线程没有限制
- 适合执行定时任务和具有固定周期的重复任务
<h5>4.SingleThreadExecutor</h5>
- 内部只有一个核心线程
- 他的意义在于统一所有的外界任务到一个线程中去。