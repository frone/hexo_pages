---
title: 名片信息拓展系统
date: 2017-04-28 10:14:01
tags:
    - 名片
    - 基础信息

categories: IMORA
---
## 名片信息拓展系统


### 一.业务整体流程

名片信息拓展系统主要包括两大任务：一.从json格式的数据源提取所需要字段，入dm库供数据挖掘等使用。二.在基础信息上进行拓展，以获得更丰富的数据。

现用到的json字段有：姓名，手机，公司名，公司电话，职位
拓展字段：手机→城市，公司名→行业，公司电话→城市，职位+行业→职能

1.**数据来源：**
可能涉及到如下数据表

* 名片基本信息，json格式
oradt_cloud_test2.contact_card
oradt_cloud_test2.contact_card_extend
* 手机、座机区号
dm_test.province_city_unique_code
* 行业职能对照表
oradt_cloud_test2.account_basic_category

2.**整体处理流程**
本系统的处理流程如下图：
![](http://i4.buimg.com/4851/24a68975106791ab.png)
图片备份位置：/home/samba/share/mengqian/handover/名片信息拓展/step.png

* 对应到代码，整体的入口为jobsAssigner，主要用来通过命令行参数实现运行每日任务，还是只计算给定区间的数据任务，将时间区间传给makeResultTables。
* makeResultTables则查询不同数据源，拼合计算所需的数据格式，处理完毕后再拼合入库所需的数据格式，并执行入等库操作。
* 其中makeResultTables会调用jsonPreprocess，jsonPreprocess主要任务为接受从makeResultTables所传递的时间区间，取出该区间中json数据的结果，拼合为后续信息拓展流程所需的数据格式。



###  二.功能模块说明
主要功能模块的包结构如下图：
![](http://i1.piimg.com/4851/4d54fb562381507a.jpg)
图片备份位置：/home/samba/share/mengqian/handover/名片信息拓展/pakages.png

信息拓展的各个模块，结构类似，输入数据格式为pandas.Dataframe，通过prepare()方法转换格式后，传递给transform()方法进行计算。


### 三.项目部署

* 当前每日任务部署于： 101.251.193.28:/home/mqian/3366
由crontab每天凌晨3点执行(3388的暂时弃用)
```
#每天凌晨3点整执行，名片数据解析
#0 3 * * *  /usr/local/bin/python /home/mqian/3388/jobsAssigner.py -d 2>> /home/mqian/3388/log3388
0 3 * * *  /usr/local/bin/python /home/mqian/3366/jobsAssigner.py -d 2>> /home/mqian/3366/log3366
```
* git位置
http://192.168.30.251:10086/git/mengqian/namecard_info_extra_basic.git
