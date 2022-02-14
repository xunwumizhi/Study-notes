| 参考文献

深入理解分布式消息队列: https://mp.weixin.qq.com/s/ZDVcIsqryXzRyFQofMDCgw

Kafka简明教程 - 知乎: https://zhuanlan.zhihu.com/p/37405836

# 消息队列MQ

基于OS的MQ：使用单机类似消息管道的技术，如Linux进程通信方式之一的管道；

基于数据库的MQ：Redis自带List数据类型

| 服务Redis的5种数据类型
* String
* list
LRANGE  切片时， start与offset都是下标，包含offset，  因此得到个数为offset-start+1
* set
* sorted set
* hash
map[string]string



# 队列功能要点

## 消费投递

- 消费模式 push、pull

 pull 模式：慢消费。消息量有限、速度不均匀，推荐使用 pull 模式，consumer 自己来 pull 消息；

 push 模式：

 - 消息投递时机

攒够了一定数量才投放。
到达了一定时间就投放。
有新的数据到来就投放。

- 通知对象

单播、广播

例如 pulsar 支持的订阅模型有：

Exclusive：独占型，一个订阅只能有一个消息者消费消息。
Failover：灾备型，一个订阅同时只有一个消费者，可以有多个备份消费者。一旦主消费者故障则备份消费者接管。不会出现同时有两个活跃的消费者。
Shared：共享型，一个订阅中同时可以有多个消费者，多个消费者共享 Topic 中的消息。
Key_Shared：键共享型，多个消费者各取一部分消息。
通常会在公共存储上维护广播关系，如 config server、zookeeper 等。

## 可靠投递，不丢消息

## 消费确认



# Kafka

## 概念

- Broker

Kafka 集群包含一个或多个服务器，这种服务器被称为 broker。

- Topic

Topic 在逻辑上可以被认为是一个 queue，每条消费都必须指定它的 Topic，可以简单理解为必须指明把这条消息放进哪个 queue 里。为了使得 Kafka 的吞吐率可以线性提高，物理上把 Topic 分成一个或多个 Partition，每个 Partition 在物理上对应一个文件夹，该文件夹下存储这个 Partition 的所有消息和索引文件。

- Partition

是物理意义上的队列，每个 Topic 包含一个或多个 Partition。

一个partition可以认为是一个队列，只存放一个topic，但一个topic可以存放在不同的partition里面，一个topic对应多个partition

单个 partition 只能由同个group内一个consumer来消费。

- Producer & Consumer

Producer负责发布消息到 Kafka broker。
Consumer消息消费者，向 Kafka broker 读取消息的客户端。

- Consumer Group

每个 Consumer 属于一个特定的 Consumer Group（可为每个 Consumer 指定 group name，若不指定 group name 则属于默认的 group）。

单个 partition 只能由同个group内一个consumer来消费。

Topic -> Partition -> Broker Partition Leader

## kafka不足

- 无法弹性扩容：对 partition 的读写都在 partition leader 所在的 broker，如果该 broker 压力过大，也无法通过新增 broker 来解决问题

- 扩容成本高：集群中新增的 broker 只会处理新 topic，如果要分担老 topic-partition 的压力，需要手动迁移 partition，这时会占用大量集群带宽

- 消费者新加入和退出会造成整个消费组 rebalance：导致数据重复消费，影响消费速度，增加延迟

- partition 过多会使得性能显著下降：ZK 压力大，broker 上 partition 过多让磁盘顺序写几乎退化成随机写

