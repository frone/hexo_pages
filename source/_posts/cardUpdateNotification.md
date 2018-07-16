---
title: 人脉信息更新
date: 2016-09-27 14:07:19
tags:
    - 人脉信息更新
    - 系统提示
---
<Excerpt in index | 首页摘要>
“对于当前用户持有的非注册用户的名片，系统根据一定规则检测有新的数据更新时提示用户。”<!-- more -->
<The rest of contents | 余下全文>

## 源码
- 版本库地址：
  - OFFICE:   `git@192.168.30.251:cardUpdateNotification.git`

  - IDC: `git@172.17.4.151:cardUpdateNotification.git (云主机 /data/git/ 目录下)`

- 部署位置:

  `BJIDC-Oradt-DMMySQL-1:/opt/workspace/cardUpdateNotification` (IP: `172.17.4.1`/`192.168.1.138`)


## 背景
  > 对于当前用户持有的非注册用户的名片，系统根据一定规则检测有新的数据更新时提示用户。并推荐持有人（未来）

  > 判断规则：

  > 1）5个以上其他用户也持有（名片 姓名、手机号一致）

  > 2）其他用户持有名片主要信息一致（姓名、手机号、公司名称、职位），但与我持有名片信息不一致（公司名称、职位有至少一项不一致）

  > 3）其他用户是在我对名片的最后修改时间后  获取或更新

  > 提示时间：系统定期（一天）检测，发现有不一致信息后，在每天19点（可配置），为用户 推送该消息 （实现有问题全库轮询？？）

详见黄小明邮件(2016年9月14日13时)

## 数据源及说明

- 获取名片信息
    - 目标数据库: `172.17.4.3:3306/oradt_cloud`

    - 数据表
      - **contact_card**: 名片状态表（来源等）

      - **contact_card_vcardinfo**: 名片具体信息表

      - **account_basic**: 注册用户信息表

  从以上数据表中获取所要操作的名片

- 过滤规则

    `cellphone`字段：从名片信息表中获取vCard格式的`TEL`字段，使用正则表达式抽取**第一个**CELL作为该名片的`cellphone`

    - 过滤已注册用户：

      - 从注册用户信息表中获取出已注册手机号集合

      - 过滤掉`cellphone`在已注册手机号集合里的名片

    - `cellphone`不能为空

    如果`cellphone`为空，根据`fn`和`cellphone`进行分组，就退化成**仅用**`fn`进行分组，无法处理重名的名片。

- 名片持有人
> 5个以上其他用户也持有（名片 姓名、手机号一致）

  这里5个*其他用户*用5个**（姓名，手机号）** 组对代替，而非5个不同的`user_id`。同一用户可能拥有不同**（姓名，手机号）** 组对。

## 整体思路

- 获取数据库表中的名片信息数据，并进行过滤（过滤规则：未注册，且`cellphone`不能为空）

- 先使用`fn`和`cellphone`分组，得到一级分组Group_1L，每组中的名片都属于同一个人，但`org`, `title`可能不同

- 过滤掉一级分组Group_1L中每组数量小于预设值（*`5`个以上其他在用户也持有*）的名片

- 在一级分组Group_1L之上，对每组再使用`org`和`title`进行分组，得到二级分组Group_2L，每组中的名片都是一张特定的名片，`fn`, `cellphone`, `org`, `title`都一样

- 过滤掉二级分组Group_2L中每组数量小于预设值（大于等于`2`，避免名片自身有误带来的影响）

- 从二级分组Group_2L中选取名片最旧的一张（即最先被录入的那张名片），加入候选集

- 从候选集中选出最新的一张（即该用户最新的一张名片），使用该张名片作为最后的目标名片

- 对一级分组Group_1L中**该组**（`fn`和`cellphone`相同）未拥有该张名片（`org`**或**`title`不同）的所有用户提醒更新

- 将提醒消息【`to_user`, `card_id`, `new_card_id`】插入数据库中

## 运行概况

  在BJIDC-Oradt-DMMySQL-1（IDC部署环境 ）上运行该脚本，以下是运行结果：

  从数据库表中获取数据量：**2919117**，过滤后，一级分组输入数据量：**1855257**

  > Total Messages: 128549
  > python cardUpdateNotification.py  678.20s user 3.95s system 98% cpu 11:34.90 total

  共产生**128549**条消息，用时**11分钟35秒**。

  **注：** 一次产生的消息量`10w` 级，如果每天更新，插入数据库，请考虑可行性。

  *建议： 对于消息通知结果表中，如果插入的数据相同【`to_user`, `card_id`, `new_card_id`】，则不再插入（联合主键可以确保插入相同的数据时，插入操作失败），加入一个`是否通知`列或`最后通知时间`列说明状态，但需根据业务情况考虑什么情况下更新这些状态列（每天一次？谁更新？）。*

## 补充消息推送的格式
####   ——by mazf
- 消息json格式（参考黄小明的消息文档 push_message_list.xlsx）：
{"messagetype":124,"params":
{
"accountid":"所属人clientid",
"imid":"11",
"vcardid":"名片id",
"picture":"名片缩略图",
"contactname": "名片姓名"
}
}
- 数据库 message_queue 表：
这个表中需要写入的数据有如下4个字段：
type, to_uid, content, created_time
- Python 代码中 notice 函数，返回推送消息列表，循环调用 sendMsg 函数。


插入数据接口已预留，但现在数据库还没有建表，具体写入位置还待确定；

## 配置文件依照需求适当修改。
配置文件示例：
{
  "r":{ # 读数据配置
    "host": "192.168.1.138",
    "port": 3306,
    "user": "dm_rw",
    "password": "dmrw4142",
    "db": "oradt_cloud",
    "cardsQuery": "select t1.user_id, t1.uuid, FN, TEL, ORG, TITLE, last_modified, t1.picture from contact_card t1 left join contact_card_vcardinfo t2 on t1.uuid = t2.card_id where t1.status = 'active' and t1.self = 'false' limit 100000;",
    "regUsersQuery": "select distinct mobile from account_basic;"
  },
  "w":{ # 写数据配置
    "host": "localhost",
    "port": 3306,
    "user": "root",
    "password": "mazf",
    "db": "test"
  }
}

----
