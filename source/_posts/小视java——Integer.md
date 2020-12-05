title: 小视java——Integer
date: 2017-07-21 23:27:24
tags: java
---
> 上篇写了有关String对比的一些事，用到了 == 和 equals 的对比，其实对比Integer类型的也有区别，来看看整型对比的差别的<!--more-->

---
首先还是先来看看一个例子（来自知乎）:
```
public static TestInteger{
	public static void main(String[] args){
		Integer i1 = 100;
		Integer i2 = 200;
		System.out.println(i1 == i2);

		Integer i3 = 1000;
		Integer i4 = 1000；
		System.out.println(i3 == i4);
	}
}
```
首先还是来猜想一下结果是什么？运行之后的结果依然是一个true，一个false。为什么呢？先别百度哦。
Integer在创建时为了避免重复创建，会对Integer的值进行缓存，如果这个值在缓存的范围之内，会直接返回缓存中的值，否则就会new出一个新的对象返回。接下来来看看代码里的值比较。
> i1和i2的值，分别为100，首先Integer的缓存范围是-128~127，而100在这个缓存范围之内，所以会取同一个数组里的对象返回，所以i1、i2指向的是同一个对象，所以i1==i2返回true

> 而i3、i4，值分别是1000，这个值的范围超出了缓存的范围，所以会新new出对象来返回，也就是i3和i4分别new了对象，指向的不是同一个对象，所以i3==i4返回的false

---
结尾
> 比较Integer跟String的差不多，==比较的还是是否来自同一对象，所以比较Integer也尽量使用equals来进行。
