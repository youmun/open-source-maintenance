# Elasticsearch数据迁移说明

## 数据迁移实现

es的数据迁移，依据迁移的数据量的不同，有以下几种方式：

>  小数据量（指定索引）

- 使用elasticdump等工具，查询导出需要迁移的索引数据

>  大数据量（指定多个索引，或全量数据迁移）

- 全量复制es存储数据的目录到需要迁移的环境后，es读取恢复

- 生成es数据快照后，复制快照数据到需要迁移的环境后，es读取恢复

下面依据具体场景分析具体实现方法及相关步骤

**场景1：单个索引数据迁移到新环境**

推荐方法：直接使用elasticdump工具将该索引数据导入新环境

相关语句介绍：

```shell
#ip1:index->ip2:index

elasticdump --input=http://ip1:9310/test --output=http://ip2:9310/test
```



**场景2：大数据量迁移**

大数据量的迁移不推荐使用elasticdump，迁移的时间长，时间成本高，大数据量可以细分为2个场景：单机和集群，从数据迁移角度来看，两者存在很大的区别，具体说明如下：

> 单机大数据量迁移到单机场景

该迁移场景较为灵活，既可以直接复制数据目录到新环境，也可以生成数据快照后复制快照数据进行迁移

如果采用第一种方式，建议使用后台执行多开CP命令的方式，充分调用机器的IO资源，参考迁移脚本实现如下

/opt/data/region/platformData/hd0/elk/elk-data ---> /opt/data/region/platformData/hd1/elk/elk-data

```shell
#!/bin/bash

mkdir -p /opt/data/region/platformData/hd0/elk/elk-data/nodes/0
\cp -a /opt/data/region/platformData/hd0/elk/elk-data/nodes/0/node.lock /opt/data/region/platformData/hd1/elk/elk-data/nodes/0
\cp -a /opt/data/region/platformData/hd0/elk/elk-data/nodes/0/_state /opt/data/region/platformData/hd1/elk/elk-data/nodes/0

mkdir -p /opt/data/region/platformData/hd1/elk/elk-data/nodes/0/indices
cd /opt/data/region/platformData/hd0/elk/elk-data/nodes/0/indices

for dir in `ls -l |awk '{print $9}'`
do
{
cp -a $dir /opt/data/region/platformData/hd1/elk/elk-data/nodes/0/indices
}&

done
```

> 集群大数据迁移场景

由于集群场景下，数据分布在各个集群节点存储，故不方便采用直接对数据目录进行复制的方式进行数据迁移，推荐使用官方提供的数据快照功能，生成数据快照后，复制快照文件到新集群环境即可

es生成快照相关步骤：

1、配置文件elasticsearch.yml设置快照目录（用于存储快照文件）：需要重启es生效

```shell
path.repo: /opt/data/elk/elastic-backup/

chown -R elk:elk /opt/data/elk/elastic-backup/
```

注意：集群生成的快照文件需要设置在共享文件路径，全部集群节点都能正常访问才可正常生成

2、创建仓库：

```shell
PUT _snapshot/my_backup 
{
    "type": "fs", 
    "settings": {
        "location": "/opt/data/elk/elastic-backup", 
        "max_snapshot_bytes_per_sec" : "50mb", 
        "max_restore_bytes_per_sec" : "50mb"
    }
}
```

3.备份索引

```shell
PUT _snapshot/my_backup/snapshot_1
{
"indices": "index_1,index_2"
}
```

获得单个快照的信息：
`GET _snapshot/my_backup/snapshot_1`
监听快照进度：
`GET _snapshot/my_backup/snapshot_1/_status`

4.从快照中恢复：

```shell
curl -XPOST "http://localhost:9310/_snapshot/my_backup/snapshot_1/_restore"
```

查看所有索引的恢复进度

```shell
curl -XGET http://localhost:9310/_recovery/
```

