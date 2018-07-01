## Consumer Group

1) High Level Consumer 将从某个Partition是读取最火一条消息的offer存于zookeeper(从0.8.2开始同时支持将offset存于zookeeper中和专用的kafka topic中)

2) 这个offset基于客户程序提供的kafka的名字来保存，这个名字呗称为consumer group

3) consumer Group是整个kafka集群全局唯一的，而非针对某个Topic的

4) 每个High Level Consumer实例都属于一个consumer Group,若不指定则玉玉默认Group

### offset 存储方式
![kakfa_5]()

1) 每次提交offset都会记录一条数据，导致offset文件过多，kafka 采用compaction方式压缩
2) 将相同的key的offset压缩保留最后（最大）的offset的值

### 消费者以消费组(consumer group为单位进行消费)

- 1) 一个topic可以被多个消费组消费，一个消费组可以消费多个topic
- 2) 消息被消费后，并不会被删除，只是相应的offset加一
- 3) 对于每个消息，在同一个Consumer Group里只会被一个Consumer消费
- 4) 不同consumer Group可消费同一个消息
- 5) 一个消息在同一Group中是单播的，在不同的group中是多播的

![kakfa_6]()

### kafka 使用场景
- 1) kafka 的设计理念之一就是同时提供对离线处理和在线流处理
- 2） 可同时使用Hadoop系统进行离线处理，strom或其他处理系统进行流处理
- 3）可使用kafka 的mirror Maker 将消息从一个数据中心镜像到另一个数据中心

![kakfa_7]()

### High Level consumer Rebalance
consumer Reabalance算法
- 1)将目标Topic下的所有Partition排序，存于Pt
- 2)对于consumer Group 下所有consumer排序，存于Cg,第i个consumer记为Ci
- 3)N=size(Pt)/size(Cg),向上取整
- 4)解除Ci对原来分配的Partition的消费权(i从0开始) 
- 5)将第ixN到N-1个partition分配给Ci


