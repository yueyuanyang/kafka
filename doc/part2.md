## kafka常用命令
### kafka 操作命令
以下是kafka常用命令行总结：
```
0.查看有哪些主题： ./kafka-topics.sh --list --zookeeper 127.0.0.1:12181

1.查看topic的详细信息
./kafka-topics.sh -zookeeper 127.0.0.1:2181 -describe -topic test

2、为topic增加副本
./kafka-reassign-partitions.sh -zookeeper 127.0.0.1:2181 -reassignment-json-file json/partitions-to-move.json 
-execute

3、创建topic
./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic testKJ1

4、为topic增加partition
./bin/kafka-topics.sh –zookeeper 127.0.0.1:2181 –alter –partitions 20 –topic test

5、kafka生产者客户端命令
./kafka-console-producer.sh --broker-list localhost:9092 --topic test

6、kafka消费者客户端命令
./kafka-console-consumer.sh -zookeeper localhost:2181 --from-beginning --topic test

7、kafka服务启动
./kafka-server-start.sh -daemon ../config/server.properties 

8、下线broker
./kafka-run-class.sh kafka.admin.ShutdownBroker --zookeeper 127.0.0.1:2181 --broker #brokerId# --num.retries 3 
--retry.interval.ms 60 shutdown broker

9、删除topic
./kafka-run-class.sh kafka.admin.DeleteTopicCommand --topic testKJ1 --zookeeper 127.0.0.1:2181
./kafka-topics.sh --zookeeper localhost:2181 --delete --topic testKJ1

10、查看consumer组内消费的offset
./kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --zookeeper localhost:2181 --group test --topic test
 ./kafka-consumer-offset-checker.sh --zookeeper 192.168.0.201:12181 --group group1 --topic group1
```

### kafka 管理工具命令
Kafka内部提供了许多管理脚本
```
1. 消费者偏移量检查(Consumer Offset Checker)
kafka-consumer-offset-checker.sh，会显示出Consumer的Group、Topic、分区ID、
分区对应已经消费的Offset、logSize大小，Lag以及Owner等信息
./kafka-consumer-offset-checker.sh --zookeeper 127.0.0.1:2181 --topic test --group group1 --broker-info
-------------------------------------------------------------------------------------
Group           Topic      Pid Offset          logSize         Lag             Owner
group1    test       0   34666914        34674392        7478            none
group1    test       1   34670481        34678029        7548            none
-------------------------------------------------------------------------------------

2. 导出日志(Dump Log Segment)
有时候我们需要验证日志索引是否正确，或者仅仅想从log文件中直接打印消息，我们可以使用 kafka.tools.DumpLogSegments类
来实现

./kafka-run-class.sh kafka.tools.DumpLogSegments --files /data/A/test-4/00000000000034245135.log

显示日志内容
./kafka-run-class.sh kafka.tools.DumpLogSegments --files /data/A/test-4/00000000000034245135.log --print-data-log

我们在使用kafka.tools.DumpLogSegments的时候必须输入--files，这个参数指的就是Kafka中Topic分区所在的绝对路径。
分区所在的目录由config/server.properties文件中log.dirs参数决定这个命令将Kafka中Message中Header的相关信息
和偏移量都显示出来了，但是没有看到日志的内容，我们可以通过--print-data-log来设置。如果需要查看多个日志文件，可以以逗号分割。

 2. 导出Zookeeper中Group相关的偏移量
 
 我们需要导出某个Consumer group各个分区的偏移量，我们可以通过使用Kafka的kafka.tools.ExportZkOffsets类来满足。
 我们需要输入Consumer group，Zookeeper的地址以及保存文件路径：
 
 ./kafka-run-class.sh kafka.tools.ExportZkOffsets --group spark --zkconnect 127.0.0.1:2181 --output-file ~/offset
 
 注意，--output-file参数必须在指定，否则会出错
 
 3. 通过JMX获取metrics信息
 
 我们可以通过kafka.tools.JmxTool类打印出Kafka相关的metrics信息
 
 bin/kafka-run-class.sh kafka.tools.JmxTool --jmx-url service:jmx:rmi:///jndi/rmi://www.silent.com:1099/jmxrmi
 
 运行上面命令前提是在启动kafka集群的时候指定export JMX_PORT= , 这样才会开启JMX。
 然后就可以通过上面命令打印出Kafka所有的metrics信息。
 
 4. Kafka数据迁移工具
 
这个工具主要有两个：
kafka.tools.KafkaMigrationTool和kafka.tools.MirrorMaker。第一个主要是用于将Kafka 0.7上面的数据迁移到Kafka0.8(https://cwiki.apache.org/confluence/display/KAFKA/Migrating+from+0.7+to+0.8);
而后者可以同步两个Kafka集群的数据(https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27846330)。
都是从原端消费Messages，然后发布到目标端。

./kafka-run-class.sh kafka.tools.KafkaMigrationTool --kafka.07.jar kafka-0.7.19.jar --zkclient.01.jar 
zkclient-0.2.0.jar --num.producers 16 --consumer.config=sourceCluster2Consumer.config 
--producer.config=targetClusterProducer.config --whitelist=.*
 
./kafka-run-class.sh kafka.tools.MirrorMaker --consumer.config sourceCluster1Consumer.config 
--consumer.config sourceCluster2Consumer.config --num.streams 2 
--producer.config targetClusterProducer.config --whitelist=".*"

5. 日志重放工具

这个工具主要作用是从一个Kafka集群里面读取指定Topic的消息，并将这些消息发送到其他集群的指定topic中：
 bin/kafka-replay-log-producer.sh 

6. Simple Consume脚本

kafka-simple-consumer-shell.sh 工具主要是使用Simple Consumer API从指定Topic的分区读取数据并打印在终端：

bin/kafka-simple-consumer-shell.sh --broker-list www.iteblog.com:9092 --topic test --partition 0

7. 更新Zookeeper中的偏移量

kafka.tools.UpdateOffsetsInZK 工具可以更新Zookeeper中指定Topic所有分区的偏移量，可以指定成 earliest或者latest：

./kafka-run-class.sh kafka.tools.UpdateOffsetsInZK

USAGE: kafka.tools.UpdateOffsetsInZK$ [earliest | latest] consumer.properties topic


需要指定是更新成earliest或者latest，consumer.properties文件的路径以及topic的名称

```

> http://blog.csdn.net/wuliusir/article/details/51062904
