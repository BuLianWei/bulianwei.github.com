---
title: "Flink"
date: 2020-08-19T20:50:34+08:00
draft: fasle
---

# Flink

## 部署
- On Yarn
	- yarn-session
		开启session
		yarn-session.sh -jm 1024m -tm 4096m
		在session上提交作业
		flink run -m yarn-cluster -p 4 -yjm 1024m -ytm 4096m ./examples/batch/WordCount.jar
		关闭session
		echo "stop" | ./bin/yarn-session.sh -id <appId>
		
		> -D 使用-D将要设置的参数进行设置（-Dtaskmanager.memory.network.min=536346624）
		> -d,--detached 		开启分离模式（session启动后client自动关掉）
		> -jm,--jobManagerMemory <arg>    jobmanager 内存大小(默认: MB)
		> -nm,--name                      设置应用程序的名称
		> -at,--applicationType           设置应用程序的类型
		> -q,--query                      查询使用的Yarn的资源（Memory，Core）
		> -qu,--queue <arg>               设置在Yarn上使用的队列.
		> -s,--slots <arg>                设置每个TaskManager的Slot数量
		> -tm,--taskManagerMemory <arg>   设置每个TaskManager的内存大小(默认: MB)
		> -z,--zookeeperNamespace <arg> 
	- pro-job
	flink run -m yarn-cluster ./examples/batch/WordCount.jar
	
		> run
		> -c,--class 		程序全路径名称
		> -C,--classpath <url>
		> -n,--allowNonRestoredState		忽略保存点状态不能被restored的错误
		> -p,--parallelism <parallelism>	程序并行度(会覆盖进群的配置，集群配置配置 < 程序里配置文件配置 < 运行时指定 ）
		> -s,--fromSavepoint <savepointPath>	设置应用程序保存点文件路径
		> ------------for yarn---------------------------
		> -d,--detached 
		> -m,--jobmanager <arg>	 设置jobmanger地址
		> -yat,--yarnapplicationType <arg>		设置应用程序的类型
		> -yD <property=value>						设置配置参数
		> -yid,--yarnapplicationId <arg>			将应用程序提交到yarn-session上运行
		> -yj,--yarnjar <arg>								flink jar 文件地址
		> -yjm,--yarnjobManagerMemory <arg>	设置yarn模式下jobmanager 内存大小（默认：MB）
		> -ynl,--yarnnodeLabel <arg>					设置node标签
		> -ynm,--yarnname <arg>							设置应用程序的名称
		> -yq,--yarnquery										查询在yarn上的资源（memory，core）
		> -yqu,--yarnqueue <arg>							设置应用程序队列
		> -ys,--yarnslots <arg> 								设置每个taskmanager的slot数量
		> -yt,--yarnship <arg>
		> -ytm,--yarntaskManagerMemory <arg>		设置每个taskmanager 的内存大小（默认：MB）

	- application
	
## 查看集群作业
- flink list	//列出计划和正在运行的作业
- flink list -s	//列出计划的作业
- flink list -r	//列出正在执行的作业
- flink list -a	//列出所有的作业
## 算子
### Map
数据流一对一的转换，作用在每一条数据上，对流经算子的每一条数据进行相应的操作，输入输出类型可变。
```java
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        DataSource<String> elements = env.fromElements("hello","flink");
        MapOperator<String, String> map = elements.map(new MapFunction<String, String>() {
            @Override
            public String map(String value) throws Exception {
                return "ods-" + value;
            }
        });

        map.print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
ods-hello
ods-flink
```
### FlatMap
数据流一对多的转换，作用在每一条数据上，将流经算子的数据拆分成相应的数据进行输出，输入输出数据类型可变。
```java
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        DataSource<String> elements = env.fromElements("hello flink");
        FlatMapOperator<String, String> flatMap = elements.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
                for (String s : value.split(" ")) {
                    out.collect("ods-"+s);
                }
            }
        });

        flatMap.print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
ods-hello
ods-flink
```
### MapPartition
数据流多对一的转换，作用在每一个分区上，将分区内的多条数据通过迭代操作然后输出结果，输入输出数据类型可变。
```java
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        DataSource<String> elements = env.fromElements("hello","flink");
        MapPartitionOperator<String, String> mapPartition = elements.mapPartition(new MapPartitionFunction<String, String>() {
            @Override
            public void mapPartition(Iterable<String> values, Collector<String> out) throws Exception {
                values.forEach(new Consumer<String>() {
                    @Override
                    public void accept(String s) {
                        out.collect("ods-" + s);
                    }
                });
            }
        });
        mapPartition.print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
ods-hello
ods-flink
```
### Filter
数据流一对一的转换，作用在每一条数据上，根据操作函数返回的true和false来确定是否让数据通过进入下一算子
```java
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        DataSource<String> elements = env.fromElements("hello","flink");
        FilterOperator<String> filter = elements.filter(new FilterFunction<String>() {
            @Override
            public boolean filter(String value) throws Exception {
                return value.startsWith("h");
            }
        });
        filter.print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
hello
```
### Project
数据流一对一的转换，作用在每一条数据上，将带有元组的数据流根据序号选取，只针对java api调用
```java
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer,String,Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer,String,Boolean>(1,"a",true);
        list.add(tuple3);
        DataSet<Tuple3<Integer,String,Boolean>> source = env.fromCollection(list);
        DataSet<Tuple2<Boolean,Integer>> project = source.project(2,0);
        project.print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
(true,1)
```
### Reduce
- 不基于groupBy
  ```java
          ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
          ArrayList<Tuple3<Integer,String,Boolean>> list = new ArrayList<>();
          Tuple3 tuple3 = new Tuple3<Integer,String,Boolean>(1,"a",true);
          list.add(tuple3);
           tuple3 = new Tuple3<Integer,String,Boolean>(2,"a",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(3,"b",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(4,"b",true);
          list.add(tuple3);
          DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
          ReduceOperator<Tuple3<Integer, String, Boolean>> reduce = source.reduce(new ReduceFunction<Tuple3<Integer, String, Boolean>>() {
              @Override
              public Tuple3<Integer, String, Boolean> reduce(Tuple3<Integer, String, Boolean> value1, Tuple3<Integer, String, Boolean> value2) throws Exception {
                  return new Tuple3<>(value1.f0 + value2.f0, value1.f1, false);
              }
          });
          reduce.print();
  ------------------------------------------------------------------------------------------------------------------------------
  结果：
  (10,a,false)
  ```
- 基于groupBy
  ```java
          ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
          ArrayList<Tuple3<Integer,String,Boolean>> list = new ArrayList<>();
          Tuple3 tuple3 = new Tuple3<Integer,String,Boolean>(1,"a",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(2,"a",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(3,"b",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(4,"b",true);
          list.add(tuple3);
          DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
          UnsortedGrouping<Tuple3<Integer, String, Boolean>> groupBy = source.groupBy(1);
          ReduceOperator<Tuple3<Integer, String, Boolean>> reduce = groupBy.reduce(new ReduceFunction<Tuple3<Integer, String, Boolean>>() {
              @Override
              public Tuple3<Integer, String, Boolean> reduce(Tuple3<Integer, String, Boolean> value1, Tuple3<Integer, String, Boolean> value2) throws Exception {
                  return new Tuple3<>(value1.f0+value2.f0,value1.f1,false);
              }
          });
          reduce.print();
  ------------------------------------------------------------------------------------------------------------------------------
  结果：
  (3,a,false)
  (7,b,false)
  ```
### GroupBy
- 不基于groupBy
  ```java
          ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
          ArrayList<Tuple3<Integer,String,Boolean>> list = new ArrayList<>();
          Tuple3 tuple3 = new Tuple3<Integer,String,Boolean>(1,"a",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(2,"a",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(3,"b",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(4,"b",true);
          list.add(tuple3);
          DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
          UnsortedGrouping<Tuple3<Integer, String, Boolean>> groupBy = source.groupBy(new KeySelector<Tuple3<Integer, String, Boolean>, String>() {
              @Override
              public String getKey(Tuple3<Integer, String, Boolean> value) throws Exception {
                  return value.f1;
              }
          });
          ReduceOperator<Tuple3<Integer, String, Boolean>> reduce = groupBy.reduce(new ReduceFunction<Tuple3<Integer, String, Boolean>>() {
              @Override
              public Tuple3<Integer, String, Boolean> reduce(Tuple3<Integer, String, Boolean> value1, Tuple3<Integer, String, Boolean> value2) throws Exception {
                  return new Tuple3<>(value1.f0+value2.f0,value1.f1,false);
              }
          });
          reduce.print();
  ------------------------------------------------------------------------------------------------------------------------------
  结果：
  (3,a,false)
  (7,b,false)
  ```
- 基于groupBy
  ```java
          ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
          ArrayList<Tuple3<Integer,String,Boolean>> list = new ArrayList<>();
          Tuple3 tuple3 = new Tuple3<Integer,String,Boolean>(1,"a",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(2,"a",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(3,"b",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(4,"b",true);
          list.add(tuple3);
          DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
          UnsortedGrouping<Tuple3<Integer, String, Boolean>> groupBy = source.groupBy(1);
          ReduceOperator<Tuple3<Integer, String, Boolean>> reduce = groupBy.reduce(new ReduceFunction<Tuple3<Integer, String, Boolean>>() {
              @Override
              public Tuple3<Integer, String, Boolean> reduce(Tuple3<Integer, String, Boolean> value1, Tuple3<Integer, String, Boolean> value2) throws Exception {
                  return new Tuple3<>(value1.f0+value2.f0,value1.f1,false);
              }
          });
          reduce.print();
  ------------------------------------------------------------------------------------------------------------------------------
  结果：
  (3,a,false)
  (7,b,false)
  ```
### ReduceGroup
与reduce的区别在于，reducegroup 可以获取一个组内所有的数据，可以单用也可以和groupby进行组合使用，输入输出类型可变
- 不基于groupBy
  ```java
          ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
          ArrayList<Tuple3<Integer,String,Boolean>> list = new ArrayList<>();
          Tuple3 tuple3 = new Tuple3<Integer,String,Boolean>(1,"a",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(2,"a",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(3,"a",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(4,"b",true);
          list.add(tuple3);
         tuple3 = new Tuple3<Integer,String,Boolean>(5,"b",true);
          list.add(tuple3);
         tuple3 = new Tuple3<Integer,String,Boolean>(6,"b",true);
          list.add(tuple3);
          DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
          GroupReduceOperator<Tuple3<Integer, String, Boolean>, Tuple3<Integer, String, Boolean>> reduceGroup = source.reduceGroup(new
                                                                                                                                      RichGroupReduceFunction<Tuple3<Integer, String, Boolean>, Tuple3<Integer, String, Boolean>>() {
              @Override
              public void reduce(Iterable<Tuple3<Integer, String, Boolean>> values, Collector<Tuple3<Integer, String, Boolean>> out) throws Exception {
                  final int[] tmp = {0};
                  final String[] key = {""};
                  values.forEach(new Consumer<Tuple3<Integer, String, Boolean>>() {
                      @Override
                      public void accept(Tuple3<Integer, String, Boolean> in) {
                          if (in.f0%2==0){
                              tmp[0] +=in.f0;
                              key[0] =in.f1;
                          }

                      }
                  });
                  out.collect(new Tuple3<>(tmp[0], key[0],false));
              }
          });
          reduceGroup.print();
  ------------------------------------------------------------------------------------------------------------------------------
  结果：
  (12,b,false)
  ```
- 基于groupBy
  ```java
          ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
          ArrayList<Tuple3<Integer,String,Boolean>> list = new ArrayList<>();
          Tuple3 tuple3 = new Tuple3<Integer,String,Boolean>(1,"a",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(2,"a",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(3,"a",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(4,"b",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(5,"b",true);
          list.add(tuple3);
          tuple3 = new Tuple3<Integer,String,Boolean>(6,"b",true);
          list.add(tuple3);
          DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
          UnsortedGrouping<Tuple3<Integer, String, Boolean>> groupBy = source.groupBy(1);
          GroupReduceOperator<Tuple3<Integer, String, Boolean>, Tuple3<Integer, String, Boolean>> reduceGroup = groupBy.reduceGroup(new
                                                                                                                                      RichGroupReduceFunction<Tuple3<Integer, String, Boolean>, Tuple3<Integer, String, Boolean>>() {
              @Override
              public void reduce(Iterable<Tuple3<Integer, String, Boolean>> values, Collector<Tuple3<Integer, String, Boolean>> out) throws Exception {
                  final int[] tmp = {0};
                  final String[] key = {""};
                  values.forEach(new Consumer<Tuple3<Integer, String, Boolean>>() {
                      @Override
                      public void accept(Tuple3<Integer, String, Boolean> in) {
                          if (in.f0%2==0){
                              tmp[0] +=in.f0;
                              key[0] =in.f1;
                          }

                      }
                  });
                  out.collect(new Tuple3<>(tmp[0], key[0],false));
              }
          });
          reduceGroup.print();
  ------------------------------------------------------------------------------------------------------------------------------
  结果：
  (2,a,false)
  (10,b,false)
  ```
### SortGroup
组内排序，前提在使用分组聚合以后（groupBy，keyBy）以后，对组内数据进行指定顺序排序操作
```java
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer,String,Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer,String,Boolean>(1,"a",true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer,String,Boolean>(2,"a",true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer,String,Boolean>(3,"a",true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer,String,Boolean>(4,"b",true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer,String,Boolean>(5,"b",true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer,String,Boolean>(6,"b",true);
        list.add(tuple3);
        DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
        UnsortedGrouping<Tuple3<Integer, String, Boolean>> groupBy = source.groupBy(1);
        SortedGrouping<Tuple3<Integer, String, Boolean>> sortGroup = groupBy.sortGroup(0, Order.ASCENDING);
        sortGroup.first(2).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
(1,a,true)
(2,a,true)
(4,b,true)
(5,b,true)
```
### GroupCombine
对数据流进行一次预聚合运算，可以减少数据的传输，该运算通常只有本分数据被聚合不能代替reduce操作，一般都是配合reduce或者reduceGroup进行操作，输入输出类型可变
```java
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        DataSource<String> elements = env.fromElements("hello", "flink");
        MapOperator<String, Tuple2<String, Integer>> map = elements.map(word -> new Tuple2<String, Integer>(word, 1)).returns(Types.TUPLE(Types.STRING,
                Types.INT));
        UnsortedGrouping<Tuple2<String, Integer>> groupBy = map.groupBy(0);
        GroupCombineOperator<Tuple2<String, Integer>, Tuple2<String, Integer>> combineGroup = groupBy.combineGroup((Iterable<Tuple2<String, Integer>> values, Collector<Tuple2<String, Integer>> out) -> {
            final String[] key = new String[1];
            final int[] cnt = {0};
            values.forEach(x -> {
                key[0] = x.f0;
                cnt[0]++;
            });
            out.collect(new Tuple2<String, Integer>(key[0], cnt[0]));
        }).returns(Types.TUPLE(Types.STRING, Types.INT));
        ;
        combineGroup.print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
(flink,1)
(hello,1)
```
### Aggregate
将数据流根据（SUM，MIN，MAX）将某个字段聚合操作，有and运算符时，如果对同一个字段进行操作，只会进行最后操作符（MAX，MIN，SUM）的操作
```java
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(1, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(2, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(6, "b", false);
        list.add(tuple3);

        DataSet<Tuple3<Integer,String,Boolean>> source = env.fromCollection(list);
        source.aggregate(SUM,0).print();
        source.aggregate(MAX,0).print();
        source.aggregate(MIN,0).print();
        System.out.println("============");
        source.aggregate(SUM,0).and(MIN,0).and(MAX,0).print();
        source.aggregate(MIN,0).and(MAX,0).print();
        source.aggregate(MIN,0).andMax(0).print();
        System.out.println("============");
        source.aggregate(MAX,0).and(MIN,0).print();
        System.out.println("============");
        source.aggregate(MAX,0).and(MIN,1).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
(21,b,false)
(6,b,false)
(1,b,false)
============
(6,b,false)
(6,b,false)
(6,b,false)
============
(1,b,false)	//根据第一个字段取最小的，其他字段取最后一条
============
(6,a,false)	//根据第一个字段取最大的，根据第二个字段取最小的，其他的取最后一条
```
### MinBy 
根据某几个字段求数据里面最小的一条，相当于order by col1,col2 asc 
```java
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(2, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        DataSet<Tuple3<Integer,String,Boolean>> source = env.fromCollection(list);
        source.minBy(0,0).print();
        source.minBy(0,1).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
(0,a,false)	//只根据第一个字段取最小的那条记录
(0,a,false)	//根据第一个字段和第二个字段取最小的那条记录
```
### MaxBy
根据某几个字段求数据里面最大的一条，相当于order by col1,col2 desc
```java
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(6, "a", false);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(2, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(6, "b", false);
        list.add(tuple3);

        DataSet<Tuple3<Integer,String,Boolean>> source = env.fromCollection(list);
        source.maxBy(0,0).print();
        source.maxBy(0,1).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
(6,a,false)	//只根据第一个字段取最大的那条记录
(6,b,false)	//根据第一个和第二个字段取最大的那条记录

```
### Distinct
distinct 可以根据字段序号，keyselect或者对象的属性名字直接进行去重
```java
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(2, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
        source.distinct(0).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
(3,a,true)
(0,a,false)
(5,b,true)
(2,a,true)
(4,b,true)
```
### Join
- join
	```java
	      ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(2, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
        source.join(source).where(0).equalTo(0).print();
	-----------------------------------------------------------------------------------------------------------------------------
	结果：
	((3,a,true),(3,a,true))
	((0,a,false),(0,a,false))
  ((0,b,true),(0,a,false))
  ((5,b,true),(5,b,true))
  ((0,a,false),(0,b,true))
  ((0,b,true),(0,b,true))
  ((2,a,true),(2,a,true))
  ((4,b,true),(4,b,true))
	```
	
	input1.join(input2, JoinHint.BROADCAST_HASH_FIRST)
	- JoinHint.OPTIMIZER_CHOOSES
		任系统自己选择
	- JoinHint.BROADCAST_HASH_FIRST
		对第一个表进行hash，广播第一个表，小表join大表
	- JoinHint.BROADCAST_HASH_SECOND
		对第二个表进行hash，广播第二个表，大表join小表
	- JoinHint.REPARTITION_HASH_FIRST
		对每个表都进行分区，并且对第一个表进行hash，小表join大表，但是两个表任然很大
		如果无法进行大小估计并且无法重新使用现有的分区和排序顺序，这是系统使用的默认后备策略
	- JoinHint.REPARTITION_HASH_SECOND
		对每个表都进行分区，并且对第二个表进行hash，大表join小表，但是两个表任然很大
	- JoinHint.REPARTITION_SORT_MERGE
		对每个表都进行分区，对每个表都进行排序，对有排序的表性能会提高

- joinWithTiny
	JoinHint.BROADCAST_HASH_FIRST
	
	```java
	-----------------------------------------------------------------------------------------------------------------------------
	结果：
	```
- joinWithHuge
	JoinHint.BROADCAST_HASH_SECOND
	```java
	-----------------------------------------------------------------------------------------------------------------------------
	结果：
	```


### OuterJoin
- LeftOuterJoin
	```java
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(2, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        ArrayList<Tuple3<Integer, String, Boolean>> list1 = new ArrayList<>();
        Tuple3 tuple4 = new Tuple3<Integer, String, Boolean>(0, "a1", false);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(2, "a1", true);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(3, "a1", true);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(4, "b1", true);
        list1.add(tuple4);

        DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
        DataSource<Tuple3<Integer, String, Boolean>> source1 = env.fromCollection(list1);
    
        JoinFunctionAssigner<Tuple3<Integer, String, Boolean>, Tuple3<Integer, String, Boolean>> equalTo = source.leftOuterJoin(source1).where(0)
                .equalTo(0);
        equalTo.with(new JoinFunction<Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>>() {
            @Override
            public Tuple3<Integer, String, Boolean> join(Tuple3<Integer, String, Boolean> first, Tuple3<Integer, String, Boolean> second) throws Exception {
                return second ==null ? first: new Tuple3<Integer, String, Boolean>(first.f0+second.f0,first.f1+"-"+second.f1,first.f2);
            }
        }).print();
    
        System.out.println("===============");
        equalTo.with(new FlatJoinFunction<Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>>() {
            @Override
            public void join(Tuple3<Integer, String, Boolean> first, Tuple3<Integer, String, Boolean> second, Collector<Tuple3<Integer, String, Boolean>> out) throws Exception {
                out.collect(second ==null ? first: new Tuple3<Integer, String, Boolean>(first.f0+second.f0,first.f1+"-"+second.f1,first.f2));
            }
        }).print();
    -----------------------------------------------------------------------------------------------------------------------------
    结果：
    (6,a-a1,true)
    (0,a-a1,false)
    (5,b,true)
    (0,b-a1,true)
    (4,a-a1,true)
    (8,b-b1,true)
    ===============
    (6,a-a1,true)
    (0,a-a1,false)
    (5,b,true)
    (0,b-a1,true)
    (4,a-a1,true)
    (8,b-b1,true)
	```
	- OPTIMIZER_CHOOSES
	- BROADCAST_HASH_SECOND
	- REPARTITION_HASH_SECOND
	- REPARTITION_SORT_MERGE

- RightOuterJoin
	```java
	      ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(2, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        ArrayList<Tuple3<Integer, String, Boolean>> list1 = new ArrayList<>();
        Tuple3 tuple4 = new Tuple3<Integer, String, Boolean>(0, "a1", false);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(2, "a1", true);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(3, "a1", true);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(4, "b1", true);
        list1.add(tuple4);

        DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
        DataSource<Tuple3<Integer, String, Boolean>> source1 = env.fromCollection(list1);
    
        JoinFunctionAssigner<Tuple3<Integer, String, Boolean>, Tuple3<Integer, String, Boolean>> equalTo = source.rightOuterJoin(source1).where(0)
                .equalTo(0);
        equalTo.with(new JoinFunction<Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>>() {
            @Override
            public Tuple3<Integer, String, Boolean> join(Tuple3<Integer, String, Boolean> first, Tuple3<Integer, String, Boolean> second) throws Exception {
                return second ==null ? first: new Tuple3<Integer, String, Boolean>(first.f0+second.f0,first.f1+"-"+second.f1,first.f2);
            }
        }).print();
    
        System.out.println("===============");
        equalTo.with(new FlatJoinFunction<Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>>() {
            @Override
            public void join(Tuple3<Integer, String, Boolean> first, Tuple3<Integer, String, Boolean> second, Collector<Tuple3<Integer, String, Boolean>> out) throws Exception {
                out.collect(second ==null ? first: new Tuple3<Integer, String, Boolean>(first.f0+second.f0,first.f1+"-"+second.f1,first.f2));
            }
        }).print();
    -----------------------------------------------------------------------------------------------------------------------------
    结果：
  (6,a-a1,true)
  (0,a-a1,false)
  (0,b-a1,true)
  (4,a-a1,true)
  (8,b-b1,true)
  ===============
  (6,a-a1,true)
  (0,a-a1,false)
  (0,b-a1,true)
  (4,a-a1,true)
  (8,b-b1,true)
	```
	- OPTIMIZER_CHOOSES
	- BROADCAST_HASH_FIRST
	- REPARTITION_HASH_FIRST
	- REPARTITION_SORT_MERGE

- FullOuterJoin
	```java
	      ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        ArrayList<Tuple3<Integer, String, Boolean>> list1 = new ArrayList<>();
        Tuple3 tuple4 = new Tuple3<Integer, String, Boolean>(0, "a1", false);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(2, "a1", true);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(3, "a1", true);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(4, "b1", true);
        list1.add(tuple4);

        DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
        DataSource<Tuple3<Integer, String, Boolean>> source1 = env.fromCollection(list1);
    
        JoinFunctionAssigner<Tuple3<Integer, String, Boolean>, Tuple3<Integer, String, Boolean>> equalTo = source.fullOuterJoin(source1).where(0)
                .equalTo(0);
        equalTo.with(new JoinFunction<Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>>() {
            @Override
            public Tuple3<Integer, String, Boolean> join(Tuple3<Integer, String, Boolean> first, Tuple3<Integer, String, Boolean> second) throws Exception {
                return first ==null ? second:second==null ? first: new Tuple3<Integer, String, Boolean>(first.f0+second.f0,first.f1+"-"+second
                        .f1,first
                        .f2);
            }
        }).print();
    
        System.out.println("===============");
        equalTo.with(new FlatJoinFunction<Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>>() {
            @Override
            public void join(Tuple3<Integer, String, Boolean> first, Tuple3<Integer, String, Boolean> second, Collector<Tuple3<Integer, String, Boolean>> out) throws Exception {
                out.collect(first ==null ? second:second==null ? first: new Tuple3<Integer, String, Boolean>(first.f0+second.f0,first.f1+"-"+second.f1,first.f2));
            }
        }).print();
    -----------------------------------------------------------------------------------------------------------------------------
    结果：
  (6,a-a1,true)
  (0,a-a1,false)
  (0,b-a1,true)
  (5,b,true)
  (2,a1,true)
  (8,b-b1,true)
  ===============
  (6,a-a1,true)
  (0,a-a1,false)
  (0,b-a1,true)
  (5,b,true)
  (2,a1,true)
  (8,b-b1,true)
	```
	- OPTIMIZER_CHOOSES
	- REPARTITION_SORT_MERGE

### Cross
笛卡尔积
- cross
	```java
	      ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        ArrayList<Tuple3<Integer, String, Boolean>> list1 = new ArrayList<>();
        Tuple3 tuple4 = new Tuple3<Integer, String, Boolean>(0, "a1", false);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(2, "a1", true);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(3, "a1", true);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(4, "b1", true);
        list1.add(tuple4);

        DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
        DataSource<Tuple3<Integer, String, Boolean>> source1 = env.fromCollection(list1);
    
        source.cross(source1).with(new CrossFunction<Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>>() {
            @Override
            public Tuple3<Integer, String, Boolean> cross(Tuple3<Integer, String, Boolean> val1, Tuple3<Integer, String, Boolean> val2) throws Exception {
                return new Tuple3<>(val1.f0+val2.f0,val1.f1+"-"+val2.f1,val1.f2);
            }
        }).print();
    -----------------------------------------------------------------------------------------------------------------------------
    结果：
  (0,a-a1,false)
  (2,a-a1,false)
  (3,a-a1,false)
  (4,a-b1,false)
  (3,a-a1,true)
  (5,a-a1,true)
  (6,a-a1,true)
  (7,a-b1,true)
  (4,b-a1,true)
  (6,b-a1,true)
  (7,b-a1,true)
  (8,b-b1,true)
  (5,b-a1,true)
  (7,b-a1,true)
  (8,b-a1,true)
  (9,b-b1,true)
  (0,b-a1,true)
  (2,b-a1,true)
  (3,b-a1,true)
  (4,b-b1,true)
	```
- crossWithTiny
	
	```java
	-----------------------------------------------------------------------------------------------------------------------------
	结果：
	```
- crossWithHuge
	```java
	-----------------------------------------------------------------------------------------------------------------------------
	结果：
	```
### CoGroup
coGroup 会将两个流的所有相等不相等的数据都可以输出，可以看做是针对所有key的group，两个流的类型可以不一样
```java
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        ArrayList<Tuple3<Integer, String, Boolean>> list1 = new ArrayList<>();
        Tuple3 tuple4 = new Tuple3<Integer, String, Boolean>(0, "a1", false);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(2, "a1", true);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(3, "a1", true);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(4, "b1", true);
        list1.add(tuple4);


        DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
        DataSource<Tuple3<Integer, String, Boolean>> source1 = env.fromCollection(list1);

        source.coGroup(source1).where(0).equalTo(0).with(new CoGroupFunction<Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>>() {
            @Override
            public void coGroup(Iterable<Tuple3<Integer, String, Boolean>> first, Iterable<Tuple3<Integer, String, Boolean>> second, Collector<Tuple3<Integer, String, Boolean>> out) throws Exception {

               first.forEach(out::collect);
               second.forEach(out::collect);
            }
        }).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
(3,a,true)
(3,a1,true)
(0,a,false)
(0,b,true)
(0,a1,false)
(5,b,true)
(2,a1,true)
(4,b,true)
(4,b1,true)
```
### Union
两个或者多个流必须一样
```java
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        ArrayList<Tuple3<Integer, String, Boolean>> list1 = new ArrayList<>();
        Tuple3 tuple4 = new Tuple3<Integer, String, Boolean>(0, "a1", false);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(2, "a1", true);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(3, "a1", true);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(4, "b1", true);
        list1.add(tuple4);


        DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
        DataSource<Tuple3<Integer, String, Boolean>> source1 = env.fromCollection(list1);

        source.union(source1).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
(0,a,false)
(0,a1,false)
(3,a,true)
(2,a1,true)
(4,b,true)
(3,a1,true)
(5,b,true)
(4,b1,true)
(0,b,true)
```
### Rebalance
```java
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);


        DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);

        source.mapPartition(new RichMapPartitionFunction<Tuple3<Integer,String,Boolean>, Tuple4<Integer,String,Boolean,String>>() {
            @Override
            public void mapPartition(Iterable<Tuple3<Integer, String, Boolean>> values, Collector<Tuple4<Integer, String, Boolean,String>> out)
                    throws
                    Exception {
                values.forEach(x->{
                    out.collect(new Tuple4<Integer, String, Boolean,String>(x.f0,x.f1,x.f2,getRuntimeContext().getIndexOfThisSubtask()+""));
                });
            }
        }).rebalance().mapPartition(new RichMapPartitionFunction<Tuple4<Integer,String,Boolean,String>, Tuple5<Integer,String,Boolean,String,String>>() {
            @Override
            public void mapPartition(Iterable<Tuple4<Integer,String,Boolean,String>> values, Collector<Tuple5<Integer, String, Boolean,String,
                    String>> out)
                    throws
                    Exception {
                values.forEach(x->{
                    out.collect(new Tuple5<Integer, String, Boolean,String,String>(x.f0,x.f1,x.f2,x.f3,getRuntimeContext().getIndexOfThisSubtask()
                            +""));
                });
            }
        }).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
(0,a,false,0,0)
(3,a,true,0,1)
(4,b,true,0,2)
(5,b,true,0,3)
(0,b,true,0,4)
```
### PartitionBy

- partitionByHash
	```java
	      ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

  
        DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
    
        source.mapPartition(new RichMapPartitionFunction<Tuple3<Integer,String,Boolean>, Tuple4<Integer,String,Boolean,String>>() {
            @Override
            public void mapPartition(Iterable<Tuple3<Integer, String, Boolean>> values, Collector<Tuple4<Integer, String, Boolean,String>> out)
                    throws
                    Exception {
                values.forEach(x->{
                  out.collect(new Tuple4<Integer, String, Boolean,String>(x.f0,x.f1,x.f2,getRuntimeContext().getIndexOfThisSubtask()+""));
                });
            }
        }).partitionByHash(0).mapPartition(new RichMapPartitionFunction<Tuple4<Integer,String,Boolean,String>, Tuple5<Integer,String,Boolean,
                String,String>>() {
            @Override
            public void mapPartition(Iterable<Tuple4<Integer,String,Boolean,String>> values, Collector<Tuple5<Integer, String, Boolean,String,
                    String>> out)
                    throws
                    Exception {
                values.forEach(x->{
                    out.collect(new Tuple5<Integer, String, Boolean,String,String>(x.f0,x.f1,x.f2,x.f3,getRuntimeContext().getIndexOfThisSubtask()
                            +""));
                });
            }
        }).print();
    -----------------------------------------------------------------------------------------------------------------------------
    结果：
  (3,a,true,0,1)
  (0,a,false,0,6)
  (5,b,true,0,6)
  (0,b,true,0,6)
  (4,b,true,0,7)
  ```
- partitionByRange
	```java
	      ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

  
        DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
    
        source.mapPartition(new RichMapPartitionFunction<Tuple3<Integer,String,Boolean>, Tuple4<Integer,String,Boolean,String>>() {
            @Override
            public void mapPartition(Iterable<Tuple3<Integer, String, Boolean>> values, Collector<Tuple4<Integer, String, Boolean,String>> out)
                    throws
                    Exception {
                values.forEach(x->{
                  out.collect(new Tuple4<Integer, String, Boolean,String>(x.f0,x.f1,x.f2,getRuntimeContext().getIndexOfThisSubtask()+""));
                });
            }
        }).partitionByRange(0).mapPartition(new RichMapPartitionFunction<Tuple4<Integer,String,Boolean,String>, Tuple5<Integer,String,Boolean,
                String,String>>() {
            @Override
            public void mapPartition(Iterable<Tuple4<Integer,String,Boolean,String>> values, Collector<Tuple5<Integer, String, Boolean,String,
                    String>> out)
                    throws
                    Exception {
                values.forEach(x->{
                    out.collect(new Tuple5<Integer, String, Boolean,String,String>(x.f0,x.f1,x.f2,x.f3,getRuntimeContext().getIndexOfThisSubtask()
                            +""));
                });
            }
        }).print();
    -----------------------------------------------------------------------------------------------------------------------------
    结果：
  (0,a,false,0,1)
  (0,b,true,0,1)
  (3,a,true,0,3)
  (4,b,true,0,5)
  (5,b,true,0,6)
  ```
- sortPartition
	
	```java
	      ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);

        source.mapPartition(new RichMapPartitionFunction<Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>>() {
            @Override
            public void mapPartition(Iterable<Tuple3<Integer, String, Boolean>> values, Collector<Tuple3<Integer, String, Boolean>> out) throws Exception {
                values.forEach(x-> {
                    System.out.println(x);
                    out.collect(x);
                });
                System.out.println("==============");
            }
        }).sortPartition(0,Order.ASCENDING).mapPartition(new RichMapPartitionFunction<Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>>() {
            @Override
            public void mapPartition(Iterable<Tuple3<Integer, String, Boolean>> values, Collector<Tuple3<Integer, String, Boolean>> out) throws Exception {
                values.forEach(x-> {
                    System.out.println(x);
                    out.collect(x);
                });
            }
        }).count();
    -----------------------------------------------------------------------------------------------------------------------------
    结果：
  (0,a,false)
  (3,a,true)
  (4,b,true)
  (5,b,true)
  (0,b,true)
  ==============
  (0,a,false)
  (0,b,true)
  (3,a,true)
  (4,b,true)
  (5,b,true)
	```
### First
```java
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        DataSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);

        source.first(2).print();
        System.out.println("=====================");
        source.groupBy(1).first(2).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
(0,a,false)
(3,a,true)
=====================
(0,a,false)
(3,a,true)
(4,b,true)
(5,b,true)
```
+++
### KeyBy
```java
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        DataStreamSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
        source.map(new RichMapFunction<Tuple3<Integer,String,Boolean>, Tuple4<Integer,String,Boolean,String>>() {
            @Override
            public Tuple4<Integer, String, Boolean,String> map(Tuple3<Integer, String, Boolean> value) throws Exception {
                return new Tuple4<Integer, String, Boolean,String>(value.f0,value.f1,value.f2,getRuntimeContext().getIndexOfThisSubtask()+"");
            }
        }).keyBy(1).process(new KeyedProcessFunction<Tuple, Tuple4<Integer,String,Boolean,String>, Tuple5<Integer,String,Boolean,String,
                String>>
                () {
            @Override
            public void processElement(Tuple4<Integer, String, Boolean,String> value, Context ctx, Collector<Tuple5<Integer, String, Boolean,String,
                    String>> out)
                    throws Exception {
                out.collect(new Tuple5<>(value.f0,value.f1,value.f2,value.f3,getRuntimeContext().getIndexOfThisSubtask()+""));

            }
        }).print();

        env.execute();
---------------------------------------------------------------------------------------------------------------------------------
结果：
2> (0,b,true,0,1)
2> (4,b,true,6,1)
2> (5,b,true,7,1)
6> (0,a,false,4,5)
6> (3,a,true,5,5)
```
>`2>` 2表示执行该任务的线程号，就是getRuntimeContext().getIndexOfThisSubtask()+1
### Fold
```java
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        DataStreamSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
        source.keyBy(1).fold("start", new FoldFunction<Tuple3<Integer, String, Boolean>, String>() {
            @Override
            public String fold(String accumulator, Tuple3<Integer, String, Boolean> value) throws Exception {
                return accumulator+"-"+value.f0+","+value.f1+","+value.f2;
            }
        }).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
2> start-4,b,true
6> start-0,a,false
2> start-4,b,true-5,b,true
6> start-0,a,false-3,a,true
2> start-4,b,true-5,b,true-0,b,true
```
>fold is deprecated
### Window
- window
	
	需要对数据流先进行keyBy操作，将数据流按照每个key进行分区，再在每个key分区上进行window操作，每个key分区一个window会有很多window。
	
	- timeWindow
		- timeWindow(size)  
  		window(TumblingProcessingTimeWindows.of(size)) or window(TumblingEventTimeWindows.of(size))
  		
  		```java
              StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
      
              ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
              Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);
      
              tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
              list.add(tuple3);
              tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
              list.add(tuple3);
              tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
              list.add(tuple3);
              tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);
      
        DataStreamSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
      
              source.assignTimestampsAndWatermarks(new AscendingTimestampExtractor<Tuple3<Integer, String, Boolean>>() {
                  @Override
                  public long extractAscendingTimestamp(Tuple3<Integer, String, Boolean> element) {
                      return System.currentTimeMillis();
                  }
        }).keyBy(1).timeWindow(Time.milliseconds(1))
      //                .window(TumblingEventTimeWindows.of(Time.milliseconds(1)))
              .apply(new WindowFunction<Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>, Tuple, TimeWindow>() {
                  @Override
                  public void apply(Tuple tuple, TimeWindow window, Iterable<Tuple3<Integer, String, Boolean>> input, Collector<Tuple3<Integer, String, Boolean>> out) throws Exception {
	                    input.forEach(out::collect);
	                }
	            }).print();
  		-----------------------------------------------------------------------------------------------------------------------
  		结果：
  		2> (4,b,true)
      2> (5,b,true)
	    2> (0,b,true)
	    6> (0,a,false)
	    6> (3,a,true)
			```

		- timeWindow(size,slide)
			window(SlidingProcessingTimeWindows.of(size, slide)) or window(SlidingEventTimeWindows.of(size, slide))
			
			```java
			        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
				env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
			        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
			        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
			        list.add(tuple3);
			
		        	tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
			        list.add(tuple3);
			        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
			        list.add(tuple3);
			        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
			        list.add(tuple3);
			        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
			        list.add(tuple3);
			
			        DataStreamSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
			
			        source.assignTimestampsAndWatermarks(new AscendingTimestampExtractor<Tuple3<Integer, String, Boolean>>() {
			            @Override
			            public long extractAscendingTimestamp(Tuple3<Integer, String, Boolean> element) {
			                return System.currentTimeMillis();
			            }
			        }).keyBy(1).timeWindow(Time.milliseconds(1),Time.milliseconds(1))
			//                .window(SlidingEventTimeWindows.of(Time.milliseconds(1), Time.milliseconds(1)))
			        .apply(new WindowFunction<Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>, Tuple, TimeWindow>() {
			            @Override
			            public void apply(Tuple tuple, TimeWindow window, Iterable<Tuple3<Integer, String, Boolean>> input, Collector<Tuple3<Integer, String, Boolean>> out) throws Exception {
			                input.forEach(out::collect);
			            }
			        }).print();
			-----------------------------------------------------------------------------------------------------------------------
			结果：
			2> (4,b,true)
			6> (0,a,false)
			2> (5,b,true)
			2> (0,b,true)
			6> (3,a,true)
			```
	- countWindow
		```java
		        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
		        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
		
		        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
		        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
		        list.add(tuple3);
		
		        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
		        list.add(tuple3);
		        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
		        list.add(tuple3);
		        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
		        list.add(tuple3);
		        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
		        list.add(tuple3);
		
		        DataStreamSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
		
		        source.assignTimestampsAndWatermarks(new AscendingTimestampExtractor<Tuple3<Integer, String, Boolean>>() {
		            @Override
		            public long extractAscendingTimestamp(Tuple3<Integer, String, Boolean> element) {
		                return System.currentTimeMillis();
		            }
		        }).keyBy(1).countWindow(2).apply(new WindowFunction<Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>, Tuple, GlobalWindow>() {
		            @Override
		            public void apply(Tuple tuple, GlobalWindow window, Iterable<Tuple3<Integer, String, Boolean>> input, Collector<Tuple3<Integer, String, Boolean>> out) throws Exception {
		                input.forEach(out::collect);
		            }
		        }).print();
		-----------------------------------------------------------------------------------------------------------------------
		结果：
		2> (4,b,true)
		6> (0,a,false)
		2> (5,b,true)
		6> (3,a,true)
		```
	
- windowAll
	
	不用进行分区直接对数据流进行window操作，所有key都在这一个window上进行操作，只有一个window
	
	```java
	        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
	        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
	
	        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
	        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
	        list.add(tuple3);
	
	        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
	        list.add(tuple3);
	        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
	        list.add(tuple3);
	        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
	        list.add(tuple3);
	        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
	        list.add(tuple3);
	
	        DataStreamSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
	
	        source.windowAll(GlobalWindows.create()).trigger(PurgingTrigger.of(CountTrigger.of(3))).apply(new AllWindowFunction<Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>, GlobalWindow>() {
	            @Override
	            public void apply(GlobalWindow window, Iterable<Tuple3<Integer, String, Boolean>> values, Collector<Tuple3<Integer, String, Boolean>> out) throws Exception {
	                values.forEach(out::collect);
	            }
	        }).print();
	-----------------------------------------------------------------------------------------------------------------------------
	结果：
	1> (3,a,true)
	8> (0,a,false)
	2> (4,b,true)
	```

### Connect
将两个具有相同或者不同结构的流，融合成一个流。两个流的元素类型可以不同。
```java
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        ArrayList<Tuple3<Integer, String, Boolean>> list1 = new ArrayList<>();
        Tuple3 tuple4 = new Tuple3<Integer, String, Boolean>(0, "a1", false);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(2, "a1", true);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(3, "a1", true);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(4, "b1", true);
        list1.add(tuple4);

        DataStreamSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
        DataStreamSource<Tuple3<Integer, String, Boolean>> source1 = env.fromCollection(list1);
        
        source.connect(source1).process(new CoProcessFunction<Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>, Tuple4<Integer,
                String,Boolean,String>>() {
            @Override
            public void processElement1(Tuple3<Integer, String, Boolean> value, Context ctx, Collector<Tuple4<Integer, String, Boolean,String>> out)
                    throws Exception {
                out.collect(new Tuple4<>(value.f0,value.f1,value.f2,"1"));
            }

            @Override
            public void processElement2(Tuple3<Integer, String, Boolean> value, Context ctx, Collector<Tuple4<Integer, String, Boolean,String>> out) throws Exception {
                out.collect(new Tuple4<>(value.f0,value.f1,value.f2,"2"));
            }
        }).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
6> (4,b1,true,2)
1> (4,b,true,1)
4> (2,a1,true,2)
7> (0,a,false,1)
3> (0,b,true,1)
2> (5,b,true,1)
3> (0,a1,false,2)
8> (3,a,true,1)
5> (3,a1,true,2)
```
### CoMap
```java
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        ArrayList<Tuple3<Integer, String, Boolean>> list1 = new ArrayList<>();
        Tuple3 tuple4 = new Tuple3<Integer, String, Boolean>(0, "a1", false);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(2, "a1", true);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(3, "a1", true);
        list1.add(tuple4);
        tuple4 = new Tuple3<Integer, String, Boolean>(4, "b1", true);
        list1.add(tuple4);

        DataStreamSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);
        DataStreamSource<Tuple3<Integer, String, Boolean>> source1 = env.fromCollection(list1);

        source.connect(source1).map(new CoMapFunction<Tuple3<Integer,String,Boolean>, Tuple3<Integer,String,Boolean>, Tuple4<Integer,String,
                Boolean,String>>() {
            @Override
            public Tuple4<Integer,String, Boolean,String> map1(Tuple3<Integer, String, Boolean> value) throws Exception {
                return new Tuple4<>(value.f0,value.f1,value.f2,"1");
            }

            @Override
            public Tuple4<Integer,String, Boolean,String> map2(Tuple3<Integer, String, Boolean> value) throws Exception {
                return new Tuple4<>(value.f0,value.f1,value.f2,"2");
            }
        }).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
8> (4,b,true,1)
3> (2,a1,true,2)
6> (0,a,false,1)
1> (5,b,true,1)
2> (0,b,true,1)
7> (3,a,true,1)
4> (3,a1,true,2)
5> (4,b1,true,2)
2> (0,a1,false,2)
```
### CoFlatMap
```java
---------------------------------------------------------------------------------------------------------------------------------
结果：
```

### Split
```java
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        DataStreamSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);

        source.split(new OutputSelector<Tuple3<Integer, String, Boolean>>() {
            @Override
            public Iterable<String> select(Tuple3<Integer, String, Boolean> value) {
                ArrayList<String> list = new ArrayList<>();
                if (value.f0%2==0){
                    list.add("1");
                }else {
                    list.add("2");
                }
                return list;
            }
        }).select("1" ).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
5> (4,b,true)
6> (0,b,true)
4> (0,a,false)
```
### Select
```java
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        DataStreamSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);

        source.split(new OutputSelector<Tuple3<Integer, String, Boolean>>() {
            @Override
            public Iterable<String> select(Tuple3<Integer, String, Boolean> value) {
                ArrayList<String> list = new ArrayList<>();
                if (value.f0%2==0){
                    list.add("1");
                }else {
                    list.add("2");
                }
                return list;
            }
        }).select("1" ).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
5> (4,b,true)
6> (0,b,true)
4> (0,a,false)
```
### PartitionCustom
```java
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        DataStreamSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);

        source.map(new RichMapFunction<Tuple3<Integer,String,Boolean>, Tuple4<Integer,String,Boolean,String>>() {
            @Override
            public Tuple4<Integer,String,Boolean,String> map(Tuple3<Integer, String, Boolean> value) throws Exception {
                return new Tuple4<>(value.f0,value.f1,value.f2,getRuntimeContext().getIndexOfThisSubtask()+"");
            }
        }).partitionCustom(new Partitioner<String>() {
            @Override
            public int partition(String key, int numPartitions) {
                return key.hashCode()%numPartitions;
            }
        },1).map(new RichMapFunction<Tuple4<Integer,String,Boolean,String>, Tuple5<Integer,String,Boolean,String,String>>() {
            @Override
            public Tuple5<Integer,String,Boolean,String,String> map(Tuple4<Integer,String,Boolean,String> value) throws Exception {
                return new Tuple5<>(value.f0, value.f1, value.f2, value.f3,getRuntimeContext().getIndexOfThisSubtask()+"");
            }
        }).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
2> (3,a,true,6,1)
2> (0,a,false,5,1)
3> (0,b,true,1,2)
3> (5,b,true,0,2)
3> (4,b,true,7,2)
```
### Shuffle
```java
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        DataStreamSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);

        source.map(new RichMapFunction<Tuple3<Integer,String,Boolean>, Tuple4<Integer,String,Boolean,String>>() {
            @Override
            public Tuple4<Integer,String,Boolean,String> map(Tuple3<Integer, String, Boolean> value) throws Exception {
                return new Tuple4<>(value.f0, value.f1, value.f2,getRuntimeContext().getIndexOfThisSubtask()+"");
            }
        }).shuffle().map(new RichMapFunction<Tuple4<Integer,String,Boolean,String>, Tuple5<Integer,String,Boolean,String,String>>() {
            @Override
            public  Tuple5<Integer,String,Boolean,String,String> map(Tuple4<Integer, String, Boolean, String> value) throws Exception {
                return new Tuple5<>(value.f0, value.f1, value.f2, value.f3,getRuntimeContext().getIndexOfThisSubtask()+"");
            }
        }).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
8> (0,b,true,0,7)
8> (4,b,true,6,7)
8> (3,a,true,5,7)
8> (0,a,false,4,7)
8> (5,b,true,7,7)
```
### Rebalance
```java
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        DataStreamSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);

        source.map(new RichMapFunction<Tuple3<Integer,String,Boolean>, Tuple4<Integer,String,Boolean,String>>() {
            @Override
            public Tuple4<Integer,String,Boolean,String> map(Tuple3<Integer, String, Boolean> value) throws Exception {
                return new Tuple4<>(value.f0, value.f1, value.f2,getRuntimeContext().getIndexOfThisSubtask()+"");
            }
        }).rebalance().map(new RichMapFunction<Tuple4<Integer,String,Boolean,String>, Tuple5<Integer,String,Boolean,String,String>>() {
            @Override
            public  Tuple5<Integer,String,Boolean,String,String> map(Tuple4<Integer, String, Boolean, String> value) throws Exception {
                return new Tuple5<>(value.f0, value.f1, value.f2, value.f3,getRuntimeContext().getIndexOfThisSubtask()+"");
            }
        }).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
2> (5,b,true,6,1)
6> (0,b,true,7,5)
7> (0,a,false,3,6)
5> (3,a,true,4,4)
3> (4,b,true,5,2)
```
### Rescale
```java
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        DataStreamSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);

        source.map(new RichMapFunction<Tuple3<Integer,String,Boolean>, Tuple4<Integer,String,Boolean,String>>() {
            @Override
            public Tuple4<Integer,String,Boolean,String> map(Tuple3<Integer, String, Boolean> value) throws Exception {
                return new Tuple4<>(value.f0, value.f1, value.f2,getRuntimeContext().getIndexOfThisSubtask()+"");
            }
        }).rescale().map(new RichMapFunction<Tuple4<Integer,String,Boolean,String>, Tuple5<Integer,String,Boolean,String,String>>() {
            @Override
            public  Tuple5<Integer,String,Boolean,String,String> map(Tuple4<Integer, String, Boolean, String> value) throws Exception {
                return new Tuple5<>(value.f0, value.f1, value.f2, value.f3,getRuntimeContext().getIndexOfThisSubtask()+"");
            }
        }).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
4> (0,b,true,3,3)
3> (5,b,true,2,2)
2> (4,b,true,1,1)
1> (3,a,true,0,0)
8> (0,a,false,7,7)
```
### Broadcast
```java
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

        ArrayList<Tuple3<Integer, String, Boolean>> list = new ArrayList<>();
        Tuple3 tuple3 = new Tuple3<Integer, String, Boolean>(0, "a", false);
        list.add(tuple3);

        tuple3 = new Tuple3<Integer, String, Boolean>(3, "a", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(4, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(5, "b", true);
        list.add(tuple3);
        tuple3 = new Tuple3<Integer, String, Boolean>(0, "b", true);
        list.add(tuple3);

        DataStreamSource<Tuple3<Integer, String, Boolean>> source = env.fromCollection(list);

        source.map(new RichMapFunction<Tuple3<Integer,String,Boolean>, Tuple4<Integer,String,Boolean,String>>() {
            @Override
            public Tuple4<Integer,String,Boolean,String> map(Tuple3<Integer, String, Boolean> value) throws Exception {
                return new Tuple4<>(value.f0, value.f1, value.f2,getRuntimeContext().getIndexOfThisSubtask()+"");
            }
        }).broadcast().map(new RichMapFunction<Tuple4<Integer,String,Boolean,String>, Tuple5<Integer,String,Boolean,String,String>>() {
            @Override
            public  Tuple5<Integer,String,Boolean,String,String> map(Tuple4<Integer, String, Boolean, String> value) throws Exception {
                return new Tuple5<>(value.f0, value.f1, value.f2, value.f3,getRuntimeContext().getIndexOfThisSubtask()+"");
            }
        }).print();
---------------------------------------------------------------------------------------------------------------------------------
结果：
5> (0,a,false,1,4)
6> (5,b,true,4,5)
8> (5,b,true,4,7)
5> (5,b,true,4,4)
3> (4,b,true,3,2)
5> (3,a,true,2,4)
5> (4,b,true,3,4)
3> (3,a,true,2,2)
5> (0,b,true,5,4)
4> (3,a,true,2,3)
3> (0,a,false,1,2)
4> (0,a,false,1,3)
3> (5,b,true,4,2)
4> (5,b,true,4,3)
3> (0,b,true,5,2)
4> (4,b,true,3,3)
1> (0,a,false,1,0)
4> (0,b,true,5,3)
2> (5,b,true,4,1)
2> (4,b,true,3,1)
2> (3,a,true,2,1)
1> (5,b,true,4,0)
2> (0,a,false,1,1)
2> (0,b,true,5,1)
8> (0,a,false,1,7)
8> (3,a,true,2,7)
8> (0,b,true,5,7)
8> (4,b,true,3,7)
6> (3,a,true,2,5)
7> (3,a,true,2,6)
6> (4,b,true,3,5)
1> (4,b,true,3,0)
7> (0,a,false,1,6)
6> (0,a,false,1,5)
7> (5,b,true,4,6)
1> (3,a,true,2,0)
6> (0,b,true,5,5)
1> (0,b,true,5,0)
7> (0,b,true,5,6)
7> (4,b,true,3,6)
```

## 分区策略

### ForwardPartitioner
数据从一个算子一对一的转换到下游另一个算子，同机器无网络传送,同一个OperationChain中上下游算子之间的数据转发

### ShufflePartitioner
随机分区
```shel
dataStream.shuffle()
```
### RebalancePartitioner
数据均匀分配至下游算子
```shell
dataStream.rebalence()
```
### RescalingPartitioner
根据下游算子数量，上游算子被分配固定数量的下游算子，上游算子使用Round-robin的方式将数据均匀分配到下游算子，上游算子不会将数据分配到其他不属于它的下游算子
```shell
dataStream.rescale()
```
### BroadcastPartitioner
数据复制一份从上一个算子发送到下游每一个算子
```shell
dataStream.broadcast()
```
### KeyGroupStreamPartitioner
根据KeyGroup素银编号进行分区，该分区器不支持用户使用，是系统层面上的
### CustomPartitionerWrapper
用户自定义分区器
```shell
datastream.partitionCustom(partitioner,"name")
datastream.partitionCustom(partitioner,0)
```
## 组件
### JobManager
### ResourceManager
### TaskManager
### Dispatcher
+++
## Application
+ 启动：flink run -c mainclasspath jarpath
+ 取消：flink cancel jobid
+ 停止：flink stop jobid
### Job
## Task
job中的一个阶段就是一个task，一个task包括链条连接的多个subtask，一个task运行在一个线程里面，task平分slot里面的内存资源共享slot里面的cpu资源
## SubTask
flink job中最小执行单元
## 算子

### Source

使用EventTime，划分滚动窗口，如果使用的是并行的Source，例如KafkaSource，创建Kafka的Topic时有多个分区，每个Source的分区都要满足触发的条件，整个窗口才会被触发

### Transformation
+ map	//DataStream - > DataStream
+	flatMap	//DataStream -> DataStream
+	filter	//DataStream -> DataStream
+	keyBy	//DataStream -> KeyStream
### Sink

writeAsCsv必须是元组才能正常写入

## WaterMark
决定一个窗口什么时候激活（触发），这时的窗口的最大长度为

materMark>=上一个窗口的结束边界就会触发窗口执行

watermark是flink中窗口延迟触发的机制

在每个算子内部都自己有一个事件时间时钟，事件时间时钟是根据watermark来更新的，。流入算子的数据可能是单分区也可能是多分区的，每个流入算子的分区端都会有一个自己的partition watermark标记，当该分区内进入新的高于之前watermark的watermark数据时，partition watermark标记才会被更新，task内部也维护一个task的watermark数据。如果某个分区的partition watermark < task watermark，那么task watermark会更新为该partition watermark数据，然后把task 把当前更新的task watermark数据发向下游task。

### AssignerWithPeriodicWatermarks
windowmax=watermark+windowsize
waterMark=数据携带的时间（窗口中最大的时间）-延迟执行的时间

周期性的生成watermark，定期向分区数据流中插入时间水印。默认周期时间为200毫秒，可以使用setAutoWatermakrIntaval（）来设置
BoundedOutOfOrdernessTimestampExtractor继承自AssignerWithPeriodicWatermarks 属于周期性watermark

>周期性数据水印在特定条件下可能会造成数据错误
>例如：env.fromCollection(List((1, "a1", 158324361000l), (1, "a2", 158324369000l), (1, "a3", 158324364000l), (1, "a4", 158324361000l), (1, "a5",158324365000l), (1, "a6", 158324362000l), (1, "a7",158324367000l))) 
>.assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor[(Int, String, Long)](Time.milliseconds(0)) {
>override def extractTimestamp(element: (Int, String, Long)): Long = {
> element._3
> }
> })
> .keyBy(0)
> .window(TumblingEventTimeWindows.of(Time.seconds(4)))
> .sum(2)
> .print()
> 当使用周期性水印AssignerWithPeriodicWatermarks时就会造成数据的错误计算
> 是因为周期性水印是定期产生的（默认200毫秒）但是在这个周期里可能会出现有多个数据已经过去，这多个数据用于一个水印从而造成数据计算错误

