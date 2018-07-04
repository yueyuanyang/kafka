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
