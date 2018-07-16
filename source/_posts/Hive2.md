---
title: Hive入门讲解
date: 2017-02-10 18:00:00
tags:
    - etl tool
categories: Hive
---
# Hive入门讲解
Hive定位：ETL（数据仓库）工具

将数据从来源端经过抽取（extract）、转换（transform）、加载（load）至目的端的工具,如像：kettle	

## DML

**批量插入/批量导入**
```
LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)]
注：filepath可以是hdfs路径或者是S3路径，如hdfs://namenode:9000/user/hive/project/data1
```
```
1.从本地文件导入到表
load data local inpath 'test.txt' into table test;
```
```
2.从hdfs导入到表
load data inpath '/home/test/add.txt' into table test;
```
```
3.从表查询中导入到表
insert into table test select id, name, tel from test;
```
```
4.将查询数据导入到多个表
from source_table
insert into table test select id, name, tel from dest1_table select src.* where src.id < 100
insert into table test select id, name, tel from dest2_table select src.* where src.id < 100
insert into table test select id, name, tel from dest3_table select src.* where src.id < 100;
```
```
5.建表时导入
create table test4 as select id, name, tel from test;
```

**指定分隔符导出数据**

```
insert overwrite local directory '/home/hadoop/export_hive' 
row format delimited 
fields terminated by '\t' 
select * from test;
```

**删除/清空**

```
1.删除table1中不符合条件的数据
insert overwrite table table1 
select * from table1 where XXXX;
```
```
2.清空表
insert overwrite table t_table1 
select * from t_table1 where 1=0;
```
```
3.截断表（注：不能截断外部表）
truncate table table_name;
```
```
4.删除hdfs对应的表数据达到清空表（表结构依然存在）
hdfs dfs -rmr /user/hive/warehouse/test

注：1和2本质是覆写表来实现清除数据
```

**delete 与 update**

在hive中默认不支持事务，因此默认不支持delete与update，如果需要支持必须在hive-site.xml中配置打开

## DDL

库/表/索引/视图/分区/分桶

#### 数据库

列出/创建/修改/删除/查看信息

```
1.列出所有数据库
show databases;
```
```
2.创建数据库
create database test;
```
```
3.删除
drop database test;

处于安全原因，直接drop有数据的数据库会报错，此时需要cascade关键字忽略报错删除
drop database if exists test cascade;
```
```
4.查看数据库信息
describe database test;

```

#### 表
列出/创建/修改/删除/查看信息
```
1.列出所有表

当前数据库的所有表
show tables;

指定数据库的所有表
show tables in db_name;

支持正则
show tables '.*s';
```
```
2.创建表
create table test
(id int,
a string
)
ROW FORMAT DELIMITED        行分割
FIELDS TERMINATED BY ‘,’    字段分隔符
LINES TERMINATED BY ‘\n’    行分隔符
STORED AS TEXTFILE;         作为文本存储

```
```
创建基于正则切分行字段的表
add jar ../build/contrib/hive_contrib.jar;

CREATE TABLE apachelog (
host STRING,
identity STRING,
user STRING,
time STRING,
request STRING,
status STRING,
size STRING,
referer STRING,
agent STRING)
ROW FORMAT SERDE 'org.apache.hadoop.hive.contrib.serde2.RegexSerDe'
WITH SERDEPROPERTIES (
"input.regex" = "([^ ]*) ([^ ]*) ([^ ]*) (-|//[[^//]]*//]) ([^ /"]*|/"[^/"]*/") (-|[0-9]*) (-|[0-9]*)(?: ([^ /"]*|/"[^/"]*/") ([^ /"]*|/"[^/"]*/"))?",
"output.format.string" = "%1$s %2$s %3$s %4$s %5$s %6$s %7$s %8$s %9$s"
)
STORED AS TEXTFILE;
```
```
3.修改
加一个新列
ALTER TABLE test ADD COLUMNS (new_col2 INT COMMENT 'a comment');

改表名
ALTER TABLE old_name RENAME TO new_name;

```
```
4.删除
drop table test;
```
```
5.查看信息

显示列信息
desc test;

显示详细表信息
desc formatted test;

```
#### 索引

