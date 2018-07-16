---
title: Imora用户行为日志采集系统
date: 2016-01-27 12:27:32
tags: 
    - NameCard
    - Levenstein Distance
---
<Excerpt in index | 首页摘要> 
- 源代码
```bash
	git clone git@192.168.30.251@imora.git
```

Imora手机客户端及网页客户端采集用户行为信息后，向该系统发送采集到的操作日志，该系统接收信息并转发到kafka集群中。<!-- more -->
<The rest of contents | 余下全文>

## 运行环境
- Java 1.7+
- Scala 2.11.7
- SBT 0.13.8 （[官方文档](http://www.scala-sbt.org/0.13/docs/index.html)）

环境安装请参考相关文档。

## 设计结构

- ![逻辑结构图](https://dn-abcxy.qbox.me/OradtImora.jpg)

使用Scala Spray框架，相关知识见[Spray官方文档](http://spray.io/introduction/what-is-spray/)

## 部署
- 部署方式使用[SBT Native Packager](http://www.scala-sbt.org/sbt-native-packager/)插件

**详见代码**。

