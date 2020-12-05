---
title: linux配置web环境
date: 2017-08-08 09:40:53
tags: java
---
> 现在很多公司的web项目都部署在了linux上，昨天在linux上自己部署了一个javaweb的环境，相比较还是比较顺利的，就安装MySQL的时候不知道一个什么原因，一直错误。<!--more-->

---
> * linux环境(centos)
> * java环境
> * tomcat linux版
> * MySQL linux版

### java环境：
首先查看自己是否有java环境 "java -version"
如果没有就安装java环境：
```
yum list java*
```
选择一个自己适合的java版本，我选的是1.7.0版本的："yum install java-1.7.0-openjdk* -y"
安装完成后就查看java版本   

---
### tomcat安装配置：
我把tomcat安装在"/usr/local"路径下，通过
```
wget http://mirror.bit.edu.cn/apache/tomcat/tomcat-8/v8.0.45/bin/apache-tomcat-8.0.45.tar.gz
```
下载tomcat，下的是tomcat8版本
解压tomcat：
```
tar -zxvf apache-tomcat-8.0.45.tar.gz
```
重命名为tomcat8:
```
mv apache-tomcat-8.0.45 tomcat8
```
tomcat的相关启动文件在bin的路径下，
```
cd /usr/local/tomcat8/bin
```
startup.sh即为启动文件
为这个路径下的所有.sh文件赋予权限：
```
chmod 777 *.sh
```
最后就是开启tomcat：
```
./startup.sh
```
在浏览器"http://<linux ip>:8080"，查看tomcat是否安装成功   
   
---
### 安装MySQL：
我用的linux系统是Centos，而centos7会将默认的数据库MySQL替换成Mariadb，所以我们先要做的就是卸载Mariadb:
```
rpm -qa|grep mariadb  	//查询出来已安装的mariadb
rpm -e --nodeps 文件名  //卸载mariadb，文件名为上述命令查询出来的文件
```
然后看看/etc目录下有没有my.cnf文件，有的话就删除该文件

下载相对应的[MySQL数据库](https://dev.mysql.com/downloads/mysql/)并解压，路径还是在 "/usr/local"中，并重命名为"mysql"

创建mysql用户组和用户：
```
groupadd mysql
useradd -r -g mysql -s /bin/false mysql
```
进入mysql目录，并修改目录拥有者为mysql用户：
```
cd mysql/
chown -R mysql:mysql ./
```
接下来就是安装了：
```
./scripts/mysql_install_db --user=mysql
```
安装中有可能会出现一个错误，我就遇到了这样一个错误
```
FATAL ERROR: please install the following Perl modules before executing scripts/mysql_install_db:
Data::Dumper
```
出现这个错误你安装之前需要先安装一个软件包：
```
yum install -y perl-Data-Dumper
```
再进行安装就能成功。

先将当前目录(mysql目录)拥有者为root用户：
```
chown -R root:root ./
```
将mysql目录下的data拥有者改为mysql：
```
chown -R mysql:mysql data
```
<img src="/images/java/2017080801.jpg" style="width: 400px;"/>
好了安装数据库方面已经完成了，现在就来配置一下。

添加开机自启，将启动脚本放在开机初始化目录中：
```
cp support-files/mysql.server /etc/init.d/mysql
# 赋予可执行权限
chmod +x /etc/init.d/mysql
# 添加服务
chkconfig --add mysql 
# 显示服务列表
chkconfig --list 
```
<img src="/images/java/2017080802.jpg" style="width: 400px;"/>
只要345项为开即可，如果不为开就开启：
```
chkconfig --level 345 mysql on
```

然后启动mysql服务：
```
mkdir /var/log/mariadb
service mysql start
```
最后连接数据库(默认密码为空)：
```
mysql -uroot -p
```
<img src="/images/java/2017080803.jpg" style="width: 400px;"/>

---
好了数据库安装完成了，不过中途有个错误一直纳闷，下面就是这个错：
```
The server quit without updating PID file(......)
```
网上查了一些资料，[>>直通资料<<](http://www.jb51.net/article/48625.htm)，出现这错误的原因一看还挺多的，自己也尝试了所有的解决方法，不过还是一直报这个错误，然后自己就卸载了mysql，然后又按照上面的再重装了一遍，然后就成功了，我估摸着原因可能是之前有装过mysql，然后系统中有两个mysql服务，所以卸载重装了就成功了。