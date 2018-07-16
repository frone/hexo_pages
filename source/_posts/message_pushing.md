---
title: 资讯推送|用户推广|活动管理
date: 2017-02-14 02:14:14
tags:
    - msyql
    - message queue
categories: IMORA
---

### 整体启动

main.py

通过多线程同时进行三个任务 (确保每个环境只有一个进程在运行)
nohup python main.py&

#### 相关配置
配置写在文件 db.conf中
只需要根据运行环境更改数据库的配置名称即可，如：
oa = operationActivity.OperationActivity("database_dev") // 192 
oa = operationActivity.OperationActivity("database_prd") // 125

```
# !/Users/xieqj/py_env/venv python
# -*- coding: utf-8 -*-


import threading
from mg_handling import msgPushing, userPromotion, operationActivity
import util.comm as cm
import time

# 资讯推送
def msg_pushing():
    while (True):
        mp = msgPushing.MsgPushing()
        msg_contents = mp.scanDBForSending()
        mp.insert_msg2mq(msg_contents)
        mp.mark_msg_as_sent(msg_contents)
        mp.close_database()
        time.sleep(8)

# 用户推广
def user_promotion():
    while (True):
        mp = userPromotion.UserPromotion()
        msg_contents = mp.scanDBForSending()
        mp.insert_msg2mq(msg_contents)
        mp.mark_msg_as_sent(msg_contents)
        mp.close_database()
        time.sleep(8)

# 活动管理
def operation_activity():
    while (True):
        oa = operationActivity.OperationActivity()
        msg_contents = oa.scanDBForSending()
        oa.insert_msg2mq(msg_contents)
        oa.mark_msg_as_sent(msg_contents)
        oa.close_database()
        time.sleep(8)


# 通过多线程同时启动两个步骤
if __name__ == "__main__":
    cm.initLog("pushing.log", "INFO")
    threads = []
    t1 = threading.Thread(target=msg_pushing)
    threads.append(t1)
    t2 = threading.Thread(target=user_promotion)
    threads.append(t2)
    t3 = threading.Thread(target=operation_activity)
    threads.append(t3)
    for t in threads:
        t.setDaemon(True)
        t.start()
    t.join()

```

### 资讯推送

读取表 sns_qa_push_setting 中的资讯，将其按照region，industry，func（职能）推送给对应群组的用户。
具体方法就是将表 sns_qa_push_setting 中的 content 1对1的推送给表 account_basic 中对应群组的用户，即插入 message_queue 表。

需要注意控制推送时间，和在推送后 update 表 sns_qa_push_setting 中 status 字段

#### 仓库位置

git@192.168.30.251:message_handling.git

{"messagetype": 230,"params": { "data": [{"newsid": " EwtNMSASOSvxjerhni98uQyEXnDIp9fS","title": "发布到审核","url": "资讯  URL", "coverurl": "https: //oradtdev.s3.cn-north-  1.amazonaws.com.cn/ukIv0BqxtnjRgjuhJcGKO6jze1liPmZE/gm9YtnBtvQ20160921132139.jpg","createdtime": "1474435299" } ]}}


#### 数据表

```
CREATE TABLE `industry_category` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `category_id` char(8) NOT NULL,
  `parent_id` char(8) NOT NULL,
  `name` varchar(128) NOT NULL COMMENT '行业名称',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='国家统计局的两级行业分类标准'

# 保存从名片的json数据解析出来的数据 name * region
CREATE TABLE `card_info_jsonver` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `card_id` char(32) DEFAULT '' COMMENT '名片id',
  `user_id` char(40) DEFAULT '' COMMENT '用户id',
  `side` tinyint(4) DEFAULT '0' COMMENT '名片的正反面，0：正面，1：反面',
  `name` varchar(128) DEFAULT '' COMMENT '名片上的名字',
  `mobile` varchar(20) DEFAULT '' COMMENT '手机号',
  `provinceM` varchar(64) DEFAULT '' COMMENT '手机号解出来的省份',
  `provinceM_code` varchar(16) DEFAULT '' COMMENT '省份编码',
  `cityM` varchar(64) DEFAULT '' COMMENT '城市',
  `cityM_code` varchar(16) DEFAULT '' COMMENT '城市编码',
  `created_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '名片创建时间',
  `updated_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '名片更新时间',
  `handle_state` varchar(16) DEFAULT '' COMMENT '处理状态',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=42333 DEFAULT CHARSET=utf8 COMMENT='保存从名片的json数据解析出来的数据'

# 从名片json数据解析出来的公司信息 companyName * func
CREATE TABLE `company_info_jsonVer` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `card_id` char(32) DEFAULT NULL COMMENT '名片id',
  `user_id` char(40) DEFAULT NULL COMMENT '用户id',
  `companyName` varchar(128) DEFAULT NULL COMMENT '公司名称',
  `industry` varchar(128) DEFAULT NULL COMMENT '二级行业',
  `ind_code` char(8) DEFAULT NULL COMMENT '二级行业编码',
  `title` char(64) DEFAULT NULL COMMENT '职位',
  `func_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '职能名称',
  `func_code` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '职能编号',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='从名片json数据解析出来的公司信息'


