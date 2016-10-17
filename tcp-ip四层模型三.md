# TCP IP 应用层 TCP和UDP 协议

> 作者:Bruce
> 
> 日期:2016年10月17日
> 
> 约读对象:Web程序员，\(虽然是低级部分,Web安全员，网络工程师，架构高性能也可看看\)
> 
> 说明:协议这些看上去非常枯燥但对我们以后web的性能优化，安全防护有非常大的帮助。在TCP IP四层模型\(二\)中，IP协议已经帮助我们根据IP来找到目标的MAC地址进行通信，这篇就来说说在IP协议至上我们传输层TCP,UDP这2个协议相关

我们在第一篇文章中简单的对TCP,UDP进行了说明定义

* TCP 协议是一个面向连接的、可靠的协议。\(有额外开销\)
* UDP 协议是一个不可靠的、无连接协议。\(无额外开销速度快\)

简单的说就是TCP协议更可靠\(下面要讲到三次握手\),而我们Web程序用的HTTP协议也是基于TCP协议,所以主要讲讲TCP协议，当然UDP的特征不确保可靠，但是无太多额外开销所以更剩流量和速度更快。不同的应用场景选择不同

## 1.TCP头格式

> 我们在上面看过IP协议的头格式,这里我们就直接看看TCP的头格式

