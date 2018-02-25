##  kafka 简介
### kafka概念
/.,mnbvcxz 
### kafka特点
- 同时为帆布和订阅提供高吞吐量。kafk每秒可以生产约25万消息(50MB)每秒处理55万消息(110MB)
- 可机型持久化，将消息持久化到磁盘。因此，可用于批量消费，例如ETL，以及实时应用程序。通过数据持久化到磁盘以及replication防止数据丢失
- 分布式系统，易于向外扩展。所有product,broker和consumer都会有多个，均为分布式的，无需停机即可扩展机器
- 消息被处理的状态在consumer端维护，而不是由server端维护，当失败是能自动平衡
- 支持online和offline

注：borker ： 只负责消息的存储和删除

   consumer ： 负责消费状态的维护offSet
   
   zookeeper: 负责路由和负载均衡
   
### kafka 架构
入学习Kafka之前，必须了解主题（Topic）、经纪人（Broker）、生产者（Producer）或者发布者，以及消费者（Consumer）或者订阅者等主要术语。 下图说明了主要术语，表格详细描述2图表组件。

![kafka_1](https://github.com/yueyuanyang/kafka/blob/master/img/kafka_1.jpg)

![kafka_2](https://github.com/yueyuanyang/kafka/blob/master/img/kafka_2.jpg)

![kafka_3](https://github.com/yueyuanyang/kafka/blob/master/img/kafka_3.jpg)

### kafka 核心概念
（1）Topics（主题） 
属于特定类别的消息流称为主题。 数据存储在主题中。Topic相当于Queue。 
主题被拆分成分区。 每个这样的分区包含不可变有序序列的消息。 分区被实现为具有相等大小的一组分段文件。 
（2）Partition（分区） 
- 一个Topic可以分成多个Partition，这是为了平行化处理。
- 每个Partition内部消息有序，其中每个消息都有一个offset序号。
- 一个Partition只对应一个Broker，一个Broker可以管理多个Partition。
（3）Partition offset（分区偏移） 
每个分区消息具有称为 offset 的唯一序列标识。 
（4）Replicas of partition（分区备份） 
副本只是一个分区的备份。 副本从不读取或写入数据。 它们用于防止数据丢失。 
（5）Brokers（经纪人）
- 代理是负责维护发布数据的简单系统。 每个代理可以每个主题具有零个或多个分区。 假设，如果在一个主题和N个代理中有N个分区，每个代理将有一个分区。
- 假设在一个主题中有N个分区并且多于N个代理(n + m)，则第一个N代理将具有一个分区，并且下一个M代理将不具有用于该特定主题的任何分区。
- 假设在一个主题中有N个分区并且小于N个代理(n-m)，每个代理将在它们之间具有一个或多个分区共享。 由于代理之间的负载分布不相等，不推荐使用此方案。

（6）Kafka Cluster（Kafka集群） 
Kafka有多个代理被称为Kafka集群。 可以扩展Kafka集群，无需停机。 这些集群用于管理消息数据的持久性和复制。 
（7）Producers（生产者） 
生产者是发送给一个或多个Kafka主题的消息的发布者。 生产者向Kafka经纪人发送数据。 每当生产者将消息发布给代理时，代理只需将消息附加到最后一个段文件。实际上，该消息将被附加到分区。 生产者还可以向他们选择的分区发送消息。 
（8）Consumers（消费者） 
Consumers从经纪人处读取数据。 消费者订阅一个或多个主题，并通过从代理中提取数据来使用已发布的消息。
- Consumer自己维护消费到哪个offet
- 每个Consumer都有对应的group
- group内是queue消费模型：各个Consumer消费不同的partition，因此一个消息在group内只消费一次
- group间是publish-subscribe消费模型：各个group各自独立消费，互不影响，因此一个消息被每个group消费一次。


