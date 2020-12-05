---
title: 每日java题
date: 2017-09-13 19:51:59
tags: java
---
一切程序语言都离不开最基本的基础，虽然这篇或许看着会很基础，不过平时空闲下来回过头来看看也还是有必要的，时间久了也可能会忘记了<!--more-->

---
### java
* & 与 && 的区别：
> & 位运算符， && 逻辑运算符，逻辑“与”

* error与exception的区别：
> error：表示恢复不是不可能的一种严重的问题，例如内存溢出
> exception：表示程序运行时异常

* 线程同步的方法：
> wait()：使一个线程处于等待状态，并且释放所持有的对象的lock
> sleeo()：使一个正在运行的线程处于睡眠状态。
> notify()：唤醒一个处于等待状态的线程，注意的是在调用此方法的时候并不能确切的唤醒一个等待状态的线程，而是由JVM确定唤醒哪个线程，而且不是按优先级
> notifyAll()：唤醒所有处于等待状态的线程，注意不是给所有唤醒的线程一个对象，而是竞争

* java基本数据类型及所占位数：
> 逻辑型：boolean(false/true 1byte)
> 文本型：char(2byte)
> 整数型：byte(1byte), short(2byte), int(4byte), long(8byte)
> 浮点数型：float(4byte), double(8byte)

* HashMap和HashTable的区别：
> 1.HashMap允许空的键值对，HashTable不允许
> 2.HashMap不是线程安全的，HashTable线程安全
> 3.HashMap直接实现Map接口，HashTable集成Dictionary类

* Jap的四种范围
> page：代表与一个页面相关的对象和属性，作用域在当前页面
> request：代表与web客户机发出的一个请求相关的对象或属性
> session：只要浏览器不关闭，作用域一直存在
> application：只要访问的服务器不关闭，作用域一直存在

* ArrayList, Vector, LinkedList存储性能和特性：
> ArrayList和Vector都是基于数组实现的
> LinkedList是基于双向循环链表实现的（查询效率低，添加删除效率高）
> ArrayList不是线程安全的，而Vector是线程安全的，所以速度上ArrayList高于Vector

* final、finally和finalize的区别：
> final: 用于声明属性，方法和类，子代不可改变
> finally: 是异常处理的一部分，不管是否有异常都会执行
> finalize: 是Object的一个方法，在垃圾收集器执行的时候会调用被回收对象的方法，可以覆盖此方法提供垃圾收集时的其他资源，如关闭文件等

* 5个启动时异常
> RunTimeException
> |---NullPointerException
> |——ArrayIndexOutOfBoundsException
> |——ArithmeticException
> |——ClassCastException
> |——NumberFormatException


### 线程相关：
* 实现线程安全的两种方法：
> synchronized方法
> synchronized块，通过synchronized关键字来生命synchronized块

* sleep()和wait()的区别
> sleep是线程类的方法，让线程暂停执行指定时间，给其他线程执行机会，但监控状态保持，到时候会自动恢复。调用sleep不会释放对象锁
> wait是Object类的方法，会导致线程放弃对象锁，进入等待此对象的等待锁定池，使用notify()或notifyAll()方法来唤醒

### servlet相关：
* jsp与servlet的区别和联系：
> jsp是servlet的技术的扩展本质上讲就是servlet的简易方式。jsp编译后是“类servlet”。
> jsp和servlet的区别在于，servlet的应用逻辑实在java文件中，并且完全从表示层中的html里分离出来；而jsp的情况是java代码和html可以组合成一个扩展名为.jsp的文件。jsp侧重于视图，servlet侧重于逻辑

* servlet生命周期
> web容器加载servlet，生命周期开始
> init()方法进行servlet的初始化
> servive()方法，根据请求的不同调用不同的doGet()或doPost()
> destory()方法结束服务

* B/S结构和C/S结构
> C/S：Client/Server的缩写，桌面应用程序，服务器通常采用高性能的PC、工作站或小型机，并采用大型数据库系统
> B/S：Brower/Server的缩写，客户机上只要安装一个浏览器，服务器安装数据库，通过浏览器来访问

---
### SQL相关：
* 删除一张表的所有数据的方式：
> truncate table
> delete

* 用一条sql语句取出所有姓名有重复的学院姓名和重复记录数：
```
select name, count(*) from student
group by name
having count(*)>1
order by count(*) desc
```
