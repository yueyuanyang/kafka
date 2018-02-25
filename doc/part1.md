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
![kafka_1](http3://github.com/yueyuanyang/kafka/tree/master/img/kafka_1.jpg)
![kafka_2](https://github.com/yueyuanyang/kafka/tree/master/img/kafka_2.jpg)
![kafka_3](https://github.com/yueyuanyang/kafka/tree/master/img/kafka_3.jpg)