```
创建索引
CREATE INDEX index_name   
ON TABLE base_table_name (col_name, ...)  
AS 'index.handler.class.name'

如：DROP INDEX index_name ON table_name  

重建索引
ALTER INDEX index_name ON table_name [PARTITION (...)] REBUILD  

如：alter index index1_index_test on index_test rebuild; 

删除索引
DROP INDEX index_name ON table_name  

列出索引
show index on index_test; 
```

#### 视图
```
CREATE VIEW [IF NOT EXISTS] view_name [ (column_name [COMMENT column_comment], ...) ][COMMENT view_comment][TBLPROPERTIES (property_name = property_value, ...)] AS SELECT

注：hive只支持逻辑视图，不支持物化视图
•增加视图
•如果没有提供表名，视图列的名字将由定义的SELECT表达式自动生成
•如果修改基本表的属性，视图中不会体现，无效查询将会失败
•视图是只读的，不能用LOAD/INSERT/ALTER
•删除视图  DROP VIEW view_name

```


#### 分区（重点）
列出/创建/修改/删除

```
1.列出一个表的所有分区
show  partitions test;
```
```
2.创建分区表
create table test
(id int,
a string,
)
partitioned by (b string,c int) 
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ‘,’ 
LINES TERMINATED BY ‘\n’ 
STORED AS TEXTFILE;
```
```
3.对现有表添加分区
ALTER TABLE test ADD IF NOT EXISTS
PARTITION (year = 2017) LOCATION ‘/hiveuser/hive/warehouse/data_zh.db/data_zh/2017.txt’;
```
```
4.删除分区
ALTER TABLE test DROP IF EXISTS PARTITION(year =2017);
```
```
5.加载数据到分区表
LOAD DATA INPATH ‘/data/2017.txt’ INTO TABLE test PARTITION(year=2017);
```
```
6.未分区表数据导入分区表
insert overwrite table part_table partition (YEAR,MONTH) select * from no_part_table;
```
```
7.动态分区指令
set hive.exec.dynamic.partition=true;
set hive.exec.dynamic.partition.mode=nonstrict;
#set hive.enforce.bucketing = true;

开启动态分区后导入数据时可以省略指定分区的步骤
LOAD DATA INPATH ‘/data/2017.txt’ INTO TABLE test PARTITION(year);
```

#### 分桶
```
CREATE TABLE bucketed_user (id INT) name STRING)
CLUSTERED BY (id) INTO 4 BUCKETS;
```
对于每一个表（table）或者分区， Hive可以进一步组织成桶，也就是说桶是更为细粒度的数据范围划分。Hive也是 针对某一列进行桶的组织。Hive采用对列值哈希，然后除以桶的个数求余的方式决定该条记录存放在哪个桶当中。

把表（或者分区）组织成桶（Bucket）有两个理由：

（1）获得更高的查询处理效率。桶为表加上了额外的结构，Hive 在处理有些查询时能利用这个结构。具体而言，连接两个在（包含连接列的）相同列上划分了桶的表，可以使用 Map 端连接 （Map-side join）高效的实现。比如JOIN操作。对于JOIN操作两个表有一个相同的列，如果对这两个表都进行了桶操作。那么将保存相同列值的桶进行JOIN操作就可以，可以大大较少JOIN的数据量。

（2）使取样（sampling）更高效。在处理大规模数据集时，在开发和修改查询的阶段，如果能在数据集的一小部分数据上试运行查询，会带来很多方便。


## DQL

