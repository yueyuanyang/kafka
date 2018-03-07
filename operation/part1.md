## kafka 数据迁移实践
## 介绍kafka的两类常见数据迁移方式

- 1、broker内部不同数据盘之间的分区数据迁移

- 2、不同broker之间的分区数据迁移

### 一、broker 内部不同数据盘之间进行分区数据迁移

如果kafka broker内部的topic分区数据存储分布不均匀，导致部分磁盘100%耗尽，而部分磁盘只有40%的消耗量。

**分析原因**：发现存在部分topic的分区数据过于集中在某些磁盘导致，比如，以下截图显示的/data5 数据盘

> df -h

根据分布式系统的特点，很容易想到采取数据迁移的办法，对broker内部不同数据盘的分区数据进行迁移。在进行线

上集群数据迁移之前，为了保证生产集群的数据完整和安全，必须先在测试集群进行测试。

#### 1.2 测试broker内部不同数据盘进行分区数据迁移

1.2.1 建立测试topic并验证生产和消费正常

我们搭建的测试集群，Kafka 有三个broker，hostname分别为：

- vm-gs-201
- vm-gs-202
- vm-gs-203

每个broker配置了两块数据盘，缓存数据分别存储在 /data/kafka-logs/ 和 /data1/kafka-logs/。

首先建立测试topic：

```
./kafka-topics.sh --create --zookeeper vm-gs-201:2181 --replication-factor 2 --partitions 3 --topic test_topic
```

然后向topic生产发送500条数据，发送的时候也同时消费数据。然后查看topic的分区数据情况：

```

```

