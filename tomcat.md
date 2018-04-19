# Tomcat

> 作者 : Bruce
> 日期 : 2018年04月19


记录学习 Tomcat 相关知识。

## Talbe of Contents
  - [Tomcat连接器线程模型](#Tomcat连接器线程模型)
  - [Tomcat连接数参数](#Tomcat连接数参数)
  - [TCP三次握手四次挥手图](#TCP三次握手四次挥手图)
  - [Keep-Alive示意图](#Keep-Alive示意图)



### Tomcat连接器线程模型

  Tomcat 有一个acceptor接收线程来接收Socket连接，Tomcat 有多个工作线程处理Acceptor接收的连接。一次请求传入的处理流程:

    1. 操作系统和客户端之间建立连接的TCP握手。根据操作系统的实施情况，可以有一个用于保持连接的单个队列，也可以有多个队列。在多个队列的情况下，一个拥有尚未完成tcp握手的不完整连接。完成后，连接将移动到已完成的连接队列以供应用程序使用。 tomcat配置中的“acceptCount”参数用于控制这些队列的大小。
    
    2. Tomcat acceptor 接收线程 接收来自已完成连接队列的连接。
    
    3. 检查空闲线程池是否有可用工作线程，如果没有可用工作线程就检查当前工作线程数量是否大于maxThreads
        
           - 大于 maxThreads : 
           
           继续呆在 accept 队列等待有空闲的工作线程,如果accept队列也满了,此时tomcat会直接拒绝此次请求，返回connection refused
             
           - 小于 maxThreads : 
           
           Tomcat 创建一个工作线程进行处理
           
    4. 一旦找到一个空闲的工作线程,acceptor接收线程就会讲连接传递给它，然后监听接收新的已完成队列的连接。
    
    5. 工作线程将从socket conn 中读取数据流，处理request 并返回 response 等等；然后如果该连接不是Keep-Alive的话，则关闭连接回到空闲线程池，如果是Keep-Alive的话，则等待下一个数据包的到来直到keepAliveTimeout，然后关闭该连接释放回线程池。[查看下方 Keep-Alive示意图](#Keep-Alive示意图)
    
### Tomcat连接数参数    

Tomcat有关连接数容易混淆的几个参数:acceptCount、maxConnections、maxThreads、minSpareThreads。

按层次区分:
* 应用层
  
  maxThread : 指工作线程池最大多少工作线程数，默认值200。
  minSpareThreads: 指工作线程池初始化多少也即最少多少工作线程数。
  
* 内核层

  maxConnections: Tomcat最多能并发处理的请求连接，NIO模式下默认是10000。
  acceptCount：TCP 完成三次握手后，进入的accept队列大小，默认100。
  
![](/assets/244379816-5873ab2a162f4_articlex.jpeg)

结合[Tomcat连接器线程模型](#Tomcat连接器线程模型)流程：

* Step 1: TCP 三次握手
    
  [TCP三次握手四次挥手图](#TCP三次握手四次挥手图)
  
  * 第一次握手（**client端的socket等待队列**） : client 向 server 发送第一个FIN 包此时client 会维护一个 socket 队列，如果 socket 等待队列满了，而 client 也会由此返回 connection time out，只要是 client 没有收到 第二次握手SYN+ACK，3s 之后，client 会再次发送，如果依然没有收到，9s 之后会继续发送。
    
  * 第二次握手(**syn队列**)：Server 接收FIN 回复SYN+ACK, 此时server 会维护一个 SYN 队列，半连接 syn 队列的长度为 max(64, /proc/sys/net/ipv4/tcp_max_syn_backlog)  ，在机器的tcp_max_syn_backlog值在/proc/sys/net/ipv4/tcp_max_syn_backlog下配置，当 server 收到 client 的 SYN 包后，会进行第二次握手发送SYN＋ACK 的包加以确认，client 的 TCP 协议栈会唤醒 socket 等待队列，发出 connect 调用。
  
  * 第三次握手(**accpet队列**): 当server接收到ACK 报之后， 会进入一个新的叫 accept 的队列，该队列的长度为 min(backlog, somaxconn)，默认情况下，somaxconn 的值为 128，表示最多有 129 的 ESTAB 的连接等待 accept()，而 backlog 的值则应该是由 int listen(int sockfd, int backlog) 中的第二个参数指定，listen 里面的 backlog 可以有我们的应用程序去定义的。
  
 > 当accept队列满了之后，即使client继续向server发送ACK的包，也会不被相应，此时，server通过/proc/sys/net/ipv4/tcp_abort_on_overflow来决定如何返回，0表示直接丢丢弃该ACK，1表示发送RST通知client；相应的，client则会分别返回read timeout 或者 connection reset by peer。
  

### TCP三次握手四次挥手图

 ![](https://camo.githubusercontent.com/289b75e598a895c61fd8330f83864dc062e8fd36/687474703a2f2f636f6f6c7368656c6c2e636e2f2f77702d636f6e74656e742f75706c6f6164732f323031342f30352f7463705f6f70656e5f636c6f73652e6a7067)


### Keep-Alive示意图

#### No Keep-Alive

![](/assets/no-keep-alive.png)

#### Keep-Alive

![](/assets/keep-alive.png)
    
    
    
    
    
    