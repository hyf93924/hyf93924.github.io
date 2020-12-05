---
title: redis实现二级缓存
date: 2017-08-22 16:49:19
tags: java
---
> 缓存的作用就是降低数据库的使用率，来减轻数据库的负担。我们平常的操作一般都是查>改，所以数据库的有些查操作是重复的，如果一直使用数据库就会有负担。Mybatis也会做缓存，也会有一级缓存和二级缓存：<!--more-->
> * 一级缓存：是SqlSession级别的缓存，使用HashMap数据结构来用于存储缓存数据的
> * 二级缓存：是mapper级别的缓存，其作用域是mapper的同一个namespace，不同的SqlSession执行两次相同namespace下的sql语句，并且传递的参数相同，返回的结果也相同时，第一次执行sql语句是会将数据从数据库中取出并存入缓存中，第二次查询时便会从缓存中直接获取数据
> 二级缓存的实现大大的降低了数据库的负担，这里就来实现以下使用redis实现Mybatis的二级缓存。

# 准备：
> 我这里使用springboot快速搭建的项目[>>>直通<<<](http://heyif.win/2017/08/03/springboot%E5%BF%AB%E9%80%9F%E6%90%AD%E5%BB%BA/)，基本的环境如下：
> jdk 1.7+
> springboot maven mybatis项目
> redis
> mysql

# 二级缓存的实现
## 概述
redis二级缓存的实现，主要是重写了Cache.java的方法，自定义缓存，先来看看Cache的方法有哪些：
```
String getId();
void putObject(Object var1, Object var2);
Object getObject(Object var1);
Object removeObject(Object var1);
void clear();
int getSize();
ReadWriteLock getReadWriteLock();
```
这里有put、get、clear等方法，将在我们自己的缓存方法中重写这些方法

## 自定义缓存方法
```
public class MybatisRedisCache implements Cache {

    private static final Logger logger = LoggerFactory.getLogger(MybatisRedisCache.class);
    private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    private RedisTemplate redisTemplate;
    private String id;
    //redis过期时间
    private static final long EXPIRE_TIME_IN_MINUTES = 30;


    public MybatisRedisCache(String id){
        if (id==null){
            throw new IllegalArgumentException("Cache instances require an ID");
        }
        logger.info("=====================================Redis cache id = "+id);
        this.id = id;
    }

    @Override
    public String getId() {
        return id;
    }

    @Override
    public void putObject(Object key, Object value) {
        logger.debug("==============================redis put= "+key);
        RedisTemplate redisTemplate = getRedisTemplate();
        ValueOperations opsForValue = redisTemplate.opsForValue();
        opsForValue.set(key, value, EXPIRE_TIME_IN_MINUTES, TimeUnit.MINUTES);
    }

    @Override
    public Object getObject(Object key) {
        logger.debug("================================redis get================================");
        RedisTemplate redisTemplate = getRedisTemplate();
        ValueOperations opsForValue = redisTemplate.opsForValue();
        return opsForValue.get(key);
    }

    @Override
    public Object removeObject(Object key) {
        logger.debug("==========================================redis remove==========================");
        RedisTemplate redisTemplate = getRedisTemplate();
        redisTemplate.delete(key);
        return null;
    }

    @Override
    public void clear() {
        logger.debug("=====================================clear redis================================");
        RedisTemplate redisTemplate = getRedisTemplate();
        redisTemplate.execute(new RedisCallback() {
            @Override
            public Object doInRedis(RedisConnection connection) throws DataAccessException {
                connection.flushDb();
                return "OK";
            }
        });
    }

    @Override
    public int getSize() {
        return 0;
    }

    @Override
    public ReadWriteLock getReadWriteLock() {
        return readWriteLock;
    }

    public RedisTemplate getRedisTemplate() {
        if (redisTemplate == null) {
            redisTemplate = ApplicationContextHolder.getBean("redisTemplate");
        }
        return redisTemplate;
    }

    public void setRedisTemplate(RedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

}
```

这里需要用到redisTemplate，一开始我们初始化的是空的redisTemplate，这样的话redisTemplate.opsForValue()就是一个空指针异常，所以我们这里要getBean来获取，所以这里有一个ApplicationContextHolder工具类。

ApplicationContextHolder.java:
```
@Component
public class ApplicationContextHolder implements ApplicationContextAware {
    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext ctx) throws BeansException {
        applicationContext = ctx;
    }

    /**
     * Get application context from everywhere
     *
     * @return
     */
    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }

    /**
     * Get bean by class
     *
     * @param clazz
     * @param <T>
     * @return
     */
    public static <T> T getBean(Class<T> clazz) {
        return applicationContext.getBean(clazz);
    }

    /**
     * Get bean by class name
     *
     * @param name
     * @param <T>
     * @return
     */
    @SuppressWarnings("unchecked")
    public static <T> T getBean(String name) {
        return (T) applicationContext.getBean(name);
    }
}
```

# 二级缓存的使用
在mapper中添加自定义cache：
```
<cache type="com.yif.utils.MybatisRedisCache" eviction="LRU"/>
```
其中的eviction有4个不同的参数：
> LRU ：最近最少使用的：移除最长时间不被使用的对象
> FIFO：先进先出：按进入缓存的顺序来移除他们
> SOFT：软引用：移除基于垃圾回收器状态和软引用规则的对象
> WEAK：弱引用：更积极的移除基于垃圾收集器状态和弱引用规则的对象

## 测试
测试前先在数据库中插入几条数据，用户测试二级缓存，其实二级缓存就是在原先完整的数据库操作程序中添加了一个自定义cache并在数据库mapper中引入使用。
1.启动redis，查看redis中的keys，可以看到现在是空的
<img src="/images/java/2017082201.jpg" style="width: 400px;"/>
2.启动程序，首先会运行MybatisRedisCache中的构造函数，在日志中看到id=com.yif.redis.model.User，即你的实例

3.运行查询步骤，MybatisRedisCache的运行顺序是：
第一次查询：
getObject()-->putObject()-->数据库查询
<img src="/images/java/2017082203.jpg" style="width: 400px;"/>
再看看redis中的keys，可以看出已经将存入了一个新的key
<img src="/images/java/2017082205.jpg" style="width: 400px;"/>
第二次相同的查询：
getObject()
<img src="/images/java/2017082204.jpg" style="width: 400px;"/>
这样就可以看出第一次查询是从数据库中查出然后将结果存入缓存中，第二次相同的查询是从缓存中就查出了，不再通过数据库查询。

## 注意：
在mapper文件中，主要两个参数：flushCache和useCache:
> 在select中，flushCache默认是false，不会清除本地缓存和二级缓存；useCache默认为true，表示进行二级缓存
> 在update、insert和delete中，flushCache默认为true，会清空本地缓存和二级缓存；而useCache在此是不存在的

所以在update语句中加上flushCache后更新数据库会清除redis的二级缓存，这样在进行下一次查询时，从redis中获取的便是自动更新后的数据（会将数据库的数据put到redis中），这里还有一个情况，也是自己遇到的。网上有很多资料写着MybatisRedisCache中的clear()方法是无用的，所以很多都没有写，这里我也没写，不过这样的话更新后就不会自动更新，所以自己又写了clear()方法，在数据库更新之后便会执行这个clear()方法。