---
title: springMVC+dubbo+zookeeper
date: 2017-07-31 19:16:58
tags: java
---
> Dubbo是阿里的一个支持RPC服务的框架，最大的特点是按照分层的方式来架构，可以使各个层之间解耦和。因为公司的项目中有涉及到Dubbo来做，所以自己就借此机会看了看Dubbo。不过[github上的Dubbo](https://github.com/alibaba/dubbo)一年前就已经停止更新了，里面有全部的dubbo，有后台、有demo，不过既然停止更新了我们都不能保证以后不会出现问题，说不定哪天就停止运行了呢。好了，接下来看来看看具体的吧。<!--more-->

---
dubbo的背景啥的这里就不说了，感兴趣的可以去百度看看，来看看dubbo的整体简介吧：
<img src="/images/java/2017073101.png" style="width: 400px;"/>
Provider: 暴露服务的服务提供方。
Consumer: 调用远程服务的服务消费方。
Registry: 服务注册与发现的注册中心。
Monitor: 统计服务的调用次调和调用时间的监控中心。
Container: 服务运行容器

调用关系说明：
0. 服务容器负责启动，加载，运行服务提供者。
1. 服务提供者在启动时，向注册中心注册自己提供的服务。
2. 服务消费者在启动时，向注册中心订阅自己所需的服务。
3. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
4. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
5. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心

准备：（[dubbo文档](http://dubbo.io)）
> 搭建好springMVC+maven框架
> 下载zookeeper，并启动（启动文件：/bin/zkServer.cmd）
> 下载dubbo后台控制中心，将dubbo后台部署到tomcat中并启动（前提是zookeeper启动）
> 用户名、密码：root, root
> <img src="/images/java/2017073102.png" style="width: 400px;"/>

dubbo消息提供者：
目录结构：
<img src="/images/java/2017073103.jpg" style="width: 400px;"/>
* maven pom文件中加载进dubbo所需的jar包：dubbo、zookeeper、zkclient:
```
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>dubbo</artifactId>
  <version>2.5.3</version>
</dependency>
<dependency>
  <groupId>org.apache.zookeeper</groupId>
  <artifactId>zookeeper</artifactId>
  <version>3.4.8</version>
</dependency>
<dependency>
  <groupId>com.github.sgroschupf</groupId>
  <artifactId>zkclient</artifactId>
  <version>0.1</version>
</dependency>
```
* 写一个service来提供服务：TestService.java:
```
public interface TestService {
    String say(String name);
}
```
TestServiceImpl.java:
```
@Service("testService")
public class TestServiceImpl implements TestService {
    @Override
    public String say(String name) {
        return "hello: "+name;
    }
}
```
其实服务提供方就是想外界提供一个可用的服务接口，我们现在这个简单的服务提供就已经完成，接下来就是暴露这个服务：
* 新建一个dubbo.xml：
```
<!-- 提供方应用名称信息，这个相当于起一个名字，我们dubbo管理页面比较清晰是哪个应用暴露出来的 -->
<dubbo:application name="dubbo_provider"></dubbo:application>
<!-- 使用zookeeper注册中心暴露服务地址 -->
<dubbo:registry address="zookeeper://127.0.0.1:2181" check="false" subscribe="false" register=""></dubbo:registry>
<!-- 要暴露的服务接口 -->
<!--没有这个bean会报错-->
<bean id="testService" class="com.yif.ssm.service.impl.TestServiceImpl"/>
<dubbo:service interface="com.yif.ssm.service.TestService" ref="testService" />
```
注：这里的<bean id="testService" class="com.yif.ssm.service.impl.TestServiceImpl"/>不加会报一个bean未注入异常，还没解决
* 将dubbo.xml关联到application.xml里，启动程序：
<img src="/images/java/2017073104.jpg" style="width: 400px;"/>
可以看到我们的服务已经暴露出来了
