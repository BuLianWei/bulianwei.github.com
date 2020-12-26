---
title: "Hive"
date: 2020-08-19T20:37:43+08:00
draft: true
---

# Hive
Hive 不存储数据其实就是将hdfs存储系统上的文件映射成具有结构的表
## 数据导入
- 使用load命令
	```sql
	LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1,partcol2=val2 ...)]
	
	load data inpath '/data/test.txt' overwrite into table tablename partition (day_id='20200801') //将hdfs上的data目录下的test.txt文件用覆盖的方式导入到tablename表的day_id为20200801的分区下
	
	load data local inpath '/root/test.txt' overwrite into table tablename partition (day_id='20200801') //将本地root目录下的test.txt文件用覆盖的方式导入到tablename表的day_id为20200801的分区下
	```
- 使用hdfs dfs -put（hadoop fs -put）
	```shell
	hdfs dfs -mkdir /data
	hdfs dfs -put test.txt /data/
	msck repair table tablename
	```
- 使用insert
	```shell
	insert into table tablename 
	select * from tablename1
	```
## 数据导出
```sql
insert overwrite [local] directory 'filename'
[row format delimited fields terminated by '\t']
select col， col1 from tablename

//local 与 row。。 只能在导出到本地时使用

insert overwrite local directory '/root/test.txt'
row format delimited fields terminated by '\t'
select col,col1 form tablename	

```

### 多维计算
- grouping sets
- cube
- rollup

## 优化
- Join优化
	```sql
	hive.optimize.skewjoin=true
	hive.skewjoin.key=n //当key的数量到达n时会主动进行优化
	```
- MapJoin优化
	```sql
	hive.atuo.conwert.join=true
	hive.mapjoin.smalltable.filesize=n
	select /*+mapjoin(a) */,col1,col2 form a join b
	注：a表要非常小，mapjoin因为是在map端操作可以进行不等值操作
	```
- Group By优化
	```sql
	hive.groupby.skewindata=true
	hive.groupby.mapaggr.checkinterval=n
	```
- Job优化
	```sql
	hive.exec.parallel=true. //设置job的并行化
	hive.exec.parallel.thread.numbe=n //设置最大线程数
	hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat //合并文件，文件受mapred.max.split.size大小的限制
	hive.merge.smallfiles.avgsize=n  //设置输出文件平均大小小于改值，启动job合并文件
	hive.merge.size.per.task=n //合并之后文件的大小
	```
	
- 场景一
	union 去重
	union all 不去重
- 场景二
	union all 减少job个数
	```sql
	select count(1) from a;
	select count(2) from b;
	select count(3) from c;
	------------------------------------
	select type,count(1) from (
	select 'a' as type ,id from a
	union all 
	select 'b' as type ,id from b
	union all
	select 'c' as type ,id from c)tmp
	group by type;
	```
	多表同join条件，减少job数
	a left join b on a.id=b.id
	left join c on a.id=c.id
- 场景三
	开启任务并行
	`set hive.exec.parallel=true`
	允许最大并行线程数
	`set hive.exec.parallel.thread.number=8`
	union all
	join
- 场景四
	任务控制
	- map
    每个map最大输入大小
    `set mapred.max.split.size=256000000`
    一个节点上的split的至少大小
    `set mapred.min.split.size.per.node=100000000`
    一个交换机下split的至少大小
    `set mapred.min.split.size.per.rack=100000000`
    执行map前进行小文件合并
    `set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat`
    
	- reduce
		每个reduce处理的数据量
		`set hive.exec.reducers.bytes.per.reducer=500000000`
		制定reduce数量
		`set mapred.reduce.tasks=20`
		reduce输出文件数
		在maping的任务结束时合并小文件（map-join）
		`set hive.merge.mapfiles=true`
		在map-reduce的任务结束时合并小文件
		`set hive.merge.mapredfiles=true`
		合并文件的大小
		`set hive.merge.size.per.task=256*1000*1000`
		输出文件的平均大小小于该值时，启动一个独立的map-reduce任务进行文件合并
		`set hive.merge.smallfiles.avgsize=16000000`
	
- 场景五
	排序
	order by
	sort by
- 场景六
	map多承担任务，减少数据传输成本和reducer，计算成本
	map join 
	map aggr
- 场景七
	数据倾斜
		空值
		数据类型
		业务数据本身
- 场景八
	数据裁剪
		记录数裁剪
			分区，分桶
		列裁剪
			剔除无效，非计算范围内的列数据
			列式存储（直接跳过无效列）
```sql
select * from a
left join b
on a.id=b.id and a.time >n
-----------------
(select id ,time from a where a.itme >n)
left join 
(select id ,time from b where b.time >n)
```


## SQL
### 连续天数计算

先将数据去重，保留每个id一天只有一天数据，然后使用row_number函数根据id分组后进行分区 根据day_id进行排序 将数据做标记，然后使用

```sql
select
id,day_time,date_sub(date_time,rank) as date_group,count(1) as continuous_days
from (
	select 
	id,date_time,row_number() over(partition by id order by date_time) as rank
	from table_name
)tmp
group by id,date_sub(date_time,rank)
```

