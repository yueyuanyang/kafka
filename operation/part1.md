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
./kafka-consumer-offset-checker.sh --zookeeper vm-gs-201:2181 --topic test_topic --group group1 --broker-info
```

发现test_topic生产和消费数据都正常。

1.2.2 将分区数据在磁盘间进行迁移

现在登录tbds-172-16-16-12这台broker节点，将test_topic的分区数据目录 /data1/kafka-logs/test_topic-0/ 移动到 /data/kafka-logs/ ：
```
mv /data1/kafka-logs/test_topic-0/ /data/kafka-logs/
```

查看 /data/kafka-logs/ 目录下，分区test_topic-0 的数据：

1.2.3 重启kafka

重启kafka集群，重启完成后，发现vm-gs-201这台broker节点的编号为0的分区缓存数据目录内的数据也增加到正常水平。

1.3 结论

Kafka broker 内部不同数据盘之间可以自由迁移分区数据目录。迁移完成后，重启kafka即可生效。

###  二、不同broker之间传输分区数据

当对kafka集群进行扩容之后，由于新扩容的broker没有缓存数据，容易造成系统的数据分布不均匀。因此，需要将原来集群broker的分区数据迁移到新扩容的broker节点。

不同broker之间传输分区数据，可以使用kafka自带的kafka-reassign-partitions.sh脚本工具实现。

我们在kafka测试集群原有的3台broker基础上，扩容1台broker。

2.1 获取test_topic的分区分布情况

执行命令：
```
./kafka-topics.sh --zookeeper vm-gs-201:2181 --topic test_topic --describe
```

2.2 获取topic重新分区的配额文件

编写分配脚本：move_kafka_topic.json内容如下：

{"topics": [{"topic":"test_topic"}], "version": 1}

执行分配计划生成脚本：

```
./kafka-reassign-partitions.sh --zookeeper vm-gs-201:2181 --topics-to-move-json-file 
/tmp/move_kafka_topic.json --broker-list "1001,1002,1003,1004" --generate
```
命令里面的broker-list填写kafka集群4个broker的id。不同kafka集群，因为部署方式不一样，选择的broker id也不一样。我们的测试集群broker id是1001,1002,1003,1004。读者需要根据自己的kafka集群设置的broker id填写。

2.3 对topic分区数据进行重新分布

执行重新分配命令：

```
./kafka-reassign-partitions.sh --zookeeper vm-gs-201:2181 --reassignment-json-file /tmp/move_kafka_topic_result.json --execute
```

2.4 查看分区数据重新分布进度

检查分配的状态，执行命令：

```
./kafka-reassign-partitions.sh --zookeeper vm-gs-201:2181 --reassignment-json-file /tmp/move_kafka_topic_result.json --verify
```

2.5 再次获取test_topic的分区分布情况

```
./kafka-topics.sh --zookeeper vm-gs-201:2181 --topic test_topic --describe
```

### 三、测试结论

- Kafka broker 内部不同数据盘之间可以自由迁移分区数据目录。迁移完成后，重启kafka即可生效；

- Kafka 不同broker之前可以迁移数据，使用kafka自带的kafka-reassign-partitions.sh脚本工具实现。


> https://www.toutiao.com/a6513706403587162638/tt_from=mobile_qq&utm_campaign=client_share&timestamp=1520390042&app=news_article&utm_source=mobile_qq&iid=27068802467&utm_medium=toutiao_ios


