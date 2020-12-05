title: nginx负载均衡、反向代理初识
date: 2017-06-14 20:52:48
tags: nginx
---
> 这几天在学习nginx的一些东西，知道nginx能负载均衡和反向代理，所以一直想学习这这类知识,这篇博文只是初识nginx有关负载均衡和反向代理的一些点<!--more-->

---
首先你要安装nginx，相关配置在nginx下的nginx.conf配置中（本博文在linux系统下）
* 查看nginx的路径
	```
	whereis nginx
	```
<img src="/images/nginx/whereis.jpg" style="width: 400px;"/>
* 然后进入nginx的配置中
	```
	cd /usr/local/nginx/conf
	```
<img src="/images/nginx/conf.jpg" style="width: 500px;"/>
* 编辑nginx.conf配置文件
	```
	vi nginx.conf
	```
* 添加相关配置
> 主要配置的是负载均衡的ip地址，这里要注意的是upstream的名字和server location/中proxy_pass中的url地址要统一
<img src="/images/nginx/fzjh.jpg" style="width: 500px;"/>
上图中upstream中的weight是代表权重
* 重启nginx和nginx端口（启动程序在sbin中，我的目录实在nginx下的sbin中）
	```
	cd ../sbin
	./nginx -t
	./nginx
	```
<img src="/images/nginx/startnginx.jpg" style="width: 580px;"/>
---
好了，如果启动成功了，就能根据你配置的负载ip和端口能访问了。这就是我初识nginx的相关内容，还得不断的深入，不断的学习。