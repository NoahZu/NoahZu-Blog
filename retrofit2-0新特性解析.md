title: retrofit2.0新特性解析
date: 2015-12-01 13:22:21
tags: [android,retrofit]
categories: android
---
译者注：原文链接：http://inthecheesefactory.com/blog/retrofit-2.0/en 第一次翻译一篇文章，只是因为觉得这篇文章帮助认识Retrofit2.0挺不错的。翻译的不好的地方，还请多多指正，谢谢。  
<!--more-->
>Retrofit是最流行的Android网络请求库之一，因为它与其他类库比起来更加简洁，但是功能却更加强劲。但是它的缺点是在Retrofit 1.x.的版本中没有一个直接的方法来取消正在进行的事务。如果你想那么做，你不得不在开一个线程调用它，然后手动结束他。那样非常难以管理。Square几年前曾经许诺这个特性将会出现在Retrofit 2.0中，但是，几年过去了，仍旧没有任何这方面更新的消息。  

>直到上周，Retrofit2.0 升级到Beta1并且公开发布了！经过尝试后，我不得不说，我被它的新特性和新模式感动了。新版本有很多地方的改进之处，下面我将在这片文章中说一下这些不同之处，让我们开始吧。
  
<h4>新版本中的包（package）与旧版本保持一致</h4>
>&emsp;如果你想在你的项目中导入Retrofit2.0，添加这句到你的`build.gradle`文件中的`dependencies`部分：

```
compile 'com.squareup.retrofit:retrofit:2.0.0-beta1'
```

<h4>新的服务的，没有更多的synchronous和Asynchronous方法</h4>

>在1.9版本中，如果你想声明一个同步（synchronous ）方法，你需要向下面这样来声明：


	/* Synchronous in Retrofit 1.9 */
	 
	public interface APIService {
	 
	    @POST("/list")
	    Repo loadRepo();
	 
	}

>如果你想声明一个异步方法：

	/* Asynchronous in Retrofit 1.9 */
	 
	public interface APIService {
	 
	    @POST("/list")
	    void loadRepo(Callback<Repo> cb);
	 
	}
>但是在2.0中，简单到你只需要声明一个方法

	import retrofit.Call;
	 
	/* Retrofit 2.0 */
	 
	public interface APIService {
	 
	    @POST("/list")
	    Call<Repo> loadRepo();
	 
	}
	
>创建service的方式也变成了相同的方式okhttp，如果想调用同步的请求，只需调用execute方法，而enqueue则会调用一个异步请求。


<h4>同步请求</h4>

	// Synchronous Call in Retrofit 2.0
	 
	Call<Repo> call = service.loadRepo();
	Repo repo = call.execute();

>上面的代码会阻塞线程，所以在Android里边不能在主线程中调用它，否则会抛出`NetworkOnMainThreadException`异常，如果你想非要调用这个方法，必须在后台线程中调用它。  


<h4>异步请求</h4>
	
	
	// Synchronous Call in Retrofit 2.0
	 
	Call<Repo> call = service.loadRepo();
	call.enqueue(new Callback<Repo>() {
	    @Override
	    public void onResponse(Response<Repo> response) {
	        // Get result Repo from response.body()
	    }
	 
	    @Override
	    public void onFailure(Throwable t) {
	 
	    }
	});
	

>上面的代码将会在后台线程中发送请求，并且将结果在回调接口中返回一个Response，你请求的数据可以在Response中获得。需要注意的是，onResponse和onFailure方法将会在主线程中调用。我建议你用enqueue这个方法他最适合Android系统的特点。
<h4>正在进行的事务的取消</h4>

```
call.cancel();
```
事务将会被直接取消，是不是很简单。

<h4>当新的服务创建的时候，Converter从Retrofit中移除了。</h4>

>在Retrofit1.9中，GsonConverter是默认包含在包里面的并且会在RestAdapter创建的时候自动创建。结果就是，如果返回的是Json字符串将会自动地被转换为数据存取对象Data Access Object（DAO）。    