### AssignerWithPunctuatedWatermarks
根据事件生成watermark。可以用于根据具体数据来生成watermark，
## Window

### Keyed Window
使用keyby后流的窗口
#### GlobalWindow
#### CountWindow
#### TimeWindow
+ Tumbling
+ Sliding
+ Session

### Non-Keyed Windows
未使用keyby后流的窗口
#### windowAll

## Window 之后的算子
### Trigger
window 数据触发器，keyed or non-keyed window 都可以使用
+ EventTimeTrigger：事件时间触发器
+ ProcessingTimeTrigger：程序时间触发器
+ CountTrigger：数量出发器。只发送窗口触发信号
+ PurgingTrigger：代理模式触发器，发送窗口触发和数据清理信号
### Evictor
window 数据剔除器，可以在window执行前或者执行后剔除window内的元素
+ CountEvictor：数量剔除器。在Window中保留指定数量的元素，并从窗口头部开始丢弃其余元素。
+ DeltaEvictor： 阈值剔除器。计算Window中最后一个元素与其余每个元素之间的增量，丢弃增量大于或等于阈值的元素。
+ TimeEvictor：时间剔除器。保留Window中最近一段时间内的元素，并丢弃其余元素。
### AllowedLateness

决定一个窗口什么时候销毁，window 延迟数据是否保留计算，可能会造成窗口的二次触发，会导致结果数据的更新，造成数据不一致。这时窗口的最大长度为windowmax=waterma+windowsize+allowedleteness

