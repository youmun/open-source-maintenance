## 文档版本

最新编辑时间：2021.06.24

平台发布版本：V6.1SP5



## 前言

beats不是一个组件，而是代指一系列相关组件，目前我们使用过的beat组件有：

- metricbeat 

  功能：采集系统硬件信息及支持的基础组件信息

- packetbeat （6.1sp5版本废弃使用）

  功能：抓取指定端口，指定协议的数据包，并解析发送到指定目的

- auditbeat（7.0sp3版本增加使用）

  功能：审计安全信息采集

  

## metricbeat

- 版本说明

  当前使用版本: 	`Ver.6.5.4`

  官方文档地址：https://www.elastic.co/guide/en/beats/metricbeat/6.5/index.html

- 启停方式

  启动命令：`service metricbeat start`

  停止命令：`service metricbeat stop`

  服务状态：`service metricbeat status`

- 关键配置

  配置文件路径：

  1.X86版本

  `/10-common/version/release/linux/beats/metricbeat/conf.d`

  ![1624522492170](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1624522492170.png)

  2.国产化飞腾64版本

  `/10-common/version/release/linux_arm_ft64/beats/metricbeat`

  其中metricbeat.yml和metricbeat-nojdb.yml的配置主要区别为采集数据和输出地址不同

  | 配置文件             | 采集数据说明    | 输出目的说明 | 关键配置段   |
  | -------------------- | --------------- | ------------ | ------------ |
  | metricbeat.yml       | 系统硬件数据    | JDB中kafka   | output.kafka |
  | metricbeat-nojdb.yml | rabbitmq中q数据 | 本地文件     | output.file  |

- 更改记录

  1. 增加过滤mq中任意q积压数据量超过100条的记录

     实现方法：在metricbeat-nojdb.yml最后，增加代码段：

     ```shell
     processors:
     
     - drop_event:
       when:
          or:
             - range:
                  rabbitmq.queue.messages.ready.count.lte: 100 
             - not:
                  equals:
                    metricset.module: rabbitmq
     - include_fields:
       when:
          equals:
              metricset.module: rabbitmq
       fields: ["rabbitmq"]
     ```

     2.增加按照指定进程名称过滤采集进程cpu/内存数据

     实现方法：修改modules.d文件夹中的system.yml文件，增加如下设置：

     ```shell
     processes：['.*java.*','.*cmu.*','.*metricbeat.*','.*mpcadp.*','.*callmanager.*','.*sipagent.*','.*webrtcagent.*','.*beam.smp.*','.*epmd.*','.*dssmaster.*','.*sfumaster.*','.*media-master.*','.*cmdataproxy.*', '.*aps.*','.*nms_collector.*','.*mcmp-svr.*','.*mediaresource.*','.*dssworker.*','.*sfumaster.*','.*sfuresource.*','.*redis.*','.*vernemq.*','.*cmdataproxy.*','.*syslog-ng*','.*mysql.*']
     processors:
        - drop_event:
             when:
                  - contains:
                       system.process.name: "monitor"
     ```

     当前指定进程名称列表如下：（7.0后续可能更新）

     ```shell
     "java",
     "syslog-ng",
     "metricbeat",
     "mpcadp",
     "callmanager",
     "sipagent",
     "webrtcagent",
     "ejabberd",
     "dssmaster",
     "sfumaster",
     "media-master",
     "cmdataproxy",
     "aps",
     "cmu",
     "nms_collector"
     "mcmp-svr"  
     "mediaresource"
     "dssworker",
     "sfumaster",
     "sfuresource",
     基础云所有组件 mysql redis vernemq rabbitmq 
     ```



## auditbeat

- 版本说明

  当前使用版本: 	`Ver.6.5.4`

  官方文档地址：	https://www.elastic.co/guide/en/beats/auditbeat/6.5/index.html

- 关键配置

  配置文件路径：

  1.X86版本

  `/10-common/version/release/linux/beats/auditbeat/conf.d`

- 启停方式

  启动命令：`service auditbeat start`

  停止命令：`service auditbeat stop`

  服务状态：`service auditbeat status`

- 更改记录

  暂无更改，使用默认配置