>但是在Retrofit2.0中，Converter将不会被包含在包中了。你可以自己插入一个Converter或者不插入Retrofit将只能接受字符串结果。因此，retrofit2.0不再依赖于Gson。  

>如果你想接收json字符串并将它转换为DAO，你必须将Gson Converter作为一个单独的依赖库添加进来。

```
compile 'com.squareup.retrofit:converter-gson:2.0.0-beta1'
```
>然后通过`addConverterFactory`方法添加到Retrofit。需要注意的是`RestAdapter`现在也被改名为`Retrofit`.


	Retrofit retrofit = new Retrofit.Builder()
	        .baseUrl("http://api.nuuneoi.com/base/")
	        .addConverterFactory(GsonConverterFactory.create())
	        .build();
	 
	service = retrofit.create(APIService.class);

>这里是一个由Square提供的Convert modules的清单，根据你的需求选择一个最合适的。  

	Gson: com.squareup.retrofit:converter-gson  
	Jackson: com.squareup.retrofit:converter-jackson  
	Moshi: com.squareup.retrofit:converter-moshi  
	Protobuf: com.squareup.retrofit:converter-protobuf  
	Wire: com.squareup.retrofit:converter-wire  
	Simple XML: com.squareup.retrofit:converter-simplexml



>你也可以创建一个自定义的Converter通过实现`Converter.Factory `接口。我比较提倡这种新的方式，它会让Retrofit更加清楚到底在做什么。
<h4>自定义的Gson对象</h4>
>假如你需要调整json里面的一些格式。比如Date格式。你可以通过创建Gson对象然后把它传给`GsonConverterFactory.create()`


	Gson gson = new GsonBuilder()
	        .setDateFormat("yyyy-MM-dd'T'HH:mm:ssZ")
	        .create();
	 
	Retrofit retrofit = new Retrofit.Builder()
	        .baseUrl("http://api.nuuneoi.com/base/")
	        .addConverterFactory(GsonConverterFactory.create(gson))
	        .build();
	 
	service = retrofit.create(APIService.class);

####新的URL分解观念，与```<a href>```一样####

>Retrofit2.0带来了新的URL分解方式，Base URL和@URL不能再像以前一样简单的拼接起来了，但是他和```<a href="...">```的组合方式是一样的。请看下边的使用样例来进一步了解：



