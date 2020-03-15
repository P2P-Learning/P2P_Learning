## RabbitMQ 面试

### 预备知识

#### RabbitMQ的基本概念

**channel 信道**：信道是生产消费者与rabbit通信的渠道，生产者publish或是消费者subscribe一个队列都是通过信道来通信的，信道是建立在TCP连接上的虚拟连接，什么意思呢？就是说rabbitmq在一条TCP上建立成百上千个信道来达到多个线程处理，这个TCP被多个线程共享，每个线程对应一个信道，信道在rabbit都有唯一的ID ,保证了信道私有性，对应上唯一的线程使用。

##### RabbitMQ的六种模式

**simple模式**：

![](https://www.rabbitmq.com/img/tutorials/python-one-overall.png)

​	1.消息产生消息，将消息放入队列

​	2.消息的消费者(consumer) 监听 消息队列,如果队列中有消息,就消费掉,消息被拿走后,自动从队列中删除	(隐患 消息可能没有被消费者正确处理,已经从队列中消失了,造成消息的丢失，这里可以设置成手动的ack,	但如果设置成手动ack，处理完后要及时发送ack消息给队列，否则会造成内存溢出)。

**work工作模式(资源的竞争)**

![](https://www.rabbitmq.com/img/tutorials/python-two.png)

1.生产者产生消息放入队列，多个消费者消费消息，这里是由队列调度采用轮询（默认）的调度算法分配消息给消费者。

**publish/subscribe发布订阅(共享资源)**

![](https://www.rabbitmq.com/img/tutorials/exchanges.png)

1、每个消费者监听自己的队列；

2、生产者将消息发给exchange（交换机），由交换机将消息转发到绑定此交换机的每个队列，每个绑定交换机的队列都将接收到消息

**exchange的作用**就是类似路由器，消息发送会指定exchange和routing_key，队列声明的时候会绑定exchange和routing_key，那么消息生产的时候只要指定exchange和routing_key就可以路由到对应的队列了。

**exchange** 有三种模式:

​		1、直接发送模式（direct）1:1, 完全匹配，直接转发到绑定队列。

​       2、扇出模式（fanout）1：N, 一个消息可以发送到多个队列，类似于广播。

​       3、主题模式（topic)N:1 ,可以支持routing_key是通配符模式，多个生产者的消息只要匹配通配符就可以发送到对应队列消费。



**routing路由模式**

​	![](https://www.rabbitmq.com/img/tutorials/direct-exchange.png)

1.消息生产者发送给exchange，exchange根据绑定的规则转发到对应的消息队列

topic 主题模式(路由模式的一种)**

![](https://www.rabbitmq.com/img/tutorials/python-five.png)

1.消息生产者发送给exchange，exchange根据通配符规则判断是否符合路由规则，符合路由规则就把消息转发到指定的队列中。

**RPC模式**

![](https://www.rabbitmq.com/img/tutorials/python-six.png)

1.生产者会发送消息到声明的rpc队列，并且指定reply_to的回调队列，设置`correlation_id`唯一id，消费者受到消息处理完毕后，发送回复消息到生产者指定的回调队列，同时也给回复消息设置一样的`correlation_id`，生产者接受到回调，判断id是否一致即可。

#### RabbitMQ的模型

![](https://github.com/P2P-Learning/P2P_Learning/blob/master/%E4%B8%AD%E9%97%B4%E4%BB%B6/images/rabbitMQ-mode.png?raw=true)



### 1.MQ的使用场景是什么？

主要场景3个：异步、解耦、削锋。

**异步：**：

![](https://github.com/doocs/advanced-java/blob/master/images/mq-3.png?raw=true)

使用MQ：

![](https://github.com/doocs/advanced-java/raw/master/images/mq-4.png)

**解耦**：



![](https://github.com/doocs/advanced-java/blob/master/images/mq-1.png?raw=true)



**使用MQ**

![](https://github.com/doocs/advanced-java/blob/master/images/mq-2.png?raw=true)

**削锋**：

假设用户请求20000，那20000请求都会打入到MySQL

  ```
-------             --------          ------
| 用户 |----------->| A系统 |--------->|MySQL|
——-----             --------          ------
  ```



使用MQ：

用户请求20000，消费者只消费2000，打入到数据库也就2000。

```
-------             --------          ------            -----------         ---------
| 用户 |----------->| A系统 |--------->|  MQ | --------->|A系统消费者|-------->| MySQL |
——-----             --------          ------            -----------         ---------
```



### 2.你选型RabbitMQ的依据是什么？

 **Kafka、ActiveMQ、RabbitMQ、RocketMQ 对比**

| 特性                     | ActiveMQ                              | RabbitMQ                                           | RocketMQ                                                     | Kafka                                                        |
| ------------------------ | ------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 单机吞吐量               | 万级，比 RocketMQ、Kafka 低一个数量级 | 万级，比 RocketMQ、Kafka 低一个数量级              | 10 万级，支撑高吞吐                                          | 10 万级，高吞吐，一般配合大数据类的系统来进行实时数据计算、日志采集等场景 |
| topic 数量对吞吐量的影响 |                                       |                                                    | topic 可以达到几百/几千的级别，吞吐量会有较小幅度的下降，这是 RocketMQ 的一大优势，在同等机器下，可以支撑大量的 topic | topic 从几十到几百个时候，吞吐量会大幅度下降，在同等机器下，Kafka 尽量保证 topic 数量不要过多，如果要支撑大规模的 topic，需要增加更多的机器资源 |
| 时效性                   | ms 级                                 | 微秒级，这是 RabbitMQ 的一大特点，延迟最低         | ms 级                                                        | 延迟在 ms 级以内                                             |
| 可用性                   | 高，基于主从架构实现高可用            | 高，基于主从架构实现高可用                         | 非常高，分布式架构                                           | 非常高，分布式，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用 |
| 消息可靠性               | 有较低的概率丢失数据                  | 基本不丢                                           | 经过参数优化配置，可以做到 0 丢失                            | 同 RocketMQ                                                  |
| 功能支持                 | MQ 领域的功能极其完备                 | 基于 erlang 开发，并发能力很强，性能极好，延时很低 | MQ 功能较为完善，还是分布式的，扩展性好                      | 功能较为简单，主要支持简单的 MQ 功能，在大数据领域的实时计算以及日志采集被大规模使用 |

答题思路：可以根据上图对比的维度来回答

举例：我们的项目做的是一个消息系统，对系统的延时要求比较高，对比了同款的其他MQ，RabbitMQ的延时很低，比较符合我们的要求，同时RabbitMQ的文档比较全，团队比较熟悉，开箱即用，另外还有一点就是RabbitMQ的支持语言非常广泛，在目前我们微服务架构条件下方便扩展支持多语言，所以综合考虑我们使用RabbitMQ。

## 3.RabbitMQ有什么缺点

1、引入MQ后导致系统复杂度提升，比如如何保证消息消费的有序性？如何保证消息不丢失？如何保证不重复消费？

2、性能相对于kafka和RocketMQ比较差

3、开发语言小众，是Erlang，对二开的要求比较高。




## 4.RabbitMQ如何保证消费的有序性



场景说明：

![](https://github.com/doocs/advanced-java/blob/master/images/rabbitmq-order-01.png?raw=true)

解决方案：

1、牺牲性能，把上图中的消费者只保留一个，使得队列和消费者是1：1,这样可以保证消息的有序性

2、在多消费者或者单消费者多线程的情况下，另外依赖一个公共内存去排队。可以依赖redis去排序，也可以在单实例多线程情况下依赖一个公共的内存队列。



## 5.RabbitMQ如何保证消息不被重复消费

1、以消息id作为数据库主键，依赖数据库主键唯一性保证不被重复消费

2、设置消息全局唯一id，消费消息前先查询redis有没有该消息id，如果有就不消费，没有就消费消息，同时把该消息id存入Redis。

## 6. RabbitMQ如何保证消息不丢失

消息丢失的三种情况：

![](https://github.com/doocs/advanced-java/blob/master/images/rabbitmq-message-lose.png?raw=true)

解决方案：

![](https://github.com/doocs/advanced-java/blob/master/images/rabbitmq-message-lose-solution.png?raw=true)

开启RabbitMQ持久化需要保证

1.队列设置为持久化

2.交换机（exchange）设置为持久化

3.消息持久化`deliveryMode`=2



## 7.RabbitMQ的持久化

**消息持久化的三个条件：**

1. 消息投体时使用持久化投递模式
2. 目标交换器是配置为持久化的
3. 目标队列是配置为持久化的

**消息持久化的过程：**

1. 当一条持久化消息发送到持久化的Exchange上时，RabbitMQ会在消息提交到日志文件后，才发送响应
2. 一旦这条消息被消费后，RabbitMQ会将会把日志中该条消息标记为等待垃圾收集，之后会从日志中清除
3. 如果出现故障，自动重建Exchange,Bindings和Queue，同时通过重播持久化日志来恢复消息。



**RabbitMQ结构**

![](http://dingyue.nosdn.127.net/G7FVm50dZ65bV06ppeVXQM8AIizdmM8JWHqfUFiQwNoF=1544076799852compressflag.png)

BackingQueue由Q1,Q2,Delta,Q3,Q4五个子队列构成，在Backing中，消息的生命周期有四个状态：

1. Alpha：消息的内容和消息索引都在RAM中。（Q1，Q4）
2. Beta：消息的内容保存在Disk上，消息索引保存在RAM中。（Q2，Q3）
3. Gamma：消息的内容保存在Disk上，消息索引在DISK和RAM上都有。（Q2，Q3）
4. Delta：消息内容和索引都在Disk上。(Delta）

这里以持久化消息为例，从Q1到Q4，消息实际经历了一个RAM->DISK->RAM这样的过程，BackingQueue这么设计的目的有点类似于Linux的Swap，当队列负载很高时，通过将部分消息放到磁盘上来节省内存空间，当负载降低时，消息又从磁盘回到内存中，让整个队列有很好的弹性。因此触发消息流动的主要因素是：1.消息被消费；2.内存不足。



## 8.RabbitMQ的高可用

RabbitMQ高可用的方式可以选择集群部署和镜像模式部署。

**集群部署：**

![](http://dingyue.nosdn.127.net/OFqjExcshZpj=tsMaQJXIBApa06sCdts0=91EuBoQhpl31544076799852compressflag.png)

普通集群模式下，RabbitMQ的队列实例子只存在在一个节点上，既不能保证该节点崩溃的情况下队列还可以继续运行，也不能线性扩展该队列的吞吐量。虽然RabbitMQ的队列实际只会在一个节点上，但元数据可以存在各个节点上。举个例子来说，当创建一个新的交换器时，RabbitMQ会把该信息同步到所有节点上，这个时候客户端不管连接的那个RabbitMQ节点，都可以访问到这个新的交换器，也就能找到交换器下的队列。



上图中队列A的实例实际只在一个RabbitMQ节点上，其它节点实际存储的是只想该队列的指针。

为什么RabbitMQ不在各个节点间做复制了，《RabbitMQ实战》给出了两个原因：

1. 存储成本-RabbitMQ作为内存队列，复制对存储空间的影响，毕竟内存是昂贵而有限的
2. 性能损耗-发布消息需要将消息复制到所有节点，特别是对于持久化队列而言，性能的影响会很大



**镜像队列：**

为什么需要镜像队列？在原有集群模式下，实际上队列只存在主节点上，其他节点不会保存队列数据，只是做转发而已，如果真的保存队列数据挂掉之后，整个集群还是不可用的。所以后面RabbitMQ就把队列数据在每个节点都保存一份，这就是镜像队列。

![](https://github.com/P2P-Learning/P2P_Learning/blob/master/%E4%B8%AD%E9%97%B4%E4%BB%B6/images/mirror-queue.png?raw=true)





这里需要注意的是：就算是镜像队列，最终的读写都是在master完成的，比如上面Q3，假设消费者请求的是broker2，那broker2中的Q3是从节点，它会转发到broker3中的Q3来进行真正的消费操作，然后再同步给从节点返回。



## 9.惰性队列

RabbitMQ3.6之后引入了惰性队列（Lazy Queue）的概念,惰性队列是尽可能地将消息存入磁盘中，而在消费者消费到相应的消息时才会加入到内存，它的重要设计目标就是为了能够支持更长的队列，防止消息堆积在内存。惰性队列会把消息直接存入到磁盘，而不管队列是否设置持久化，但是如果队列设置的是非持久化，使用惰性队列重启后一样会丢失数据，因为他不会重放日志恢复数据。

总结：惰性队列比较适用于大批量消息场景，可以用来支持存放更多消息。



参考资料：

1. https://sq.163yun.com/blog/article/229026816937607168

2. https://github.com/doocs/advanced-java