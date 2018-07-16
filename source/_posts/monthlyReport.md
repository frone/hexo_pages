---
title: 月报各统计指标计算方法
date: 2017-02-15 10:14:01
tags:
    - 月报
    - 统计指标
categories: IMORA
---


## 月报各项目计算（初版）


### 一.消费记录

1.**数据来源：月报表**
<pre><font size=2>CREATE TABLE `orange_monthly_report` (
  `id` int(10) NOT NULL AUTO_INCREMENT,
  `user_id` varchar(42) NOT NULL COMMENT '用户ID',
  `content` text NOT NULL COMMENT '月报内容',
  `created_time` int(10) NOT NULL COMMENT '月报创建时间',
  `modify_time` int(10) NOT NULL COMMENT '处理时间',
  `report_month` varchar(6) NOT NULL COMMENT '月报生成年月',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8 COMMENT='月报数据表';
 
 
CREATE TABLE `orange_consume_detail` (
  `id` int(10) NOT NULL AUTO_INCREMENT,
  `user_id` char(42) NOT NULL COMMENT '用户id',
  `card_id` char(32) NOT NULL COMMENT '卡片id',
  `bankcard_id` char(32) NOT NULL COMMENT '银行卡号',
  `bank_name` varchar(90) NOT NULL COMMENT '银行名',
  `bill_type` tinyint(1) NOT NULL COMMENT '消费类型：1、手机银行支付，2、支取，3、转账等',
  `bill` float(10,2) NOT NULL COMMENT '消费金额',
  `balance` float(10,2) NOT NULL COMMENT '余额',
  `created_time` int(10) NOT NULL COMMENT '创建时间',
  `modify_time` int(10) NOT NULL COMMENT '修改时间',
  `report_month` varchar(6) NOT NULL COMMENT '月报生成年月',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=utf8 COMMENT='消费详情表';
</font></pre>


2.**消费记录详情SQL**

得到指定月份的用户消费详情
提供各卡片当月消费金额
<pre><font size=2>SELECT user_id, bankcard_id,bill_time,cons_type,bill FROM consume_details
WHERE  MONTH(bill_time) == "%s" AND user_id = "%s"</font></pre>

<font color=red size=2>待定：短信去重</font>


### 二.我在旅途
1.**数据来源：里程卡，酒店卡**

<font color=red size=2>待定：</font>

    2.里程卡中可能无法得到飞行次数数据。
    3.里程数可能无法涵盖所有公司。
    4.酒店卡数据来源？？

5.抵达城市算法，只用手机GPS位置   

### 三.人脉动态
1.新增名片数，新增好友数
从数据库中获取新增名片数，新增好友数

新增好友数计算：contact_card(is_friend,from_uid,created_time)。从该表中按创建时间分组后取好友uid最早的记录，看是否落入统计区间。

2.被关注人数
来自谁看过我的消费结果
contact_card_see_me_source(未去重）
统计自然月的关注过用户的人数

3.参加会议时间 

http://192.168.30.191/example/orange_schedule.txt
日程相关说明

    oradt_cloud1520.contact_card_schedule 取会议开始结束时间；

4.关注公司

    oradt_cloud1520.contact_card_schedule_map 取uuid，即参与人；    
    与DM库中公司表关联得到参与人公司,统计公司前三位。如有参与人公司排名相同的情况，取名片更新时间最新的公司。


### 四.卡片统计
日志中获取卡片展示超过5秒的所有卡片，每找到一对展示开始时间超过5秒的记录，认为是一次使用，记入DM库
<pre><font size=2>CREATE TABLE `card_used_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `user_id` char(40) DEFAULT '' COMMENT '用户id',
  `card_id` char(32) DEFAULT '' COMMENT '卡片id',
  `daily_used_count` int(10) DEFAULT '' COMMENT '卡片当日使用次数',
  `card_type` char(32) DEFAULT '' COMMENT '卡片类型',
  `used_time` datetime COMMENT '卡片使用时间',
  `batch_time` datetime DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='卡片使用次数表'</font></pre>

展示结果为按日别、卡别、用户统计使用次数
各类卡使用占比则先分组
常用卡列表来自于hive
### 附录
月报json存储格式
<pre><font size=2>{
    "consume": {
        "totle": 58097.86,
        "bank": [
            {
                "bank_name": "招商银行",
                "card_num": 1234567898765,
                "amount": 300.23
            },
            {
                "bank_name": "招商银行",
                "card_num": 9876543212345,
                "amount": 451.8
            },
            {
                "bank_name": "工商银行",
                "card_num": 66436743634424,
                "amount": 670.83
            },
            {
                "bank_name": "建设银行",
                "card_num": 63762343254654,
                "amount": 1270.31
            }
        ]
    },
    "travel": {
        "flight": 28,
        "mileage": 10867,
        "stay": 15,
        "city": "北京、东京、纽约、斯德哥尔摩"
    },
    "connection": {
        "meeting_hour": 28,
        "meeting_people": 15,
        "new_friend": 98,
        "new_card": 887,
        "followed": 185,
        "follow_comp": "苹果、google、三星"
    },
    "card": {
        "category": [
            {
                "type": "行程卡",
                "freq": 90
            },
            {
                "type": "名片",
                "freq": 50
            },
            {
                "type": "酒店卡",
                "freq": 50
            },
            {
                "type": "系统卡",
                "freq": 10
            },
            {
                "type": "other",
                "freq": 100
            }
        ],
        "recent_use": "33829923329,53829923329,9829923380"
    }
}

** 此处的 other是取使用在4次以上，并不在前4位的卡片，非卡片类型的其它卡片
</font></pre> 

