## kafka配置文件详细

### 1. producer.properties
```
#指定kafka节点列表，用于获取metadata，不必全部指定
#需要kafka的服务器地址，来获取每一个topic的分片数等元数据信息。
<font color=#A52A2A size=7 face="黑体">metadata.broker.list=kafka01:9092,kafka02:9092,kafka03:9092</font>

#生产者生产的消息被发送到哪个block，需要一个分组策略。
#指定分区处理类。默认kafka.producer.DefaultPartitioner，表通过key哈希到对应分区
#partitioner.class=kafka.producer.DefaultPartitioner

#生产者生产的消息可以通过一定的压缩策略（或者说压缩算法）来压缩。消息被压缩后发送到broker集群，
#而broker集群是不会进行解压缩的，broker集群只会把消息发送到消费者集群，然后由消费者来解压缩。
#是否压缩，默认0表示不压缩，1表示用gzip压缩，2表示用snappy压缩。
#压缩后消息中会有头来指明消息压缩类型，故在消费者端消息解压是透明的无需指定。
#文本数据会以1比10或者更高的压缩比进行压缩。
compression.codec=none

#指定序列化处理类，消息在网络上传输就需要序列化，它有String、数组等许多种实现。
serializer.class=kafka.serializer.DefaultEncoder

#如果要压缩消息，这里指定哪些topic要压缩消息，默认empty，表示不压缩。
#如果上面启用了压缩，那么这里就需要设置
#compressed.topics= 
#这是消息的确认机制，默认值是0。在面试中常被问到。
#producer有个ack参数，有三个值，分别代表：
#（1）不在乎是否写入成功；
#（2）写入leader成功；
#（3）写入leader和所有副本都成功；
#要求非常可靠的话可以牺牲性能设置成最后一种。
#为了保证消息不丢失，至少要设置为1，也就
#是说至少保证leader将消息保存成功。
#设置发送数据是否需要服务端的反馈,有三个值0,1,-1，分别代表3种状态：
#0: producer不会等待broker发送ack。生产者只要把消息发送给broker之后，就认为发送成功了，这是第1种情况；
#1: 当leader接收到消息之后发送ack。生产者把消息发送到broker之后，并且消息被写入到本地文件，才认为发送成功，这是第二种情况；#-1: 当所有的follower都同步消息成功后发送ack。不仅是主的分区将消息保存成功了，
#而且其所有的分区的副本数也都同步好了，才会被认为发动成功，这是第3种情况。
request.required.acks=0

#broker必须在该时间范围之内给出反馈，否则失败。
#在向producer发送ack之前,broker允许等待的最大时间 ，如果超时,
#broker将会向producer发送一个error ACK.意味着上一次消息因为某种原因
#未能成功(比如follower未能同步成功)
request.timeout.ms=10000

#生产者将消息发送到broker，有两种方式，一种是同步，表示生产者发送一条，broker就接收一条；
#还有一种是异步，表示生产者积累到一批的消息，装到一个池子里面缓存起来，再发送给broker，
#这个池子不会无限缓存消息，在下面，它分别有一个时间限制（时间阈值）和一个数量限制（数量阈值）的参数供我们来设置。
#一般我们会选择异步。
#同步还是异步发送消息，默认“sync”表同步，"async"表异步。异步可以提高发送吞吐量,
#也意味着消息将会在本地buffer中,并适时批量发送，但是也可能导致丢失未发送过去的消息
producer.type=sync

#在async模式下,当message被缓存的时间超过此值后,将会批量发送给broker,
#默认为5000ms
#此值和batch.num.messages协同工作.
queue.buffering.max.ms = 5000

#异步情况下，缓存中允许存放消息数量的大小。
#在async模式下,producer端允许buffer的最大消息量
#无论如何,producer都无法尽快的将消息发送给broker,从而导致消息在producer端大量沉积
#此时,如果消息的条数达到阀值,将会导致producer端阻塞或者消息被抛弃，默认为10000条消息。
queue.buffering.max.messages=20000

#如果是异步，指定每次批量发送数据量，默认为200
batch.num.messages=500

#在生产端的缓冲池中，消息发送出去之后，在没有收到确认之前，该缓冲池中的消息是不能被删除的，
#但是生产者一直在生产消息，这个时候缓冲池可能会被撑爆，所以这就需要有一个处理的策略。
#有两种处理方式，一种是让生产者先别生产那么快，阻塞一下，等会再生产；另一种是将缓冲池中的消息清空。
#当消息在producer端沉积的条数达到"queue.buffering.max.meesages"后阻塞一定时间后,
#队列仍然没有enqueue(producer仍然没有发送出任何消息)
#此时producer可以继续阻塞或者将消息抛弃,此timeout值用于控制"阻塞"的时间
#-1: 不限制阻塞超时时间，让produce一直阻塞,这个时候消息就不会被抛弃
#0: 立即清空队列,消息被抛弃
queue.enqueue.timeout.ms=-1


#当producer接收到error ACK,或者没有接收到ACK时,允许消息重发的次数
#因为broker并没有完整的机制来避免消息重复,所以当网络异常时(比如ACK丢失)
#有可能导致broker接收到重复的消息,默认值为3.
message.send.max.retries=3

#producer刷新topic metada的时间间隔,producer需要知道partition leader
#的位置,以及当前topic的情况
#因此producer需要一个机制来获取最新的metadata,当producer遇到特定错误时,
#将会立即刷新
#(比如topic失效,partition丢失,leader失效等),此外也可以通过此参数来配置
#额外的刷新机制，默认值600000
topic.metadata.refresh.interval.ms=60000

```