## State
### Managed State
#### Operator State
operator state 绑定到每个算子的实例上，各个实例拥有自己的state，一个实例无法获取同并行的其他实例的state数据

operator state ：记录的是每一个分区的偏移量

#### Keyed State

keyedstate：在一个subtask中可能有多个state，一个组对应一个key的状态



### Raw State
## CheckPoint
全自动程序管理，轻量快捷算子级数据快照

开启flink checkpoint 后设置精准一次消费，kafka的offset会保存在savepoint设置的路径里面，还会降offset保存在kafka 特殊topic里面，如果程序重启时没有指定savepiont保存数据的地址会默认根据kafka 特殊topic保存的偏移量消费数据，可以设置不降offset保存在kafka 特殊topic里面使用，setCommitOffsetOnCheckpoints(false)

开启检查点机制

```java
// start a checkpoint every 1000 ms
env.enableCheckpointing(1000);

// advanced options:

// set mode to exactly-once (this is the default)
env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);

// make sure 500 ms of progress happen between checkpoints
env.getCheckpointConfig().setMinPauseBetweenCheckpoints(500);

// checkpoints have to complete within one minute, or are discarded
env.getCheckpointConfig().setCheckpointTimeout(60000);

// allow only one checkpoint to be in progress at the same time
env.getCheckpointConfig().setMaxConcurrentCheckpoints(1);

// enable externalized checkpoints which are retained after job cancellation
env.getCheckpointConfig().enableExternalizedCheckpoints(ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);

// allow job recovery fallback to checkpoint when there is a more recent savepoint
env.getCheckpointConfig().setPreferCheckpointForRecovery(true);
```
### Barrier
算子checkpoint的依据，是exactly-once 和at-least-once语义的根据
## SavePoint

