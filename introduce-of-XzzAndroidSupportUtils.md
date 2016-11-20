title: introduce of XzzAndroidSupportUtils
date: 2016-01-13 21:05:48
tags: [android]
categories: [android,IDE]
---

<h3>介绍：</h3>
本项目将自己以前使用过的一些常用的工具类提取出来组成一个Library module，下次导入可以直接使用，避免重复造轮子。
<h3>功能：</h3>
这个类库主要包含以下内容：

- BaseActivity和ActivityCollector，可用于直接关闭所有Activity
- 获取App version的工具
- 度量转换工具
- 图片资源转换工具
- 日志工具
- 网络检测工具
- SharedPreference工具
<h3>用法：</h3>
	
1. 查看当前是哪个Activity  

	>只要继承BaseActivity就会在日志里边打印当前的activity 
2. 一次关闭所有的Activity

	>首先，需要继承BaseActivity  
	然后，可以调用ActivityCollector的finishAll方法结束所有的Activity
3. 获取当前App版本  

	>AppVersion.getAppVersion()
4. dp转换为px  
	>DimensUtil.dp2px()

5. Bitmap转换为Drawable
	>ImageUtil.bitmap2Drawable
6. 日志工具

	1. LogUtil.setTag（）设置tag  
	2. LogUtil.v();  
		LogUtil.d();  
		LogUtil.i();  
		LogUtil.w();  
		LogUtil.e();
7. 网络状态监测
	>NetUtil.isNetworkConnected

8. SharedPreference工具
