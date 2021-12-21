## 文档版本

最新编辑时间：2021.06.24

平台发布版本：V6.1SP5，v7.0SP3

## 前言

- 版本说明

  使用版本：kafka = 2.5.0 scala = 2.12

  官方文档：https://kafka.apache.org/25/documentation.html#introduction

- 启停方式

  启动命令：`/opt/mcu/kafka/shells/start.sh`

  停止命令：`/opt/mcu/kafka/shells/stop.sh`

- 关键配置

  配置文件路径：`10-common\version\release\linux\kafka\config`

  安装部署脚本路径：`10-common\script\linux\kafka`

  集群拆搭等功能脚本：`10-common\version\release\linux\kafka\cluster`

## KAFKA 数据操作命令

```bash
kafka 创建topic
/opt/midware/kafka/bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 10 --topic test
/opt/midware/kafka/bin/kafka-topics.sh --create --zookeeper localhost:2180 --replication-factor 1 --partitions 1 --topic xjx-test

删除topic
/opt/midware/kafka/bin/kafka-topics.sh --delete  --zookeeper 127.0.0.1:2180  --topic ads.timing.ap

查看top内容
/opt/midware/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic ods.formatlog --from-beginning  
/opt/midware/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic ods.formatlog --offset 4311719103 --partition 0 --isolation-level read_committed --consumer.config /opt/data/config/kafka/config/consumer.properties
/opt/midware/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic ods.nginx --from-beginning  --consumer.config /opt/data/config/kafka/config/consumer.properties
发送message:
/opt/midware/kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test --producer.config /opt/data/config/kafka/config/producer.properties
seq 20000 | ./bin/kafka-console-producer.sh --topic test --broker-list localhost:9092
/opt/midware/kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --producer.config /opt/data/config/kafka/config/producer.properties --topic xjx-test < /home/xjx/mpcadp.log
后台启动：
/opt/midware/kafka/bin/zookeeper-server-start.sh /opt/midware/kafka/config/zookeeper.properties &
/opt/midware/kafka/bin/kafka-server-start.sh /opt/midware/kafka/config/server.properties 1>/dev/null 2>&1 &

查看topic列表
/opt/midware/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092 --producer.config /opt/data/config/kafka/config/producer.properties
/opt/midware/kafka/bin/kafka-topics.sh --list --bootstrap-server 10.67.18.100:9092
/opt/midware/kafka/bin/kafka-topics.sh --list --zookeeper localhost:2180

查看topic最大offset:
/opt/midware/kafka/bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic ods.formatlog --time -1
/opt/midware/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic ods.nginx  --offset 28496 --partition 0

查看topic 信息：
/opt/midware/kafka/bin/kafka-topics.sh --describe --zookeeper 127.0.0.1:2180 --topic __consumer_offsets
/opt/midware/kafka/bin/kafka-topics.sh --describe --zookeeper 127.0.0.1:2180 --topic __transaction_state
/opt/midware/kafka/bin/kafka-topics.sh --describe --zookeeper 127.0.0.1:2180 --topic ods.*
/opt/midware/kafka/bin/kafka-topics.sh --describe --zookeeper 127.0.0.1:2180 --topic connect.*
/opt/midware/kafka/bin/kafka-topics.sh --describe --zookeeper 127.0.0.1:2180 --topic dwd.mixswitch 


/opt/midware/kafka/bin/kafka-configs.sh --describe --zookeeper 127.0.0.1:2180 --entity-type topics  --entity-name ods.h323

登陆zk删除数据：
/opt/midware/kafka/bin/zookeeper-shell.sh localhost:2180
ls /
rmr /controller 

删除zk中flink数据：
/opt/midware/kafka/bin/zookeeper-shell.sh localhost:2180 deleteall /flink
查看zk谁是leader:
谁有监听2888端口谁就是leader

修改保存数据时间：86400000ms=24h  604800000ms=7days  1296000000ms=15days 2592000000ms=30days
/opt/midware/kafka/bin/kafka-topics.sh --zookeeper 127.0.0.1:2180 -topic ods.nms --alter --config retention.ms=86400000
/opt/midware/kafka/bin/kafka-topics.sh --zookeeper 127.0.0.1:2180 -topic ods.metricbeat --alter --config retention.ms=604800000

/opt/midware/kafka/bin/kafka-configs.sh --zookeeper 127.0.0.1:2180 --alter --entity-name topicName --entity-type ods.metricbeat --add-config retention.ms=1296000000

修改topic默认partition
/opt/midware/kafka/bin/kafka-topics.sh --zookeeper 127.0.0.1:2180 --alter --topic ads.h323 --partitions 10
/opt/midware/kafka/bin/kafka-topics.sh --zookeeper 127.0.0.1:2180 --alter --topic dwd.h323 --partitions 10
/opt/midware/kafka/bin/kafka-topics.sh --zookeeper 127.0.0.1:2180 --alter --topic ods.h323 --partitions 5
/opt/midware/kafka/bin/kafka-topics.sh --zookeeper 127.0.0.1:2180 --alter --topic dwd.logjson.nginx --partitions 10
/opt/midware/kafka/bin/kafka-topics.sh --zookeeper 127.0.0.1:2180 --alter --topic ods.metricbeat --partitions 5

/opt/midware/kafka/bin/kafka-topics.sh --zookeeper 127.0.0.1:2180 --alter --topic ads.h323 --partitions 10
/opt/midware/kafka/bin/kafka-topics.sh --zookeeper 127.0.0.1:2180 --alter --topic ods.sip --partitions 10

/opt/midware/kafka/bin/kafka-topics.sh --zookeeper 127.0.0.1:2180 --alter --topic dwd.nms.dss-worker --partitions 5



/opt/midware/kafka/bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic testtest

导出topic数据到文件
json需要设置:
key.converter.schemas.enable:false
value.converter.schemas.enable:false

bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-sink.properties
修改Connect的REST API默认端口
在connect-distributed.properties中的rest.port=8083
kafka的性能测试：
bin/kafka-producer-perf-test.sh --messages 100000 --message-size 1000  --batch-size 10000 --topics test4 --threads 4 --broker-list hadoop234:9092,hadoop237:9092,hadoop238:9092

bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic nms.amc --from-beginning
```

