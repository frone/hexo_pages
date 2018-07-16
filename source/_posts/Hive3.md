---
title: Hive优化以及数据倾斜问题
date: 2017-02-10 18:00:00
tags:
    - etl tool
categories: Hive
---
# Hive优化以及数据倾斜问题

## HQL优化

1.调整Join顺序，确保以大表作为驱动表,即大表放在后面

2.去除查询中不需要的column
    
3.Where条件判断等在TableScan阶段就进行过滤
    
4.利用Partition信息，只读取符合条件的Partition

5.Map端join，以大表作驱动，小表载入所有mapper内存中

6.对于数据分布不均衡的表Group by时，为避免数据集中到少数的reducer上，分成两个map-reduce阶段。第一个阶段先用Distinct列进行shuffle，然后在reduce端部分聚合，减小数据规模，第二个map-reduce阶段再按group-by列聚合。
    
7.在map端用hash进行部分聚合，减小reduce端数据处理规模。

## 数据倾斜问题

数据倾斜
概念：数据倾斜是指，map /reduce程序执行时，reduce节点大部分执行完毕，但是

有一个或者几个reduce节点运行很慢，导致整个程序的处理时间很长，这是因为某一

个key的条数比其他key多很多（有时是百倍或者千倍之多），这条key所在的reduce

节点所处理的数据量比其他节点就大很多，从而导致某几个节点迟迟运行不完。

执行操作：

  1.其中一个表较小，但是key集中，可能会导致分发到一个或几个Reduce上的数据远高于平均值
  2.大表与大表关联，但是空值或者0比较多，这些控制都由一个reduce处理，非常慢。
  3.group by维度过小，某值的数量过多。处理某值的Reduce非常耗时。
  4.count distinct 某特殊值过多，处理此特殊值的Reduce耗时。
  
原因：

    1.key分布不均
    2.业务数据本身的特性
    3.建表时考虑不周
    4.某些语句本身就有数据倾斜
  

解决方案：

  1.参数调节
    hive.map.aggr=true
    Map端部分聚合，相当于Combiner(合并器)。
    hive.groupby.skewindata=true
    有数据倾斜的时候进行负载均衡，当选项设定为 true，生成的查询计划会有两个 MR Job。

    第一个 MR Job 中，Map 的输出结果集合会随机分布到 Reduce 中，每个 Reduce 做部分

    聚合操作，并输出结果，这样处理的结果是相同的 Group By Key 有可能被分发到不同的 

    Reduce 中，从而达到负载均衡的目的；第二个 MR Job 再根据预处理的数据结果按照 Group By Key 

    分布到 Reduce 中（这个过程可以保证相同的 Group By Key 被分布到同一个 Reduce 中），最后完

    成最终的聚合操作。

  2.如何join？关于驱动表的选取，选用join key分布最均匀的表做为驱动表。做好列裁剪和filter操作，

    以达到两表做join的时候数据量相对变小的效果。

  3.使用mapjoin让小的维度表先进内存。在map端完成reduce。

  4.大表join大表时，把空值的key变成一个字符串加上随机数，把倾斜的数据随机分布到不同的reduce上，

    由于null值关联不上，处理后并不影响最终结果。

  5.count distinct时，将值为空的情况单独处理，如果是计算count distinct，可以不用处理，直接过滤，

    在最后结果中加1。如果还有其它计算，需要进行group by，可以先将值为空的记录单独处理，再和其他

    结果进行union。

  6.采用sum() group by的方式来替换count(distinct )进行计算。

  7.在业务逻辑优化效果不大的情况下，有些时候是可以将倾斜的数据单独拿出来处理，最后union回去。

  8.对于控制产生的倾斜问题，解决方案1：为空的数据不参与关联；方案2：随机（rand()）赋予空值新的key。

    把空值的key变成一个字符串加上随机数，就能把倾斜的数据分到不同的reduce上，解决数据的倾斜问题。

  9.不同数据类型关联产生数据倾斜，默认的Hash操作会按int型的id来分配reduce，这样会导致所有的String

    类型数据分配到同一个reduce。

  10.使用 mapjoin 解决小表(记录数少)关联大表的数据倾斜问题，这个方法使用的频率非常高，但如果小表

     很大，大到map join会出现bug或异常，这时就需要特别的处理。