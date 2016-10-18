# TCP IP 应用层 HTTP 协议

> 作者:Bruce
> 
> 日期:2016年10月17日
> 
> 最底层协议解决了MAC,数据广电转换问题，网络互连层IP协议解决了IP问题，数据传输层解决了数据的传输问题，那么离我们Web开发最近的就是应用层的HTTP协议，基本上HTTP协议也是网页开发必备知识，万物归宗这块了解越深入对以后学习其他的开发语言及框架\(php:ThinkPHP,python:django,java:ssh,spring,springmvc等等\)是非常有帮助的，而且对现在正流行的前端构建也非常有帮助\(vuejs,react,angularjs\)

首先我们回顾并整理下我们上面几篇文章中的知识。

### 1.1 回顾 TCP\/IP 网络协议栈分为应用层（Application）、传输层（Transport）、网络层（Network）和链路层（Link）四层。

![](http://pic002.cnblogs.com/images/2012/467431/2012111621035424.jpg)

### 1.2 回顾 信过程中，每层协议都要加上一个数据首部（header），称为封装（Encapsulation）

![](http://image.beekka.com/blog/201205/bg2012052913.png)
注意:

1. 以太网的标头，属于第一层协议，MAC地址在这一层
2. IP标头，属于第二场IP协议，IP地址在这一层
3. TCP标头，属于第三层传输层，端口在这一层

### 1.3 回顾 总结 一个完整的数据包

![](http://pic002.cnblogs.com/images/2012/467431/2012111621050639.jpg)
我们前面没有对以太网报文格式做详细说明这里稍微说说

1. 以太网头部14字节由：目标地址6字节+源地址6字节
2. 数据字段包含IP,TCP的头和数据，长度46~1500字节
3. 以太网尾部4字节

> 注意:我们的IP包数据固定长度为20字节，加上可变数据理论长度65535字节，TCP固定长度为20字节，理论上已经超过以太网给的1500字节，所以IP包和TCP包会分包会有那么复杂的协议来支持分包

### 1.4 回顾 总结

基本上最底层的3层已经能保障2台电脑能够数据通信了，打个比喻我们要让一个集装箱从武汉到上海，现在有了高速路，有了车，有了集装箱，就差集装箱的货物。那么进入今天的主题HTTP协议。

## 2 HTTP协议

> 现在我们站在了前三层的肩膀上,基本上就关注数据的内容业务上。所以这套设计模式,在我们设计软件项目的时候还是非常有参考价值，一层只做一层的事。

### 2.1 HTTP协议定义

http协议\(超文本传输协议HyperText Transfer Protocol\)，它是基于TCP协议的应用层传输协议，简单来说就是客户端和服务端进行数据传输的一种规则。直接上图:
![](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016081901.jpg)

> 因为是基于TCP的所以那些繁复都都让TCP去做了

但我们在浏览器输入一个网址，浏览器直接是发送一个request请求流,服务器也是直接返回一个response流，那么很自然各种参数和规则就在request流和response流里面,这里我直接实战使用Burp Suite抓包工具来获取我自己站点[http:\/\/www.jsh315.com](http://www.jsh315.com)的request,response流:

\(request流内容\)
![](/assets/request1.jpg)

由上图可以看到，http请求由请求行，消息报头，请求正文三部分构成 \(由于是get请求没有正文\)

### 2.11 Request 请求行

第一行就是请求头:GET \/shop\/home\/goods\/doGoodsList?mainId=1&parentId=8&classId=15 HTTP\/1.1。该行包含3个字段:请求方法,请求资源，HTTP版本

**请求方法**有多种,如下:

1. GET     请求获取Request-URI所标识的资源
2. POST    在Request-URI所标识的资源后附加新的数据
3. HEAD    请求获取由Request-URI所标识的资源的响应消息报头
4. PUT     请求服务器存储一个资源，并用Request-URI作为其标识
5. DELETE  请求服务器删除Request-URI所标识的资源
6. TRACE   请求服务器回送收到的请求信息，主要用于测试或诊断
7. CONNECT 保留将来使用
8. OPTIONS 请求查询服务器的性能，或者查询与资源相关的选项和需求

> 传统的网站常用的是GET和POST,配合现在流行的前后分离后台提供REST Full Api 的风格常用的就是,GET,POST,PUT,DELETE

**请求资源就是路径**

**最后一个就是HTTP版本** 目前常用的是HTTP1.1,HTTP2.0已经出来了

### 2.11 Request 消息报头

> 消息报头由一系列的键值对，每一个报头域都是由名字+“：”+空格+值 组成,也允许客户端向服务器端发送一些附加信息或者客户端自身的信息。在做REST Full Api接口时安全相关的信息可以放到头里面

常用的请求报头:







\(response流内容\)
![](/assets/response1.jpg)

