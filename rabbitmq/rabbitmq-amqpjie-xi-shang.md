AMQP\(Advanced Message Queuing Protocol\)**高级消息队列协议**是一个规范，它规定了

* 系统模型，定义了AMQP中各个**实体**的名称，交互流程

* 通讯规范，定义了实体之间交互的数据包格式、**实体**之间通讯的具体命令

* 序列化标准，定义了通讯数据的序列化和反序列化格式

《AMQP解析》分为上下，主要讨论系统模型和通讯规范。这两部分是AMQP的核心内容，理解这两部分内容也就掌握了AMQP的精髓。

## AMQP模型



也许你见过下面这幅图![](http://mmbiz.qpic.cn/mmbiz_jpg/nfxUjuI2HXiatibXdLvUF8cx4IL6JEBDx88otLmhM7qbzL4pVGu7IaTwV02YScmI8ybod8AiblwicEuvqYv6FvUlzA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)  
这就是AMQP的**实体**模型，它把整个系统分代表**客户端**的Publisher\(生产者\)、Consumer\(消费者\)；代表**服务器端**的Broker。不同于我们用过的其他MQ系统"消息=队列"，AMQP模型解耦和**消息**和**队列**，我们模拟一个Publisher发送log到Consumer。![](http://mmbiz.qpic.cn/mmbiz_gif/nfxUjuI2HXiatibXdLvUF8cx4IL6JEBDx8m49JgBZYbVo3h0UPr2icib3Cia9R5jsWnsJWsvjiaCFvAJ4VL8aPzRsbmg/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)  


* Publisher产生一条数据，发送到Broker。例子中就是发送到服务器

* Broker中的Exchange可以理解为一个**规则表**（routing key和queue的映射关系——Binding），Broker收到消息后根据**routing key**查询投递的目标**queue**

* Consumer向Broker发送**订阅**消息的时候会指定自己监听哪个queue，当有数据到达queue的时broker会推送数据到consumer。

**Publisher指明谁可以收到消息（routing key）而不具体指定某个Queue；Consumer只是纯粹的监听Queue而不会关心数据从而来**。Exchange非常像计算机网络通讯交换机，**收到数据后查询转发表决定下一跳出口**；实际上Exchange并不真实的存在它仅仅是一个规则表。匹配-&gt;投递的动作是由Broker内部的进程完成的。Publisher、Consumer连接到Broker都是通过**Connection**完成的，它代表了一个TCP连接。TCP连接对于操作系统来说是非常昂贵的资源，如果每秒钟千上万个消息发送给Broker，每次发送都是一个TCP连接这是一笔非常大的系统开销。为了提高通道的利用率AMQP规定了channel的概念，可以把它理解为和broker的一次会话。如果你接触过Http2.0应该知道多路复用（Multiplexing），这个特性和AMQP是一模一样的。

## Exchange的类型



Exchange代表了消息和Queue的映射关系，它非常灵活。AMQP定义了三种类型的关联方式

* 直接关联\(direct\)

![](http://mmbiz.qpic.cn/mmbiz_png/nfxUjuI2HXiatibXdLvUF8cx4IL6JEBDx8b6piaXxNjlQjd8c4ZGybpc4jPBgVcr5ibpeH691AVrpbQCG0qJiaUxUibA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)  
这种类型的Exchange看似简单其实非常有威力，它的逻辑是：Exchange根据routing key寻找到对应的Queue。比如我定义下面的Exchange

![](http://mmbiz.qpic.cn/mmbiz_jpg/nfxUjuI2HXiatibXdLvUF8cx4IL6JEBDx8jjLibC9978xPZFp4n25WBvTdtib1vh96lX0kxwTDuquCYYSghpr0icwQw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)  
我们需要把“性能数据”分别存放在Mysql和InfluxDB，可以定义一个“metric”Exchange，把“mysql”、“InfluxDB”和“mertic”做绑定\(binding\)。**Broker收到发送给Exchange的数据后会判断routing key，如果publisher没有指定routing key则分别投递给InfluxDB、mysql。所以看起来就像是一条数据被镜像成两份分别发给两个Queue。“空”也是一种routing key。设置为“空”其实还是会发生“比较”，只不过publisher刚好也是“空”所以才会往两个队列中同时发送。**如果我们把mysql的绑定修改一下

![](http://mmbiz.qpic.cn/mmbiz_jpg/nfxUjuI2HXiatibXdLvUF8cx4IL6JEBDx8Ruq16vruUzA2muwSyrYPljo0n3l9LnDBqwAe3T3FC2UDOZB5fibk30w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)  
**Broker收到发送给Exchange的数据后会判断routing key，如果publisher没有指定routing key则投递给InfluxDB；如果routing key是mysql则投递给mysql。**

* 广播\(fanout\)

![](http://mmbiz.qpic.cn/mmbiz_png/nfxUjuI2HXiatibXdLvUF8cx4IL6JEBDx8VWDXLcG5FzfR4Cic1wbld9wcMHkgIUku0tjlQ8aZbpibSY1ZibyX5BQEw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)  
这种类型的Exchange最简单，无视routing key，有多少个Queue和Exchange绑定就直接推送到Queue。比如上面的例子中如果Exchange是fanout类型的，则无论你设置不设置routing key，publisher发送不发送routing key；Exchange都会分别投递到两个Queue。一般这种类型的Exchange很少用。

* 模糊匹配\(topic\)

和direct很像，direct比较routing key的时候是严格比较。topic则允许你用“模糊匹配”。![](http://mmbiz.qpic.cn/mmbiz_jpg/nfxUjuI2HXiatibXdLvUF8cx4IL6JEBDx87hY9feX1ibSic9ZIBwIDTqAr8dr4SMHS4c6SQ4Ns0Lia8IZ9Nmmibs6qRw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)  
上面的映射可以翻译为：

* 所有cpu、mem开头的数据给mysql Queue

* 所有disk、network开头的数据给InfluxDB Queue

发送数据的时候我们可以这样

![](http://mmbiz.qpic.cn/mmbiz_jpg/nfxUjuI2HXiatibXdLvUF8cx4IL6JEBDx8gT5WHKV66H1SjIFFTYmN3zF7wmeK3HnMz1iaJeD5EjTmFqTYbEPUxicg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)  


内存数据我们可以用“mem.free”、“mem.usage”，它们都会被投递到mysql queue。同样的道理，disk、network开头的数据会被投递到InfluxDB队列中。需要特别注意的是

![](http://mmbiz.qpic.cn/mmbiz_jpg/nfxUjuI2HXiatibXdLvUF8cx4IL6JEBDx8NNnFCzsEnEXaYoG1DOEjKz1B3f2C2cyWx9nOictqUTIlMoqmfMznkAw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)  
如果你routing key什么都不设置，它的含义不是指“匹配所有的消息”；而是指匹配“routing key为空”的数据（publisher发送的routing key是空）。如果你要匹配所有的数据，正确的方法应该是——“\#”。**提示：RabbitMQ还有一个扩展类型的Exchange——header。它会根据publisher的发送数据的头部信息选择Queue。比如可以根据header部分的userId选择Queue其实有点像IM了（userId代表的是发送者用户名）**

## 小技巧



学习的最好方式是实践，AMQP的规范并不像想想的那么复杂，特别是RabbitMQ的实现代码非常简洁，甚至有人怀疑AMQP协议是专门为Erlang定制的（其实二者真没有任何关系）RabbitMQ提供了一个漂亮的Web界面可以让我们不用写一行代码就能实验AMQP的所有功能，修改rabbitmq的配置文件启用rabbitmq\_management插件

```
配置文件enabled_plugins


[rabbitmq_management].
```

你可以用命令行搞定`rabbitmq-plugins enable rabbitmq_management`重启RabbitMQ后访问15672端口

![](http://mmbiz.qpic.cn/mmbiz_jpg/nfxUjuI2HXiatibXdLvUF8cx4IL6JEBDx8Bxib3YvoXTXdNhI6kn3TeGofCPPXJSgQbNZEwWfIIxwf8mqmJBlAe7A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)  
通过命令行新建一个用户

```
rabbitmqctl add_user fireflyc 123 #新建用户fireflyc，密码123


rabbitmqctl set_user_tags fireflyc administrator #设置fireflyc是超级管理员
```

登录

![](http://mmbiz.qpic.cn/mmbiz_jpg/nfxUjuI2HXiatibXdLvUF8cx4IL6JEBDx8iav3jClrxHibVp5ZSaejg8libk3jpqkg2ZibOwXLLq63c3tfB0F9UGt6qw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)  


* Exchange标签我们可以看到所有的Exchange，最底下可以Add a new exchange

* Queue标签我们可以看到所有的Queue，最底下可以Add a new queue

* 点击某个Exchange我们可以看到Binding，可以通过Publish Message发送消息

* 点击某个Queue我们可以通过Get message获取队列中的消息

好了，可以开始你的AMQP之旅了。

来源:[写程序的康德](https://mp.weixin.qq.com/s?__biz=MzIxMjAzMDA1MQ==&mid=2648945635&idx=1&sn=966633eeba2567e7759b597e43568054&chksm=8f5b54efb82cddf9678821ad9708fc404c087034471f3385ccac09dae0392a146b3673e3ccbd##)

