title: Android中利用Gson解析Json
date: 2015-12-04 19:19:15
tags: [json,gson,android]
categories: android
---
在Android开发中，Json是一种客户端与服务器端交互的一种语言，它语法简单，最好的是目前市面上有很便捷的轮子可以对他进行解析。例如，Gson就是google提供的一款用于解析或者生成Json的库，可以直接将Json字符串映射成对应的实体类，十分方便。下面我总结一下利用Gson解析Json的用法以及我遇到的问题。

###最简单对象的解析：
例如下边这段Json字符串：
```
{
    text: "Love",
	img:"http://img2.imgtn.bdimg.com/it/u=2346639372,3797158284&fm=21&gp=0.jpg"
}  
```
如果我要解析它，可以直接定义一个对象，比如：

```
class LoveJson{
	String text;//这个名字要与json字符串中的相对应才能解析出来
	String img;
}
```
然后在需要解析的时候调用Gson来解析：
```
String jsonString;//待解析的json字符串
Gson gson = new Gson();
LoveJson love = gson.from(jsonString,Love.class);
```
短短几行就解析完了，有轮子就是如此方便。那么我们再看几种比较复杂的解析，最后，我再说下我遇到的问题以及解决方案。
###其他比较复杂的解析：
####嵌套类型的解析
	嵌套的Json类型无非就是这样的：
	

```
{
	school:"12级一班"
	student_number:54
	{
		student_name:"zhangsan"
		id:21
	}
}
```
那么解析的类应该这样写：

```
class SchoolJson{
	String school;
	String studentNumber;
	public static class Student{
		String studentName;
		int id;
	}
}
```
如果你有强迫症，变量非得用驼峰命名法，那么就用注解：
```
class SchoolJson{
	String school;
	@SerializedName("studen_number")
	String studentNumber;
	public static class Student{
		@SerializedName("student_name")
		String studentName;
		int id;
	}
}
```
####数组类型的解析
 也是很简单的，都是一个套路，比如：
 

```
[
	{
		student_name:"zhangsan"
		id:1
	},
	{
		student_name:"lisi"
		id:2
	},
	...
]
```

```
class Student{
	@SerializedName("student_name")
	String studentName;
	int id;
}
```
解析的时候可以这样：

```
Gson gson = new Gson();
Student[] students = gson.from(jsonString,Student[].class);
```
####解析Json的原则
以后可能会遇见更加复杂的，比如对象里边嵌套数组，数组里边嵌套对象，对象里边又嵌套数组...所以这里边有一个原则：
>当遇到对象也就是大括号的时候，写成一个对象，遇到数组的时候，可以定义成一个List<>然后再这个List<>下边定义一个表示数组元素的类（如果数组元素还是对象的话）。如此循环下去即可。
下面我举个稍微复杂点的例子：
json字符串：

```
{
    stories: [
        {
            images: [
              "http://pic1.zhimg.com/84dadf360399e0de406c133153fc4ab8_t.jpg"
            ],
            type: 0,
            id: 4239728,
            title: "前苏联监狱纹身百科图鉴"
        },
        {
	      "type": 0,
	      "id": 7086807,
	      "title": "职人介绍所 ·  自闭儿童的解锁人"
	    },
	    {
	      "images": [
	        "http://pic2.zhimg.com/afecdc04983a8e261326386995150599_t.jpg"
	      ],
	      "type": 0,
	      "id": 7066097,
	      "title": "家庭的生命周期：关于「离家」"
	    },
        ...
    ],
    description: "为你发现最有趣的新鲜事，建议在 WiFi 下查看",
    background: "http://pic1.zhimg.com/a5128188ed788005ad50840a42079c41.jpg",
    color: 8307764,
    name: "不许无聊",
    image: "http://pic3.zhimg.com/da1fcaf6a02d1223d130d5b106e828b9.jpg",
    editors: [
        {
            url: "http://www.zhihu.com/people/wezeit",
            bio: "微在 Wezeit 主编",
            id: 70,
            avatar: "http://pic4.zhimg.com/068311926_m.jpg",
            name: "益康糯米"
        },
        ...
    ],
    image_source: ""
}
```
用于解析的类可以这样写：

```
public class SubjectDailyContentJson{
    public List<Story> stories = new ArrayList<>();
    public static class Story implements Serializable{
        List<String> images;
        int type;
        int id;
        String title;
    }
    public String description;
    public String background;
    public int color;
    public String name;
    public String image;
    public List<Editor> editors;
    public static class Editor{
        String url;
        String bio;
        int id;
        String avator;
        String name;
    }
    @SerializedName("image_source")
    public String imageSource;
}

```
只要遵循那个原则，是不是很简单呢？可是最近我遇到一个问题令我很不开心，他的数组里的元素居然不是一样的样式，观察一下上边的那个json字符串，他的stories数组里边的元素的格式居然不是统一的，有的是这样：

```
{
            images: [
              "http://pic1.zhimg.com/84dadf360399e0de406c133153fc4ab8_t.jpg"
            ],
            type: 0,
            id: 4239728,
            title: "前苏联监狱纹身百科图鉴"
},
```
有的是这样：
```
	{
      "type": 0,
      "id": 7086807,
      "title": "职人介绍所 ·  自闭儿童的解锁人"
    }
```
那要怎么办呢，我蛋疼了一阵子，也终于找到了一个方法解决，他们两个的区别无非就是有没有images那个数组，那我干脆只要有的全写在我的类里边，然后将那些有可能没有的属性，我提前给他new一个值，这个样子的话，如果没有的话json解析成一个类时这个属性就为null，我们只要判断一下就可以了。你看，我那个images那个链表我就给他new了一个。

今天的Gson的用法的总结就到这里了，以后需要用到新的知识的话再做补充。