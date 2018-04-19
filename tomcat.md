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

    Tomcat 有一个Acceptor来接收Socket连接，Tomcat 有多个工作线程处理Acceptor接收的连接。