![](http://inthecheesefactory.com/uploads/source/blog/retrofit2/apiservice1.png)
![](http://inthecheesefactory.com/uploads/source/blog/retrofit2/apiservice2.png)
![](http://inthecheesefactory.com/uploads/source/blog/retrofit2/apiservice3.png)
  
下面是我个人的使用建议：

- Base Url：总是以/结尾
- @Url 不要以/开始

举个例子：

	public interface APIService {
	 
	    @POST("user/list")
	    Call<Users> loadUsers();
	 
	}
	 
	public void doSomething() {
	    Retrofit retrofit = new Retrofit.Builder()
	            .baseUrl("http://api.nuuneoi.com/base/")
	            .addConverterFactory(GsonConverterFactory.create())
	            .build();
	 
	    APIService service = retrofit.create(APIService.class);
	}

>上述代码中的loadUser将会从```http://api.nuuneoi.com/base/user/list```获取数据
>此外，在Retrofit2.0中我们还可以在@Url中写完整的Url

	public interface APIService {
	 
	    @POST("http://api.nuuneoi.com/special/user/list")
	    Call<Users> loadSpecialUsers();
	 
	}
这种情况下，Base Url将会被忽视。

>也许你已经发现，在URL拼接这块Retrofit2.0确实有很大的改变，它与之前的方式是完全不同的。如果你想把你的代码中Retrofit升级到2.0，那么别忘了修改一下Url部分的代码哦~

<h4>OkHttp现在是必须的</h4>
>在1.9版本中，OkHttp是可选择的，如果你想Retrofit使用OkHttp作为HTTP连接的客户端，你可以手动引入OkHttp。
>但是在Retrofit2.0中，OkHttp是必须的，他将会自动作为Retrofit的客户端，下面的代码节选自Retrofit2.0的pom文件，由于是自动添加的，所以你自己不必做任何操作。

	<dependencies>
	  <dependency>
	    <groupId>com.squareup.okhttp</groupId>
	    <artifactId>okhttp</artifactId>
	  </dependency>
	 
	  ...
	</dependencies>
>OkHttp is automatically used as a HTTP interface in Retrofit 2.0 in purpose to enabling the OkHttp's Call pattern as decribed above.

<h4>即使Response有问题，onResponse仍旧会被调用</h4>
>在Retrofit1.9中，如果获取到的数据不能够被解析成我们定义的实体类，```failure```将会被调用。但是在Retrofit2.0中，不管response能否被成功解析，onResponse总是会被调用。但是，由于结果不能被解析成实体类，`response.body()`将会返回null。不要忘记处理这种情况哦~
>不管response有任何错误，比如404 not found。OnResponse也将会被调用，你可以从```response.errorBody().string()```取出错误信息。

![](http://inthecheesefactory.com/uploads/source/blog/retrofit2/response.jpg)

Response/Failure的处理逻辑与1.9有很大的不同。如果你想从1.9升级为2.0，那么在处理这些情况的时候要注意了。

<h4>没添加网络权限引起的SecurityException异常将不复存在</h4>
>在Retrofit1.9中，如果你忘记了在`AndroidManifest.xml`添加`INTERNET `权限，异步请求将会由于没有网络权限而立即失败调用，并回调到`failure`,不会有异常抛出。
>但是在Retrofit2.0中，当你执行`call.enqueue`或者`call.execute`，将会立即抛出异常`SecurityException `，如果你没有用`try-catch`来处理的话，将会崩溃。
![](http://inthecheesefactory.com/uploads/source/blog/retrofit2/sec.png)
>就跟你没加网络权限而去调用`HttpURLConnection`一样的错误。总之，如果你老老实实的在`AndroidManifest.xml`添加上网络权限的话，这里没有什么值得注意的。

<h4>使用来自Okhttp的拦截器</h4>
>在Retrofit1.9 你可以使用`RequestInterceptor`来拦截一个请求，但是自从Retrofit2.0使用了OkHttp以后他就被移除了。
>因此，从现在开始，我们就必须使用来自OkHttp的`Interceptor `来作为拦截器了。第一步，你首先创建一个带有拦截器的OkHttpClient对象：

	OkHttpClient client = new OkHttpClient();
	client.interceptors().add(new Interceptor() {
	    @Override
	    public Response intercept(Chain chain) throws IOException {
	        Response response = chain.proceed(chain.request());
	 
	        // Do anything with response here
	 
	        return response;
	    }
	});
>将我们刚才创建的`client`添加到Retrofit的 Builder链里边：

	Retrofit retrofit = new Retrofit.Builder()
	        .baseUrl("http://api.nuuneoi.com/base/")
	        .addConverterFactory(GsonConverterFactory.create())
	        .client(client)
	        .build();

这样就好了。
如果你想了解更多关于`OkHttp Interceptor`可以作什么，请浏览[OkHttp Interceptors](https://github.com/square/okhttp/wiki/Interceptors)

<h4>添加授权</h4>
>与Interceptor一样，如果你想在你的Connection中添加授权的信息，那么你也需要创建一个OkHttp Client的实例。下面是示例代码：

	OkHttpClient client = new OkHttpClient.Builder()
	        .certificatePinner(new CertificatePinner.Builder()
	                .add("publicobject.com", "sha1/DmxUShsZuNiqPQsX2Oi9uv2sCnw=")
	                .add("publicobject.com", "sha1/SXxoaOSEzPC6BgGmxAt/EAcsajw=")
	                .add("publicobject.com", "sha1/blhOM3W9V/bVQhsWAcLYwPU6n24=")
	                .add("publicobject.com", "sha1/T5x9IXmcrQ7YuQxXnxoCmeeQ84c=")
	                .build())
	        .build();

	Retrofit retrofit = new Retrofit.Builder()
	        .baseUrl("http://api.nuuneoi.com/base/")
	        .addConverterFactory(GsonConverterFactory.create())
	        .client(client)
	        .build();

>如果想了解跟多关于sha1 授权的信息，google一下...

<h4>将CallAdapter与RxJava结合使用</h4>

>除了声明`Call<T>`接口以外，我们还可以声明我们自己的接口类型，比如`MyCall<T>`。手动调用`"CallAdapter"`在Retrofit2.0是可用的。
>有一些现成的来自Retrofit Team的CallAdapter模板可以供我们使用。其中一个最流行的就是CallAdapter for RxJava 了，他返回的是`Observable<T>`。要使用他，下面这两个模板必须导入：

	compile 'com.squareup.retrofit:adapter-rxjava:2.0.0-beta2'
	compile 'io.reactivex:rxandroid:1.0.1'
Sync Gradle并且添加`addCallAdapterFactory`在Retrofit的创建；链中，就像这样：

	Retrofit retrofit = new Retrofit.Builder()
	        .baseUrl("http://api.nuuneoi.com/base/")
	        .addConverterFactory(GsonConverterFactory.create())
	        .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
	        .build();

>你的请求接口就可以使用RxJava了，以为他返回的是`Observable<T>`
	public interface APIService {
	 
	    @POST("list")
	    Call<DessertItemCollectionDao> loadDessertList();
	 
	    @POST("list")
	    Observable<DessertItemCollectionDao> loadDessertListRx();
	 
	}

>可用于强制使用RxJava的地方。另外，如果你想让你的代码在subscribe部分的Main Thread中调用`observeOn(AndroidSchedulers.mainThread())`也需要添加到这个创建链中。

	Observable<DessertItemCollectionDao> observable = service.loadDessertListRx();
	 
	observable.subscribeOn(Schedulers.io())
	    .observeOn(AndroidSchedulers.mainThread())
	    .unsubscribeOn(Schedulers.io())
	    .subscribe(new Subscriber<DessertItemCollectionDao>() {
	        @Override
	        public void onCompleted() {
	            Toast.makeText(getApplicationContext(),
	                    "Completed",
	                    Toast.LENGTH_SHORT)
	                .show();
	        }
	 
	        @Override
	        public void onError(Throwable e) {
	            Toast.makeText(getApplicationContext(),
	                    e.getMessage(),
	                    Toast.LENGTH_SHORT)
	                .show();
	        }
	 
	        @Override
	        public void onNext(DessertItemCollectionDao dessertItemCollectionDao) {
	            Toast.makeText(getApplicationContext(),
	                    dessertItemCollectionDao.getData().get(0).getName(),
	                    Toast.LENGTH_SHORT)
	                .show();
	        }
	    });

完成了，我相信喜欢RxJava的人一定对此非常满意。

<h4>结论</h4>
>还有其他许多改变，你可以在官方的[Change Log](https://github.com/square/retrofit/blob/master/CHANGELOG.md)中查看。总是，我相信这篇文章覆盖了大多数的改变。
>你可能会疑惑是否应该升级到Retrofit2.0，因为他现在还是在beta版本，你可能想先用着1.9版本，但也不排除你像我一样迫不及待的要使用新的版本。Retrofit2.0用起来真的不错，在我使用的过程中还没有发现什么bug。
>请注意，Retrofit1.9的官方文档早就从Square github 的网站去掉了。我建议你马上开始学习Retrofit2.0并且考虑在未来的一段时间将你的Retrofit升级到2.0