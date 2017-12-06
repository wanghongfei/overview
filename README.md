# OpenAdv总览



## 图解

OpenAdv包含 GAE(广告投放引擎)、GAE-DAS(Binlog监听)、log-collector(日志收集)、gae-report(报表计算任务)四个子系统，它们之间的业务流程如下：

![arc](http://ovbyjzegm.bkt.clouddn.com/all-arc4.jpg)

- GAE-DAS

通过监听业务系统mysql binlog实时生成增量索引

- GAE

实时读取增量索引，响应广告请求并产出检索日志; 每个GAE实例都会启动FileBeat进程监测检索日志文件增量并实时发送到Kafka中

- gae-log-streaming

使用Spark Streaming实时分析日志。将检索日志与曝光日志通过`sid`字段执行`JOIN`操作，使用`JOIN`结果进行扣费，并实时下线广告。同时每小时产出一套报表数据，按维度(推广单元、推广计划、创意、地域、标签、广告位)产出7个文本文件，业务系统定时将报表入库以供用户查询



## 部署

每个子系统会提供Docker一键部署方式，并配详细文档。

