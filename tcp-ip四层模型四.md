# TCP IP 应用层 HTTP 协议


> 作者:Bruce
>
> 日期:2016年10月17日
>
> 最底层协议解决了MAC,数据广电转换问题，网络互连层IP协议解决了IP问题，数据传输层解决了数据的传输问题，那么离我们Web开发最近的就是应用层的HTTP协议，基本上HTTP协议也是网页开发必备知识，万物归宗这块了解越深入对以后学习其他的开发语言及框架(php:ThinkPHP,python:django,java:ssh,spring,springmvc等等)是非常有帮助的，而且对现在正流行的前端构建也非常有帮助(vuejs,react,angularjs)

首先我们回顾并整理下我们上面几篇文章中的知识。

### 1.1 回顾 TCP/IP 网络协议栈分为应用层（Application）、传输层（Transport）、网络层（Network）和链路层（Link）四层。
![](http://pic002.cnblogs.com/images/2012/467431/2012111621035424.jpg)

### 1.2 回顾 信过程中，每层协议都要加上一个数据首部（header），称为封装（Encapsulation）
![](http://image.beekka.com/blog/201205/bg2012052913.png)
注意:

1. 以太网的标头，属于第一层协议，MAC地址在这一层
2. IP标头，属于第二场IP协议，IP地址在这一层
3. TCP标头，属于第三层传输层，端口在这一层

### 1.3 回顾 总结 一个完整的数据包
![](http://pic002.cnblogs.com/images/2012/467431/2012111621050639.jpg)