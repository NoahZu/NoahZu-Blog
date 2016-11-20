title: '《thinking in java》读书笔记-第15章:泛型(一)'
date: 2016-01-05 15:24:21
tags: [java]
categories: [《thinking in java》读书笔记]
---
java的泛型与C++的比较、简单泛型、泛型方法、泛型接口
<!--more-->
泛型有优点，也有其局限性
<h4>一. 与C++比较：</h4>
&nbsp;&nbsp;了解java中泛型的局限性，也就是他的边界。只有了解了边界所在才能够称为程序高手（原因是不必在死胡同打转）

<h4>二. 简单泛型：</h4>
&nbsp;&nbsp;泛型出现的原因之一就是为了创造容器类，泛型出现之前都是利用Object来存储不同的对象，当使用的时候在进行强制转换，这种做法是不安全的，无法保证强制转换的时候不出错。通常情况下，我们的容器中只存一种元素，泛型的主要目的之一就是指定容器要持有什么类型的对象，并且由编译器来保证类型的正确性。<br><br>
&nbsp;&nbsp;当使用的时候我们倾向于先不指定什么类型，然后稍后再决定使用什么类型，要达到这个目的，可以适用类型参数：需要使用的类型参数放到类的后边用<T>尖括号括起来，再类的内部定义的时候我们可以用类型参数T来替代，当实例化这个类的时候，在用实际的类型来替代。代码如下：

	public class Holder3<T> {
	     private T a;
	     public Holder3(T a){this.a = a ;}
	     public void set(T a ){ this.a = a ;}
	     public T get(){return this .a ;}
	     public static void main(String[] args) {
	          Holder3<Automobile> holder3 = new Holder3<Automobile>(new Automobile());
	           holder3.set( new Automobile());
	          Automobile automobile = holder3 .get();
	     }
	     static class Automobile{ }
	}







<h6>1.一个元组类库</h6>
&nbsp;&nbsp;&nbsp;元组：元组就是将一组对象直接打包存储进一个单一对象，这个容器只允许存，不允许放入新的对象（这个概念也成为数据传送对象或信使）。可以解决想在一个return语句中返回多个对象的问题。泛型可以让我们很容易的创建元组下面是利用泛型实现元组的一个实例：

	public class TwoTuple<A,B> {
	     public final A first ;
	     public final B second ;
	     public TwoTuple(A first,B second){
	           this.first = first ;
	           this.second = second ;
	     }
	     public String toString() { return first +"," +second ;}
	}
	
>说明：元组不可赋值，直接读取

<h6>2.一个堆栈类</h6>
利用泛型来实现自己的堆栈类：
	
	public class LinkedStack<T> {
	     private static class Node<U>{
	          U item;
	          Node<U> next;
	          Node() { item = null;next = null;}
	          Node(U item,Node<U> next){
	               this.item = item ;
	               this.next = next ;
	          }
	           boolean end(){return item == null && next == null;}
	     }
	     private Node<T> top = new Node<T>();
	     public void push(T item ){
	           top = new Node<T>(item ,top );
	     }
	     public T pop(){
	          T result = top. item;
	           if (!top .end()) {top = top.next;}
	           return result ;
	     }
	     public static void main(String[] args) {
	          LinkedStack<String> linkedStack = new LinkedStack<>();
	           for(String s : "I am smallzoo I love java".split(" ")){
	               linkedStack.push(s );
	          }
	          String s ;
	           while ((s = linkedStack .pop()) != null) {
	              System. out.println(s );
	          }
	     }
	}
>要注意末端哨兵的使用


<h4>三. 泛型接口</h4>
接口使用泛型和类使用泛型没有什么不同。下面展示一个泛型接口的应用：生成器

		 public interface Generator<T> {T next();}
		 public class Fibonacci implements Generator<Integer>{
	     private int count = 0;
	     private int fib(int n ){
	           if (n < 2){return 1;}
	           return fib(n -2)+fib(n -1);
	     }
	     @Override
	     public Integer next() {
	           return fib(count ++);
	     }
	     public static void main(String[] args) {
	          Fibonacci fibonacci = new Fibonacci();
	           for(int i = 0;i <100;i ++){
	              System. out.print(fibonacci .next()+" " );              
	          }
	     }
	}


<h4>四.泛型方法</h4>
泛型方法与泛型类无关，也就是说是否拥有泛型方法与所在的类是否是泛型类无关<br><br>
**基本原则**：如果使用泛型方法能够取代整个类泛型化，那么就应该只使用泛型方法<br>
**定义方法**：将泛型参数列表至于返回值之前


