---
title: cloudera集群环境说明文档
date: 2016-09-26 16:53:17
tags:
    - Cloudera
    - Kafka
---
<Excerpt in index | 首页摘要>
CDH集群环境说明（含IDC环境和OFFICE环境） <!-- more -->
<The rest of contents | 余下全文>


环境搭建文档：http://192.168.30.251:4000/2016/01/27/BigdataPlatform/

----
IDC Cloudera集群环境

|简称|节点名|IDC IP| 内网映射IP|在集群|Kafka Broker|备注(其他主要功能)
|-----|--
|Cloud  |BJCloud-Spark-Slave-1	  | 172.17.4.151  |101.251.193.28 | 否 |否 |web service
|Mongo1 |BJIDC-Oradt-MongoDB-1	  | 172.17.4.51   |192.168.1.137  | 否 |否 |web service
|Mongo2 |BJIDC-Oradt-MongoDB-2	  | 172.17.4.52   |192.168.1.132  | 否 |否 |web service
||
|MySQL1 |BJIDC-Oradt-DMMySQL-1	  | 172.17.4.1    |192.168.1.138  | 是 | 是 |mysql数据库
|MySQL2 |BJIDC-Oradt-DMMySQL-2	  | 172.17.4.2    |192.168.1.139  | 是 | 是 |mysql数据库
|Master1|BJIDC-Oradt-SprakMaster-1|	172.17.4.101  |192.168.1.133  | 是 |**否**|管理界面（admin/CMadmin）
|Master2|BJIDC-Oradt-SparkMaster-2|	172.17.4.102  |192.168.1.134  | 是 | 是 |--
|Slave1 |BJIDC-Oradt-SparkSlave-1 | 172.17.4.153  |192.168.1.135  | 是 | 是 |--
|Slave2 |BJIDC-Oradt-SparkSlave-2 | 172.17.4.154  |192.168.1.136  | 是 | 是 |spark job提交

----
OFFICE Cloudera集群测试环境

root密码：123456

|简称|节点名|内网IP|备注
|-----|--
| vm150| BJOFFICE-Oradt-Spark04 |192.168.30.150 |CDM管理界面（admin/ADMIN）
| vm149| BJOFFICE-Oradt-Spark03 |192.168.30.149 |--
| vm148| BJOFFICE-Oradt-Spark02 |192.168.30.148 |--
| vm147| BJOFFICE-Oradt-Spark01 |192.168.30.147 |--
----

## 常见问题及解决方案

- 1 未找到第三方jar包？

  - 解决方案一.

    将第三方jar包所在路径添加到/etc/spark/conf/classpath文件末尾， 同步到集群中所有节点

  - 解决方案二.

    通过cloudera manager界面，找到spark下的"配置"-->"Gateway Default Group"-->"高级"，修改spark-defaults.conf，将第三方jar包加入到spark.driver.extraClassPath和spark.executor.extraClassPath选项中

- 2 hdfs文件不存在

  解决方案： 使用命令 hdfs dfs -ls /path/to/file  确认是否存在改文件 （spark分布式系统要确保数据在HDFS上，而非本地文件系统上）

- 3 文件或文件夹无读写权限

  解决方案：确保当前用户在spark job的数据读取路径或数据写入路径中有相应权限；如果没有，添加相应权限或者将数据复制或写入到别处。

- 4 spark job提交后一直等待资源

  解决方案：spark集群中资源被占用，当前的job无法分配到足够的资源以运行任务。如果必要，重启spark集群。

- 5 cloudera升级

  - 方法一：

    登录Cloudera Manager界面，"主机" --> "Parcel"，进入软件包界面后可看到先用软件版本， 右上角"检查新Parcel"可发现新parcel，"下载"-->"分配"-->"安装"

  - 方法二：

    从cloudera官网下载（http://archive.cloudera.com/cdh5， http://archive.cloudera.com/cm5） Parcel包构建本地库（详见文章‘环境搭建文档’），手动更新


提示：**错误信息请注意查看log和控制台屏显日志**

---
