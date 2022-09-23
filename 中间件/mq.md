# 应用场景

## Kafka 为什么不像 Redis 和 MySQL 那样支持读写分离？

- 第一，这和它们的使用场景有关。对于那种读操作很多而写操作相对不频繁的负载类型而言，采用读写分离是非常不错的方案——我们可以添加很多 Follower 横向扩展，提升读操作性能。反观 Kafka，它的主要场景还是在消息引擎，而不是以数据存储的方式对外提供读服务，通常涉及频繁地生产消息和消费消息，这不属于典型的读多写少场景。因此，读写分离方案在这个场景下并不太适合。

- 第二，Kafka 副本机制使用的是异步消息拉取，因此存在 Leader 和 Follower 之间的不一致性。如果要采用读写分离，必然要处理副本滞后引入的一致性问题，比如如何实现 Read-your-writes、如何保证单调读（Monotonic Reads）以及处理消息因果顺序颠倒的问题。相反，如果不采用读写分离，所有客户端读写请求都只在 Leader 上处理，也就没有这些问题了。当然，最后的全局消息顺序颠倒的问题在 Kafka 中依然存在，常见的解决办法是使用单分区，其他的方案还有 Version Vector，但是目前 Kafka 没有提供。

## 消息可靠投递，不丢数据

？？？给每个消息加上序号，消费者检查序号是否有序；要求指定Partition，指定Producer，consumer与Partition一一对应

容易丢消息的阶段：

生产阶段：Producer 确认是否投递到broker返回成功

存储：单节点broker刷盘后返回成功，多节点可以通过消息复制保证多数节点保存成功

消费阶段：执行完业务逻辑再commit

## 只消费一次，避免重复消费

业务保证幂等

MQ都只能保证 At Least Once

而kafka中的exactly once是指：在流计算中，用 Kafka 作为数据源，并且将计算结果保存到 Kafka 这种场景下，数据从 Kafka 的某个主题 A 中消费，在计算集群中计算，再把计算结果保存在 Kafka 的其他主题 B 中。这样的过程中，保证每条消息都被恰好计算一次，确保计算结果正确。


## 顺序消费

同个用户的消息，哈希处理投递到指定Partition

## 容灾、弹性

- RocketMQ

两种复制模式：

同步双写。主从节点都写成功，才返回成功。

主从异步复制，主从关系配置固定，不支持动态切换，主节点宕机，producer不能再生产消息，consumer切换到从节点消费。

Dledger 集群异步复制：可选举切换主节点，由于消息要至少复制到 2 个节点上才会返回写入成功，即使主节点宕机了，也至少有一个节点上的消息是和主节点一样的。Dledger 在选举时，总会把数据和主节点一样的从节点选为新的主节点，这样就保证了数据的一致性，既不会丢消息，还可以保证严格顺序。

Dledger 采用raft协议选主，选举期间服务不可用

- kafka

复制基本单位是partition，partition分布在不同broker上，一主多从

![](https://miszibu.github.io/2020/08/07/Components/Kafka/Kafka%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6/data_copy.png)

集群异步复制：足够多写入成功副本数 ISR（In Sync Replicas)，包含主节点。写入ISR个partition后，返回成功。

共 5 副本(partition)，可有 3 个ISR

kafka 分布式锁选主，broker启动后向ZK注册/controller，成为leader，其他后续启动broker watch 这个临时节点，leader宕机会收到ZK通知，开启新一轮抢占

- pulsar

消息数据由 BookKeeper 集群负责存储，元数据由 ZooKeeper 集群负责存储，Pulsar 的 Broker 上就不需要存储任何数据了，这样 Broker 就成为了无状态的节点。

## 服务发现broker、partition

- RocketMQ

NameServer，broker向NameServer注册地址，client 访问NameServer获取路由表并缓存，定期拉取更新。也可以从任意 broker 拉取路由表

- kafka

Kafka 的客户端并不会去直接连接 ZooKeeper，它只会和 Broker 进行远程通信，ZooKeeper 上的元数据（topic/partition/broker）通过 Broker 中转给每个客户端的。client不能直接访问ZK

