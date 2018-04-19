# Tomcat

> 作者 : Bruce
> 日期 : 2018年04月19


记录学习 Tomcat 相关知识。

## Talbe of Contents
  - [Tomcat连接器线程模型](#Tomcat连接器线程模型)
  - [Tomcat连接数参数](#Tomcat连接数参数)
  - [关于maxConnections与maxThreads](#关于maxConnections与maxThreads)
  - [TCP三次握手四次挥手图](#TCP三次握手四次挥手图)
  - [SYN攻击](#SYN攻击)
  - [Keep-Alive示意图](#Keep-Alive示意图)
  - [总结](#总结)
  - [参考文档](#参考文档)



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

  maxConnections: 表示有多少个socket连接到tomcat上。NIO模式下默认是10000。
  acceptCount：TCP 完成三次握手后，进入的accept队列大小，默认100。
  
![](/assets/244379816-5873ab2a162f4_articlex.jpeg)

结合[Tomcat连接器线程模型](#Tomcat连接器线程模型)流程：

* Step 1: TCP 三次握手
    
  [TCP三次握手四次挥手图](#TCP三次握手四次挥手图)
  
  * 第一次握手（**client端的socket等待队列**） : client 向 server 发送第一个FIN 包此时client 会维护一个 socket 队列，如果 socket 等待队列满了，而 client 也会由此返回 connection time out，只要是 client 没有收到 第二次握手SYN+ACK，3s 之后，client 会再次发送，如果依然没有收到，9s 之后会继续发送。
    
  * 第二次握手(**syn队列**)：Server 接收FIN 回复SYN+ACK, 此时server 会维护一个 SYN 队列，半连接 syn 队列的长度为 max(64, /proc/sys/net/ipv4/tcp_max_syn_backlog)  ，在机器的tcp_max_syn_backlog值在/proc/sys/net/ipv4/tcp_max_syn_backlog下配置，当 server 收到 client 的 SYN 包后，会进行第二次握手发送SYN＋ACK 的包加以确认，client 的 TCP 协议栈会唤醒 socket 等待队列，发出 connect 调用。
  
  * 第三次握手(**accpet队列**): 当server接收到ACK 报之后， 会进入一个新的叫 accept 的队列，该队列的长度为 min(backlog, somaxconn)，默认情况下，somaxconn 的值为 128，表示最多有 129 的 ESTAB 的连接等待 accept()，而 backlog 的值则应该是由 int listen(int sockfd, int backlog) 中的第二个参数指定，listen 里面的 backlog 可以有我们的应用程序去定义的。
  
 > 当accept队列满了之后，即使client继续向server发送ACK的包，也会不被相应，此时，server通过/proc/sys/net/ipv4/tcp_abort_on_overflow来决定如何返回，0表示直接丢丢弃该ACK，1表示发送RST通知client；相应的，client则会分别返回read timeout 或者 connection reset by peer。在TCP连接的过程中Server端有如下两个队列:
 1. 一个是半连接队列：(syn queue) queue(max(tcp_max_syn_backlog, 64))，用来保存 SYN_SENT 以及 SYN_RECV 的信息。
 2. 另外一个是完全连接队列：accept queue(min(somaxconn, backlog))，保存 ESTAB 的状态，那么建立连接之后，我们的应用服务的线程就可以accept()处理业务需求了。
 
* Step 2: Acceptor 处理

acceptor 接收到连接后会做一些处理，NioEndpoint$Acceptor代码:
```java
// --------------------------------------------------- Acceptor Inner Class
    /**
     * The background thread that listens for incoming TCP/IP connections and
     * hands them off to an appropriate processor.
     */
    protected class Acceptor extends AbstractEndpoint.Acceptor {
        @Override
        public void run() {
            int errorDelay = 0;
            // Loop until we receive a shutdown command
            while (running) {
                // Loop if endpoint is paused
                while (paused && running) {
                    state = AcceptorState.PAUSED;
                    try {
                        Thread.sleep(50);
                    } catch (InterruptedException e) {
                        // Ignore
                    }
                }
                if (!running) {
                    break;
                }
                state = AcceptorState.RUNNING;
                try {
                    //if we have reached max connections, wait
                    countUpOrAwaitConnection();

                    SocketChannel socket = null;
                    try {
                        // Accept the next incoming connection from the server
                        // socket
                        socket = serverSock.accept();
                    } catch (IOException ioe) {
                        //we didn't get a socket
                        countDownConnection();
                        // Introduce delay if necessary
                        errorDelay = handleExceptionWithDelay(errorDelay);
                        // re-throw
                        throw ioe;
                    }
                    // Successful accept, reset the error delay
                    errorDelay = 0;
                    // setSocketOptions() will add channel to the poller
                    // if successful
                    if (running && !paused) {
                        if (!setSocketOptions(socket)) {
                            countDownConnection();
                            closeSocket(socket);
                        }
                    } else {
                        countDownConnection();
                        closeSocket(socket);
                    }
                } catch (SocketTimeoutException sx) {
                    // Ignore: Normal condition
                } catch (IOException x) {
                    if (running) {
                        log.error(sm.getString("endpoint.accept.fail"), x);
                    }
                } catch (Throwable t) {
                    ExceptionUtils.handleThrowable(t);
                    log.error(sm.getString("endpoint.accept.fail"), t);
                }
            }
            state = AcceptorState.ENDED;
        }
    }
```

这里countUpOrAwaitConnection()判断的就是当前的连接数是否超过maxConnections。 超过了就会阻塞等待。没有超过就会进入Tomcat 线程模型的4,5步。



### 关于maxConnections与maxThreads
maxConnections表示有多少个socket连接到tomcat上。NIO模式下默认是10000。而maxThreads则是woker线程并发处理请求的最大数。也就是虽然client的socket连接上了，但是可能都在tomcat的task queue里头，等待worker线程处理返回响应。

比如maxThreads=1000，maxConnections=800，假设某一瞬间的并发时1000，那么最终Tomcat的线程数将会是800，即同时处理800个请求，剩余200进入队列“排队”，如果acceptCount=100，那么有100个请求会被拒掉。
  
  


### TCP三次握手四次挥手图

 ![](https://camo.githubusercontent.com/289b75e598a895c61fd8330f83864dc062e8fd36/687474703a2f2f636f6f6c7368656c6c2e636e2f2f77702d636f6e74656e742f75706c6f6164732f323031342f30352f7463705f6f70656e5f636c6f73652e6a7067)

### SYN攻击
 
在三次握手过程中，Server发送SYN-ACK之后，收到Client的ACK之前的TCP连接称为半连接（half-open connect），此时Server处于SYN_RCVD状态，当收到ACK后，Server转入ESTABLISHED状态。SYN攻击就是 Client在短时间内伪造大量不存在的IP地址，并向Server不断地发送SYN包，Server回复确认包，并等待Client的确认，由于源地址 是不存在的，因此，Server需要不断重发直至超时，这些伪造的SYN包将产时间占用未连接队列，导致正常的SYN请求因为队列满而被丢弃，从而引起网络堵塞甚至系统瘫痪。SYN攻击时一种典型的DDOS攻击，检测SYN攻击的方式非常简单，即当Server上有大量半连接状态且源IP地址是随机的，则可以断定遭到SYN攻击了，使用如下命令可以让之现行：
```shell
netstat -nap | grep SYN_RECV
```

### Keep-Alive示意图

#### No Keep-Alive

![](/assets/no-keep-alive.png)

#### Keep-Alive

![](/assets/keep-alive.png)
    
    
### 总结

* maxThreads:Tomcat线程池最多能起的线程数；
* minSpareThreads:Tomcat初始化的线程池大小或者说Tomcat线程池最少会有这么多线程。
* maxConnections:Tomcat最多能并发处理的请求（连接）；
* acceptCount:Tomcat维护最大的对列数；
* tomcat最大连接数取决于maxConnections这个值加上acceptCount这个值，在连接数达到了maxConenctions之后，tomcat仍会保持住连接，但是不处理，等待其它请求处理完毕之后才会处理这个请求。
* tomcat server在tcp的accept队列的大小设置的基础上，对请求连接多做了一层保护，也就是maxConnections的大小限制。
    1. 当client端的大量请求过来时，首先是OS层的tcp的accept队列帮忙挡住，accept队列满了的话，后续的连接无法进入accept队列，无法交由工作线程处理，client将得到read timeout或者connection reset的错误。
    2. 第二层保护就是，在acceptor线程里头进行缓冲，当连接的socket超过maxConnections的时候，则进行阻塞等待，控制acceptor转给worker线程连接的速度，稍微缓缓，等待worker线程处理响应client。
    


    
### 参考文档
---

* 《Tomcat内核设计》

* [Tuning Tomcat For A High Throughput, Fail Fast System](http://techblog.netflix.com/2015/07/tuning-tomcat-for-high-throughput-fail.html)

* [PerformanceTuningApacheTomcat-Part2](http://www.tomcatexpert.com/sites/default/files/PerformanceTuningApacheTomcat-Part2.pdf)

* [Apache Tomcat 8 Configuration Reference](https://tomcat.apache.org/tomcat-8.5-doc/config/http.html)

* [杜绝假死，Tomcat容器做到自我保护，设置最大连接数](https://yq.aliyun.com/articles/2779)

* [聊下并发和Tomcat线程数（Updated](http://www.cnblogs.com/zhanjindong/p/concurrent-and-tomcat-threads.html)

* [tomcat的acceptCount与maxConnections](https://segmentfault.com/a/1190000008064162)


    
    
    