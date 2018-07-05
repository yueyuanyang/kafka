## Kafka之数据存储

**本文主要讲述以下两部分内容**：

- kafka数据的存储方式；
- kafka如何通过offset查找message。

### 1.前言

写介绍kafka的几个重要概念（可以参考之前的博文Kafka的简单介绍）：

- Broker：消息中间件处理结点，一个Kafka节点就是一个broker，多个broker可以组成一个Kafka集群；
- Topic：一类消息，例如page view日志、click日志等都可以以topic的形式存在，Kafka集群能够同时负责多个topic的分发；
- Partition：topic物理上的分组，一个topic可以分为多个partition，每个partition是一个有序的队；
- Segment：每个partition又由多个segment file组成；
- offset：每个partition都由一系列有序的、不可变的消息组成，这些消息被连续的追加到partition中。partition中的每个消息都有一个连续的序列号叫做offset，用于partition唯一标识一条消息；
- message：这个算是kafka文件中最小的存储单位，即是 a commit log。

kafka的message是以topic为基本单位，不同topic之间是相互独立的。每个topic又可分为几个不同的partition，每个partition存储一部的分message。topic与partition的关系如下：

![kafka1_1]()

**其中，partition是以文件夹的形式存储在具体Broker本机上。**

### 2.partition中的数据文件
有了上面的介绍，下面我们开始介绍Topic中partition的数据文件类型。

#### 2.1.segment中的文件

对于一个partition（在Broker中以文件夹的形式存在），里面又有很多大小相等的segment数据文件（这个文件的具体大小可以在config/server.properties中进行设置），这种特性可以方便old segment file的快速删除。

下面先介绍一下partition中的segment file的组成：

- segment file 组成：由2部分组成，分别为index file和data file，这两个文件是一一对应的，后缀”.index”和”.log”分别表示索引文件和数据文件；
- segment file 命名规则：partition的第一个segment从0开始，后续每个segment文件名为上一个segment文件最后一条消息的offset,ofsset的数值最大为64位（long类型），20位数字字符长度，没有数字用0填充。如下图所示：

![kafka1_2]()

关于segment file中index与data file对应关系图，这里我们选用网上的一个图片，如下所示：

![kafka1_3]()

segment的索引文件中存储着大量的元数据，数据文件中存储着大量消息，索引文件中的元数据指向对应数据文件中的message的物理偏移地址。以索引文件中的3，497为例，在数据文件中表示第3个message（在全局partition表示第368772个message），以及该消息的物理偏移地址为497。

注：Partition中的每条message由offset来表示它在这个partition中的偏移量，这个offset并不是该Message在partition中实际存储位置，而是逻辑上的一个值（如上面的3），但它却唯一确定了partition中的一条Message（可以认为offset是partition中Message的id）。

#### 2.2.message文件
message中的物理结构为：

![kafka1_4]()

**参数说明：**

| 关键字  | 解释说明  |
|---|---|
| 8 byte offset  | 在parition(分区)内的每条消息都有一个有序的id号，这个id号被称为偏移(offset),它可以唯一确定每条消息在parition(分区)内的位置。offset表示partiion的第多少message  |
| 4 byte message size  | message大小 |
|  4 byte CRC32 | 用crc32校验message  |
|  1 byte “magic” | 表示本次发布Kafka服务程序协议版本号  |
|  1 byte “attributes” | 用crc32校验message  |
|  4 byte key length | 表示key的长度,当key为-1时，K byte key字段不填  |
|  K byte key | 	可选  |
|  K byte key | 表示实际消息数据  |