## 事务与事务消息

对于订单系统来说，它创建订单的过程中实际上执行了 2 个步骤的操作：

在订单库中插入一条订单数据，创建订单；

发消息给消息队列，消息的内容就是刚刚创建的订单。

【购物车系统】订阅相应的主题，接收订单创建的消息，然后清理购物车，在购物车中删除订单中的商品。【订单系统】创建订单和发送消息这两个步骤要么都操作成功，要么都操作失败，不允许一个成功而另一个失败的情况出现。

Order 开启本地事务，然后给MQ发送一个半消息（对consumer不可见），发送消息成功然后执行本地事务，创建订单，如果执行成功提交本地DB事务，成功则提交事务消息。如果失败则回滚事务消息。

![](https://static001.geekbang.org/resource/image/27/e6/27ebf12e0dc79e00e1e42c8ff0f4e2e6.jpg)


- RocketMQ 事务消息

在 RocketMQ 中的事务实现中，增加了事务反查的机制来解决事务消息提交失败的问题。如果 Producer 也就是订单系统，在提交或者回滚事务消息时发生网络异常，RocketMQ 的 Broker 没有收到提交或者回滚的请求，Broker 会定期去 Producer 上反查这个事务对应的本地事务的状态，然后根据反查结果决定提交或者回滚这个事务。

业务代码需要实现一个反查本地事务状态的接口，告知 RocketMQ 本地事务是成功还是失败。
![](https://static001.geekbang.org/resource/image/11/7a/11ea249b164b893fb9c36e86ae32577a.jpg)

- kafka 没有事务消息

kafka 事务是指的，发送多条消息，要么都成功，要么都失败

- kafka 的 exactly once

在流计算中，用 Kafka 作为数据源，并且将计算结果保存到 Kafka 这种场景下，数据从 Kafka 的某个主题中消费，在计算集群中计算，再把计算结果保存在 Kafka 的其他主题中。这样的过程中，保证每条消息都被恰好计算一次，确保计算结果正确。

## 海量client

RocketMQ 它的元数据是保存在 NameServer 的内存中，Kafka 是保存在 ZooKeeper 中，这些存储都不擅长保存大量数据，所以也支撑不了太多的客户端和主题。

## MQ 数据存储

- kafka

存储以partition为单位，每个partition包含一组消息文件(segment file)和索引（index）。

写消息到WAL(write ahead log)找到最近生成的文件，写满了则新建吗，文件尾部追加。
读消息时索引二分查找文件，然后往下遍历，找到目标消息，往下顺序读取

- rocketMQ

RocketMQ 的存储以 Broker 为单位。它的存储也是分为消息文件和索引文件，但是在 RocketMQ 中，每个 Broker 只有一组消息文件，它把在这个 Broker 上的所有主题的消息都存在这一组消息文件中。

索引文件和 Kafka 一样，是按照主题和队列分别建立的，每个队列对应一组索引文件，这组索引文件在 RocketMQ 中称为 ConsumerQueue。

Broker 上所有主题、所有队列的消息按照自然顺序追加写入到同一个消息文件中，一个文件写满了再写下一个文件。查找消息的时候，可以直接根据队列的消息序号，计算出索引的全局位置（索引序号 x 索引固定长度 20），然后直接读取这条索引，再根据索引中记录的消息的全局位置，找到消息。可以看到，这里两次寻址都是绝对位置寻址，比 Kafka 的查找是要快的。

![](https://static001.geekbang.org/resource/image/34/60/343e3423618fc5968405e798b7928660.png)

在消息文件的存储粒度上，Kafka 以分区为单位，粒度更细，优点是更加灵活，很容易进行数据迁移和扩容。

RocketMQ 以 Broker 为单位，较粗的粒度牺牲了灵活性，带来的好处是，在写入的时候，同时写入的文件更少，有更好的批量（不同主题和分区的数据可以组成一批一起写入），更多的顺序写入，尤其是在 Broker 上有很多主题和分区的情况下，有更好的写入性能。

索引设计上，RocketMQ 和 Kafka 分别采用了稠密和稀疏索引，稠密索引需要更多的存储空间，但查找性能更好，稀疏索引能节省一些存储空间，代价是牺牲了查找性能。可以看到，两种消息队列在存储设计上，有不同的选择。

大多数场景下，这两种存储设计的差异其实并不明显，都可以满足需求。但是在某些极限场景下，依然会体现出它们设计的差异。比如，在一个 Broker 上有上千个活动主题的情况下，RocketMQ 的写入性能就会体现出优势。再比如，如果我们的消息都是几个、十几个字节的小消息，但是消息的数量很多，这时候 Kafka 的稀疏索引设计就能节省非常多的存储空间。

## 计算与存储分离

消息队列没有什么计算逻辑，当做数据存储的代理，写入与读取数据，那为什么要pulsar这样计算存储分离呢？

1. 无状态扩容方便，云原生多租户模式

2. 方便扩展broker计算能力


## 流式计算 kafka + flink 

kafka + flink 流式计算。计算准确，数据只计算一次，实现Exactly once
flink 使用 checkpoint，保存：
1. 整个计算任务的状态，计算过程中的临时状态
2. 数据源的信息，计算了哪些数据

计算失败重启时，每个子任务从最近的checkpoint读取数据恢复自己的状态，然后整个任务从记录的数据源处继续消费

![](https://static001.geekbang.org/resource/image/0c/fa/0c301d798341dc53515611c31e9031fa.png)

Flink 在创建一个 CheckPoint 的时候，同时开启一个 Kafka 的事务，完成 CheckPoint 同时提交 Kafka 的事务。Flink 基于两阶段提交这个常用的分布式事务算法，实现了一分布式事务的控制器来保证 完成checkpoint + 提交kafka事务 同时完成或取消。


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


## kafka不足

- 无法弹性扩容：对 partition 的读写都在 partition leader 所在的 broker，如果该 broker 压力过大，也无法通过新增 broker 来解决问题

- 扩容成本高：集群中新增的 broker 只会处理新 topic，如果要分担老 topic-partition 的压力，需要手动迁移 partition，这时会占用大量集群带宽

- 消费者新加入和退出会造成整个消费组 rebalance：导致数据重复消费，影响消费速度，增加延迟

- partition 过多会使得性能显著下降：ZK 压力大，broker 上 partition 过多让磁盘顺序写几乎退化成随机写




## 为什么需要消息队列，MQ能解决那些问题

- 上下游处理速度不一致，异步处理

秒杀场景中，风控、锁库存完成即可返回，后续生成订单、通知、数据上报可以异步完成

- 流量控制，削峰填谷

秒杀系统网关收到请求，放入队列后端慢慢处理，消费到老数据滞后认为。
更好方法使用流控组件、令桶牌

- 服务一定程度的解耦

多个后续流程，可以从生产者中抽离，在多组消费者中实现：

一个新订单创建时：支付系统需要发起支付流程；风控系统需要审核订单的合法性；客服系统需要给用户发短信告知用户；经营分析系统需要更新统计数据；

- 引入的问题

系统复杂度；时延

## 消息队列对比

### RocketMQ

时延优化：处理消息的QPS几十万

### Kafka

异步批量处理，吞吐量最高，“先攒一波再一起处理”的设计。
Kafka 这种异步批量的设计带来的问题是，它的同步收发消息的响应时延比较高，因为当客户端发送一条消息的时候，Kafka 并不会立即发送出去，而是要等一会儿攒一批再发送，在它的 Broker 中，很多地方都会使用这种“先攒一波再一起处理”的设计。
当你的业务场景中，每秒钟消息数量没有那么多的时候，Kafka 的时延反而会比较高。所以，Kafka 不太适合在线业务场景。

单节点极限消息处理能力：每秒 2000W 条消息

- 批处理：

Kafka 的 Producer 只提供了单条发送的 send() 方法，并没有提供任何批量发送的接口。当你调用 send() 方法发送一条消息之后，无论你是同步发送还是异步发送，Kafka 都不会立即就把这条消息发送出去。它会先把这条消息，存放在内存中缓存起来，然后选择合适的时机把缓存中的所有消息组成一批，一次性发给 Broker。简单地说，就是攒一波一起发。

在消费时，消息同样是以批为单位进行传递的，Consumer 从 Broker 拉到一批消息后，在客户端把批消息解开，再一条一条交给用户代码处理。

- 数据压缩

一批消息进行压缩，投递给consumer，由consumer自行解压

- 磁盘IO顺序读写

顺序读写相比随机读写省去了大部分的寻址时间，它只要寻址一次，就可以连续地读写下去，所以说，性能要比随机读写要好很多。

Kafka 就是充分利用了磁盘的这个特性。它的存储设计非常简单，对于每个Partition，它把从 Producer 收到的消息，顺序地写入对应的 log 文件中，一个文件写满了，就开启一个新的文件这样顺序写下去。

消费的时候，也是从某个全局的位置开始，也就是某一个 log 文件中的某个位置开始，顺序地把消息读出来。这样一个简单的设计，充分利用了顺序读写这个特性，极大提升了 Kafka 在使用磁盘时的 IO 性能。

- PageCache 缓存加速读写

PageCache 就是操作系统在内存中给磁盘上的文件建立的缓存。应用程序在写入文件的时候，操作系统会先把数据写入到内存中的 PageCache，然后再一批一批地写到磁盘上。

读取文件的时候，也是从 PageCache 中来读取数据，这时候会出现两种可能情况。一种是 PageCache 中有数据，那就直接读取，这样就节省了从磁盘上读取数据的时间；另一种情况是，PageCache 中没有数据，这时候操作系统会引发一个缺页中断，应用程序的读取线程会被阻塞，操作系统把数据从文件中复制到 PageCache 中，然后应用程序再从 PageCache 中继续把数据读出来，这时会真正读一次磁盘上的文件，这个读的过程就会比较慢。

用户的应用程序在使用完某块 PageCache 后，操作系统并不会立刻就清除这个 PageCache，而是尽可能地利用空闲的物理内存保存这些 PageCache，除非系统内存不够用，操作系统才会清理掉一部分 PageCache。清理的策略一般是 LRU 或它的变种算法，大部分情况下，消费读消息都会命中 PageCache，读最近写入的消息

PageCache 是读写缓存，更新数据时经过缓存。读写缓存是不可靠的

Kafka 它更依赖的是，在不同节点上的多副本来解决数据可靠性问题，这样即使某个服务器掉电丢失一部分文件内容，它也可以从其他节点上找到正确的数据，不会丢消息。

- ZeroCopy 零拷贝

从文件复制数据到 PageCache 中，如果命中 PageCache，这一步可以省掉；

从 PageCache 复制到应用程序的内存空间中，

从应用程序的内存空间复制到 Socket 的缓冲区，这个过程就是我们调用网络应用框架的 API 发送数据的过程。

Kafka 使用零拷贝技术可以把这个复制次数减少一次，上面的 2、3 步骤两次复制合并成一次复制。减少内核切换到用户空间的，数据复制

直接从 PageCache 中把数据复制到 Socket 缓冲区中，这样不仅减少一次数据复制，更重要的是，由于不用把数据复制到用户内存空间，DMA 控制器可以直接完成数据复制，不需要 CPU 参与，速度更快。



- Kafka 高性能总结

使用批量处理的方式来提升系统吞吐能力。

基于磁盘文件高性能顺序读写的特性来设计的存储结构。

利用操作系统的 PageCache 来缓存数据，减少 IO 并提升读性能。

使用零拷贝技术加速消费流程。

### Pulsar

存储计算分离

### 总结
如果你的系统使用消息队列主要场景是处理在线业务，比如在交易系统中用消息队列传递订单，那 RocketMQ 的低延迟和金融级的稳定性是你需要的。

如果你需要处理海量的消息，像收集日志、监控信息或是前端的埋点这类数据，或是你的应用场景大量使用了大数据、流计算相关的开源产品，那 Kafka 是最适合你的消息队列。

## 概念

消费模式：发布订阅

Topic、队列(RocketMQ) 分区(Partition, Kafka)、消费组

数据流处理、消费的一致性保障等级：
- 至多一次（At most once）
- 至少一次（At least once）
- 精确一次（Exactly once）


### kafka

02 | 一篇文章带你快速搞定Kafka术语: https://time.geekbang.org/column/article/99318?utm_term=zeusGVR04&utm_source=geektime&utm_medium=xiaoxiduilie

consumer 属于 consumer group

consumer 独占一个或多个 Partition

每个group都有 coordinator 负责维护 consumer 与 partition 对应关系，consumer与partition发生变更时，coordinator 重新组织 consumer 与 partition 对应关系，即rebalance

client：consumer，producer

server：broker进程。多个 Broker 进程能够运行在同一台机器上，但更常见的做法是将不同的 Broker 分散运行在不同的机器上，这样如果集群中某一台机器宕机，即使在它上面运行的所有 Broker 进程都挂掉了，其他机器上的 Broker 也依然能够对外提供服务。

Kafka 定义了两类副本：领导者副本（Leader Replica）和追随者副本（Follower Replica）。前者对外提供服务，这里的对外指的是与客户端程序进行交互；而后者只是被动地追随领导者副本而已，不能与外界进行交互。副本是 partition 维度的

概念分层：
1. 第一层是主题层，每个主题可以配置 M 个分区，而每个分区又可以配置 N 个副本。
2. 第二层是分区层，每个分区的 N 个副本中只能有一个充当领导者角色，对外提供服务；其他 N-1 个副本是追随者副本，只是提供数据冗余之用。
3. 第三层是消息层，分区中包含若干条消息，每条消息的位移从 0 开始，依次递增。
4. 最后，客户端程序只能与分区的领导者副本进行交互。

![](https://static001.geekbang.org/resource/image/58/91/58c35d3ab0921bf0476e3ba14069d291.jpg)


# kafka

## 运维CMD

```bash
# 创建主题 --partition 指定分区个数，--replication-factor 指定分区总副本数
bin/kafka-topics.sh --bootstrap-server <broker_host:port> --create --topic <my_topic_name> --partitions 1 --replication-factor 1

# 主题 list
bin/kafka-topics.sh --bootstrap-server <broker_host:port> --list

# 主题详细
bin/kafka-topics.sh --bootstrap-server <broker_host:port> --describe --topic <topic_name>
# bin/kafka-topics.sh --zookeeper <broker_host:port> --describe --topic <topic_name>

# 生产消息
```bash
bin/kafka-console-producer.sh --broker-list <kafka-host:port> --topic <test-topic> --request-required-acks -1 --producer-property compression.type=lz4
>

# 消费
bin/kafka-console-consumer.sh --bootstrap-server <kafka-host:port> --topic <test-topic> --group <test-group> --from-beginning --consumer-property enable.auto.commit=false

# 查看消费情况
bin/kafka-consumer-groups.sh --bootstrap-server <ip:port> --describe --group <group_name> 
```



## client 客户端

### 为什么分区

1. 主要是为了负载均衡，
2. 容灾处理，方便扩展，提高吞吐量

### partition 分区

分区策略：消息发送至哪个分区；默认两种：
1. 轮询
2. 按key指定
3. 地理位置分区

支持用户自定义分区插件

### 消费位移 __consumer_offsets

Consumer 端有个位移的概念，它和消息在分区中的位移不是一回事儿，虽然它们的英文都是 Offset。今天我们要聊的位移是 Consumer 的消费位移，它记录了 Consumer 要消费的**下一条**消息的位移。假设一个分区中有 10 条消息，位移分别是 0 到 9。某个 Consumer 应用已消费了 5 条消息，这就说明该 Consumer 消费了位移为 0 到 4 的 5 条消息，此时 Consumer 的位移 offset 是 5，指向了下一条消息的位移。

想想 consumer 初始化时，位移是 0，即将消费的消息offset则是 0

内部topic: __consumer_offsets


老版本位移管理依托于 ZK，将 group 每个 partition 消费位移数据提交到 ZK，这样式的 broker 不需要保存唯一数据，减少 broker 的状态

新版本 Consumer 的位移管理机制其实也很简单，就是将 Consumer 的位移数据作为一条条普通的 Kafka 消息，提交到 __consumer_offsets 中。可以这么说，__consumer_offsets 的主要作用是保存 Kafka 消费者的位移信息。它要求这个提交过程不仅要实现高持久性，还要支持高频的写操作。显然，Kafka 的主题设计天然就满足这两个条件，因此，使用 Kafka 主题来保存位移这件事情，实际上就是一个水到渠成的想法了。

位移主题的 Key 中应该保存 3 部分内容：<Group ID，主题名，分区号 >。

### rebalance
协调 consumer group 中 consumer 消费 topic 中的 partition

触发rebalance的情况：
1. consumer数量变更
2. 分区数partition变化
3. group 订阅主题数变更

在 Rebalance 过程中，所有 Consumer 实例都会停止消费，等待 Rebalance 完成。这是 Rebalance 为人诟病的一个方面。

coordinator 负责 rebalance。 所有 Broker 都有各自的 Coordinator 组件。那么，Consumer Group 如何确定为它服务的 Coordinator 在哪台 Broker 上呢？答案就在我们之前说过的 Kafka 内部位移主题 __consumer_offsets 身上。目前，Kafka 为某个 Consumer Group 确定 Coordinator 所在的 Broker 的算法有 2 个步骤。
- 第 1 步：确定由位移主题 __consumer_offsets 的哪个分区来保存该 Group 数据：partitionId=Math.abs(groupId.hashCode() % offsetsTopicPartitionCount)。
- 第 2 步：找出该分区数据副本 Leader 所在的 Broker，该 Broker 即为对应的 Coordinator。

当 Consumer Group 完成 Rebalance 之后，每个 Consumer 实例都会定期地向 Coordinator 发送心跳请求，表明它还存活着。如果某个 Consumer 实例不能及时地发送这些心跳请求，Coordinator 就会认为该 Consumer 已经“死”了，从而将其从 Group 中移除，然后开启新一轮 Rebalance。

Consumer 还提供了一个允许你控制发送心跳请求频率的参数，就是 `heartbeat.interval.ms`。这个值设置得越小，Consumer 实例发送心跳请求的频率就越高。频繁地发送心跳请求会额外消耗带宽资源，但好处是能够更加快速地知晓当前是否开启 Rebalance，因为，目前 Coordinator 通知各个 Consumer 实例开启 Rebalance 的方法，就是将 REBALANCE_NEEDED 标志封装进心跳请求的响应体中。

 - 设置 session.timeout.ms = 6s。
 - 设置 heartbeat.interval.ms = 2s。

保证 Consumer 实例在被判定为“dead”之前，能够发送至少 3 轮的心跳请求，即 session.timeout.ms >= 3 * heartbeat.interval.ms。

除了以上两个参数，Consumer 端还有一个参数，用于控制 Consumer 实际消费能力对 Rebalance 的影响，即 max.poll.interval.ms 参数。它限定了 Consumer 端应用程序两次调用 poll 方法的最大时间间隔。它的默认值是 5 分钟，表示你的 Consumer 程序如果在 5 分钟之内无法消费完 poll 方法返回的消息，那么 Consumer 会主动发起“离开组”的请求，Coordinator 也会开启新一轮 Rebalance。

consumer 要把控好每条消息的消费耗时

| 消费者端重平衡流程

分别是 `加入组` 和 `等待领导者消费者（Leader Consumer）分配方案`。这两个步骤分别对应两类特定的请求：JoinGroup 请求和 SyncGroup 请求。

重平衡时需要从消费者实例中选择一个leader，让leader来发起重平衡方案，那为啥不直接让协调者组件来处理呢？
作者回复: 客户端自己确定分配方案有很多好处。比如可以独立演进和上线，不依赖于服务器端


### consumer TCP 连接过程

1. 确定协调者和获取集群元数据。
2. 连接协调者，令其执行组成员管理操作。
3. 执行实际的消息获取。

当第三类 TCP 连接成功创建后，消费者程序就会废弃第一类 TCP 连接，之后在定期请求元数据时，它会改为使用第三类 TCP 连接。也就是说，最终你会发现，第一类 TCP 连接会在后台被默默地关闭掉。对一个运行了一段时间的消费者程序来说，只会有后面两类 TCP 连接存在。

### 消费进度监控

consumer lag, lag 是分区上的概念，指的是当前分区还有多少条 message 没被消费

## kafka 服务端

### ZK 作用

保存集群元数据信息；

提供协调者服务；

提供集群broker管理服务；

### 分区副本管理 In-sync Replicas

追随者副本是不对外提供服务的。副本是分区下的概念

In-sync Replicas

Unclean Leader Election）

### 副本同步中的水位与epoch

| 高水位(HW High Watermark) 低水位（Low Watermark）

在 Kafka 中，高水位的作用主要有 2 个。
- 定义消息可见性，即用来标识分区下的哪些消息是可以被消费者消费的。
- 帮助 Kafka 完成副本同步。

![](https://static001.geekbang.org/resource/image/45/db/453ff803a31aa030feedba27aed17ddb.jpg)

高水位上的数据不可见，也不能被消费。分区HW以下的消息是已提交消息commited，consumer只能消费已提交的消息

| 日志末端位移的概念，即 Log End Offset，简写是 LEO。它表示副本将写入下一条消息的位移值。

| 高水位更新

![](https://static001.geekbang.org/resource/image/80/cb/8066e72733f14d2732a054ed56e373cb.jpg)

1. leader 收到 producer 一条消息，0上有数据了，Leo变成1(下一条消息写到1上)；
2. follower 来拉数据，leader告知0上的数据，follower 便也更新自己Leo为1
3. follower 再来拉数据，并告知上次0上数据拉取成功，leader则将Remote Leo 也更新为1，于是0上数据同步到follower成功，0 对消费者可见了，则leader将HW更新为1，同时告知follower也更新自己的HW，至此双方 HW == LEO == 1

| leader epoch

所谓 Leader Epoch，我们大致可以认为是 Leader 版本。它由两部分数据组成：
Epoch。一个单调增加的版本号。每当副本领导权发生变更时，都会增加该版本号。小版本号的 Leader 被认为是过期 Leader，不能再行使 Leader 权力。
起始位移（Start Offset）。Leader 副本在该 Epoch 值上写入的首条消息的位移。

leader切换时，会增加epoch版本号


### 控制器 controller

集群中任意一台 Broker 都能充当控制器的角色，但是，在运行过程中，只能有一个 Broker 成为控制器，行使其管理和协调的职责。换句话说，每个正常运转的 Kafka 集群，在任意时刻都有且只有一个控制器。

控制器中到底保存的数据，其实在 ZooKeeper 中也保存了一份。每当控制器初始化时，它都会从 ZooKeeper 上读取对应的元数据并填充到自己的缓存中。

有了这些数据，控制器就能对外提供数据服务了。这里的对外主要是指对其他 Broker 而言，控制器通过向这些 Broker 发送请求的方式将这些数据同步到其他 Broker 上。

ZooKeeper 本身的 API 提供了同步写和异步写两种方式。之前控制器操作 ZooKeeper 使用的是同步的 API，性能很差，集中表现为，当有大量主题分区发生变更时，ZooKeeper 容易成为系统的瓶颈。

## kafka 流处理

| 流处理与批处理

流处理平台（Streaming System）是处理无限数据集（Unbounded Dataset）的数据处理引擎，而流处理是与批处理（Batch Processing）相对应的。

kafka Steam 业务自定义算子处理数据，利用kafak批量消息事务性
![](https://static001.geekbang.org/resource/image/09/51/09e4dc28ea338ee69b5662596c0b8751.jpg)