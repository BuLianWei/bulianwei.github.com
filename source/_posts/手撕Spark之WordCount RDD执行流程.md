---
title: 手撕Spark之WordCount RDD执行流程
date: 2019-12-18 10:03:14
categories: Spark
tags: Spark
---

##  手撕Spark之WordCount RDD执行流程

[TOC]



### 写在前面

一个Spark程序在初始化的时候会构造DAGScheduler、TaskSchedulerImpl、MapOutTrackerMaster等对象，DAGScheduler主要负责生成DAG、启动Job、提交Stage等操作，TaskSchedulerImpl主要负责Task Set的添加调度等，MapOutTrackerMaster主要负责数据的Shuffle等，这里不再赘述。

**注意几个概念：**

- Application   //一个Spark程序会有一个Application，也就拥有了唯一的一个applicationId
- Job    //调用Action 算子 触发runJob，触发一次runJob就会产生一个Job
- Stage  //遇到一次宽依赖就会生成一个Stage
- Task  //Spark程序运行的最小单元

> 注：一个Spark程序会有1个Application，会有1～N 个Job，会有1～N 个Stage，会有1～N 个Task
>
> 1 Application = [1 ~ N  ] Job
> 1 Job = [ 1 ~ N ] Stage
> 1 Stage = [ 1 ~ N ] Task
>
> Stage数 = Shuffle数 +1



### 软件环境

+ Spark：2.3.0

### 代码

写一个简单的WordCount计算代码

data.txt

~~~txt
hello world
hello java
hello scala
hello hadoop
hello spark
~~~

WCAnalyzer.scala

~~~scala

    //设置日志输出级别，便于观察日志
    Logger.getLogger("org.apache").setLevel(Level.ALL)

    //创建sc
    val sc = new SparkContext(new SparkConf().setMaster("local[1]")
                              .setAppName("WCAnalyzer"))

    //从文件读取数据
    sc.textFile("data/data.txt", 1)
      //将数据按照空格进行切分（切分出单个单词）
      .flatMap(_.split(" "))
      //将每个单词和1组成一个Tuple
      .map((_, 1))
      //按照相同的单词进行聚合
      .reduceByKey(_ + _)
      //将聚合后的结果将（key，value）数据进行倒置 转换成（value，key）便于排序
      .map(v => (v._2, v._1))
      //按照聚合后的单词数量进行降序排序
      .sortByKey(false)
      //将排序后的数据进行倒置
      .map(v => (v._2, v._1))
      //将数据收集到driver
      .collect()
      //输出数据
      .foreach(println)

    //关闭sc
    sc.stop()
  }
~~~



### 过程分析

本代码只会生成一个Job，3个Stage，8个RDD。

+ 划分Stage

  Stage的划分要从后向前，每遇到一次宽依赖就划分一个Stage，因此这个简单的WC代码可以分为3个Stage，分别是由textFile、flatMap、map算子组成的第一个Stage 0；由reduceByKey、map算子组成的Stage 1；由sortByKey、map算子组成的Stage 2。

+ RDD的生成 

  textFile（HadoopRDD [0] ，MapPartitionsRDD [1] ）  //[ ] 内为该rdd的序号

  flatMap（MapPartitionsRDD [2] ）

  map（MapPartitionsRDD [3] ）

  reduceByKey（ShuffledRDD [4] ）

  map（MapPartitionsRDD [5] ）

  sortByKey（ShuffledRDD [6] ）

  map （MapPartitionsRDD [7] ）

