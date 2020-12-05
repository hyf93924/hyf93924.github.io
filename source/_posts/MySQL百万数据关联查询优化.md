---
title: MySQL百万数据关联查询优化
date: 2019-01-08 19:49:46
tags:
---
前段时间写过一篇[MySQL Join的底层实现原理](https://www.jianshu.com/p/16ad9669d8a9)，里面稍微有提到怎么通过索引优化，即Index Nested-Loop Join，今天在获取数据时，正好做到了优化一下。<!--more-->

---
表1（T1）：
![T1 count.png](https://upload-images.jianshu.io/upload_images/1005338-6f5a51694dafc46a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
表2（T2）：
![T2 count.png](https://upload-images.jianshu.io/upload_images/1005338-06e78b50237a330b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到T1有33万数据，T2有50万数据，sql语句是：
```
select tbj.OWNER_TYPE, tpo.OWNER_NAME, tpo.PRINCIPAL, tbj.LICENSE_NUMBER, 
tpo.ADDRESS, tpo.CANCODE, tbj.SCOPE_BUSINESS from tf_pt_owner tpo
left JOIN tf_bs_jurctc tbj
on tpo.OWNER_ID = tbj.OWNER_ID;
```
因为要查出所有数据，所以没有where条件，这样，在没有任何优化的情况下，查询所需时间为：

为什么呢？因为没有索引之类的，left join会使用Block Nested-Loop Join的方式去关联表查询，T1有30万数据，T2有50万数据，那么sql需要扫描的数据就相当于300000*500000，可想而知这是非常恐怖的。
![未加索引所用时间](https://upload-images.jianshu.io/upload_images/1005338-c217cbf469601de9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
实在等不下去了，用了1572s的时间只查出来17236条数据，总数据有540000条，全部查出来不知道等到什么时候去了。

一般我们会将数据量小的表当作驱动表（T1），这也就是一般情况下Join的效率高于Left Join的原因，因为mysql默认会将小表作为驱动表，而在匹配表（T2）中跟驱动表关联的字段我们就可以在此创建一个索引，那么效果如何呢？
![加索引查询时间](https://upload-images.jianshu.io/upload_images/1005338-d34d28ed974004ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到效率提高了不是一点两点，虽然现在还需要10s的查询时间。

---

最后，如何优化left join
1、条件中尽量能够过滤一些行将驱动表变得小一点，用小表去驱动大表 

2、右表的条件列一定要加上索引（主键、唯一索引、前缀索引等），最好能够使type达到range及以上（ref,eq_ref,const,system） 

3、无视以上两点，一般不要用left join~~！