![](/assets/QQ截图20161017105911.jpg)
\(中文版\)
![](http://coolshell.cn//wp-content/uploads/2014/05/TCP-Header-01.jpg)
\(英文版\)

> * 做程序这行有时候英文还是方便，所以这里我们尽量看英文版,可以对照中文看
> * 1 比特 符号为 1bit
> * 1 字节 符号为 1byte
> * 1字节=8比特 即 1byte = 8bit
> * 跟IP协议一样每行32bits\(等于4bytes\)固定部分有5行共160bits\(等于20bytes\)

### 1.1 TCP头格式 各个字段说明

1. Source Port（来源端口），长度16bits。发送链接端口，范围0~65525
2. Destination Port（目标端口）,长度16bits。接受链接端口，范围0~65525
3. Sequence Number（序列号码SYN），长度32bits。数据序列号码，TCP 连接中传送的数据流中的每一个字节都编上一个序号。如果没有同步化旗标（SYN），则此为第一个数据比特的序列码。
  如果含有同步化旗标（SYN），则此为最初的序列号；第一个数据比特的序列码为本序列号加一。
4. Acknowledgment Number\(确认号码ACK\),长度32bit,期望收到对方的下一个报文段的数据的第一个字节的序号,也即已经收到的数据的字节长度加1。
5. Offset（偏移），长度4bits,单位为字节，它指出报文数据距TCP报头的起始处有多远\(TCP报文头长度\)。
6. Reserved（保留字段）,长度6bits,保留今后使用，目前设置为0
7. TCP Flags（标识符）

  > * URG—紧急比特，1bit，当 URG=1 时，表明紧急指针字段有效。它告诉系统此报文段中有紧急数据，应尽快传送\(相当于高优先级的数据\)
  > * ACK—确认比特，1bit，只有当 ACK=1时确认号字段才有效。当 ACK=0 时，确认号无效
  > * PSH—推送比特，1bit，接收方 TCP 收到推送比特置1的报文段，就尽快地交付给接收应用进程，而不再等到整个缓存都填满了后再向上交付
  > * RST—复位比特，1bit，当RST=1时，表明TCP连接中出现严重差错\(如由于主机崩溃或其他原因\)，必须释放连接，然后再重新建立运输连接
  > * SYN—同步比特，1bit，同步比特 SYN 置为 1，就表示这是一个连接请求或连接接受报文
  > * FIN—终止比特，1bit，用来释放一个连接。当FIN=1 时，表明此报文段的发送端的数据已发送完毕，并要求释放运输连接

8. Window\(窗体大小，ps:不是微软操作系统\),长度16bits，窗口字段用来控制对方发送的数据量，单位为字节。TCP 连接的一端根据设置的缓存空间大小确定自己的接收窗口大小，然后通知对方以确定对方的发送窗口的上限。用于流量控制。\(计划在以后写带宽时详细解释\)

9. Checksum\(校验和\),长度16bits,对整个的TCP报文段，包括TCP头部和TCP数据，以16位字进行计算所得。这是一个强制性的字段。

10. Urgent Pointer（紧急指针）,长度16bits,紧急指针指出在本报文段中的紧急数据的最后一个字节的序号。
11. Options（选项字段），长度可变。TCP首部可以有多达40字节的可选信息，用于把附加信息传递给终点，或用来对齐其它选项。 这部分最多包含40字节，因为TCP头部最长是60字节（其中还包含前面讨论的20字节的固定部分）

> * 第一个字段kind-选项类型
> * 第二个字段length\(如果有的话\)指定该选项的总长度，该长度包括kind字段和length字段占据的2字节。
> * 第三个字段info（如果有的话）是选项的具体信息.
> * 1. kind=0是选项表结束选项
> 
> * 1. kind=1是空操作（nop）选项，没有特殊含义，一般用于将TCP选项的总长度填充为4字节的整数倍
> 
> * 1. kind=2是最大报文段长度选项,TCP连接初始化时，通信双方使用该选项来协商最大报文段长度（Max Segment Size，MSS）。TCP模块通常将MSS设置为（MTU-40）字节（减掉的这40字节包括20字节的TCP头部和20字节的IP头部）。这样携带TCP报文段的IP数据报的长度就不会超过MTU（假设TCP头部和IP头部都不包含选项字段，并且这也是一般情况），从而避免本机发生IP分片。对以太网而言，MSS值是1460（1500-40）字节。

注意点:

> * TCP包没有IP地址,因为那是上篇IP协议的事，但有源端口和目标端口
> * 一个TCP链接需要4个元组来表示是同一个链接\(src\_ip,scr\_post,dst\_ip,dst\_port\)
> * **Sequence Number** 是包的序号，**用来解决网络包乱序\(reordering\)问题**
> * **Acknowledgement Number** 就是**ACK**--用于确认收到,**用来解决不丢包问题**
> * \*\*Window又叫Advertised-Window,用于解决流控的。\(我以后在写关于带宽的时候详细说明，类似于路由器限速\)
> * **TCP Flag**, 也就是包的类型，主要用于操控TCP的状态机

### 1.2 TCP 状态机

> TCP为一个连接定义了11种状态，简单理解就是一个TCP连接的生命周期\(搞过Android开发的对比Activity的生命周期一样理解\),下面放一张中文版和英文版的TCP状态机图，先大致看看主要看下面解释

![](http://s3.51cto.com/wyfs02/M00/76/C2/wKioL1Zb_J6gV08KAAHRjgX486s686.png)
\(英文版\)
![](http://s5.51cto.com/wyfs02/M00/76/C4/wKiom1ZcA_WBfjVXAALhkWgijbk565.jpg)
 \(中文版\)

> 这里看不懂的就先过知道有这么个周期就行了

图中得知TCP有11种状态,这些可以在电脑命令行中使用netstat命令查询\(非常有用,在安全排查,性能排查,这往往是第一个命令\)

* CLOSED : 最开始,也标识结束，因为结束也是回到这里
* LISTEN : 监听,服务器正在等待连接进入
* SYN\_RCVD : 收到一个请求\(收到一个SYN,返回SYN\/ACK\),尚未确认
* SYN\_SENT : 已经发出连接请求\(SYN\),尚未确认
* ESTABLISHED : 建立连接,数据正常传输

  > 1.SYN\_RCVD收到ACK后,状态为ESTABLISHED
  > 
  > 2 SYN\_SENT 在收到SYN\/ACK,发送ACK后状态为ESTABLISHED

* FIN\_WAIT\_1 : （主动关闭）收到关闭通知\(接受到FIN\)，发出关闭请求（发出ACK），等待确认

* FIN\_WAIL\_2 ：（主动关闭） FIN\_WAIL\_1只收到ACK没有收到FIN\(说明还有数据传输,如果也收到了FIN就直接进入TIME\_WAIL\)，等待对方关闭请求

* TIME\_WAIT ：完成双向关闭,等待所有分组死掉
* CLOSING ： 两边同时尝试关闭
* CLOSE\_WAIT : \(被动关闭\)收到关闭请求，已确认
* LAST\_ACK:\(被动关闭\)等待最有一个确认\(收到FIN\)，等待所有分组死掉

> ps:看了这个图反正我已经吐槽,这个状态机太复杂了，协议也非常复杂,但有非常重要，下面我们从3个实例\(TCP创建、TCP传输数据、TCP断链接\)来一一对应。所幸这些是网络工程师必须熟悉必须会，WEB程序员相对了解越多越好先发一张整图
> ![](http://coolshell.cn//wp-content/uploads/2014/05/tcp_open_close.jpg)

### 1.2.1 TCP 3次握手链接
![TCP创建3次握手](http://s4.51cto.com/wyfs02/M00/76/C3/wKiom1Zb-1bhSsrQAAECUogyt_0364.jpg)

1. Client(客户端) 发送SYN报文(SYN为1),告诉Server(服务器端)打算链接服务器端口,以及初始化序号(Inital Sequence Number 缩写为:ISN)，客户端进入SYN_SENYT状态
2. Server返回保护服务器的初始化seq(sequence Number),同时将确认号(ACK)设置为Client发送来的ISN+1,及ACK=ISN+1(在上面整图中可以看到ACK=X+1)，服务端进入SYN_RCVD状态
3. 客户端收到SYN,ACK后，将确认号(ACK)设置为服务器的seq+1,发送给服务端，客户端进入ESTABLISHED状态
4. 最后服务器接受到确认的ACK报文后,完成三次握手进入ESTABLISHED状态

### 1.2.2 TCP 4次握手断开
![](http://s5.51cto.com/wyfs02/M02/76/C3/wKioL1Zb_mCCYBXFAAEdquxxit4101.jpg)
> 由于TCP连接是全双工的，因此每个方向都必须单独进行关闭,发送方和接收方都需要Fin和Ack,意思就是客户端，服务端都要确认和关闭。这原则是当一方完成它的数据发送任务后就能发送一个FIN来终止这个方向的连接。收到一个 FIN只意味着这一方向上没有数据流动，一个TCP连接在收到一个FIN后仍能发送数据。首先进行关闭的一方将执行主动关闭，而另一方执行被动关闭。

1. 客户端发送一个关闭请教FIN
2. 服务端接受到后发送ACK确认(特别注意服务器这时可能要有数据没处理完所以有了下一步)
3. 服务端发送关闭请求FIN
4. 客户端收到后发送ACK确认

思考:

1. 为什么建立连接协议是三次握手，而关闭连接却是四次握手呢？
 >这是因为服务端在LISTEN状态下的SOCKET当收到SYN报文的建连请求后，它可以把ACK和SYN（ACK起应答作用，而SYN起同步作用）放在一个报文里来发送。但关闭连接时，当收到对方的FIN报文通知时，它仅仅表示对方没有数据发送给你了；但未必你所有的数据都全部发送给对方了，所以你可以未必会马上会关闭SOCKET,也即你可能还需要发送一些数据给对方之后，再发送FIN报文给对方来表示你同意现在可以关闭连接了，所以它这里的ACK报文和FIN报文多数情况下都是分开发送的。

2. 为什么TIME_WAIT状态还需要等2MSL后才能返回到CLOSED状态？
> 这是因为虽然双方都同意关闭连接了，而且握手的4个报文也都协调和发送完毕，按理可以直接回到CLOSED状态（就好比从SYN_SEND状态到ESTABLISH状态那样）；但是因为我们必须要假想网络是不可靠的，你无法保证你最后发送的ACK报文会一定被对方收到，因此对方处于LAST_ACK状态下的SOCKET可能会因为超时未收到ACK报文，而重发FIN报文，所以这个TIME_WAIT状态的作用就是用来重发可能丢失的ACK报文。

TCP小结:这里写的都是TCP基础中的基础,这部分能完全了解最好，TCP还有很多算法及用法这里就不展开了我也没有深入研究，有兴趣的朋友可以看看
[http://coolshell.cn/articles/11564.html](http://coolshell.cn/articles/11564.html)


### 2.


参考链接:

* [https:\/\/zh.wikipedia.org\/wiki\/传输控制协议](https://zh.wikipedia.org/wiki/传输控制协议)
* [https:\/\/zhangbinalan.gitbooks.io\/protocol\/content\/tcpbao\_wen\_ge\_shi.html](https://zhangbinalan.gitbooks.io/protocol/content/tcpbao_wen_ge_shi.html)
* [http://coolshell.cn/articles/11564.html](http://coolshell.cn/articles/11564.html)
* [http://wangzan18.blog.51cto.com/8021085/1718212](http://wangzan18.blog.51cto.com/8021085/1718212)
* [http://wzgl08.blog.51cto.com/668144/1666021](http://wzgl08.blog.51cto.com/668144/1666021)

