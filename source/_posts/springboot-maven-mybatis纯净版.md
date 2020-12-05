title: springboot+maven+mybatis纯净版
date: 2017-07-26 13:31:53
tags: java
---
> springboot是一个能让你快速搭建的一个框架，能让你省了很多以往搭建框架时繁琐的配置，因为很多相关的配置springboot都依赖下来了，今天就来记录一下用springboot+maven+mybitis搭建的纯净版框架。<!--more-->

---
* 首先要做的就是用maven来创建一个工程，再在这个工程上配置springboot和mybatis，我的整体的项目结构是这样的：
<img src="/images/java/springboot.jpg" style="width: 400px;"/>
下面就是使用maven的pom.xml来添加springboot的依赖关系：
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.yif.boot</groupId>
    <artifactId>xstock-payment</artifactId>
    <version>1.0</version>
    <packaging>war</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.4.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.0</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-freemarker</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.30</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>1.5.4.RELEASE</version>
                <configuration>
                    <fork>true</fork>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <profiles>
        <profile>
            <id>test</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <log.path>/home/logs</log.path>
                <spring.profiles.active>test</spring.profiles.active>
            </properties>
            <build>
                <resources>
                    <resource>
                        <directory>src/main/resources</directory>
                        <filtering>true</filtering>
                        <includes>
                            <include>**/*</include>
                        </includes>
                    </resource>
                </resources>
            </build>
        </profile>
        <profile>
            <id>prod</id>
            <properties>
                <log.path>/home/logs</log.path>
                <spring.profiles.active>prod</spring.profiles.active>
            </properties>
            <build>
                <resources>
                    <resource>
                        <directory>src/main/resources</directory>
                        <filtering>true</filtering>
                        <includes>
                            <include>**/*</include>
                        </includes>
                        <excludes>
                            <exclude>test-db.properties</exclude>
                        </excludes>
                    </resource>
                </resources>
            </build>
        </profile>
    </profiles>
</project>
```
这里已经将整个项目所需的都配置进去了，spring-boot-start-parent是构建springboot项目的一个很好的方法。
* springboot需要一个入口文件，通过运行入口文件就能直接启动项目，完成springboot的入口文件，存在于下面的目录中（java里）：
<img src="/images/java/boot-application.jpg" style="width: 400px;"/>
```
@SpringBootApplication(scanBasePackages = "com.yif.boot")
@PropertySource("classpath:config-${spring.profiles.active}.properties")
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(Application.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }

}
```
* 来配置端口，编码，数据库之类的东西，在application.properties中：
```
server.port=80
server.session-timeout=1800
server.context-path=/
server.tomcat.compression=off
server.tomcat.uri-encoding=ISO-8859-1

spring.jmx.enabled=false
spring.freemarker.cache=false

#datasource
datasource.tomcat.jndi-name=java:comp/env/jdbc/__user

#mybatis
mybatis.config-location=classpath:mybatis/configuration.xml
mybatis.mapper-locations=classpath:mybatis/tables/*.xml

#profile
spring.profiles.active=@spring.profiles.active@

#logging
logging.config=classpath:log4j2.xml
```
这个时候springboot的基本的本地配置就已经完了，如果不需要连接数据库，在配置文件中将datasource、mybatis的配置删除即可，然后运行Application.java的main方法，看到如图效果说明已经成功:
<img src="/images/java/boot-success.jpg" style="width: 400px;"/>
这个图案就是spring，不过具体为什么要弄这样也不清楚。
* 接下来就可以来配置数据库和mybatis了，数据库我们采用的是mysql，在pom文件我们已经将mysql和mybatis的依赖放进去了，在application.properties里也已经配置好了datasource和mybatis了。我们就添加一个有关数据连接的配置文件，就是上图的test-db.properties，配置的内容：
```
db.driverClassName=com.mysql.jdbc.Driver
db.url=jdbc:mysql://localhost:3306/__user
db.username=root
db.password=a123456
db.maxActive=3
db.maxIdle=3
db.maxWait=1000
```
内容包括数据库驱动、数据库连接地址、用户名、密码最大连接数、id长度和最大等待时间等
* 弄完上面的这些，我们需要将数据库的相关信息导入到代码中，所以我在java/下创了一个config，用于获取相关的资源：
```
@Configuration
@EnableAutoConfiguration
@PropertySource("classpath:test-db.properties")
@Profile({"test","dev"})
public class TestResourceConfig {
    @Autowired
    private Environment env;
    @Bean
    @Profile({"dev","test"})
    public DataSource dataSource() throws Exception {
        Properties props = new Properties();
        props.setProperty("driverClassName",env.getProperty("db.driverClassName"));
        props.setProperty("username",env.getProperty("db.username"));
        props.setProperty("password",env.getProperty("db.password"));
        props.setProperty("url",env.getProperty("db.url"));
        props.setProperty("maxActive",env.getProperty("db.maxActive"));
        props.setProperty("maxIdle",env.getProperty("db.maxIdle"));
        DataSource ds = BasicDataSourceFactory.createDataSource(props);
        return ds;
    }

}
```
数据库和mybatis方面也配置好了，接下来就可以来练习代码了，不过在测试的时候遇到了点小问题：
> 在dao层使用mybatis往数据库添加内容时，提示了一个异常：
```
Caused by: java.lang.IllegalArgumentException: Property 'sqlSessionFactory' or 'sqlSessionTemplate' are required
	at org.springframework.util.Assert.notNull(Assert.java:134)
	at org.mybatis.spring.support.SqlSessionDaoSupport.checkDaoConfig(SqlSessionDaoSupport.java:74)
	at org.springframework.dao.support.DaoSupport.afterPropertiesSet(DaoSupport.java:44)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1687)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1624)
	... 39 more
```
就是dao的注入失败，是什么原因呢，自己找了好久没找出来，因为之前自己用springmvc时是没有遇到问题的，问了别人才知道是因为在mybatis-spring-1.2.0.jar对sqlSessionFactory和sqlSessionTemplate注入方式进行了调节，不能自动注入了，所以会出现错误，在这里我用的是重写了SqlSessionDaoSupport里的checkDaoConfig方法，将注入在checkDaoCongfig时进行，然后dao继承这个SqlSessionDaoSupport，这样就解决了：
```
public class SqlSessionDaoSupport extends org.mybatis.spring.support.SqlSessionDaoSupport {

    @Autowired
    private SqlSessionFactory sqlSessionFactory;

    @Override
    protected  void checkDaoConfig(){
        setSqlSessionFactory(sqlSessionFactory);
        super.checkDaoConfig();
    }
}
```
当然后来查资料，网上的解决方法是另一种，就是在dao里使用@Autowired 注入：
```
public abstract class AbstractDao extends SqlSessionDaoSupport{  
  
    /** 
     * Autowired 必须要有 
     */  
    @Autowired  
    public void setSqlSessionFactory(SqlSessionFactory sqlSessionFactory){  
          
        super.setSqlSessionFactory(sqlSessionFactory);  
    }  
      
}  
```

---
springboot和maven的搭建就完成了，测试代码就不介绍了，如果你会搭建之后会发现确实这个省略了很多springmvc搭建的繁琐之处，不过如果刚开始学习那还是有点困难的，还是要积累啊。
[>>代码<<](https://github.com/hyf93924/springboot-demo)