+ 日志分析

    ~~~txt
    org.apache.spark.SparkContext                     - Starting job: collect at WCAnalyzer.scala:34
    ~~~
    由collect算子触发runJob 启动一个Job，代码中的`foreach(println)`其中`foreach`并不是RDD中的算子，因此不会触发runJob，也就不会生成一个Job

    ~~~txt
    org.apache.spark.scheduler.DAGScheduler           - Got job 0 (collect at WCAnalyzer.scala:34) with 1 output partitions
    ~~~

    	生成一个Job 0 ，这个Job是由collect算子生成，在代码第34行，有一个分区

    ~~~txt
    org.apache.spark.scheduler.DAGScheduler          - Final stage: ResultStage 2 (collect at WCAnalyzer.scala:34)
    org.apache.spark.scheduler.DAGScheduler          - Parents of final stage: List(ShuffleMapStage 1)
    org.apache.spark.scheduler.DAGScheduler          - Missing parents: List(ShuffleMapStage 1)
    org.apache.spark.scheduler.DAGScheduler          - submitStage(ResultStage 2)
    org.apache.spark.scheduler.DAGScheduler          - missing: List(ShuffleMapStage 1)
    org.apache.spark.scheduler.DAGScheduler          - submitStage(ShuffleMapStage 1)
    org.apache.spark.scheduler.DAGScheduler          - missing: List(ShuffleMapStage 0)
    org.apache.spark.scheduler.DAGScheduler          - submitStage(ShuffleMapStage 0)
    org.apache.spark.scheduler.DAGScheduler          - missing: List()
    ~~~

    Job 的Final Stage 为ResultStage 0，ResultStage 的父依赖为ShuffleMapStage 1，遗留的父依赖为ShuffleMapStage 1。

    ~~~txt
    org.apache.spark.scheduler.DAGScheduler         - submitStage(ResultStage 2)
    ~~~

    尝试提交ResultStage 2

    ~~~txt
    org.apache.spark.scheduler.DAGScheduler         - missing: List(ShuffleMapStage 1)
    ~~~

    遗留一个ShuffleMapStage 1

    ~~~txt
    org.apache.spark.scheduler.DAGScheduler         - submitStage(ShuffleMapStage 1)
    ~~~

    尝试提交ShuffleMapStage 1

    ~~~txt
    org.apache.spark.scheduler.DAGScheduler         - missing: List(ShuffleMapStage 0)
    ~~~

    遗留一个ShuffleMapStage 0

     ~~~txt
      org.apache.spark.scheduler.DAGScheduler         - submitStage(ShuffleMapStage 0)
     ~~~
    
     尝试提交ShuffleMapStage 0
    
    ~~~txt
    org.apache.spark.scheduler.DAGScheduler         - missing: List()
    ~~~
  
    没有遗留的Stage
  
    ~~~txt
    org.apache.spark.scheduler.DAGScheduler         - Submitting ShuffleMapStage 0 (MapPartitionsRDD[3] at map at WCAnalyzer.scala:24), which has no missing parents
    ~~~
  
    提交ShuffleMapStage 0，该Stage的最后一个RDD是MapPartitionsRDD[3]，是由map算子生成，在代码第24行
  
    ~~~txt
    org.apache.spark.scheduler.DAGScheduler         - submitMissingTasks(ShuffleMapStage 0)
    ~~~
  
    提交Tasks，一个Stage就是一个Task Set集合
  
    ~~~txt
    org.apache.spark.scheduler.TaskSchedulerImpl    - Adding task set 0.0 with 1 tasks
    ~~~
    TaskSchedulerImpl 调度器添加一个Task Set集合
  
    ~~~txt
    org.apache.spark.scheduler.TaskSetManager       - Starting task 0.0 in stage 0.0 (TID 0, localhost, executor driver, partition 0, PROCESS_LOCAL, 7909 bytes)
    ~~~
  
    TaskSetManager 启动stage 0.0 中的task 0.0（taskid=0.0，host=localhost，executor=driver，partition=0，taskLocality=PROCESS_LOCAL，serializedTask=7909 bytes
  
    ~~~txt
    org.apache.spark.executor.Executor              - Running task 0.0 in stage 0.0 (TID 0)
    ~~~
  
    Executor 端运行task
  
    ~~~txt
    org.apache.spark.executor.Executor              - Finished task 0.0 in stage 0.0 (TID 0). 1159 bytes result sent to driver
    ~~~
  
    Executor 端 运行完成task，将序列化后大小为1159 bytes结果数据发送回driver端
  
    ~~~txt
    org.apache.spark.scheduler.TaskSetManager       - Finished task 0.0 in stage 0.0 (TID 0) in 194 ms on localhost (executor driver) (1/1)
    ~~~
  
    TaskSetManager 运行完task  完成task数量／总攻task数量
  
    ~~~txt
    org.apache.spark.scheduler.TaskSchedulerImpl     - Removed TaskSet 0.0, whose tasks have all completed, from pool 
    ~~~
  
    TaskSchedulerImpl 移除TaskSet 集合
  
    ~~~txt
    org.apache.spark.scheduler.DAGScheduler          - ShuffleMapTask finished on driver
    ~~~
  
    DAGScheduler 完成ShuffleMapTask 的计算
  
    ~~~txt
    org.apache.spark.scheduler.DAGScheduler          - ShuffleMapStage 0 (map at WCAnalyzer.scala:24) finished in 0.289 s
    ~~~
  
    DAGScheduler 完成ShuffleMapStage 的计算，用时共 0.289 s

    ~~~txt
    org.apache.spark.scheduler.DAGScheduler          - looking for newly runnable stages
    org.apache.spark.scheduler.DAGScheduler          - running: Set()
    org.apache.spark.scheduler.DAGScheduler          - waiting: Set(ShuffleMapStage 1, ResultStage 2)
    org.apache.spark.scheduler.DAGScheduler          - failed: Set()
    org.apache.spark.MapOutputTrackerMaster          - Increasing epoch to 1
    org.apache.spark.scheduler.DAGScheduler          - Checking if any dependencies of ShuffleMapStage 0 are now runnable
    org.apache.spark.scheduler.DAGScheduler          - running: Set()
    org.apache.spark.scheduler.DAGScheduler          - waiting: Set(ShuffleMapStage 1, ResultStage 2)
    org.apache.spark.scheduler.DAGScheduler          - failed: Set()
    ~~~
    
    Stage在计算完后，DAGScheduler会查询是否还有未完成的计算，直到有新的Stage提交
    
    ~~~txt
    ============================   ShuffleMapStage 1 的提交计算过程  ==========================
    org.apache.spark.scheduler.DAGScheduler          - submitStage(ShuffleMapStage 1)
    org.apache.spark.scheduler.DAGScheduler          - missing: List()
    org.apache.spark.scheduler.DAGScheduler          - Submitting ShuffleMapStage 1 (MapPartitionsRDD[5] at map at WCAnalyzer.scala:28), which has no missing parents
    org.apache.spark.scheduler.DAGScheduler          - submitMissingTasks(ShuffleMapStage 1)
    org.apache.spark.scheduler.TaskSchedulerImpl     - Adding task set 1.0 with 1 tasks
    org.apache.spark.scheduler.TaskSetManager        - Starting task 0.0 in stage 1.0 (TID 1, localhost, executor driver, partition 0, ANY, 7638 bytes)
    org.apache.spark.executor.Executor               - Running task 0.0 in stage 1.0 (TID 1)
    org.apache.spark.executor.Executor               - Finished task 0.0 in stage 1.0 (TID 1). 1331 bytes result sent to driver
    org.apache.spark.scheduler.TaskSetManager        - Finished task 0.0 in stage 1.0 (TID 1) in 102 ms on localhost (executor driver) (1/1)
    org.apache.spark.scheduler.TaskSchedulerImpl     - Removed TaskSet 1.0, whose tasks have all completed, from pool 
    org.apache.spark.scheduler.DAGScheduler          - ShuffleMapTask finished on driver
    org.apache.spark.scheduler.DAGScheduler          - ShuffleMapStage 1 (map at WCAnalyzer.scala:28) finished in 0.117 s
    ============================   ResultStage 2 的提交计算过程  =============================
    org.apache.spark.scheduler.DAGScheduler          - looking for newly runnable stages
    org.apache.spark.scheduler.DAGScheduler          - running: Set()
    org.apache.spark.scheduler.DAGScheduler          - waiting: Set(ResultStage 2)
    org.apache.spark.scheduler.DAGScheduler          - failed: Set()
    org.apache.spark.MapOutputTrackerMaster          - Increasing epoch to 2
    org.apache.spark.scheduler.DAGScheduler          - Checking if any dependencies of ShuffleMapStage 1 are now runnable
    org.apache.spark.scheduler.DAGScheduler          - running: Set()
    org.apache.spark.scheduler.DAGScheduler          - waiting: Set(ResultStage 2)
    org.apache.spark.scheduler.DAGScheduler          - failed: Set()
    org.apache.spark.scheduler.DAGScheduler          - submitStage(ResultStage 2)
    org.apache.spark.scheduler.DAGScheduler          - missing: List()
    org.apache.spark.scheduler.DAGScheduler          - Submitting ResultStage 2 (MapPartitionsRDD[7] at map at WCAnalyzer.scala:32), which has no missing parents
    org.apache.spark.scheduler.DAGScheduler          - submitMissingTasks(ResultStage 2)
    org.apache.spark.scheduler.TaskSchedulerImpl     - Adding task set 2.0 with 1 tasks
    org.apache.spark.scheduler.TaskSetManager        - Starting task 0.0 in stage 2.0 (TID 2, localhost, executor driver, partition 0, ANY, 7649 bytes)
    org.apache.spark.executor.Executor               - Running task 0.0 in stage 2.0 (TID 2)
    org.apache.spark.executor.Executor               - Finished task 0.0 in stage 2.0 (TID 2). 1387 bytes result sent to driver
    org.apache.spark.scheduler.TaskSetManager        - Finished task 0.0 in stage 2.0 (TID 2) in 44 ms on localhost (executor driver) (1/1)
    org.apache.spark.scheduler.TaskSchedulerImpl     - Removed TaskSet 2.0, whose tasks have all completed, from pool 
    org.apache.spark.scheduler.DAGScheduler          - ResultStage 2 (collect at WCAnalyzer.scala:34) finished in 0.057 s
    ~~~
    
    以上是ShuffleMapStage 1和ResultStage 2的提交计算过程，与ShuffleMapStage 0一样，不再赘述
    
    ~~~txt
    org.apache.spark.scheduler.DAGScheduler         - Job 0 finished: collect at WCAnalyzer.scala:34, took 0.770898 s
    ~~~
    
    DAGScheduler 当所有的Stage 提交计算完成 结束Job