```
hive中的order by、distribute by、sort by和cluster by

order by 
全局排序，只有一个Reduce任务

sort by 
只做jubu排序

distribute by 
用distribute by 会对指定的字段按照hashCode值对reduce的个数取模，然后将任务分配到对应的reduce中去执行

cluster by 
distribute by 和 sort by 合用就相当于cluster by，但是cluster by 不能指定排序为asc或 desc 的规则，只能是desc倒序排列

注：distribute by 和 sort by,distribute by 必须在 sort by前面
```
```
表连接
1.join只支持等值连接

2.可以 join 多于 2 个表
SELECT a.val, b.val, c.val FROM a JOIN b 
    ON (a.key = b.key1) JOIN c ON (c.key = b.key2)
注:如果join中多个表的 join key 是同一个，则 join 会被转化为单个 map/reduce 任务

3.LEFT，RIGHT和FULL OUTER
SELECT a.val, b.val FROM a LEFT OUTER JOIN b ON (a.key=b.key)

如果 d 表中找不到对应 c 表的记录，d 表的所有列都会列出 NULL，包括 ds 列。也就是说，join 会过滤 d 表中不能找到匹配 c 表 join key 的所有记录。这样的话，LEFT OUTER 就使得查询结果与 WHERE 子句无关

•解决办法
　　SELECT c.val, d.val FROM c LEFT OUTER JOIN d 
     ON (c.key=d.key AND d.ds='2009-07-07' AND c.ds='2009-07-07')
     
4.LEFT SEMI JOIN
LEFT SEMI JOIN 的限制是， JOIN 子句中右边的表只能在 ON 子句中设置过滤条件，在 WHERE 子句、SELECT 子句或其他地方过滤都不行

SELECT a.key, a.value 
  FROM a 
  WHERE a.key in 
   (SELECT b.key 
    FROM B);

可以被重写为：

SELECT a.key, a.val 
   FROM a LEFT SEMI JOIN b on (a.key = b.key)
   
5.UNION ALL
用来合并多个select的查询结果，需要保证select中字段须一致
　　select_statement UNION ALL select_statement UNION ALL select_statement ...
```
```
分析窗口函数
over (partition by * order by d desc)
```
例子：[Hive分析窗口函数(一) SUM,AVG,MIN,MAX](//www.tuicool.com/articles/zuiIVbZ)
```
其他
1.不支持EXIST ,NOT EXIST

2.Top N 关键字 为Limit

3.支持正则筛选字段，如SELECT `(ds|hr)?+.+` FROM test
```


## DCL
权限

```
Grant/revoke语法：
grant/revoke priv_type[column_list] on object_type object to/from principal_type principal_name

查看grant 定义：
show grant user user_name on table table_name;


创建/删除角色：
Create/drop Role role_name

角色分配/回收：
Grant role role_name to user user_name
Revoke role role_name from user user_name

角色授权：
Grant/revoke priv_type[col_List] on object_type object from/to role role_name

查看role定义：
show role grant role role_name

相关hive元信息表：
Db_privs:记录了User/Role在DB上的权限
Tbl_privs:记录了User/Role在table上的权限
Tbl_col_privs：记录了User/Role在table column上的权限
Roles：记录了全部创建的role
Role_map：记录了User与Role的相应关系
```

## Hive函数
UDF、UDAF和UDTF

UDF实现输入一行输出一行行的函数

UDAF实现输入多行输出一行的函数

UDTF实现输入一行输出多行的函数

自定义UDF、UDAF和UDTF

## 相关资料
hive2.0函数大全
http://www.cnblogs.com/MOBIN/p/5618747.html

## Mysql数据导入Hive

通过sqoop把mysql数据导入到hive(若hive没对应表则自动建表)
```
./bin/sqoop import --connect jdbc:mysql://172.17.4.1:3306/poi --username lizt  --table poi_amap --hive-import --password 123456
```
## python 客户端

[pyhs2](https://github.com/BradRuderman/pyhs2)
```
pip install pyhs2
```
