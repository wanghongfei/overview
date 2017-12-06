# OpenAdv总览



## 图解

OpenAdv包含 GAE(广告投放引擎)、GAE-DAS(Binlog监听)、log-collector(日志收集)、gae-report(报表计算任务)四个子系统，它们之间的业务流程如下：

![arc](http://ovbyjzegm.bkt.clouddn.com/all-arc.jpg)

GAE-DAS通过监听业务系统mysql binlog实时生成增量索引;

GAE实时读取增量索引，响应广告请求并产生检索日志;

检索日志通过FileBeat发送至Kafka;

log-collector集中收集日志数据，按小时分目录存储;

gae-report每小时运行一次分析日志文件，产出报表;

业务系统定时将报表产出拉取入库;



## 部署

每个子系统会提供Docker一键部署方式，并配详细文档。

