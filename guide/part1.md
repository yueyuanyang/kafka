## kafka 命令整理

### 一、主题操作

### 1. 创建主题(--create)

涉及到的参数：

1) 主题的名字

2) 复制系数

3) 分区

**示例：创建一个test主题，副本为2，分区数为8**
```
kafka-topic.sh --zookeeper 10.0.0.201:2181 --create --topic test --replication-factor 2 --partitions 3
```
#### 2. 增加分区(--alter)

**示例：(--alter) 将test主题的分区数从8个扩展到16**

```
kafka-topic.sh --zookeeper 10.0.0.201:2181 --alter --topic test --partition 16
```

#### 3. 删除主题(--delete)

1) 为了删除主题。broker的delete.topic.enable参数设置为true。如果设置为false，删除主题将被忽略

示例：删除test 主题
```
kafka-topic.sh --zookeeper 10.0.0.201:2181 --delete --topic test
```
#### 4. 列出集群里的所有主题(--list)

示例：列出集群的所有主题列表
```
kafka-topics.sh --zookeeper 10.0.0.201:2181 --list
```
#### 5. 列出主题详细信息(--describe)

1) 信息里包含分区数据、主题的覆盖以及每个分区的副本清单

2) --topic 参数指定特定的主题，就可以列出主题的详细信息

**示例**：
```
kafka-topics.sh --zookeeper 10.0.0.201:2181 --describe
```
3) describe命令提供了一些参数，用于过滤结果，诊断集群问题时会很有用，但无法与--list命令一起使用

--topics-with-override 参数可以找出所有包含覆盖的主题，只会列出包含了与集群不一样配置的主题。

--under-replicated-partitions 参数列出所有包含不同副本的分区

--unavailable-partitions 列出所有没有首领的分区

**示例:列出包含不同副本的分区**
```
kafka-topics.sh --zookeeper 10.0.0.201:2181 --describe --unavailable-partitions
```

### 二、消费者群组(kafka-consumer-groups)

旧版本：信息保存在zookeeper上,需要通过--zookeeper指定zookeeper地址。

新版本：信息保存在broker上,需要通过--bootstrap-server参数指定broker的主机名和端口。

#### 1.列出并描述群组

旧版本：--zookeeper --list

新版本: --bootstrap-server --list和--new-consumer

