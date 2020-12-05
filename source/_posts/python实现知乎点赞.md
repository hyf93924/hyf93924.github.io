---
title: python实现知乎点赞
date: 2017-07-27 22:24:05
tags: python
---
> 刚刚学了python爬虫，http相关的内容，就试着玩了玩知乎点赞的功能，当然只是作为我初学者的习题而已嘛，代码很简单，不过也遇到了一些小问题，下面来说明吧。<!--more-->

---
> 首先先来介绍一个工具：Anaconda，一个集成的python，因为有些软件包用pip install会出错，安装不了，使用Anaconda的conda install就可以轻松的安装，而且其自带一个工具Jupyter用来刚开始的python学习很不错的。

* 我这里使用python来给知乎上的文字点赞，其实就是使用python来模拟用户点赞这么一个动作，所以就先打开[知乎首页](https://www.zhihu.com/#signin)（这里就是遇到的第一个小问题，使用电脑版的知乎首页来做时报了个403错误，所以这里采用的是模拟手机版来做），而我测试的这条内容为[>>>点击查看<<<](https://www.zhihu.com/question/62958001/answer/204215037)，查看调试模式我们看到：
<img src="/images/python/2017072701.jpg" style="width: 400px;"/>
其中我们要用到的便是Request URL和Request Headers里的内容，Request URL提供的是目标地址，而Request Headers头文件信息就是把我们包装成用户去访问地址。其中的cookie是比较重要的，因为你的一些登录信息是存在cookie里的，还有就是User-Agent，让爬虫模拟了浏览器的行为，也是重要的。
<img src="/images/python/2017072703.jpg" style="width: 400px;"/>
Form Data里提供的就是我们接下来点赞操作的参数。
* 接下来就是使用上面的信息来做爬虫了，具体操作实在Jupyter里操作的，步骤如下：
<img src="/images/python/2017072702.jpg" style="width: 400px;"/>
将上面的所需的参数设置进去，不过在headers里将Request Headers里的所有参数设置进去会报一个400错误，这个时候就是要慢慢的调试了，你会发现头文件里有些内容是多余的：
> Accept
> Accept-Encoding
> Connection
> Content-Length
> Content-Type
最后如果成功了就会看到如图那样返回的状态码为“200”。
---
最后来看看知乎上的体现吧：
运行python之前（原始状态）：
<img src="/images/python/2017072704.jpg" style="width: 400px;"/>
运行成功后：
<img src="/images/python/2017072705.jpg" style="width: 400px;"/>
我们看到点赞数增加了，说明成功了，当然反对的做法也是一样的。这就是使用python来给知乎上的文章点赞的小demo

