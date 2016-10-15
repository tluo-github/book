# TCP\/IP

> 我们每天都在使用互联网，全世界几十亿台电脑能够通信，核心靠的就是一些列协议，总称“互联网协议\(Internet  Protocol Suite\)”.它们对电脑如何连接和组网,做出了详尽规定。因为互联网协议太复杂，太庞大，我自己也知皮毛。就只记录自己学习web相关的协议。用户浏览器能加载到服务器的网页文件。第一个必要条件是用户电脑跟服务器是相通的\(靠TCP\/IP协议\),第二个就是用户的浏览器能够获取到服务器上的网页文件并识别\(靠的是http协议\)。
> 所以一般web程序员主要碰到和打交道的是http协议，但是它又是基于TCP\/IP协议，对TCP\/IP协议的理解能够对我们后期web性能优化和防DDOS攻击起到很大作用。

首先我们看看互联网上2个电脑主机A是如何将数据发送给B

![](/assets/490px-IP_stack_connections.svg.png)

可以明确的看到数据的交流\(Data flow\)是从上到下。

  1. Application-&gt;Transport-&gt;Internet-&gt;Link
  2. Link-&gt;Internet-&gt;Transport-&gt;Application

TCP\/IP参考模型分为四个层次：应用层、传输层、网络互连层、主机到网络层。每一个层创建在第一层提供的服务上，并且为上一层提供服务。

