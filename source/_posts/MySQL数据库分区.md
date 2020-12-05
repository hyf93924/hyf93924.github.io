---
title: MySQL数据库分区
date: 2018-09-28 22:00:07
tags:
---

#### 数据库分区处理
如果一张表的数据量太大的话，myd、myi就会变得很大，查找数据就会变的很慢，我接触到的是有关温州出租车网约车GPS数据量的查询，大概数据量为1天4000万条记录，不分区查询速度慢到怀疑人生。在物理上我们可以把数据表分割成不同的小块，查找数据只需要查找需要的那一块数据即可，查询速度大大提升。<!--more-->

我们项目中我只用到了rang分区：
[参考mysql分区](https://blog.csdn.net/laoyang360/article/details/52886987)

#### range分区
##### 自动分区
```
create table cb03_cbrfid(
ZWMC  VARCHAR2(50),
DTID  VARCHAR2(50),
RFID  VARCHAR2(50),
CBSBH  VARCHAR2(50),
CBLX  VARCHAR2(50),
NUM  NUMBER(10),
ZDW  VARCHAR2(50) ,
SJ  DATE ,
ZZZ  VARCHAR2(50) ,
ZZXCG  VARCHAR2(50),
JD  NUMBER(19,16),
WD  NUMBER(19,16),
WZ  VARCHAR2(50),
RESERVED1  FLOAT(8),     
RESERVED2  FLOAT(8),    
XXLX  INTEGER ,
GXSJ  DATE )
 partition by range(gxsj)
  interval(numtodsinterval(1,'day'))
  ( partition cbrfid1 values less than(to_date('2017-10-15 00:00:00','yyyy-mm-dd hh24:mi:ss')));
```

具体的操作：
```
partition by range (字段名)
(
  partition 分区名 values less than (TO_DATE(' 2014-07-25 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
);
```

##### 手动分区
```
alter table 表名 add partition 分区名 values less than (TO_DATE(' 2014-07-27 00:00:00, 'SYYYY-MM-DD HH24:MI:SS'))
```

---
#### 最后分区示例：
```
create table T_WYC_GPS_HISTORY
(
  vehicle_no    VARCHAR2(20),
  vehicle_color VARCHAR2(8),
  alarm_id      VARCHAR2(32),
  status        VARCHAR2(32),
  lat           VARCHAR2(32),
  lon           VARCHAR2(32),
  speed         VARCHAR2(16),
  direction     VARCHAR2(8),
  geometry      MDSYS.SDO_GEOMETRY,
  load_time     DATE,
  pre_geohash   VARCHAR2(20),
  last_geohash  VARCHAR2(20),
  platform_id   INTEGER
)
partition by range (LOAD_TIME)
(
  partition T_WYC_GPS_HISTORY_20180226 values less than (TO_DATE(' 2018-02-27 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
    tablespace WZYG_WYC_GPS
    pctfree 10
    initrans 1
    maxtrans 255
    storage
    (
      initial 8M
      next 1M
      minextents 1
      maxextents unlimited
    ),
  partition T_WYC_GPS_HISTORY_20180227 values less than (TO_DATE(' 2018-02-28 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
    tablespace USERS
    pctfree 10
    initrans 1
    maxtrans 255
    storage
    (
      initial 8M
      next 1M
      minextents 1
      maxextents unlimited
    ),
  partition T_WYC_GPS_HISTORY_20180228 values less than (TO_DATE(' 2018-03-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
    tablespace USERS
    pctfree 10
    initrans 1
    maxtrans 255
    storage
    (
      initial 8M
      next 1M
      minextents 1
      maxextents unlimited
    ),
```