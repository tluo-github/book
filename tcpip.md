# TCP\\/IP

> 我们每天都在使用互联网，全世界几十亿台电脑能够通信，核心靠的就是一些列协议，总称“互联网协议\(Internet  Protocol Suite\)”.它们对电脑如何连接和组网,做出了详尽规定。因为互联网协议太复杂，太庞大，我自己也知皮毛。就只记录自己学习web相关的协议。用户浏览器能加载到服务器的网页文件。第一个必要条件是用户电脑跟服务器是相通的\(靠TCP\/IP协议\),第二个就是用户的浏览器能够获取到服务器上的网页文件并识别\(靠的是http协议\)。
> 所以一般web程序员主要碰到和打交道的是http协议，但是它又是基于TCP\/IP协议，对TCP\/IP协议的理解能够对我们后期web性能优化和防DDOS攻击起到很大作用。

## 1、TCP\\/IP四层模型

首先我们看看互联网上2个电脑主机A是如何将数据发送给电脑主机B

![](/assets/490px-IP_stack_connections.svg.png)

可以明确的看到数据的交流\(Data flow\)是从上到下。

1. Application-&gt;Transport-&gt;Internet-&gt;Link\(电脑A发送数据流程\)
2. Link-&gt;Internet-&gt;Transport-&gt;Application\(电脑B接收数据流程\)

所以TCP\/IP参考模型分为四个层：

* 应用层\( Application \)
* 传输层 \( Transport \)
* 网络互连层 \(Internet\)
* 网络接口层 \(Link\) 

| 应用层 | FTP、TENLNET、HTTP、HTTPS、SSH..等等 |
| --- | --- |
| 传输层 | TCP、UDP |
| 网络互连层 | IP |
| 网络接口层 | 以太网、Wi-Fi、MPLS |

每一层都是为了完成一种功能。为了实现这些功能，就需要大家都遵守共同的规则。每一个层创建在下一层提供的服务上，并且为上一层提供服务。可以看出我们的HTTP和FTP处于最上层应用层,个人认为web程序员，应该对网络层\(IP协议\)、传输层\(TCP,UDP协议\)，应用层（各种协议）了解。

### 1.1应用层

> 该层包括所有和应用程序协同工作,利用基础网络交换应用程序专用的数据的协议。

每一个应用层（TCP\/IP参考模型的最高层）协议一般都会使用到两个传输层协议之一： 面向连接的TCP传输控制协议和无连接的包传输的UDP用户数据报文协议。 常用的应用层协议有：

运行在TCP协议上的协议：

* HTTP（Hypertext Transfer Protocol，超文本传输协议），主要用于普通浏览。
* HTTPS（Hypertext Transfer Protocol over Secure Socket Layer, or HTTP over SSL，安全超文本传输协议）,HTTP协议的安全版本。
* FTP（File Transfer Protocol，文件传输协议），由名知义，用于文件传输。
* POP3（Post Office Protocol, version 3，邮局协议），收邮件用。
* SMTP（Simple Mail Transfer Protocol，简单邮件传输协议），用来发送电子邮件。
* TELNET（Teletype over the Network，网络电传），通过一个终端（terminal）登陆到网络。
* SSH（Secure Shell，用于替代安全性差的TELNET），用于加密安全登陆用。

运行在UDP协议上的协议：

* BOOTP（Boot Protocol，启动协议），应用于无盘设备。
* NTP（Network Time Protocol，网络时间协议），用于网络同步。
* DHCP（Dynamic Host Configuration Protocol，动态主机配置协议），动态配置IP地址。

其他：

* DNS（Domain Name Service，域名服务），用于完成地址查找，邮件转发等工作（运行在TCP和UDP协议上）。
* ECHO（Echo Protocol，回绕协议），用于查错及测量应答时间（运行在TCP和UDP协议上）。
* SNMP（Simple Network Management Protocol，简单网络管理协议），用于网络信息的收集和网络管理。
* ARP（Address Resolution Protocol，地址解析协议），用于动态解析以太网硬件的地址。

### 1.2传输层

> 传输层的功能是使源端主机和目标端主机上的对等实体可以进行会话,分为TCP,UDP

* TCP 协议是一个面向连接的、可靠的协议。\(有额外开销\)
* UDP 协议是一个不可靠的、无连接协议。\(无额外开销速度快\)

### 1.3网络互连层

> 网络层解决在一个单一网络上传输数据包的问题。我们上图主机A发送数据到主机B中间会有多个路由及网络

### 1.4网络接口层

> 网络接口层实际上并不是因特网协议组中的一部分，但是它是数据包从一个设备的网络层传输到另外一个设备的网络层的方法。

