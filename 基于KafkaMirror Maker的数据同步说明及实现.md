# 基于Kafka Mirror Maker的数据同步说明及实现

## Kafka MirrorMaker基本特点介绍

​	在目标集群没有对应Topic时，Kafka MirrorMaker会自动在目标集群上创建一个一模一样(分区数量、副本数量)的topic。如果目标集群存在相同的Topic，则不进行创建。

​	Kafka Mirror Maker允许指定需要同步的多个Topic。比如TopicA|TopicB|TopicC，也兼容使用逗号进行分隔的写法。

​	多线程支持。MirrorMaker会在每一个线程上创建一个Consumer对象。因此，如果机器性能允许，建议多创建一些线程增加同步性能。

​	多进程任意横向扩展，只要这些进程的consumer Group-id相同。无论是多进程还是多线程，Kafka Consumer Group的本身设计，可以提供任意数量的横向扩展性，具体的分配情况，即具体的Topic Partition会分派给Group中的哪个Topic负责，是Kafka自动完成的，Consumer无需关心。 

## 基础配置说明

消费者配置内容如下，需要鲁班动态替换消费目录集群为实际ip：port

```
# 消费目标集群
bootstrap.servers=<kafka1host_from>:<port>,<kafka2host_from>:<port>,...
# 消费组的ID
group.id=mm-consumer-group
# 选取镜像数据的起始设置
auto.offset.reset=earliest
# 单个poll()执行的最大record数，默认是500
max.poll.records=20000
#支持sasl认证
security.protocol = SASL_PLAINTEXT
sasl.mechanism = PLAIN
```

生成者配置如下，需要鲁班动态替换消费目录集群为实际ip：port

```
# 生产者目的集群ip
bootstrap.servers=<kafka1host_to>:<port>,<kafka2host_to>:<port>,...
#支持sasl认证
security.protocol = SASL_PLAINTEXT
sasl.mechanism = PLAIN
```

## 执行命令

执行mirror-maker脚本命令示例如下，设置commit间隔为15s（默认60s）

```
nohup bin/kafka-mirror-maker.sh --consumer.config config/consumer-mm.properties --num.streams 2 --offset.commit.interval.ms 15000 --producer.config config/producer-mm.properties --whitelist 'ads.conf' >/opt/log/kafka/logs/mirrormaker.log 2>&1 & 
```

## 部署运行方式

​	Mirror Maker可以分开部署运行在集群的各个节点中，采用”推“的方式同步到中心节点；也可以只在中心节点上运行多个Mirror Maker，采用”拉“的方式本地消费个节点数据。

​	一般业内是建议从远端机房消费向本地机房生产发送，这样可以避免当网络发生异常时，可能会产生消费了消息但是没有及时得到转发，造成数据丢失。

​	在中心节点进行数据”拉“方式的示意图如下：

![](C:\Users\Administrator\Desktop\TLPic_20210518143903.png)

​	对于需要拉取多个不同集群的kafka数据，可以通过对每一个集群启用一个mirror maker，但是实现比较繁琐。

​	同时，如果需要同步的中心节点中，kafka是单机环境，中心节点的资源消耗会随着需要同步的节点数量增加而增加，且性能会有一定影响，该中心节点所在机器的 I/O可能会成为整个消费过程的性能瓶颈，且很难充分利用集群的吞吐率

## 性能说明

​	基于测试环境，对kafka mirror maker同步性能进行测试，具体结论如下：

- 单个partition分区， 速度不超过 2000 message/sec。

- 多个partition性能会明显改善（测试 10 个partition，速度可以提升至 20000 message/sec）
  

