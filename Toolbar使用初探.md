title: Toolbar使用初探
date: 2015-12-19 14:30:42
tags: [android,material design]
categories: android
---
本文主要记载Toolbar的使用方法，以及本人遇到的一些小问题。

<h3>第一步：通过设置主题（theme）来隐藏掉actionbar</h3>
这里有一个原则就是只要保证AndroidMainfest.xml文件里边的application标签里边的theme继承自`Theme.AppCompat.Light.NoActionBar`或者其他NoActionBar的主题就可以隐藏默认的actionbar。

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".activity.Launcher">
    	   ...
        </activity>
        <activity android:name=".activity.MainActivity">
           ...
		</activity>
    </application>
这里为了方便定制与扩展，写一个App.Base：

    <style name="App.Base" parent="Theme.AppCompat.Light.NoActionBar">
	</style>
更改AppTheme继承自App.Base：

    <style name="AppTheme" parent="App.Base">
    </style>

由于AndroidMainfest.xml里边默认的主题为AppTheme，所以这样就不用改了。也可以写一个自己的主题替换掉AndroidMainfest.xml里边的主题。只要保证主题继承自NoActionBar的主题就ok了。

在我们定制的theme下边可以分别设置toolbar背景、状态栏颜色、对应EditText编辑时、RadioButton选中、CheckBox等选中时的颜色。标签分别是：

- colorPrimary 对应ActionBar的颜色。
- colorPrimaryDark对应状态栏的颜色
- colorAccent 对应EditText编辑时、RadioButton选中、CheckBox等选中时的颜色。

那么设置颜色以后的主题就是：

    <style name="App.Base" parent="Theme.AppCompat.Light.NoActionBar">
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
下有一张图说明了各个标签对应界面的位置：

