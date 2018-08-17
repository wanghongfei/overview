# OpenAdv总览

OpenAdv是开源广告系统总称，其愿景为帮助小型企业在没有专业团队的情况下以最小的成本快速搭建自己的广告系统。为了方便部署运维，所有组件均以docker镜像的方式提供，力求做到一键完成全套系统的部署。



## 技术架构

OpenAdv包含GAE(投放引擎)、GAE-DAS(增量索引生成)、gae-log-cache(检索日志缓存)、gae-log-join(检索日志、曝光/点击日志拼接)、gae-log-charge(扣费) 五个子系统构成，组合到一起就可以支持以CPM或CPC进行计费的广告投放。这些系统之间的关系如下:



### 在无条件运维Spark的情况下(推荐)

对于检索日志与曝光日志的Join部分我们使用kafka + redis的方式代替Spark Streaming完成实时的检索日志与曝光日志的拼接, 实现起来相对于Spark更简单且不需要会出运维Spark集群的代价：

![gae-overview-no-spark](http://ovbyjzegm.bkt.clouddn.com/all-arc-no-spark.jpg)

- GAE-DAS

通过监听业务系统mysql binlog实时生成增量索引。此服务**不依赖于特定的表结构**。

- GAE

实时读取增量索引，响应广告请求并产出检索日志; 每个GAE实例都会启动FileBeat进程监测检索日志文件增量并实时发送到Kafka中

- gae-log-cache

监听检索日志topic, 将日志发送至Redis并设置过期时间

- gae-log-join

监听曝光日志topic, 从Redis中查询是否有对应的检索, 如果match上则向Kafka投放扣费消息

以上两个服务可以使用轻量级的语言编写，如Go, Python，通过Kafka Partition实现横向扩展。

- 扣费服务

从kafka中接收扣费消息，操作redis进行扣费, 余额不足时操作mysql下线广告。同时每小时产出一套报表数据，按维度(推广单元、推广计划、创意、地域、标签、广告位)产出7个文本文件，业务系统定时将报表入库以供用户查询





### 有条件使用Spark

可以使用Spark-Streaming替换`gae-log-cache`和`gae-log-join`服务：

![arc](http://ovbyjzegm.bkt.clouddn.com/all-arc5.jpg)



## 部署(TODO)

每个子系统会提供Docker一键部署方式，并配详细文档;

同时提供`docker-compose`配置文件，完成全套系统的一键部署。

