### kafka 在 zookeeper的目录结构
1.topic注册信息

/brokers/topics/[topic] :

存储某个topic的partitions所有分配信息
```
Schema:
{
    "version": "版本编号目前固定为数字1",
    "partitions": {
        "partitionId编号": [
            同步副本组brokerId列表
        ],
        "partitionId编号": [
            同步副本组brokerId列表
        ],
        .......
    }
}
 
Example:
{
"version": 1,
"partitions": {
"0": [1, 2],
"1": [2, 1],
"2": [1, 2],
}
}
说明：紫红色为patitions编号，蓝色为同步副本组brokerId列表

```