![](http://img.blog.csdn.net/20150427034747930)

<h3>第二步：利用toolbar构建自己的布局</h3>
隐藏好了主题以后就可以在布局文件中使用Toolbar了，需要注意的一点是有两个Toolbar，分别是Toolbar和v7包下面的Toolbar也就是android.support.v7.widget.Toolbar。两者的区别是：
>前者只能支持5.0以上的设备，后者是兼容包，可以支持5.0以下的设备。

所以最好使用后者，但是5.0以下的设备是不能改变状态栏颜色的这里需要注意一下。ok，就让我们用一下，我的布局文件如下：  

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    	xmlns:tools="http://schemas.android.com/tools"
    	xmlns:app="http://schemas.android.com/apk/res-auto"
    	tools:context="xiaozuzu.github.io.simpleweather.activity.MainActivity"
    	android:layout_width="match_parent"
    	android:layout_height="match_parent"
    	android:orientation="vertical">
	    <android.support.v7.widget.Toolbar
	        android:id="@+id/main_toolbar"
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:navigationIcon="@mipmap/menu"
	        android:minHeight="?android:attr/actionBarSize"
	        android:background="?attr/colorPrimary"
	        android:elevation="4dp"
	        />
	    <android.support.v4.widget.DrawerLayout
	        android:id="@+id/drawer_layout"
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        >
	        <!--content-->
	        <FrameLayout
	            android:id="@+id/main_content"
	            android:layout_width="match_parent"
	            android:layout_height="match_parent"/>
	        <!--menu-->
	        <include
	            layout="@layout/left_drawer_menu"
	            android:layout_width="240dp"
	            android:layout_height="match_parent"
	            android:layout_gravity="start"
	            />
    	</android.support.v4.widget.DrawerLayout>
	</LinearLayout>
上边是一个Toolbar，下边是一个DrawerLayout。
<h3>第三步：在代码中设定toolbar：</h3>
在代码中必须要进行的操作是：

	Toolbar toolbar = (Toolbar) findViewById(R.id.id_toolbar);//获取Toolbar
	setSupportActionBar(toolbar);//将Toolbar的对象设置进去替换actionBar

还有一些关于toolbar的title、logo等等设置既可以在xml文件中设置，也可以在代码中设置，他们是：

    <android.support.v7.widget.Toolbar
	    app:title="App Title"                                        
	    app:subtitle="Sub Title"
	    app:navigationIcon="@drawable/ic_toc_white_24dp"
	    android:layout_height="wrap_content"
	    android:minHeight="?attr/actionBarSize"
	    android:layout_width="match_parent"
	    android:background="?attr/colorPrimary"
    />

     toolbar.setLogo(R.mipmap.ic_launcher);
	 toolbar.setTitle("App Title");
	 toolbar.setSubtitle("Sub title");
	 toolbar.setNavigationIcon(R.drawable.ic_toc_white_24dp);
具体更多的方法可参考官方文档。[http://developer.android.com/design/patterns/actionbar.html#ActionButtons](http://developer.android.com/design/patterns/actionbar.html#ActionButtons)
做完以上工作就可以运行一下看看效果了。

![](http://img.blog.csdn.net/20151219152438994)
是不是状态栏也改变颜色了。
<h3>第四步：Menu Item的定制</h3>
在最右边还有一个菜单的标志，下边就是菜单的添加。

首先在res/menu文件夹下边简历一个菜单文件夹：

    <menu xmlns:android="http://schemas.android.com/apk/res/android"
    	xmlns:app="http://schemas.android.com/apk/res-auto"
    	xmlns:tools="http://schemas.android.com/tools"
    	tools:context="xiaozuzu.github.io.simpleweather.activity.MainActivity"
    	>
	    <item android:id="@+id/refresh"
	        android:title="刷新"
	        android:icon="@mipmap/ic_launcher"
	        android:visible="true"
	        android:orderInCategory="100"
	        app:showAsAction="ifRoom"/>
	    <item
	        android:id="@+id/collect"
	        android:title="收藏"
	        android:icon="@mipmap/collect"
	        android:visible="true"
	        app:showAsAction="ifRoom"/>
	</menu>

在编写菜单文件的时候需要注意的一点是showAsAction属性：


- 他的作用就是指定菜单项显示在ToolBbar上还是隐藏起来。
- 他的命名空间是app而不是android

他的几个属性的意思分别是：

- always表示总是显示在Toolbar上
- ifRoom表示如果空间足够就显示在Toolbar上
- never表示始终隐藏

然后复写onCreateOptionsMenu，加载菜单布局：

 	public boolean onCreateOptionsMenu(Menu menu) {
        MenuInflater inflater = new MenuInflater(this);
        inflater.inflate(R.menu.menus,menu);
        return true;
    }
然后可以复写onOptionsItemSelected方法来给菜单指定点击事件：

    public boolean onOptionsItemSelected(MenuItem item) {
		switch (item.getItemId()) {
                    case R.id.refresh:
                        Toast.makeText(MainActivity.this, "refresh", Toast.LENGTH_SHORT).show();
                        break;
                    case R.id.collect:
                        Toast.makeText(MainActivity.this, "collect", Toast.LENGTH_SHORT).show();
                        break;
                    case R.id.garbage:
                        Toast.makeText(MainActivity.this, "garbage", Toast.LENGTH_SHORT).show();
                        break;
                }
        return ture;
    }
也可以利用toolbar.setOnMenuItemClickListener方法：

    toolbar.setOnMenuItemClickListener(new Toolbar.OnMenuItemClickListener() {
            @Override
            public boolean onMenuItemClick(MenuItem item) {
                MenuInflater menuInflater = new MenuInflater(MainActivity.this);

                switch (item.getItemId()) {
                    case R.id.refresh:
                        Toast.makeText(MainActivity.this, "refresh", Toast.LENGTH_SHORT).show();
                        break;
                    case R.id.collect:
                        Toast.makeText(MainActivity.this, "collect", Toast.LENGTH_SHORT).show();
                        break;
                    case R.id.garbage:
                        Toast.makeText(MainActivity.this, "garbage", Toast.LENGTH_SHORT).show();
                        break;
                }
                return false;
            }
        });

<h3>最后</h3>
安利一下goolge自家的一个图标资源的网站[https://design.google.com/icons/#ic_list](https://design.google.com/icons/#ic_list)

参考资料：


- [Android 5.x Theme 与 ToolBar 实战 by Hongyang](http://blog.csdn.net/lmj623565791/article/details/45303349)
- [官方教程](http://developer.android.com/design/patterns/actionbar.html#ActionButtons)