消息类型
 
{
    "messagetype": 230,
    "params": {
        "data": [
            {
                "newsid": 资讯id,   
                "title": "资讯标题",
               "url": "资讯URL",
                "coverurl": "封面URL",
            },
            ....
        ]
    }
}
注：排序 按照data中数组的顺序
```

----

### 用户推广

基本流程与咨询推送相同

#### 仓库位置

git@192.168.30.251:message_handling.git

git@192.168.30.251:effective.git

userPromotion.py

#### 推广表

```
CREATE TABLE `sys_user_promotion` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `pro_id` char(32) NOT NULL COMMENT '推广ID',
  `type` tinyint(2) NOT NULL DEFAULT '0' COMMENT '推送方式：1：内部 2：邮件 3：短信',
  `isalluser` tinyint(2) NOT NULL DEFAULT '1' COMMENT '1发送注册用户2发送全部用户',
  `push_time` int(10) NOT NULL COMMENT '设置的推送时间',
  `region` varchar(2000) NOT NULL DEFAULT '' COMMENT '地区（多个,号隔开）',
  `industry` varchar(2000) NOT NULL DEFAULT '' COMMENT '行业（多个,号隔开）',
  `func` varchar(2000) NOT NULL DEFAULT '' COMMENT '职能（多个,号隔开）',
  `title` varchar(50) CHARACTER SET utf8mb4 DEFAULT '' COMMENT '标题',
  `content` text NOT NULL COMMENT '推送的内容',
  `created_time` int(10) NOT NULL COMMENT '添加时间',
  `admin_id` char(40) NOT NULL DEFAULT '' COMMENT '设置人',
  `status` tinyint(2) NOT NULL DEFAULT '1' COMMENT '状态 1：未发 2：已发',
  `isntice` tinyint(2) DEFAULT '0' COMMENT '是否通知',
  `url` varchar(256) NOT NULL COMMENT 'url地址',
  `json_data` text NOT NULL COMMENT '推送json格式',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=58 DEFAULT CHARSET=utf8 COMMENT='运营后台-用户推广表'

-- 存储type为2，3的用户推广
CREATE TABLE `message_promotion` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `type` tinyint(4) NOT NULL DEFAULT '0' COMMENT '类型：1-内部，2-邮件，3-短信',
  `mobile` varchar(18) DEFAULT '' COMMENT '手机号',
  `email` varchar(256) DEFAULT '' COMMENT '邮箱',
  `title` varchar(100) NOT NULL DEFAULT '' COMMENT '标题',
  `content` text NOT NULL COMMENT '消息内容',
  `created_time` int(11) DEFAULT NULL COMMENT '创建时间',
  `modified_time` int(11) DEFAULT NULL COMMENT '修改时间',
  `status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '状态：0-未发送，1-已发送，2-发送失败',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8mb4 COMMENT='推广消息表'
 
推广消息类型
 
{
  "messagetype":231,
  "params":{
     "id":"文章id"，
      "title": "资讯标题",
      "content":"内容"，
      "createdtime": "1345764830"
   }
}
{
  "messagetype": 231,
  "params": {
    "content": "hello world！<br/>",
    "createdtime": 1483620281,
    "id": "zlJjJD6gDvjCdoMcibqCWOGrMo7vbB0b",
    "image": "",
    "title": "推送测试",
    "type": "user",
    "url": "h5/News/secretary?type=promotion&id=zlJjJD6gDvjCdoMcibqCWOGrMo7vbB0b"
  }
}
```

----

### 活动管理

#### 背景说明
系统中，内容管理-推送、运营管理-新增推广、运营管理-新增活动页面中，地区、行业、职能选择时，弹窗样式与“内容管理-推送”样式相同，具体请参见原型

进行推送时，匹配规则如下：
a)  行业：第一次使用APP时所选行业+当前默认身份的并集。采用1级行业-2级行业模式划分
b)  地区：第一次使用APP时所选手机号+当前默认身份的并集。采用1级地区-2级地区模式划分
c)  职能：当前默认身份。采用1级行业-职能（只有1级）模式划分

当前默认身份cardid需要通过表 account_basic_detail 的 card_id 字段得到

#### 业务流程


#### 仓库位置
git@192.168.30.251:message_handling.git

OperationActivity

#### 活动表结构

