为什么选 RocketMQ  

(1)   Kafka  之所以可以高性能是因为采用了顺序写，但如果一旦 Topic 或者 Partition 变多，则变成不断的写多个文件相当于随机写，所以性能开始大幅度下降；而 RocketMQ 则几乎没有这个问题，及时在Topic很多的（官方说可以支持5W），也不会出现明显的性能下降 

(2)   可靠性方面，RocketMQ 由于支持主从双同步刷盘机制，所以要强于 Kafka 的异步刷盘机制  

(3） RocketMQ 消费失败支持定时重试，每次重试间隔时间顺延；消费者消费失败可以直接抛出直接抛出RuntimeException，会重试

(4） Kafka 不支持分布式事务消息 

(5)   RocketMQ 去除对 zk 的依赖，使用自己开发的 NameSrv，无状态可以随意的部署多台并且更加轻量

(6)   Kafka 使用 scala 编写，而 RocketMQ 是 JAVA 系 





SpringBoot 如何整合使用 RocketMQ

(1) application.properties  配置 rocketmq.name-server 地址、生产者组 

(2) 生产者把消息同步发送到指定主题(topic) 

(3) 消费者监听某个主题，并设置属于哪个消费者组 

# RocketMQ 优点 

+ 单机吞吐量：十万级
+ 可用性：非常高，分布式架构
+ 源码是Java，我们可以自己阅读源码  
+ **RoketMQ ** 在稳定性上可能更值得信赖，这些业务场景在阿里双11已经经历了多次考验，如果你的业务有上述并发场景，建议可以选择**RocketMQ** 



# 消息模型 

+ 队列模型   队列模型就真的只是一个队列
+ **主题模型**   **发布订阅模型** 

# MQ使用的场景

+ 异步 加快响应时间

+ 解耦  系统模块解耦，通过消息队列通信，出现错误容易排查，增减功能容易

+ 削峰  突发大量请求，把请求放到队列里面，服务器根据自己处理能力消费，不至于打垮服务器



主要的缺点

+ 系统复杂性 **重复消费**、**消息丢失**、**消息的顺序消费**

+ 数据一致性   **分布式事务**   

+ 可用性   **MQ挂了咋办**


# RocketMQ 架构 

![640](G:\markdown图片\640.png)



NameServer` 、`Broker` 、`Producer` 、`Consumer 

NameServer  注册中心

+ Broker 管理 和 路由信息管理  
+ Broker  会将自己的信息注册到 NameServer  中，此时 NameServer 就存放了很多 Broker 的信息(Broker的路由表)，消费者和生产者就从 NameServer 中获取路由表然后照着路由表的信息和对应的Broker进行通信 (生产者和消费者定期会向 NameServer去查询相关的 Broker 的信息) 



Broker

+ Broker  主要负责消息的存储、投递和查询以及服务高可用保证；消息队列服务器嘛，生产者生产消息到 Broker，消费者从 Broker  拉取消息并消费
+ 一 个 Topic  中存在多个队列，一个 Topic 分布在多个 Broker 上，一个 Broker 可以配置多个 Topic ，它们是多对多的关系
+ 单个Broker 和 所有 NameServer 保持长连接 ，并且在每隔30秒 `Broker` 会向所有 `Nameserver` 发送心跳，心跳包含了自身的 `Topic` 配置信息
+ Broker 宕机，则该Broker上的消息读写都会受到影响。所以 RocketMQ提供了 master/slave 的结构，salve 定时从 master 同步数据，如果 master宕机，则 slave提供消费服务，但是不能写入消息

![image-20210927205907688](https://i.loli.net/2021/09/27/mHGX49wj6oufTs7.png)

Producer

+ 消息发布的角色，支持分布式集群方式部署 
+ Producer通过多种负载均衡模式发送到 Broker 集群  



Consumer 

+ Consumer，支持PUSH和PULL两种消费模式，支持 **集群消费**和  **广播消费**  

+ 集群消费：消费者集群共同消费一个主题的多个队列，一个队列只会被一个消费者消费，如果某个消费者挂掉，分组内其它消费者会接替挂掉的消费者继续消费  
+ 广播消费：消息会发给消费者组中的每一个消费者进行消费  





# 重复消费、顺序消费、消息丢失、分布式事务、消息堆积

重复消费：很多系统监听支付系统支付成功的消息，一个系统处理失败，让重新发送，其它系统也会收到这个消息，服务的**网络抖动**，**开发人员代码Bug**，还有**数据问题**等都可能处理失败要求重发的



### 消息去重 

接口**幂等**  幂等操作的特点是其任意多次执行所产生的影响均与一次执行的影响相同

+ 一般**幂等** **分场景去考虑**，看是**强校验**还是**弱校验**

+ 强校验  接收到消息做相应处理的同时还要加流水(写到磁盘) 消息来的时候id+业务场景去表里查，有就忽略

+ 弱校验  一些不重要的场景 

  ID + 场景唯一标识作为 **Redis** 的 key ，放到缓存里面失效时间看你场景，**一定时间内**的这个消息就去Redis判断，丢失无关紧要 



### 消息顺序消费

RocketMQ  一个 Topic 下有多个队列，为了保证发送有序， Hash取模法让同⼀个订单发送到同一个队列中，保证发送有序，再由一个消费者消费，保证消费有序 





### 消息丢失

+ 生产者（Producer） 通过网络发送消息给 Broker，当 Broker 收到之后，将会返回确认响应信息给 Producer，所以生产者只要接收到返回的确认响应，就代表消息在生产阶段未丢失

+ 要保证严格不丢失，主节点同步刷盘，主从节点同步复制，从节点同步刷盘，这时响应状态为 SendStatus.SEND_OK，则肯定不丢失  



### 分布式事务    

采取的是最终一致性，不是强一致性，采取的事务消息实现  

![img](G:\markdown图片\1090617-20190715204649244-1070060475.jpg)

Producer 已经把消息成功发送到了 Broker 端，但此消息被标记为暂不能投递状态，处于该种状态下的消息称为半消息，需要 Producer对消息的二次确认后，Consumer才能去消费它 

消息回查  

由于网络闪段，生产者应用重启等原因。导致 Producer 端一直没有对半消息进行 二次确认。这是Brock 服务器会定时扫描长期处于半消息的消息，会主动询问 Producer端该消息的最终状态 (Commit或者Rollback)，该消息即为消息回查



### 消息堆积问题 

削峰  峰值太大了导致消息堆积在队列中

+ RocketMQ 一个队列只会被一个消费者消费，增加消费者实例，同时增加每个主题的队列数量

# 刷盘和复制

同步刷盘和异步刷盘

+ 同步刷盘  

  同步刷盘中需要等待一个刷盘成功的 ACK ，同步刷盘对 MQ 消息可靠性来说是一种不错的保障，但是性能上会有较大影响

+ 异步刷盘 

  一个线程去异步地执行刷盘操作。消息刷盘采用后台异步线程提交的方式进行， 降低了读写延迟，提高了 MQ的性能和吞吐量



+ fore( ) 缓冲区是READ_WRITE模式下，此方法对缓冲区内容的修改强行写入文
+ load( ) 将缓冲区的内容载入内存，并返回该缓冲区的引用
+ isloaded( ) 如果缓冲区的内容在物理内存中，则返回真，否则返回假  



同步复制和异步复制 

+ 同步刷盘和异步刷盘是在单个结点层面的，而同步复制和异步复制主要是指的 Borker 主从模式下

  同步复制： 也叫  "同步双写" ，也就是说，只有消息同步双写到主从结点上时才返回写入成功

  异步复制： 消息写入主节点之后就直接返回写入成功



异步复制会不会也像异步刷盘那样影响消息的可靠性

+ 只影响系统的可用性，不会影响消息的可靠性，因为RocketMQ不支持主从自动切换，当主节点挂掉之后，生产者就不能再给这个主节点生产消息了，采用异步复制的方式，在主节点还未发送完需要同步的消息的时候主节点挂掉了，这个时候从节点就少了一部分消息，消费者可以自动切换到从节点进行消费，主节点挂掉的时间只会产生主从结点短暂的消息不一致的情况，降低了可用性，而当主节点重启之后，从节点那部分未来得及复制的消息还会继续复制





RocketMQ   的三种消息发送  同步发送  、异步发送 、单向发送  

同步发送  

+ 同步发送指消息发送方发出数据后会在收到接收方发回响应之后才发下一个数据包。⼀般用于重要通知消息  

异步发送

+ 异步发送指发送方发出数据后，不等接收方发回响应，接着发送下个数据包，一般用于可能链路耗时较长而对响应时间敏感的业务场景  

单向发送

+ 单向发送是指只负责发送消息而不等待服务器回应且没有回调函数触发，适用于某些耗时非常短但对可靠性要求并不高的场景

