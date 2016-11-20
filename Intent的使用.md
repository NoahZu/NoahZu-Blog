title: Intent的使用
date: 2015-12-04 19:25:39
tags: [intent,android]
categories: android
---
##Intent的使用

####一、显式Intent的基本用法
>显式的Intent实现从一个Activity跳转到另外一个Activity，并可以传送一些数据。用法如下：

```
Intent intent = new Intent(FirstActivity.this,SecondActivity.class);
startActivity(Intent);//利用Intent启动SecondActivity
```
>此外还可以利用显式Intent在两个Activity之间传送一些数据，这里先简单的说明一下，稍后详细说明。利用显式Intent传送数据：

```
//FirstActivity.java
Intent intent = new Intent(FirstActivity.this,SecondActivity.class);
intent.putExtra("extra_data", data);//利用Intent穿送数据
startActivity(Intent);//利用Intent启动SecondActivity
```

```
//SecondActivity.java
Intent intent = getIntent();
String data = intent.getStringExtra("extra_data");
```
####二、隐式Intent的大用处
#####隐式Intent用于什么地方
>显式Intent的使用方法是比较简单的，隐式Intent的使用要稍微难一些，不过隐式Intent可是有大用处的。先举几个最简单的例子吧：


- 如果我要在我的App里边启动拨号界面
- 如果我要在App中跳转到一个网页
- 如果我要在App中启动其他的App


>他们都需要隐式的Intent

#####隐式Intent怎么用
>相比于显式Intent，隐式Intent则含蓄了许多，它并不明确指出我们想要启动哪一个活动，而是指定了一系列更为抽象的action和category等信息，然后交由系统去分析这个Intent，并帮我们找出合适的活动去启动。


>每一个活动（Activity）可以配置自己的action和category,方式如下：
```
<activity android:name=".SecondActivity" >
	<intent-filter>
	<action android:name="com.example.activitytest.ACTION_START" />
	<category android:name="android.intent.category.DEFAULT" />
	</intent-filter>
</activity>  
```

>当一个活动要启动另外一个活动的时候，只需要在Intent里边指定要启动的Activity的action和category就行了。方式如

```
Intent intent = new Intent("com.example.activitytest.ACTION_START");
intent.addCategory("com.example.activitytest.MY_CATEGORY");
startActivity(intent);
```



####三、如何在Intent之间传送数据
####1.直接传送

```
//传送数据
String data = "Hello SecondActivity";
Intent intent = new Intent(FirstActivity.this, SecondActivity.class);
intent.putExtra("extra_data", data);
startActivity(intent);
```

```
//接收数据
Intent intent = getIntent();
String data = intent.getStringExtra("extra_data");
Log.d("SecondActivity", data);
```
#####2.通过Bundle传送

####四、销毁前向启动他的Activity传送数据
>A活动了B活动，在B活动中编辑了一些数据。在按下返回键要销毁B活动的时候，B活动通过Intent将数据传回给启动它的A活动。详情见下图：

![这里写图片描述](http://img.blog.csdn.net/20150712104033545)

>在A活动里边通过startActivity方法启动B活动，B活动处理完数据以后，当要返回到A活动时（可能是一个返回按钮）通过setResult方法将Intent传给A活动。在A活动里边通过onActivityResult方法接收B活动返回的Inten。这就是它的基本流程

####五、总结
>本人android刚刚入门，看完郭神的《第一行代码》受益匪浅，将自己的笔记以博客的形式发在csdn，督促自己学习，加油！