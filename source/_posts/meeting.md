---
title: 月报会议模块
date: 2017-03-02 14:31:19
tags:
    - 月报
categories: IMORA
---

### 会议相关月报样板

您xx月共参加了xx ×x小时的会议，与xx个人进行过会晤，最近较为关注的公司有xx,xx,xx(公司名) 
(数据取自日程，会晤人取日程参与人，关注公司取参与会议的人次较多的前三名公司；会议仅包括：橙子日程中从橙脉和手机日历获取的日程）


### 参加会议时间 
oradt_cloud1520.contact_card_schedule 取会议开始结束时间；
```
select user_id,SUM(meeting_time) as total_schedule_hour from(
select a.id as schedule_id,a.user_id,content,title,
FROM_UNIXTIME(start_time) as start_time,
FROM_UNIXTIME(end_time) as end_time,
(end_time - start_time)/(60*60*24) as day_diff,
case
-- 本月开始结束且一天之内结束
when (end_time - start_time)/(60*60*24) <= 1 and MONTH(FROM_UNIXTIME(start_time)) = 3 and MONTH(FROM_UNIXTIME(end_time))=3 
then round((end_time - start_time)/(60*60)) 
-- 本月开始结束且一天之内没有结束
when (end_time - start_time)/(60*60*24) > 1 and MONTH(FROM_UNIXTIME(start_time)) = 3 and MONTH(FROM_UNIXTIME(end_time))=3 
then round((end_time - start_time)/(60*60*24) * 8) 
-- 结束时间不是本月，则取月初第一天为结束时间
when (end_time - start_time)/(60*60*24) > 1 and MONTH(FROM_UNIXTIME(start_time)) = 3 and MONTH(FROM_UNIXTIME(end_time))!=3 
then round((UNIX_TIMESTAMP(last_day(curdate())) - start_time)/(60*60*24) * 8) 
-- 开始时间不是本月，则取月初第一天为开始时间 
when (end_time - start_time)/(60*60*24) > 1 
and MONTH(FROM_UNIXTIME(start_time)) != 3 and MONTH(FROM_UNIXTIME(start_time)) != 3 and MONTH(FROM_UNIXTIME(end_time))=3 
then round((end_time - UNIX_TIMESTAMP(DATE_ADD(curdate(),interval -day(curdate())+1 day)))/(60*60*24) * 8)
end as meeting_time
from oradt_cloud1520.contact_card_schedule a 
right join oradt_cloud1520.contact_card_schedule_map b on a.id = b.schedule_id
left join(
select user_id,uuid
    from oradt_cloud1520.contact_card
    where self='true' and status = 'active'
) as c on c.user_id = a.user_id and c.uuid = b.uuid 
WHERE a.start_time >0 and a.end_time > a.start_time 
and b.status != 0 and b.uuid != '' and c.uuid is null and a.status = 1
and YEAR(FROM_UNIXTIME(start_time)) = 2017 and YEAR(FROM_UNIXTIME(end_time))=2017
and (MONTH(FROM_UNIXTIME(start_time)) = 3 or MONTH(FROM_UNIXTIME(end_time))=3)
order by user_id,schedule_id
)c GROUP BY user_id;
```

### 名片角色总会见人数
先统计每个人参加了几个会议（会议需要去重）
```
select user_id, GROUP_CONCAT(DISTINCT(schedule_id)) as meeting_list from(
select a.user_id, a.id as schedule_id
from oradt_cloud1520.contact_card_schedule a
right join oradt_cloud1520.contact_card_schedule_map b on a.id = b.schedule_id
left join(
select user_id,uuid
    from oradt_cloud1520.contact_card
    where self='true' and status = 'active'
) as c on c.user_id = a.user_id and c.uuid = b.uuid  
WHERE a.start_time >0 and a.end_time > a.start_time 
and b.status != 0 and b.uuid != '' and c.uuid is null and a.status = 1
and YEAR(FROM_UNIXTIME(start_time)) = 2017 and YEAR(FROM_UNIXTIME(end_time))=2017
and (MONTH(FROM_UNIXTIME(start_time)) = 3 or MONTH(FROM_UNIXTIME(end_time))= 3)
order by schedule_id,user_id
) a GROUP BY user_id;
```
然后分别统计每个人的会议时长
```
select DISTINCT(count(uuid)) from oradt_cloud1520.contact_card_schedule_map where status != 0 and uuid != '' and
schedule_id in (3260,3890,3895,3915,4249,4252,4254,4255,4280,4822);
```

### 关注公司
oradt_cloud1520.contact_card_schedule_map 取uuid，即参与人；    
与DM库中公司表关联得到参与人公司,统计公司前三位
```
select user_id, GROUP_CONCAT(companyName ORDER BY com_count desc,last_modified desc) as com_list from(
select user_id,companyName,count(*) as com_count,max(last_modified) as last_modified from(
select d.*,FROM_UNIXTIME(g.last_modified) as last_modified from(
select a.user_id,b.schedule_id,b.uuid,c.companyName
from oradt_cloud1520.contact_card_schedule a 
right join oradt_cloud1520.contact_card_schedule_map b on a.id = b.schedule_id
left JOIN(select DISTINCT user_id,card_id,companyName from dm_test.company_info_jsonver) as c on b.uuid = c.card_id
left join(
select user_id,uuid
    from oradt_cloud1520.contact_card
    where self='true' and status = 'active'
) as f on f.user_id = a.user_id and f.uuid = b.uuid
WHERE a.start_time > 0 and a.end_time > a.start_time 
and b.status != 0 and b.uuid != '' and f.uuid is null and a.status = 1
and YEAR(FROM_UNIXTIME(start_time)) = 2017 and YEAR(FROM_UNIXTIME(end_time))=2017
and (MONTH(FROM_UNIXTIME(start_time)) = 3 or MONTH(FROM_UNIXTIME(end_time))= 3)
and c.companyName !='' and c.companyName is NOT null
)d left join oradt_cloud1520.contact_card g on g.uuid = d.uuid) h
GROUP BY user_id,companyName ORDER BY com_count desc,last_modified desc
)e GROUP BY user_id;
```

### 相关表结构
```
CREATE TABLE `contact_card_schedule` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
  `user_id` char(40) CHARACTER SET utf8 NOT NULL DEFAULT '' COMMENT '创建人user_id',
  `vcard_id` char(32) CHARACTER SET utf8 DEFAULT '' COMMENT '日程名片id',
  `content` text NOT NULL COMMENT '日程内容',
  `title` varchar(100) DEFAULT '' COMMENT '日程标题',
  `address` varchar(300) DEFAULT '' COMMENT '地址',
  `start_time` int(10) NOT NULL DEFAULT '0' COMMENT '日程开始时间',
  `end_time` int(10) NOT NULL DEFAULT '0' COMMENT '日程结束时间',
  `remind_time` int(10) NOT NULL DEFAULT '0' COMMENT '提醒时间',
  `cycle` int(10) NOT NULL DEFAULT '0' COMMENT '提醒周期（例：15分钟为900单位为秒）',
  `isallday` tinyint(1) NOT NULL DEFAULT '2' COMMENT '是否全天 ：1是  2否 ',
  `flag_time` int(10) NOT NULL DEFAULT '0' COMMENT '提醒计算字段（为时间戳）',
  `last_modify` int(10) NOT NULL DEFAULT '0' COMMENT '最后修改时间',
  `create_time` int(10) NOT NULL DEFAULT '0' COMMENT '创建时间',
  `latitude` decimal(13,8) NOT NULL DEFAULT '0.00000000' COMMENT '坐标',
  `longitude` decimal(13,8) NOT NULL DEFAULT '0.00000000' COMMENT '坐标',
  `is_remind` tinyint(1) NOT NULL DEFAULT '0' COMMENT '是否提醒',
  `status` tinyint(1) NOT NULL DEFAULT '1' COMMENT '1为添加2为删除',
  `flight_id` int(20) DEFAULT '0' COMMENT '行程卡id',
  `schedule_from` tinyint(2) NOT NULL DEFAULT '1' COMMENT '日程来源：1.为橙脉2.为手机同步3.短信同步为4.为行程卡5.为还款同步6.航旅卡',
  `schedule_info` text CHARACTER SET utf8 COMMENT '来源模块信息',
  `orange_remind` int(10) NOT NULL DEFAULT '0' COMMENT '橙子专用提醒时间',
  PRIMARY KEY (`id`),
  KEY `user_id` (`user_id`),
  KEY `v_cardid_idx` (`vcard_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=4771 DEFAULT CHARSET=utf8mb4

CREATE TABLE `contact_card_schedule_map` (
  `id` int(20) unsigned NOT NULL AUTO_INCREMENT,
  `uuid` char(32) NOT NULL DEFAULT '' COMMENT '邀请人cardid',
  `schedule_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '日程id',
  `last_modify` int(10) NOT NULL DEFAULT '0' COMMENT '最后更新时间',
  `operation` enum('delete','modify','add') NOT NULL COMMENT '参与人状态',
  `status` tinyint(1) NOT NULL DEFAULT '0' COMMENT '0 不参与 1 未接受 2 已接受 3 完成',
  PRIMARY KEY (`id`),
  KEY `sch_id` (`schedule_id`) USING BTREE,
  KEY `v_uuid_index` (`uuid`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=7093 DEFAULT CHARSET=utf8mb4

CREATE TABLE `orange_meeting_follow` (
  `id` int(10) NOT NULL AUTO_INCREMENT,
  `user_id` varchar(42) NOT NULL COMMENT '用户id',
  `meeting_hour` int(2) NOT NULL DEFAULT '0' COMMENT '会议小时数',
  `meeting_people` int(2) NOT NULL DEFAULT '0' COMMENT '面谈人数',
  `follow_comp` text COMMENT '关注公司列表',
  `report_month` int(6) NOT NULL COMMENT '月报生成年月,存储格式例：201702',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8 COMMENT='用户会议相关及关注公司统计表'
```