人工参与管理的application级别的数据快照

### 手动保存数据快照
+ flink stop jobid
  停止job并保存快照，如果在flink-conf.yaml上配置了state.savepoints.dir，停止任务后会自动将快照保存。
  
+ flink stop jobid -p dirpath
  停止job并将快照保存在dirpath
  
+ flink savepoint jobid [dirpath] 
  在不结束job的情况下保存快照。如果带有dirpath则会将快照保存在此目录否则会保存在默认配置的保存目录
### 从数据快照恢复程序
+ 直接从savepoint目录恢复
  flink run -s dirpath
  从dirpath目录恢复程序
  
+ 跳过无法恢复的算子恢复
  flink run -s dirpath -n
  
### 手动清除数据快照
+ flink savepoint -d itemdirpath
  手动将数据某个具体快照删除（itemdirpath 快照具体根目录）
  
  
  
## CEP（Complex Event Processing）

NFA（Nondeterministic Finite Automaton）

## 反压
flink 三层buff缓存（resultsubpartition，nettybuff，netty中通过高水位来控制buff是否还可以接收数据，socketbuff）

1.5 之前是基于tcp 窗口的反压机制，发送端根据接收端返回的ack和windox size 来发送数据，当window size 为0时，发送端则不会再附上数据，而是发送一个zerowindow 的探测性数据来确定是否可以再次发送数据，当接收端继续可以接收数据时，发送端才会继续发送数据

