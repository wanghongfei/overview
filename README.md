# OpenAdv总览



## 图解

OpenAdv包含 GAE(广告投放引擎)、GAE-DAS(Binlog监听)、gae-log-streaming(spark-streaming实时日志分析)和业务端四个子系统，它们之间的业务流程如下：

![arc](http://ovbyjzegm.bkt.clouddn.com/all-arc5.jpg)

- GAE-DAS

通过监听业务系统mysql binlog实时生成增量索引

- GAE

实时读取增量索引，响应广告请求并产出检索日志; 每个GAE实例都会启动FileBeat进程监测检索日志文件增量并实时发送到Kafka中

- gae-log-streaming

使用Spark Streaming实时分析日志。将检索日志与曝光日志通过`sid`字段执行`JOIN`操作，使用`JOIN`结果进行扣费，并投递扣费消息到kafka中。

- 扣费服务

从kafka中接收扣费消息，操作redis进行扣费, 余额不足时操作mysql下线广告。同时每小时产出一套报表数据，按维度(推广单元、推广计划、创意、地域、标签、广告位)产出7个文本文件，业务系统定时将报表入库以供用户查询



对于检索日志与曝光日志的Join部分，使用Spark Streaming可能较重，部署比较复杂; 还有另一种不需要使用大数据处理系统的方案，如下图：

![gae-overview-no-spark](http://ovbyjzegm.bkt.clouddn.com/all-arc-no-spark.jpg)

我们分别使用检索日志Cache和Join两个服务代替Spark:

- cache

监听检索日志topic, 将日志发送至Redis并设置过期时间

- Join

监听曝光日志topic, 从Redis中查询是否有对应的检索, 如果match上则向Kafka投放扣费消息

以上两个服务可以使用轻量级的语言编写，如Go, Python，通过Kafka Partition实现横向扩展。



## 部署

每个子系统会提供Docker一键部署方式，并配详细文档。

