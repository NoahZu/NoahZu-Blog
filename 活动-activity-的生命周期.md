title: 活动(activity)的生命周期
date: 2015-12-04 19:23:54
tags: [avtivity,android]
categories: android
---
###一、周期的各个阶段
>首先，通过谷歌官方发布的生命周期图来理解一下整个流程：

![这里写图片描述](http://img.blog.csdn.net/20150712105521969)

>首先在这里简要介绍一下周期中各个方法：
>
 onCreate():在活动被创建的时候执行 
 onStart():在活动开始可见的时候执行
onPause():在活动失去焦点，不可交互的时候执行，但此时此活动仍然可见
onStop():在活动完全被其他活动覆盖的时候执行，显然，此时活动不可见了 
onDestory():在活动销毁的时候
onRestart():当被覆盖的活动没被回收再次启动的时候调用。

###二、用三个小周期去理解整个生命周期
####1.完整的生命周期
>从onCreate()到onDestroy()之间所经历的的是完整的生命周期。所以，应该在onCreate()完成各种初始化操作，在onDestroy()完成各种资源回收的操作。
####2.可见的生命周期
>从onStart()到onStop()之间所经历的的是可见生命周期，只要是在这之间，activity的内容都是可见的。既然是从onStart()开始可见，到onStop()不可见，那么我们就可以在onStart()里边加载用户可见的资源，在onStop()里边释放这些资源。这样的话，比在onDestroy()里边释放资源要节省内存。
####3.前台生命周期
>从onResume()到onPause()之间经历的是前台生命周期，在这之间的不仅可见，而且是可以交互的。

###三、各个方法执行的时机
#####onCreate():
>这个不用多说啦，就是在Activity被创建的时候调用。如果一个Activity被回收掉，那么他再次启动的时候也会调用这个方法。
#####onStart()
>Activity被创建以后，便开始启动，执行了onStart()方法以后，这个Activity便可见。如果一个Activity被完全覆盖但是系统没有回收它，那么他再次启动的时候，便会先执行onRestart()方法，然后再次执行onStart()方法。右上边的图片可以看出
#####onResume()
>查了一下单词“Resume”的意思是恢复的意思。实在搞不明白恢复放在这里的关系。但是onResume()执行完以后，这个Activity便可以与用户进行交互了。这个Activity便处于正式的运行状态，可以与用户进行交互。
#####onPause()
>当另外一个活动来到前台的时候，此时要将当前运行的Activity覆盖，会先执行这个方法，然后在执行onStop()。不过，如果到来的是一个对话框（没有完全覆盖当前Activity）的话，当前的Activity便会执行到onPause()，不会继续执行inStop(),因为他并不是完全不可见的。此时，在这个状态下，除非是内存极其紧张，否则不会回收这个Activity。
#####onStop()
>如果当前activity被另一个Activity覆盖了，完全不可见了。那么onStop()便会被执行。如果内存紧张的时候，便会回收掉这个Activity。如果回收的话便会向下执行到onDestroy().
#####onRestart()
>如果被完全覆盖的activity没有被回收，那么他再次执行的时候便会首先执行onRestart()方法。在执行onStart()方法。
####onDestroy()
>当系统内存不足了，那么他便会首先搜寻被完全覆盖的Activity，将他们的内存释放掉，释放的时候便会执行onDestroy()方法。我们可以在onDestroy()中释放一些自己定义的变量。
###四、活动被回收了怎么办
>对于一般的活动，当被系统回收掉以后，无非就是重新创建这个活动。但是如果一个活动我们要求他再次创建的时候保持原样，就比如我们在文本框里输入了一些内容。那么此时该怎么办呢？
>自然是有办法解决的:

```
onSaveInstanceState(Bundle bundle){
	
}
```
>这个方法会保证在活动被回收前被调用，因此我们就可以在这个方法之中做一些保存操作。然后onCreate()接受的参数就是这个方法保存的那个Bundle。

```
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
```
###五、总结
>郭霖大神在他的《第一行代码》关于活动的声明周期的讲解算是比较透彻，令我受益匪浅。
