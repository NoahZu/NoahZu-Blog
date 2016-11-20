title: Android中对象的序列化
date: 2016-07-20 22:34:34
tags: [android,序列化]
categories: 《android开发艺术探索》笔记
---
介绍Android中两种序列化的方式。
<!--more-->

<h3>序列化的作用</h3>
- 通过Intent或者Binder传输数据需要序列化
- 实现对象的持久化
- 对象的网络传输

<h3>序列化的两种方式</h3>
<h4>Serializable接口</h4>
利用Serializable实现序列化只需要实现Serializable接口，并声明一个serialVersionUID即可，serialVersionUID不是必须的，后面说他的作用。下面是序列化和反序列化一个对象的代码：

	//user对象
	public class User implements Serializable{
		private String name;
		private int id;
		
		public User(String name,int id){
			this.name = name;
			this.id = id;
		}
	}

	//序列化
	User user = new User("John",1);
	ObjectOutputStream out  = new ObjectOutputStream(new FileOutputStream("user.txt"));
	out.writeObject(user);
	out.close();
	//反序列化
	ObjectInputStream in = new ObjectInputStream(new FileInputStream("user.txt"));
	User user = (User)in.readObject();
	in.close();

serialVersionUID的作用：
当序列化的时候，会把serialVersionUID写入到序列化文件中，当反序列化的时候，只有当读出的serialVersionUID和当前类一样，才能反序列化成功，所有serialVersionUID的作用便是，确保反序列化出来的类和当前是同一个类。
<h4>Parcelable接口</h4>

	public class User implements Parcelable {
    private String name;
    private int id;

    public User(String name,int id){
        this.name = name;
        this.id = id;
    }

    @Override
    public int describeContents() {
        return 0;//一般是0
    }

    @Override
    public void writeToParcel(Parcel parcel, int i) {
        parcel.writeInt(id);
        parcel.writeString(name);
    }
    
    public static final Parcelable.Creator<User> CREATOR = new Parcelable.Creator<User>(){

        @Override
        public User createFromParcel(Parcel parcel) {
            return new User(parcel.readString(),parcel.readInt());
        }

        @Override
        public User[] newArray(int size) {
            return new User[size];
        }
    };
}

观察上面的代码可以总结出：
序列化由writeToParcel来完成，反序列化由Creator来完成。

<h3>两种方式的比较</h3>
Serializable：

- Java提供的序列化
- 使用简单
- 需要开销比较大

Parcelable：
- Android提供的方式
- 使用麻烦
- 效率较高