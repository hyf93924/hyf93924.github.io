title: 小视java——字符串
date: 2017-07-21 23:04:23
tags: java
---
> 这些天看知乎上一个专栏有关java的那些事，写得一些java的基础知识，觉得很全面，挺深入的，所以将其转到自己的博客，我就是借鉴下来，将其命为《小视java》，当然这些基础的知识点，大牛们完全可以忽略，不过对自己还是有所帮助的。<!--more-->

---
现在先来看一段代码（代码例子来自知乎）：
```
public static TestString{
	public static void main(String[] args){
		String s1 = "100";
		String s2 = "200";
		System.out.println(s1 == s2);

		String s3 = new String("100");
		String s4 = new String("100")；
		System.out.println(s3 == s4);
	}
}
```
猜想一下结果是什么？运行的结果是一个true，一个false。为什么呢？
> 先来看看s1和s2，首先String s1="100";由于是第一次创建这个字符串，常量池里没有这个就会创建初始化该对象，并把引用指向它。而String s2="100"; 会发现常量池里已经存在"100"这个字符串，所以会直接引用而不会去创建。所以最后执行 == 操作时指向的是同一个引用，所以结果为true.
   
> 再来看看s3和s4，String s3=new String("100"); new就是直接开辟一个新的内存来创建字符串，而String s4=new String("100"); 也会去开辟一个新的内存去创建字符串，所以 s3==s4 因指向的不是同一个引用，所以为false.
   
---
结尾
> 比较两个字符串相等可以用 == 或者使用 equals，==比较的是是否来自同一引用，所以比较字符串尽量使用equals。s3.equals(s4)返回的就为true
