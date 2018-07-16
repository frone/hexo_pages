---
title: 名片纠正服务文档
date: 2016-09-30 18:12:41
tags:
    - NameCard
    - Levenstein Distance
---
<Excerpt in index | 首页摘要>

- 源代码
```
    git clone git@192.168.30.251@cardRectify.git
```
名片中的公司名称、职位名称、姓名等信息需要纠正或重新格式化，以获得更准确的信息，便于消除错误数据，去除冗余数据等，提升数据质量。<!-- more -->
<The rest of contents | 余下全文>

- - -

- 部署位置：

    - OFFICE：Oradt-BigData:/opt/workspace/cardRectify （IP：192.168.30.251）

    - IDC：   BJIDC-Oradt-DMMySQL-1:/opt/workspace/cardRecity （IP：172.17.4.1 / 192.168.1.138）

- 数据源
	> host： 192.168.1.139
	> database： dm
	> table： scan_card_vcardinfo

	数据源为从生产库中同步并整合的名片数据表。

- 获取逻辑
	每次执行时，以时间戳作为本次执行的批次号，任务执行成功后，记录该批次号。下次任务执行时，先获取上一次记录的批次号，从数据源中取出`updateTime`字段落在新旧批次号之间的所有记录。如果是首次执行，则取出所有`updateTime`字段小于本批次号的记录。

## 处理逻辑
- 目标
	获取与当前字段编辑距离最小的 **K** 个候选（K默认为3）。

- 处理过程
	- 获取`handled`状态名片的`ORG`和`TITLE`， 按频率从大到小排序，分别作为比较基准集合。

	- `ORG`：从数据源中获取`needhandle`状态的名片，取出该名片的`ORG`字段，与基准集合中的`ORG`依次做比较，求编辑距离，将基准集合中编辑距离小于预设阈值的`ORG`按编辑距离大小加入候选集合，**每个编辑距离的候选最多保留K个记录**（编辑距离为0的只有一个）。计算完成后，再从候选集合中取出编辑距离较小的 **K** 个。

	- `TITLE`的处理逻辑同`ORG`。

## 数据输出
- 输出位置
	> host： 172.17.3.1
	> database: oradt_cloud_test
	> table： scan_card_rectify

	将结果 **插入** 或 **更新** 到结果表中（PK：`card_id`）。


- - -
## 后记
- 分发逻辑：
	- 分发时获取生产库表中`handle_state`为`needhandle`状态的名片，在`scan_card_rectify`表中查找纠正建议（理论上`scan_card_rectify`能包含所有`needhandle`状态的记录，**存在数据同步和计算时间差**），然后分发。

