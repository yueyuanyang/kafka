## kafka监控

**kafka监控的两种方法**

- JMX监控
- 第三方监控
- cmdline-jmxclient(命令行)

## 第一种：JMX 监控

Kafka可以配置使用JMX进行运行状态的监控，既可以通过JDK自带Jconsole来观察结果，也可以通过Java API的方式来.

### 步骤一：开启JMX端口

修改bin/kafka-server-start.sh，添加JMX_PORT参数，添加后样子如下

```
if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
    export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
    export JMX_PORT="9999"
fi
```
或者
> JMX_PORT=9999 bin/kafka-server-start.sh -daemon config/server.properties

### 步骤二：通过Jconsole测试时候可以连接

![p1](https://github.com/yueyuanyang/kafka/blob/master/guide/img/p1.png)

![p2](https://github.com/yueyuanyang/kafka/blob/master/guide/img/p2.png)


## 第二种：java 代码实现

```

public class KafkaDataProvider{
    protected final Logger LOGGER = LoggerFactory.getLogger(getClass());
    private static final String MESSAGE_IN_PER_SEC = "kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec";
    private static final String BYTES_IN_PER_SEC = "kafka.server:type=BrokerTopicMetrics,name=BytesInPerSec";
    private static final String BYTES_OUT_PER_SEC = "kafka.server:type=BrokerTopicMetrics,name=BytesOutPerSec";
    private static final String PRODUCE_REQUEST_PER_SEC = "kafka.network:type=RequestMetrics,name=RequestsPerSec,request=Produce";
    private static final String CONSUMER_REQUEST_PER_SEC = "kafka.network:type=RequestMetrics,name=RequestsPerSec,request=FetchConsumer";
    private static final String FLOWER_REQUEST_PER_SEC = "kafka.network:type=RequestMetrics,name=RequestsPerSec,request=FetchFollower";
    private static final String ACTIVE_CONTROLLER_COUNT = "kafka.controller:type=KafkaController,name=ActiveControllerCount";
    private static final String PART_COUNT = "kafka.server:type=ReplicaManager,name=PartitionCount";
    public String extractMonitorData() {
        //TODO 通过调用API获得IP以及参数
        KafkaRoleInfo monitorDataPoint = new KafkaRoleInfo();
        String jmxURL = "service:jmx:rmi:///jndi/rmi://192.168.40.242:9999/jmxrmi";
        try {
            MBeanServerConnection jmxConnection = MetricDataUtils.getMBeanServerConnection(jmxURL);
            ObjectName messageCountObj = new ObjectName(MESSAGE_IN_PER_SEC);
            ObjectName bytesInPerSecObj = new ObjectName(BYTES_IN_PER_SEC);
            ObjectName bytesOutPerSecObj = new ObjectName(BYTES_OUT_PER_SEC);
            ObjectName produceRequestsPerSecObj = new ObjectName(PRODUCE_REQUEST_PER_SEC);
            ObjectName consumerRequestsPerSecObj = new ObjectName(CONSUMER_REQUEST_PER_SEC);
            ObjectName flowerRequestsPerSecObj = new ObjectName(FLOWER_REQUEST_PER_SEC);
            ObjectName activeControllerCountObj = new ObjectName(ACTIVE_CONTROLLER_COUNT);
            ObjectName partCountObj = new ObjectName(PART_COUNT);
            Long messagesInPerSec = (Long) jmxConnection.getAttribute(messageCountObj, "Count");
            Long bytesInPerSec = (Long) jmxConnection.getAttribute(bytesInPerSecObj, "Count");
            Long bytesOutPerSec = (Long) jmxConnection.getAttribute(bytesOutPerSecObj, "Count");
            Long produceRequestCountPerSec = (Long) jmxConnection.getAttribute(produceRequestsPerSecObj, "Count");
            Long consumerRequestCountPerSec = (Long) jmxConnection.getAttribute(consumerRequestsPerSecObj, "Count");
            Long flowerRequestCountPerSec = (Long) jmxConnection.getAttribute(flowerRequestsPerSecObj, "Count");
            Integer activeControllerCount = (Integer) jmxConnection.getAttribute(activeControllerCountObj, "Value");
            Integer partCount = (Integer) jmxConnection.getAttribute(partCountObj, "Value");
            monitorDataPoint.setMessagesInPerSec(messagesInPerSec);
            monitorDataPoint.setBytesInPerSec(bytesInPerSec);
            monitorDataPoint.setBytesOutPerSec(bytesOutPerSec);
            monitorDataPoint.setProduceRequestCountPerSec(produceRequestCountPerSec);
            monitorDataPoint.setConsumerRequestCountPerSec(consumerRequestCountPerSec);
            monitorDataPoint.setFlowerRequestCountPerSec(flowerRequestCountPerSec);
            monitorDataPoint.setActiveControllerCount(activeControllerCount);
            monitorDataPoint.setPartCount(partCount);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (MalformedObjectNameException e) {
            e.printStackTrace();
        } catch (AttributeNotFoundException e) {
            e.printStackTrace();
        } catch (MBeanException e) {
            e.printStackTrace();
        } catch (ReflectionException e) {
            e.printStackTrace();
        } catch (InstanceNotFoundException e) {
            e.printStackTrace();
        }
        return monitorDataPoint.toString();
    }
    public static void main(String[] args) {
        System.out.println(new KafkaDataProvider().extractMonitorData());
    }
    /**
     * 获得MBeanServer 的连接
     *
     * @param jmxUrl
     * @return
     * @throws IOException
     */
    public MBeanServerConnection getMBeanServerConnection(String jmxUrl) throws IOException {
        JMXServiceURL url = new JMXServiceURL(jmxUrl);
        JMXConnector jmxc = JMXConnectorFactory.connect(url, null);
        MBeanServerConnection mbsc = jmxc.getMBeanServerConnection();
        return mbsc;
    }
}

```

### 第三种：其他第三方工具

除了自己编写定制化的监控程序外
```
kafka-web-console
https://github.com/claudemamo/kafka-web-console
部署sbt：
http://www.scala-sbt.org/0.13/tutorial/Manual-Installation.html
http://www.scala-sbt.org/release/tutorial/zh-cn/Installing-sbt-on-Linux.html

KafkaOffsetMonitor
https://github.com/quantifind/KafkaOffsetMonitor/releases/tag/v0.2.0
java -cp KafkaOffsetMonitor-assembly-0.2.0.jar com.quantifind.kafka.offsetapp.OffsetGetterWeb --zk localhost:12181 --port 8080 --refresh 5.minutes --retain 1.day

Mx4jLoader
```

### 第四种：命令行的形式来查看某项数据

也可以通过命令行的形式来查看某项数据，不过这里要借助一个jar包：cmdline-jmxclient-0.xx.3.jar，这个请自行下载，网上很多。 将这个jar放入某一目录，博主这里放在了linux系统下的/root/util目录中，以offset举例： 

0.8.1.x版-读取topic=default_channel_kafka_zzh_demo,partition=0的Value值：

```
java -jar cmdline-jmxclient-0.10.3.jar - xx.101.130.1:9999 
'"kafka.log":type="Log",name="default_channel_kafka_zzh_demo-0-LogEndOffset"' Value
```

0.8.2.x版-读取topic=default_channel_kafka_zzh_demo,partition=0的Value值：

```
java -jar cmdline-jmxclient-0.10.3.jar - xx.101.130.1:9999 kafka.log:type=Log,name=LogEndOffset,topic=default_channel_kafka_zzh_demo,partition=0
```
## JMX 业务标准

#### broker的度量指标

#### 1.对应的非同步分区
表1： 度量指标和对应的非同步分区

| 度量指标名称 | under-replicated partitions |
| - | :-: |
| JMX MBean | kafka.server:type=ReplicaManager,name=UnderReplicatedPartition|
| 值域区 | 非负整数 |


#### 2. 活跃控制器数量

该指标表示broker是否就是当前的集群控制器，其值可以是0或1。如果1，表示broker就是当前的控制器。任何时候都只有一个控制器，而且这个broker就是当前的控制器。如果出现了两个控制器，说明一个本该退出的线程被阻塞

表2： 度量指标和对应的非同步分区

| 度量指标名称 | Active controll count |
| - | :-: |
| JMX MBean | kafka.controller:type=kafkaController,name=ActiveControllerCount |
| 值域区 | 0或1 |

#### 3. 请求处理器空闲率
kafka使用两个线程处理客户端的请求

- 网络处理器线程池：负责网络的读入和写出数据，没有太多工作，不用担心出问题
- 请求处理器线程池:接受来自客户端的请求，包括：从磁盘读取消息和往磁盘写入消息

表3： 请求处理器空闲率
| 度量指标名称 | Request handler average idle percentage |
| - | :-: |
| JMX MBean | kafka.server:type=kafkaRequestHandlerPool,name=RequestHandlerAvglePercent |
| 值域区 | 从0到1的浮点数(包括1在内) |

请求处理器平均空闲百分比这个度量表示请求处理器空闲时间的百分比。数值越低说明broker的负载越高。经验表明，如果空闲比低于20%，说明存在潜在的为题，如果低于10%说明性能问题