**示例：列出旧版本的消费者群组**
```
kafka-consumer-groups.sh  --zookeeper 10.0.0.201:2181 --list
````
**示例:列出新版本的消费者群组**
```
kafka-consumer-groups.sh  --new-consumer --bootstrap-server 10.0.0.202:9092 --list
```
#### 2. 获取特定消费组的详细信息

获取旧版本的消费群组test的详细信息
```
kafka-consumer-groups.sh --zookeeper 10.0.0.201:2181 --describe --group group1
```
**表1：输出结果中的字段**

GDOUP  消费者群组的名字

TOPIC  正在被读取的主题名字

PARTITION  正在被读取的分区ID

CURRENT-OFFSET 消费者群组最近提交的偏移量，也就是消费者在分区里读取的当前位置

LOG-END-OFFSET 当前高水位便宜量，也就是最近一个被读取消息的偏移量，同时也是最近一个被提交集群的偏移量

LAG 消费者的CURRENT-OFFSET和LOG-END-OFFSET之间的差距

OWNER 消费着群组里正在读取该分区的消费者，这是一个消费者的ID，不一定包括消费着的主机名。

#### 3.删除群组

注意：只有旧版本的消费者客户端才支持删除群组操作。删除前关闭所有的消费者

**示例:删除消费者群组test**
```
kafa-consumer-groups.sh --zookeeper 10.0.0.201:2181 --delete --group group1
```
**示例: 从消费者群组test里删除my-topic主题的偏移量**
```
kafa-consumer-groups.sh --zookeeper --group test --topic my-topic
```
#### 4. 偏移量管理

1) 导出偏移量

**示例：将群组test的偏移量导出到offsets文件里**
```
kafka-run-class.sh kafka.tools.ExportZkOffsets --zkconnect 10.0.0.201:2181 --group test --output-file offsets
```
2)导入偏移量

可以修改偏移量，不需要添加group，文件中有group信息
```
kafka-run-class.sh kafka.tools.ImportZkOffsets --zkconnect 10.0.0.201:2181 --input-file offsets
```
#### 三、动态配置变更

我们可以在集群处于运行状态时覆盖和客户端的配额参数

1) 更改主题配置的默认值
```
kafka-configs.sh --zookeeper 10.0.0.201:2181 --alter --entity-type topics --entity-name <topic name> --add-config <key><value>
```
**示例:将主题test的消息保留时间设为1h(3600 000ms)**
```
kafka-configs.sh --zookeeper 10.0.0.201:2181 --alter --entity-type topics --entity-name test --add-config retention.ms=3600000
```
2) 覆盖客户端的默认配置
```
kafka-configs.sh --zookeeper 10.0.0.201:2181 --alter --entity-type client --entity-name <client ID> --add-config <key><value>
```
可以客户端配置参数

producer_bytes_rate 单个生产者每秒钟可以往单个broker上生产的消息字节数

consumer_bytes_rate 单个消费者每秒可以从单个broker读取的消息字节数

#### 3) 列出被覆盖的配置

**示例:列出主题my-topic所有被覆盖的配置**
```
kafka-configs.sh --zookeeper 10.0.0.201:2181 --describe --entity-type topics --entity-name my-topic
```
#### 4) 移除被覆盖的配置

动态配置的完全可以被移除
```
kafka-configs.sh --zookeeper 10.0..0.201:2181 --alter --entity-type topics --entity-name my-topic  --delete-config retention.ms=3600000
```

### 四、分区管理

kafka提供了两个工具管理分区：一个用于重新选举首领。另一个用于将分区分配给broker.

结合使用两个工具，就可以实现集群流量的负载均衡。

**示例：在一个包含1个主题和8个分区的集群中启动首选的副本选举**
```
kafka-preferred-replica-election.sh --zookeeper 10.0.0.201:2181
```
如果分区的节点的元数据过大，这是将分区清单的信息写到一个JSON文件中，并将请求分为多个步骤进行

{
   "aprtitions":{
       {
	      "partition":1,
		  "topic":"foo"
	   },
	   {
	       "partition":2,
		   "topic":"foobar"
	   }
   }
}

**示例:通过在partitions.json中文件里指定分区清单来启动副本选举**
```
kafka-preferred-replica-election.sh --zookeeper 10.0.0.201:2181 --path-to-json-file partitions.json
```
2) 修改分区副本

情况：
- 1) 主题分区在整个集群中里的不均衡分布造成集群负载的不均衡
- 2) broker离线造成分区不同步
- 3) 新加入的broker需要从集群获取负载

kafka-reassign-partition.sh 工具修改分区

**步骤**:

第一步:根据broker清单和主题清单生成一组迁移步骤

第二步:执行这些迁移步骤

第三步:(可选)使用生成的迁移验证分区分配的进度和完成情况

主题清单(topics.json)：

{
   "topics":[
       {
	       "topic":"foo"
	   },
	   {
	       "topic":"foo1"
	   }
   ],
   "version":1
}

**示例1：为topics.json文件里的主题生成迁移步骤，以便将这些主题迁移到broker0和broker1**
```
kafka-reassign-partitions.sh --zookeeper 10.0.0.201:2181 --generate --topics-to-move-json-file topics.json
--broker-list 0,1
```

结果：生成两个json文件

- 1)当前的分区分配情况(保留，用在回滚)
- 2)建议的分区分配方案(reassign.sh)

**示例2:使用reassign.json来执行建议的分配方案**
```
kafka-reassign-partitions.sh --zookeeper 10.0.0.201:2181 --execute --reassignment-json-file reassign.json
```

**示例3：完成分配后可以验证，验证reassign.json文件里指定的分区重分配情况**
```
kafka-reassign-partition.sh --zookeeper 10.0.0.201:2181 --verify --reassignment-json-file reassign.sh
```
3. 修改复制系数

{
   "partitions":[
    {
	   "topic" ："my-topic",
	   "partition":0,
	   "replicas":[
	     1,
		 2
	   ]
	}
   ],
   "verson":1
}

#### 4. 转存日志片段

**示例1：解码日志片段00000000023123.log显示消息的概要信息**
```
kafka-run-class.sh kafka.tools.DumpLogSegment --files 00000000023123.log
```

**示例2：解码日志片段00000000023123.log显示消息的内容**
```
kafka-run-class.sh kafka.tools.DumpLogSegment --files 00000000023123.log --print-data-log
```
**示例3：验证日志片段00000000023123.log索引文件的正确性**
```
kafka-run-class.sh kafka.tools.DumpLogSegment --files 00000000023123.log --index-sanity-check
```

**示例4：验证日志片段00000000023123.log索引文件的检查无用的索引(检查索引的匹配度,不会打印处所有的正确性)**
```
kafka-run-class.sh kafka.tools.DumpLogSegment --files 00000000023123.log --verify-index-only
```
#### 5. 副本验证

**示例1：对broker1和broker2上以my-开头的主题副本进行验证**
```
kafka-replica-verification.sh --broker-list 10.0.0.201:9092 --topic-white-list my-*
```

### 五、消费和生产

#### 1. 控制台消费

**示例1：使用旧版本消费者读取单个主题my-topic**
```
kafka-console-consumer.sh --zookeeper 10.0.0.201:2181 --topic my-topic
```
**示例2：使用新版本消费者读取单个主题my-topic**
```
kafka-console-consumer.sh --broker-list 10.0.0.201:9092,10.0.0.202:9092 --topic my-topic --new-consumer
```
参数：(下边两个后面跟一个正则表达式)

--white-list (白名单-读取的topic)

--black-list (黑名单-过滤的topic)

其他配置：

--formatter CLASSNAME 指定消息格式化器的类型，用于解码消息，它默认值时kafka.tools.defaultFormatter

--from-beginning  指定从最旧偏移量开始读取数据，否则就从最新的偏移量开始读取

--max-message NUM 指定在退出之前最多读取NUM个消息

--partition NUM  指定只读取ID为NUM的分区(需要新版本的消费者)

#### 2.读取偏移量主题
```
kafka-console-consumer.sh --zkconnect 10.0.0.201:2181 --topic __consumer_offset
--formatter 'kafka.coordinator.GroupMetadataManager$offsetsMessageFormatter' --max-message 1
```

#### 3. 控制台生产

**示例：向主题my-topic生成两个消息**
```
kafka-console-producer.sh --broker-list 10.0.0.201:9092 --topic my-topic
```