基于tcp窗口的反压机制缺点
1.单个task造成的反压，会阻断整个TM-TM的socket，连chekcpoint barrier 也无法发送
2.反压路径较长，导致生效延迟较大


1.5 引入credit 机制实现反压，credit 反压机制是类似于tcp 窗口反压实现的另一种反压机制，resultsubpartition在发送数据时会带有resultbuff里面还存有的数据大小 backlog size，inputchanel在接收到时会计算自己当前还能接收到的数据大小，当inputchanel无法再接收数据时会将credit置为0，告诉result不能再接收消息。result每次发送消息时会检测当前自己的credit数据，当credit为0时 则不会再向netty发送数据从而实现反压机制



## Flink 保持数据一直性（Exactly-Once）
- 内部保证 —— checkpoint

- source 端 —— 可重设数据的读取位置

- sink 端 —— 从故障恢复时，数据不会重复写入外部系统
  - 幂等写入
  
    所谓幂等操作，是说一个操作，可以重复执行很多次，但只导致一次结果更改，也就是说，后面再重复执行就不起作用了
  
    从Flink程序sink到的key-value存储中读取数据的应用，在Flink从检查点恢复的过程中，可能会看到不想看到的结果。当重播开始时，之前已经发出的计算结果可能会被更早的结果所覆盖（因为在恢复过程中）。所以，一个消费Flink程序输出数据的应用，可能会观察到时间回退，例如读到了比之前小的计数。
  
  - 事务写入
  
    应用程序中一系列严密的操作，所有操作必须成功完成，否则在每个操作中所作的所有更改都会被撤消。具有原子性：一个事务中的一系列的操作要么全部成功，要么一个都不做。
  
    事务性的方法将不会遭受幂等性写入所遭受的重播不一致的问题。但是，事务性写入却带来了延迟，因为只有在检查点完成以后，我们才能看到计算结果。
  
    Flink提供了两种构建模块来实现事务性sink连接器：write-ahead-log（**WAL**，预写式日志）sink和**两阶段提交sink**。
  
    - 预写日志（Write-Ahead-Log , WAL）
    
      （1）把结果数据先当成状态保存，然后在收到chekpoint完成的通知时，一次性写入sink系统
      （2）简单易于实现，由于数据提前在状态后端中做了缓存，所以无论什么sink系统，都能用这种方式一批搞定
  （3）DataStream API 提供了一个模板类：GenericWriteAheadSink，来实现这种事务性sink
    
    - 两阶段提交（Two-Phase-Commit , 2PC）
    
    （1）对于每个checkpoint，sink任务会启动一个事务，并将接下来所有接受的数据添加到事务里
    （2）然后将这些数据写入外部sink系统，但不提交他们——这是只是“预提交”
    （3）当它收到checkpoint完成的通知时，它才正式提交事务，实现结果的真正写入
    （4）这种方式真正实现了exactly-once，它需要一个提供事务支持的外部sink系统。Flink提供了TwoPhaseCommitSinkFunction接口。
  
  >
  >
  >在使用kafka011 sink 时注意的点：
  >
  >1.为了保证事务特性，在使用其他程序去消费我们flink sink 数据的kafka时，这个consumer需要设置了`isolation.level = read_committed`，那么它只会读取已经提交了的消息。
  >
  >2.Checkpoint超时时间 必需大于 kafka 提交事务时间。
  >
  >假如checkpoint失败时间高于 kafka事务等待时间，比如，设置了一个checkpoint最多等待10分钟，10分钟后会失败这个checkpoint的保存。而kafka 的事务只能等待5分钟，5分钟后把uncommitted的事务关掉。这个时候6分钟checkpoint成功了，但是对应kafka数据的事务已经失败。这样就无法保证Exactly-once的实现







## 其他

两个流join 必须有等值字段必须都在同一个窗口里面

duplicate key update  mysql数据库的更新插入 合为一条sql

并行度： **算子级别** > **env级别** > **Client级别** > **系统默认级别**

在所有Task共享资源槽点名字相同，默认情况下 （pipline）
同一个job的同一个Task中的多个subTask不能在同一个slot槽中

>具有并行度的subtask 不能在一个slot槽中
对于同一个job，不同Task【阶段】的subTask可以在同一个资源槽中


