## 文档版本

最新编辑时间：2021.07.23

平台发布版本：V6.1SP5，v7.0SP3



## 前言

- 版本说明

  使用版本：6.8.14

  官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html

- 启停方式

  启动命令：`service elasticsearch start`

  停止命令：`service elasticsearch stop`

- 关键配置

  配置文件路径：`10-common\version\release\linux\elk\elasticsearch\config`

  安装部署脚本路径：`10-common\script\linux\elasticsearch`

  集群拆搭等功能脚本：`10-common\version\release\linux\elk\elasticsearch\cluster`

## ES安装报错解决

```shell
报错1：max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
vim  /etc/sysctl.conf
增加：
#elasticsearch config start
vm.max_map_count=655360
#elasticsearch config end

运行 sysctl -p

报错2：max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
memory locking requested for elasticsearch process but memory is not locked

vim /etc/security/limits.conf
增加：
#set custom limit parameters
- hard nofile 65536
- soft nofile 65536
- hard memlock unlimited
- soft memlock unlimited

启动一段时间后报错：
Native controller process has stopped - no new native processes can be started
修改/etc/security/limits.d/xx-nproc.conf
非root用户nproc 默认设置为4096，太小了，改成unlimited

指定es启动的配置文件路径：
ES_PATH_CONF=/opt/data/config/elk/elasticsearch/config ./bin/elasticsearch

elasticsearch 启动时指定jdk1.8版本:
在启动脚本/bin/elasticsearch里面增加下面内容：
export JAVA_HOME=/opt/midware/elk/jdk1.8.0_181/
export PATH=$JAVA_HOME/bin:$PATH
# jdk path
if [ -x "$JAVA_HOME/bin/java" ]; then
        JAVA="/opt/midware/elk/jdk1.8.0_181/bin/java"
else
        JAVA=`which java`
fi

新增用户: adduser elk; passwd elk
改变文件归属：chown -R elk:elk  /opt/log/elk
```

## ES运行状态查询方法

说明：es设置了秘钥访问后，下列查询语句需要都加上可选项：`--user username:password`

```shell
查看elastic节点大小和状态：
curl (--user username:password) http://localhost:9310/_cat/indices?pretty
curl (--user username:password) http://localhost:9310/_cat/nodes?pretty
查看elastic集群整体状态：
curl (--user username:password) http://localhost:9310/_cluster/health?pretty
curl (--user username:password) http://localhost:9310/_cluster/health?level=indices
查看最大的索引名称
curl (--user username:password) http://localhost:9310/_cat/indices?bytes=b | sort -rnk8
查看UNASSIGNED分片：
curl (--user username:password) http://localhost:9310/_cat/shards?pretty|fgrep UNASSIGNED
查看primary shard没有分配的原因：
curl (--user username:password) http://localhost:9310/_cluster/allocation/explain?pretty
重新恢复失败的分片：
curl (--user username:password) -XPOST http://localhost:9310/_cluster/reroute?retry_failed=true
查看是否只读状态：
curl (--user username:password) http://localhost:9310/_settings?pretty
只读状态修复(read only):
curl (--user username:password) -XPUT -H "Content-Type: application/json" http://localhost:9310/_all/_settings -d '{"index.blocks.read_only_allow_delete": null}'
命令查看磁盘利用率：
curl -s (--user username:password) localhost:9310/_cat/allocation?v
删除节点：
curl (--user username:password) -XDELETE "http://localhost:9310/.kibana_1"
删除全部：
curl (--user username:password) -XDELETE "http://localhost:9310/_all"
查看所有task
curl (--user username:password) -XGET 'http://127.0.0.1:9310/_tasks?detailed=true&actions=*reindex'
使用taskId查看指定task详情
curl (--user username:password) -XGET 'http://127.0.0.1:9310/_tasks/99zZQSV6ROmXDvZX3fSoPQ:491?pretty'
查看segmet占用内容
curl (--user username:password) -XGET "http://localhost:9310/_cat/nodes/?v&h=name,port,sm"
设定多节点数据存放在同一目录下（全部数据存储在单个目录）：
node.max_local_storage_nodes: 100
强制规避segmeant
curl (--user username:password)  -XPOST http://127.0.0.1:9310/logstash-2015-06.10/_forcemerge?max_num_segments=1
查看elastic执行什么
集群所有节点：
curl (--user username:password)  -XGET http://localhost:9310/_nodes/hot_threads
单个节点：
curl (--user username:password)  -XGET http://localhost:9310/_nodes/hot_threads
其中 node-name 替换成自己节点节点名称
curl (--user username:password)  localhost:9310/_nodes/hot_threads?type=wait&interval=1s
剩下的  google 去到底在执行什么代码！

es 开启慢查询日志方法，定位慢查询语句以及查询耗时：
curl  (--user username:password) -XPUT 'http://localhost:9310/_all/_settings?preserve_existing=true' -H 'Content-Type: application/json' -d '{
  "index.indexing.slowlog.threshold.index.debug" : "5s",
  "index.indexing.slowlog.threshold.index.info" : "10s",
  "index.indexing.slowlog.threshold.index.trace" : "2s",
  "index.indexing.slowlog.threshold.index.warn" : "20s",
  "index.search.slowlog.threshold.query.debug" : "5s",
  "index.search.slowlog.threshold.query.info" : "10s",
  "index.search.slowlog.threshold.query.trace" : "2s",
  "index.search.slowlog.threshold.query.warn" : "20s"
}'
curl (--user username:password) -XPUT "http://localhost:9310/_cluster/settings" -H 'Content-Type: application/json' -d'
{
    "transient" : {
        "logger.index.search.slowlog" : "DEBUG", 
        "logger.index.indexing.slowlog" : "WARN" 
    }
}'
定时关闭索引(kibana中执行)
XPOST {{path}}/.auto_close/taskInfo/
{
    "indicesPrefix": "platform-",
    "days": 30
}
```

