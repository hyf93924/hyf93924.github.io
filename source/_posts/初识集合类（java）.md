---
title: 初识集合类（java）
date: 2017-08-09 14:14:05
tags: java
---
现在一些公司面试，对于集合类的使用是比较看重的，也的确，我们平时用的也比较多，之前自己在这方面也只是会用，所以现在抽空来整理整理集合类的一些相关的内容。我们所熟知的java.util包中包含了一系列的集合类，掌握其内部结构是至关重要的。<!--more-->

### 老大“Collection接口”
Collection是最基本的集合类接口，一个Collection代表一个Object。一些Collection允许元素相同但另一些不允许，一些能够排序而有些不能排序。
所有实现Collection接口的类都必须有两个标准的构造函数：无参构造函数和有一个Collection参数的构造函数。无参构造函数用于创建一个空的Collection，有参数的构造函数用于创建一个新的Collection
主要的一个方法：boolean add(Object obj)

#### 使用iterator来实现遍历集合
Collection有一个iterator()方法来遍历集合中的所有元素，用法如下：
```
Iterator it = collection.iterator();  //获取一个迭代器
while(it.hasNext()){
	Object obj = it.next();  //得到下一个元素
}
```
要确保遍历的完成，必须确保只在一个线程中使用这个集合，或者在多线程中对遍历代码进行同步。
Collection派生出的两个非常重要的子接口：List和Set

#### List接口
List是一个有序的Collection，使用这个接口能够精确的控制每个元素插入的位置。用户可以通过索引来取出List里的元素，而且List允许元素重复（Set则不允许）。
除了Collection的iterator接口外，其自身还有listIterator方法：
<img src="/images/java/2017080901.jpg" style="width: 400px;"/>
listIterator比标准的iterator多了add()之类的方法，允许添加删除设定元素，还能向前向后遍历。
实现List接口的类有：ArrayList, LinkedList, Vector和Stack

##### ArrayList类（unsynchronized）
ArrayList实现了可变大小的数组。而且其允许null，不过ArrayList没有同步。
每个ArrayList实例都有一个容量，即用于存储元素的数组大小，这个容量可随着不断添加新元素而自动增大。
##### LinkedList类（unsynchronized）
LinkedList实现了List接口，是基于双向链表的实现（查询效率低，增加删除效率高），同样允许null，也没有同步。
如果多个线程同时访问一个List，则必须自己实现访问同步，可以在创建List时构造一个同步的List：
```
List list = Collections.synchornizedList(new LinkedList(...));
```
此外LinkedList还提供额外的get，remove，insert方法在LinkedList的首部或尾部。这些操作使LinkedList可被用作堆栈（stack），队列（queue）或双向队列（deque）。
##### Vector类（synchronized）
类似ArrayList，但是是同步的。注意的是虽然创建的Iterator和ArrayList创建的Iterator是同一个接口，但由于Vector是同步的，所以当一个Iterator被创建而且正在使用，另一个线程改变了Vector的状态（例如增加或删除），这时调用Iterator的方法时将抛出ConcurrentModificationException，因此必须捕获该异常。
##### Stack
继承自Vector，这个Stack在我大学的时候经常被拿来当口头禅，因为其先进后出的特性，其中的方法：基本的push和pop方法，还有peek方法得到栈顶的元素，empty方法测试堆栈是否为空，search方法检测一个元素在堆栈中的位置

#### Set接口
Set跟List不同，不能包含重复元素，所以可以用来存去重的一些元素，Set最多有一个null元素。
由于其不包含重复元素，所以任意两个元素e1和e2，都有e1.equals(e2)=false

---
### Map接口
注意Map接口没有继承Collection接口，Map提供的是K-V这样的键值对，一个Map中不能有重复的Key，每个Key只能映射一个Value。网上也有说Map接口其实是提供了3种集合的视图，Map的内容可以看成是一组Key集合，一组Value集合，或者一组Key-Value映射。

#### HashMap类（unsynchronized）
HashMap类是用的比较多的，非同步，允许null，即null value和null key。但是将HashMap是为Collection时（values()方法可返回Collection），其迭代开销时间和HashMap的容量成比例。
HashMap的默认容量是16，默认的加载因子是0.75，最大容量为2的30次方
<img src="/images/java/2017080902.jpg" style="width: 400px;"/>
#### HashTable类（synchronized）
HashTable实现一个key-value映射的哈希表，同步的。任何非空的（non-null）的对象都可以作为key和value。
其添加数据（put(k,v)）和取出数据（get(k,v)）时间开销都是常数
其任何作为key的对象都必须实现hashCode和equals方法。如果你自定义的类作为key的话，如果两个对象相同，即obj1.equals(obj2)=true，则他们的hashCode必须相同，但如果两个对象不同，其hashCode不一定不同，如果两个对象的hashCode相同，这种现象称为冲突。
如果相同的对象有不同的hashCode，对哈希表的操作会出现意想不到的结果（期待的get方法返回null），要避免这种问题，只需要牢记一条：要同时复写equals方法和hashCode方法，而不要只写其中一个。7190-10086