```
CREATE TABLE `operation_activity` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `activity_id` char(32) CHARACTER SET utf8 NOT NULL COMMENT '活动id',
  `type` tinyint(4) NOT NULL DEFAULT '1' COMMENT '类型 1:活动 2:资讯',
  `isnotify` tinyint(4) NOT NULL DEFAULT '1' COMMENT '1:不通知 2:通知',
  `push_state` tinyint(4) NOT NULL DEFAULT '1' COMMENT 'isnotify为2时有效，1:未推送，2:已推送',
  `region` varchar(256) CHARACTER SET utf8 NOT NULL COMMENT '地区（多个,号隔开）',
  `industry` varchar(256) CHARACTER SET utf8 NOT NULL COMMENT '行业（多个,号隔开）',
  `func` varchar(256) CHARACTER SET utf8 NOT NULL COMMENT '职能（多个,号隔开）',
  `show_id` varchar(400) CHARACTER SET utf8 DEFAULT NULL COMMENT '资讯shwo_id（多个,号隔开）type为2时有效',
  `image` varchar(256) CHARACTER SET utf8 DEFAULT NULL COMMENT '标题图片',
  `title` varchar(100) CHARACTER SET utf8 DEFAULT NULL COMMENT '活动标题 type为1时使用',
  `content` text CHARACTER SET utf8 COMMENT '活动标题 type为1时使用',
  `onlinetime` int(11) NOT NULL DEFAULT '0' COMMENT '上线时间',
  `offlinetime` int(11) NOT NULL DEFAULT '0' COMMENT '下线时间',
  `user_id` char(40) CHARACTER SET utf8 NOT NULL COMMENT '发布人用户id',
  `created_time` int(11) NOT NULL DEFAULT '0' COMMENT '创建时间',
  `modify_time` int(11) NOT NULL DEFAULT '0' COMMENT '修改时间',
  `state` tinyint(4) NOT NULL DEFAULT '1' COMMENT '状态 1:未发布 2:已发布 3:已下线 4:已删除',
  `click_count` int(11) NOT NULL DEFAULT '0' COMMENT '点击量',
  `share_count` int(11) NOT NULL DEFAULT '0' COMMENT '分享量',
  `share_user_count` int(11) NOT NULL DEFAULT '0' COMMENT '分享用户量',
  `push_count` int(11) NOT NULL DEFAULT '0' COMMENT '推送用户量',
  `url` varchar(256) NOT NULL COMMENT 'URL地址',
  `json_data` text NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `activity_id` (`activity_id`)
) ENGINE=InnoDB AUTO_INCREMENT=94 DEFAULT CHARSET=utf8mb4 COMMENT='运营活动表'

推送类型
{
  "messagetype":231,
  "params":{
      "id":"文章id"，
      "title": "标题",
      "content":"内容"，
      "createdtime": "1345764830",
      "image":"图片URL"，
      "type":"活动（activity）或者用户推广（user）"
   }
}
```
----
### 带推送消息查询语句
#### 资讯推送
select id,FROM_UNIXTIME(push_time) as push_time,status,region,industry,func,admin_id,content from oradt_cloud_test2.sns_qa_push_setting
where `status` <> 2 and industry != '' and func != '' and region != ''
and ROUND((UNIX_TIMESTAMP(SYSDATE()) - push_time)/60) >= -2;
#### 用户推广
select id,FROM_UNIXTIME(push_time) as push_time,status,region,industry,func,admin_id,title,json_data from oradt_cloud_test2.sys_user_promotion
where `status` <> 2 and industry != '' and func != '' and region != ''
and ROUND((UNIX_TIMESTAMP(SYSDATE()) - push_time)/60) >= -2;
#### 活动推广
select id,push_state, region, industry, func, title, json_data, user_id from oradt_cloud_test2.operation_activity
where push_state <> 2 and industry != '' and func != '' and region != '' and ROUND((UNIX_TIMESTAMP(SYSDATE()) - push_time)/60) >= -2;

----
### 部署情况

#### 125仿真环境 

部署在IDC 云主机 /home/xieqj/python2.7_scenario/fangzhen

```
0 */1 * * * python2.7 /home/xieqj/python2.7_scenario/fangzhen/CardDeduplication/CardDeduplication3366.py
```
名片去重 60分钟运行一次 
名片异常 5分钟运行一次
人脉更新 60分钟运行一次
消息推送 main.py 60分钟一次

python2.7 运行的是125 3366端口
python2.7 message_handling/main.py (消息推送)
python2.7 CardDeduplication/CardDeduplication.py
python2.7 CardExceptionHandling/CardException.py
python2.7 cardUpdateNotification/CardUpdateNote.py

python 运行的是3388端口
python CardDeduplication.py
python CardException.py

#### AWS 

部署在ec2 54.223.28.119
```
0 0 */1 * * python2.7 /home/ec2-user/namecard/CardDeduplication/CardDeduplication_aws.py
```

名片异常 每5分钟运行一次
名片去重 每24小时运行一次
用户推广 每1分钟运行一次
消息推送 每1分钟运行一次