## es数据操作语句

```shell
重新分配示例：
for shard in $(curl --user elastic:password -XGET http://localhost:9310/_cat/shards | grep closed | awk '{print $1}'); 
do
curl --user elastic:password -XDELETE "http://localhost:9310/$shard" 
done

curl -XPOST  -H "Content-Type: application/json" http://localhost:9310/_cluster/reroute? -d '{
  "commands" : [ {
  "allocate_empty_primary" :
  {
    "index" : "systemprofile_2020-01",
    "shard" : 0,
    "node" : "VaS26NjYTZa0dwV95J2PtA",
    "accept_data_loss" : true
  }
}]
}'
reindex：
后台方式执行，返回taskId
curl -XPOST 'http://127.0.0.1:9201/_reindex?pretty&wait_for_completion=false' -H 'content-Type:application/json' -d '
{
  "source": {
    "remote": {
      "host": "http://127.0.0.1:9200"
    },
    "index": "forum",
    "type": "article"
  },
  "dest": {
    "index": "forum_jp1"
  }
}'

for line in $(curl -s 'localhost:9310/_cat/shards' | fgrep UNASSIGNED); do
  INDEX=$(echo $line | (awk '{print $1}'))
  SHARD=$(echo $line | (awk '{print $2}'))
  number=$RANDOM
  let "number %= ${range}"

  curl -XPOST http://localhost:9310/_cluster/reroute? -d '{
  "commands" : [ {
  "allocate_empty_primary" :
  {
    "index" : '\"${INDEX}\"',
    "shard" : '\"${SHARD}\"',
    "node" : "undercooked-horse-elasticsearch-data-0",
    "accept_data_loss" : true
  }
}
]
}'
done
```

## es设置秘钥及账号权限相关语句

