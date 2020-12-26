---
title: Hive
---

1. 表的数据加载

   1. insert
   2. load

2. 创建分区表（外表）

3. 数据导出

   1. hdfs dfs -get filename

   2. hdfs dfs -text filename    //查看

      1. insert overwrite 【local】 directory 'filename'

         【row format delimited fields terminated by '\t'】

         select col， col1 from tablename

         //local 与 row。。 只能在导出到本地时使用

   3. Shell 命令加管道：hive -f/e | sed/awk/gred >filename

   4. sqoop

   5. 动态分区

      1. 设置 set hive.exec.dynamic.partition=true //设置使用动态分区
      2. 设置 set hive.exec.dynamic.partition.mode=nonstrict //使用无限制模式
      3. insert into table tablename partition(dt) select col,col1 as dt from tablename  //一个分区 动态设置
      4. Insert into table tablename partition(dt='2017',value) select col,col1 as value from tablename //两个分区 ，静态分区必须在设置的动态分区前面

   6. 修改表

      1. 重命名表 alter table tablename rename to newtablename;

      2. 修改列名 alter table tablename change column c1 c2 int comment 'xxx';

      3. 移动列的位置 alter table  tablename change column c1 c2 after c3;

      4. 增加列 alter table tablename add columns(c1 string comment 'xxx',c2 int);

      5. 删除列 alter table tablename replace columns(col string,col1 int) 

         //col col1 都是保留下来的列

      6. 修改分割符 alter table tablename【partition(dt='xxxx')】 set serdeproperties('field.delim'='\t')

      7. 修改location;

          alter table tablename 【partition()】set location 'path'

      8. 内部表改外部表 

         alter table tablename set tblproperties('EXTERNAL'='TRUE')  

         //外部表改内部表 EXTERNAL=FALSE；

   7. group by：

      select col1,col2 from tablename group by col1,col2

      //查询的列 col1，col2 必须出现在group by后面；

   8. sum(col)  //col可为int，double 等数字类型也可以为string类型

   9. 在hive中使用python脚本

      add file filenamepath //先将脚本缓存到hive集群上

      select * from ( select transform (col,col1) using 'filename' 【as coll ,coll2】from tablename  ) tablename

   10. 【inner】join，left 【outer】 join，right 【outer】 join, full 【outer】 join，

     /* +mapjoin(tablename) */,left semi join 相当于in 

   11. 函数

       1. nvl(col,0) //如果col非空则显示col否则显示0
       2. 【nvl2(col1,col2,col3) //如果col1为空则显示col2否则显示col3】
       3. 【nullif(col1,col2) //如果col1与col2相等则返回null否则返回col1】
       4. coalesce(col1,col2,col3….) //返回第一个非空（null）值，如果都为空则返回null
       5. concat(col1,col2….) //字符串拼接，如果col1或col2中有一个为null则返回null
       6. concat_ws(',',col1,col2…..) //带有分隔符的字符串拼接，如果有col2为null则不会拼接，拼接的内容可以为array
       7. cast(1 as bigint) //转化1为bigint
       8. round(0.0089,4) //保留4位小数
       9. if (condition,a,b) //若condition为真则返回a，否则返回b；
       10. explode(array(or map)) //将输入的一行数组或map转换成列输出
       11. split(str,',') //第一个参数是要分隔的字符串，第二个参数是分隔符，分隔后的数据可看成数组

   12. distribute by 打散数据 长和sort by（使每一个reduce里面的数据都有序）连用。distribute by 与group by 都是安key对数据进行划分，都使用reduce操作，但是distribute by只是将数据分散，而group by 是把相同的key聚集到一起然后进行聚合操作。sort by与order by，order by 是全局排序，只会有一个reduce ，sort by是确保每一个reduce 上的数据都是有序的，如果只有一个reduce时，sort by 和order by作用是一样的。

   13. cluster by

   14. unoin all

   15. 自定义UDF：

       重写evaluate函数

   16. 优化：

       1. join优化

          hive.optimize.skewjoin=true

          hive.skewjoin.key=n //当key的数量到达n时会主动进行优化

       2. mapjoin

          hive.atuo.conwert.join=true

          hive.mapjoin.smalltable.filesize=n

          select /*+mapjoin(a) */,col1,col2 form a join b

          注：a表要非常小，mapjoin因为是在map端操作可以进行不等值操作

       3. group by 

          hive.groupby.skewindata=true

          Hive.groupby.mapaggr.checkinterval=n

       4. job

          hive.exec.parallel=true. //设置job的并行化

          hive.exec.parallel.thread.numbe=n //设置最大线程数

          hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat //合并文件，文件受mapred.max.split.size大小的限制

          Hive.merge.smallfiles.avgsize=n  //设置输出文件平均大小小于改值，启动job合并文件

          Hive.merge.size.per.task=n //合并之后文件的大小

       5. 

       6. 

          

   17. yarn queue -status mt1

   18. mapred【hadoop】 queue -list

   19. 

   20. 

       

   21. 命令：

       1. mapred[hadoop] job -list | grep o2o4 //查job
       2. ps -ef| grep xxxx.sh | grep -v grep // 查进程
       3. mapred[hadoop] queue -list // 查队列（所有）
       4. yarn queue -status o2o4  //查队列（单个）
       5. du -h //查文件大小
       6. sed -n 's/old/new/gp' filename //-n  代表行   g 全局 p 打印  
       7. 使用”;”时，不管command1是否执行成功都会执行command2； 使用”&&”时，只有command1执行成功后，command2才会执行，否则command2不执行；使用”||”时，command1执行成功后command2 不执行，否则去执行command2，总之command1和command2总有一条命令会执行。
       8. 


* hive 分区下有数据但是查询表没有数据

  可以是表的读入设置了压缩等其他格式，单纯txt的文件，表无法读取

  STORED AS INPUTFORMAT 
    'com.hadoop.mapred.DeprecatedLzoTextInputFormat'   //设置压缩为lzo 所以只能读取lzo数据

* Hive count(distinct ) 优化

  可以先进行group by  然后再进行count

  ```sql
  select count(*),count(distinct col) from tablename where day_id=20180708
  ```


  select max(cnt), max(mdn_cnt), max(day_id) from (select null  as cnt , count(*) as mdn_cnt , day_id  as day_id from (select mdn, day_id from dwi_m.dwi_res_regn_mergelocation_msk_d where day_id=20180708 group by mdn,day_id )a group by  day_id union all select count(*) as cnt, null  as mdn_cnt , day_id as day_id from dwi_m.dwi_res_regn_mergelocation_msk_d where day_id=20180708 group by day_id )b;//union all


  select b.cnt,a.mdn_cnt ,a.day_id from (
  select count(*) as mdn_cnt ,day_id from (select mdn,day_id from dwi_m.dwi_res_regn_mergelocation_msk_d where day_id=20180708 group by mdn,day_id)a1 group by day_id)a
  left join 
  (select count(*) as cnt, day_id from dwi_m.dwi_res_regn_mergelocation_msk_d  b where day_id=20180708 group by day_id)b
  on a.day_id=b.day_id;//join


  join 和 union all 用的时间差不多  但相对 直接count distinct 要 少很多