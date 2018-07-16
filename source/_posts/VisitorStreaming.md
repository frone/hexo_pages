---
title: 谁看过我
date: 2016-09-26 11:10:54
tags:
    - Spark Streaming
    - Kafka
    - 谁看过我
---
<Excerpt in index | 首页摘要>
用户访问记录<!-- more -->
<The rest of contents | 余下全文>

## 背景

  visitor日志已经由前端APP或API发送到Kafka，需要将数据消费出，导入到MySQL数据库中。

## 源代码：

  `git@192.168.30.251:visitorStreaming.git`

## 程序逻辑
  使用Spark streaming方式消费数据，并将消费后的数据导入MySQL。

  - 2.1 定义消息格式
    - 根据visitor消息格式定义消费者需要解析的消息格式
    ```
    {
        "head":{
          "type": "XXXtype",
          "version": "XXXversion"
        }
        "body":{
          "timestamp": "XXXtimeString",
          "cardOwnerId": "XXXcardOwnerId",
          "vcardId": "XXXvcardId",
          "userId": "XXXuserId",
          "sendtime": "XXXsendtime"
        }
    }
    ```

  - 2.2 编写streaming逻辑，并提供插入MySQL方法
      - 2.2.1 解析出配置信息，创建kafka的DirectStream实例，并消费消息；
      - 2.2.2 创建插入MySQL的逻辑函数，将消费出的信息插入到数据库。

  - 2.3 配置程序环境

    - 2.3.1 kafka环境信息配置
      - brokers连接地址
      - topic信息

    - 2.3.2 MySQL环境信息配置
      - user
      - password
      - url
      - sql （insert语句）

    - 2.3.3 Spark App的环境配置
      - partitions RDD数据分片（repartition函数的参数）
      - duration： Streaming的时间间隔

## 提交Spark Job

  - 2.4 将程序打包成jar文件（注意及时将代码提交到版本库）
    - 使用sbt打包程序

      - `sbt package`

      注意Scala版本需和Spark集群中的Scala版本一致（Scala version： 2.10）

  - 2.5 在集群中提交程序

    - 将打包好的jar文件上传至spark集群中的某个节点（slave2:172.17.4.154/192.168.1.136）
    - 使用spark-submit提交job (提交时放置到后台，不然退出shell后，程序也将结束)

      `(spark-submit --class com.oradt.hank.VisitorStreaming /path/to/visitorsteaming_2.10-1.0.jar &)`
---
