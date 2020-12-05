---
title: springboot快速搭建
date: 2017-08-03 16:39:26
tags: java
---
> 之前有写过[如何搭建springboot+maven框架](http://heyif.win/2017/07/26/springboot-maven-mybatis%E7%BA%AF%E5%87%80%E7%89%88/)，那是完全手工搭建的，今天来看看如何快速搭建springboot框架，突出在“快”上。<!--more-->

---

快速构建springboot项目，可以通过一个网站来快速创建，访问[http://start.spring.io/](http://start.spring.io/)，根据自己的配置参数来选择创建。
注意：点击“Switch to the full version.”来设置java版本和项目名等具体信息
创建完之后下载压缩包。

生成的目录结构如图：
<img src="/images/java/2017080301.jpg" style="width: 400px;"/>

* pom.xml中添加支持web的模块：
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<!--去除logback依赖包-->
	<exclusions>
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-logging</artifactId>
		</exclusion>
	</exclusions>
</dependency>
```
这里将默认的logging去除，因为我们将使用的是log4j2。

这个项目Application.java也是自动生成的，不过要注意的是Application.java所处的位置要处于项目的最完成，然后它去包含其他模块。比如你的项目的包是com.yif.XX，你的Application.java因处于com.yif包下（一开始就是因为这个问题，所以项目一直报错）

启动Application.java就可以看到已经搭建成功了。

* 我们可以在application.properties中设置端口号等信息：
```
server.port=8099
server.session-timeout=1800
server.context-path=/
server.tomcat.compression=off
server.tomcat.uri-encoding=UTF-8
```

---
* springboot项目配置mybatis和mysql，其实配置和之前那篇是差不多的。
在pom.xml中添加相应的模块：
```
<!--mybatis-->
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	<version>1.3.0</version>
</dependency>

<!--mysql-->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>5.1.30</version>
</dependency>

<dependency>
	<groupId>commons-dbcp</groupId>
	<artifactId>commons-dbcp</artifactId>
	<version>1.4</version>
</dependency>
```

* 然后就是在主配置文件中配置mybatis的信息（mybatis配置文件位置，扫描文件位置）和数据库连接的信息:
```
#mybatis
mybatis.config-location=classpath:mybatis/configuration.xml
mybatis.mapper-locations=classpath:mybatis/tables/*.xml

#datasource
datasource.tomcat.jndi-name=java:comp/env/jdbc/__user

#mysql
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/__user
spring.datasource.username=root
spring.datasource.password=a123456
spring.datasource.maxActive=3
spring.datasource.maxIdle=3
spring.datasource.maxWait=100
```
这样就将mysql和mybatis配置完成了，然后在进行数据库相关操作时不要忘了“mybatis/configuration.xml”里的相关配置

---
* springboot将后台数据传到前端，推荐使用thymeleaf来代替jsp，就是不推荐想springmvc那样使用jsp：
在pom.xml文件中设置thymeleaf的模板：
```
<!--thymeleaf-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```
前后端传数据并没有发生什么改变，还是跟spring那样传。但在前端（html）中拿出数据用的标签是  th:  ：
```
<!DOCTYPE html>
<html xmlns:th="http://www.w3.org/1999/xhtml">
<head lang="en">
    <meta charset="UTF-8" />
    <title></title>
</head>
<body>
<h1 th:text="${host}">Hello World</h1>
</body>
</html>
```
html xmlns:th="http://www.w3.org/1999/xhtml" 这个就是标签库的地址。

---
好了，快速搭建springboot（+mysql+maven+mybatis）框架就这样，很轻松，因为给我们生成的就是一个大体的架构模板了，然后再根据我们所需要的再进行配置就行了。