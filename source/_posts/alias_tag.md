---
title: 名片别名标注
date: 2017-05-08 16:31:19
tags:
    - 名片
    - 别名
categories: IMORA
---

## 名片别名标注


### 一.业务整体流程

为满足整体橙子语音检索时的需要，为名片现有数据自动添加标签。目前主要包括四部分：

* 根据职位头衔为高管级姓名后添加称呼，如总、董等。
* 为公司名添加别名，如北京橙鑫数据科技有限公司的别名为橙鑫或IMORA
* 从公司名通过切词来提取中心词，为公司名增加标签
* 从公司基本信息表获取城市字段，加为标签

1.**数据来源：**
可能涉及到如下数据表

* 别名信息表
oradt_cloud_test2.contact_card_removeword_alias
* 公司别名表
oradt_cloud_test2.orange_company_alias

2.**整体处理流程**
本系统使用到名片基本信息，需要在名片基本信息拓展系统处理之后运行，处理流程如下图：
![](http://wx3.sinaimg.cn/mw690/006nSoSwly1ffe26sy9vpj30l40dc3z3.jpg)
图片备份位置：/home/samba/share/mengqian/handover/名片信息拓展/alias-step.png

* 对应到代码，整体的入口为main，主要用来通过时间区间来控制需要计算的数据起始时间。并调用jobContrl中的读取数据、处理数据及序列化的方法，控制程序流程。
* 城市标签
由于添加城市标签功能为后续添加，没有修改整体处理流程，添加到main中，其他任务结束后，追加更新别名表中的city字段。
* 由jobContrl实现数据输入输出格式的转换,业务处理逻辑调用dataPreprocessing实现。
* dataPreprocessing 调用各个功能模块，拼合结果数据。


###  二.功能模块说明
主要功能模块的包结构如下图：
![](http://wx1.sinaimg.cn/mw690/006nSoSwly1ffe1e2slwuj30q50c60vb.jpg)
图片备份位置：/home/samba/share/mengqian/handover/名片信息拓展/alias-classes.png

* 公司名关键词提取
orgKeyWords
* 公司别名
companyAlias
* 姓氏后缀
respectedTitle

### 三.项目部署

* 当前每日任务部署于： 101.251.193.28:/home/mqian/orange_namecard_tagging
由crontab每天凌晨4点执行，本系统的需要使用到名片基础信息，需要在名片拓展系统任务完成之后运行
```
####为企业打别名、人名后加称呼等打标签
0 4 * * * /usr/local/bin/python /home/mqian/orange_namecard_tagging/main.py >>
home/mqian/aliaslog.log
```
* git位置
http://192.168.30.251:10086/git/mengqian/orange_namecard_tagging.git