```shell
忘记密码：
1.创建临时超级用户：
JAVA_HOME=/opt/midware/elk/jdk1.8.0_181/ /opt/midware/elk/elasticsearch/bin/elasticsearch-users useradd admin -p my_password -r superuser
\cp -r /opt/midware/elk/elasticsearch/config/users_roles /opt/data/config/elk/elasticsearch/config/
\cp -r /opt/midware/elk/elasticsearch/config/users /opt/data/config/elk/elasticsearch/config/
临时超级用户查看状态：
curl -u admin:my_password http://localhost:9310/_cat/master
curl -u admin:my_password -XPOST http://localhost:9310/_cat/state
curl -u admin:my_password http://localhost:9310/_cat/shards?pretty

2.用临时超级用户修改elastic密码：
curl -u admin:my_password -XPUT 'http://localhost:9310/_xpack/security/user/elastic/_password?pretty' -H 'Content-Type: application/json' -d'
{
  "password" : "password"
}'

重置全部密码：
curl -u elastic:password -XDELETE "http://localhost:9310/.security*"
删除全部数据
curl -u admin:my_password -XDELETE "http://localhost:9310/_all"

新建只读用户
curl -XPOST -u elastic:password 'localhost:9310/_xpack/security/role/kedacom' -H "Content-Type: application/json" -d '{"cluster":["all"],"indices":[{"names":["*"],"privileges":["read"]}]}'
curl -XPOST -u elastic:password 'localhost:9310/_xpack/security/user/kedacom' -H "Content-Type: application/json" -d '{
  "password" : "password16#",
  "full_name" : "kibana",
  "roles" : [ "kedacom" ]
}'
```
# es内存使用记录及维护

es内存使用主要由一下几个方面，相关说明如下表：

| es内存消耗大户               | 大小              | 说明                             |
| ---------------------------- | ----------------- | -------------------------------- |
| 1. segment memory            | -                 | lucene倒排索引，和索引数量正相关 |
| 2. Request cache             | 默认10% heap      | 缓存使用过的filter的结果集的     |
| 3. field data cache          | 默认超过40%进行gc | 和查询的命中结果                 |
| 4. bulk queue                | 默认              | request在内存的排列              |
| 5. indexing buffer           | 默认10% heap      | 缓存新数据                       |
| 6. state buffer              | -                 | 集群状态的拷贝，单机可忽略       |
| 7. 超大搜索聚合结果集的fetch | -                 | 和返回结果的size相关             |

- 内存主要增大点：

  field data cache ，segment memory

- 解决方法：

  segment memory：

  ​	通过减少不需要的索引，来达到减少这部分内存占用的效果

  field data cache：

  ​	该内存默认可以无限增长，知道达到设置的**indices.breaker.fielddata.limit(默认40%heap size)**为止，主要有一下设置可以降低内存使用

  > 在配置文件设置**indices.fielddata.cache.size（静态）**

  ​	可以设置绝对值（12GB）和相对值（20% heap size）,到达设置的数量后，会清理最早的数据

  ```
  elasticsearch.yml： indices.fielddata.cache.size: 20%
  ```

  > 设置**indices.breaker.fielddata.limit（动态）**

  ​	系统设置的默认值为 40% of JVM heap，动态更改语句为

  ```
  PUT /_cluster/settings
  {
    "persistent": {
      "indices.breaker.fielddata.limit": "30%"
    }
  }
  ```

  > 手动清理cache

  ```
  #清理指定索引的指定cache:分别指定fielddata，query，request cache
  POST /my-index-000001/_cache/clear?fielddata=true  
  POST /my-index-000001/_cache/clear?query=true      
  POST /my-index-000001/_cache/clear?request=true
  
  #清理指定索引的全部cache
  POST /my-index-000001/_cache/clear
  ```
## 
## 其他信息记录

- es版本升级注意点

  es7可以兼容type为"_doc"的索引。
  如果在es6中，索引名已经为"_doc"，则升级es7过程，索引不需要做数据清洗。
  如果在es6中，索引名不为"_doc"，则升级es7过程，索引需要做数据清洗为"_doc"。执行_reindex进行数据迁移。

- es集群滚动升级

  由于es存储数据量较大，同时进行升级时，会导致恢复数据花费较长时间，业内推荐的方式是单台分别滚动升级，这样对外界访问查询，不会有不可用的时间，且不会感知es的相关变动
