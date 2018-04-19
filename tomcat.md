# Tomcat

> 作者 : Bruce
> 日期 : 2018年04月19


记录学习 Tomcat 相关知识。

## Talbe of Contents
  - [Tomcat Http 连接器线程模型](#Tomcat Http 连接器线程模型)
  - [工作流](#工作流)
  - [reset](#reset)
  - [checkout](#checkout)
  - [总结](#总结)


### Tomcat Http 连接器线程模型

  Tomcat 有一个acceptor接收线程来接收Socket连接，Tomcat 有多个工作线程处理Acceptor接收的连接。一次请求传入的处理流程:

    1. 操作系统和客户端之间建立连接的TCP握手。根据操作系统的实施情况，可以有一个用于保持连接的单个队列，也可以有多个队列。在多个队列的情况下，一个拥有尚未完成tcp握手的不完整连接。完成后，连接将移动到已完成的连接队列以供应用程序使用。 tomcat配置中的“acceptCount”参数用于控制这些队列的大小。
    
    2. Tomcat acceptor 接收线程 接收来自已完成连接队列的连接。
    
    3. 检查空闲线程池是否有可用工作线程
          
        - 没有 ：
        
        检查当前工作线程数量是否大于maxThreads
        
           - 大于 maxThreads : 
           
           继续呆在 accept 队列等待有空闲的工作线程,如果accept队列也满了,此时tomcat会直接拒绝此次请求，返回connection refused
             
           - 小于 maxThreads : 
           
           Tomcat 创建一个工作线程进行处理
           
    4. 一旦找到一个空闲的工作线程,acceptor接收线程就会讲连接传递给它，然后监听接收新的已完成队列的连接。