title: android service使用方法
date: 2016-02-19 22:25:44
tags: [android,四大组件]
categories: [android]
---
<h4>两种服务的启动方式：</h4>
Start方式（与activity无关联）：
>跟启动源没有任何关系
>无法得到服务对象  

Bind方式（与activity有关联）：
>通过Ibinder接口实例，返回一个ServiceConnection对象给启动源
>通过ServiceConnection对象的相关方法可以得到Service对象
 
两种方式的生命周期对比：
![生命周期图](http://img.blog.csdn.net/20160219224403918)
     

<h4>Start方式：</h4>
代码比较简单，步骤是：
>1. 自定义一个类继承Service
>2. 在AndroidMainifest.xml文件中注册
>3. 利用Intent和startService(intent)来启动服务

<h4>Bind方式：</h4>
此方式可以与activity进行通讯  

1、 自定义类继承Service：
2、 在自定义的service里边写一个内部类继承Binder类MyBinder，在MyBinder里边定义activity需要操纵service的方法，然后在onBind里边返回MyBinder的实例，代码如下:  
```
	package noahzu.github.io.servicedemo;
	import android.app.Service;
	import android.content.Intent;
	import android.os.Binder;
	import android.os.IBinder;
	import android.support.annotation.Nullable;
	import android.util.Log;
	public class MyService extends Service {
	    private static final String TAG = "MyService";
	    @Override
	    public void onCreate() {
	        super.onCreate();
	        Log.d(TAG,"==onCreate");
	    }
	    @Nullable
	    @Override
	    public IBinder onBind(Intent intent) {
	        Log.d(TAG,"==onBind");
	        return new MyBinder();
	    }
	    @Override
	    public boolean onUnbind(Intent intent) {
	        Log.d(TAG,"==onUnbind");
	        return super.onUnbind(intent);
	    }
	    public class MyBinder extends Binder{
	        public void play(){
	            Log.d(TAG,"play");
	        }
	        public void pause(){
	            Log.d(TAG,"pause");
	        }
	        public void next(){
	            Log.d(TAG,"next");
	        }
	        public void previous(){
	            Log.d(TAG,"previous");
	        }
	    }
	}
```  

3、 在activity中定义一个MyBinder

```MyService.MyBinder myBinder;```

4、 在activity中定义ServiceConnection的实例，并在其xx方法中返回的Service强转赋值给myBinder，这样activity就可以通过自己持有的myBinder来操纵service了

```
ServiceConnection serviceConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        myBinder = (MyService.MyBinder)service;
    }
    @Override
    public void onServiceDisconnected(ComponentName name) {

    }
};
```


5、 ServiceConnection的实例来启动服务


```
public void bindService(View view){
    Intent intent = new Intent(MainActivity.this,MyService.class);
    bindService(intent, serviceConnection, Service.BIND_AUTO_CREATE);
}
```







