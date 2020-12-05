---
title: Java百万级数据存入MySql
date: 2019-01-08 19:57:40
tags:
---
我又来了，又是百万数据的导入，其实又是项目中遇到的，那么怎么快速导入呢，因为我们项目用的是JPA，有时也使用jdbcTemplate，没错又是jdbcTemplate。对于这种大数据的处理，我们能业务代码方面最基本的能想到的就是块处理和使用线程。此处可以参考[JAVA向Mysql插入亿级别数据---测评](https://blog.csdn.net/q6834850/article/details/73726707)<!--more-->

---
首先由于我们项目用的是jpa，so我最先也使用的jpa来存数据，而且最先做的也是批量处理，一次存10000数据。在springboot1.5中，jpa的save有两个方法，先来看一下源码：
```
@NoRepositoryBean
public interface CrudRepository<T, ID extends Serializable> extends Repository<T, ID> {
    <S extends T> S save(S var1);
    <S extends T> Iterable<S> save(Iterable<S> var1);
}
```
可以看到save既可以保存一个实体，也可以保存多个实体，所以我选择的就是保存多个实体，结果发现save的速度贼慢，尤其是想存100000条数据时。而如果我save单个实体，并开启多线程，发现save的效率比批量处理要快点，我也是懵了。一开始就这样存80多万的数据用了N久，具体时间也没记录了。

然后自己就想着用最原生的方法来存一下，就是jdbc，同样一次存10000条，批量处理。

> 首先要注意的是：
> 1.在URL连接时需要开启批处理、以及预编译 
String url = “jdbc:mysql://localhost:3306/cds_credit_rtm_data?rewriteBatched 
-Statements=true&useServerPrepStmts=false”;
> 2. PreparedStatement预处理sql语句必须放在循环体外

接下来就是常规的jdbc操作：
```
//开始总计时
long bTime1 = System.currentTimeMillis();
//count=1380271
for (int i=0; i<count; i+=100000){
   List<DataToEmployee> list = //todo 每次从数据库获取10w条
   Connection conn = null;
   PreparedStatement pstm = null;
  try{
    //加载jdbc驱动
    Class.forName("com.mysql.jdbc.Driver");
    //连接mysql
    conn = DriverManager.getConnection(url, user, password);
    String sql = ".....";  //insert语句
     //关闭自动提交
//  conn.setAutoCommit(false);
    //预编译sql
    pstm = conn.prepareStatement(sql);
    if(list!=null && list.size()>0){
        long bTime = System.currentTimeMillis();
        for(DataToEmployee data : list){
            pstm.setString(1,data.getName());
            ......
            pstm.addBatch();  //批量处理
        }
        pstm.executeBatch();
        //关闭分段计时
       long eTime = System.currentTimeMillis();
       //输出
       System.out.println("成功插入10W条数据耗时："+(eTime-bTime));
    }
  }
}
//关闭总计时
long eTime1 = System.currentTimeMillis();
//输出
System.out.println("插入"+count+"数据共耗时："+(eTime1-bTime1));
```
这里统计出插入10W条数据的时间一般维持在2s左右，相比之前jpa是有很大的提升
> 成功插入10W条数据耗时：2832 
成功插入10W条数据耗时：1870 
成功插入10W条数据耗时：2328 
成功插入10W条数据耗时：2023 
成功插入10W条数据耗时：2148 
成功插入10W条数据耗时：1789 
成功插入10W条数据耗时：1987 
成功插入10W条数据耗时：2032 
...

jpa在存大量数据的时候，需要进行ORM转换，MyBatis也一样，效率相比jdbc肯定要差点，而且在使用jdbc的时候我就用了单线程，可见两者之间的差距。