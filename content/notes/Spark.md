---
title: Spark
---

# Spark 编码

## 1. map 和 mapPartitions

~~~
map是对rdd中的每一个元素进行操作；
mapPartitions则是对rdd中的每个分区的迭代器进行操作
MapPartitions的优点：
如果是普通的map，比如一个partition中有1万条数据。ok，那么你的function要执行和计算1万次。
使用MapPartitions操作之后，一个task仅仅会执行一次function，function一次接收所有
的partition数据。只要执行一次就可以了，性能比较高。如果在map过程中需要频繁创建额外的对象(例如将rdd中的数据通过jdbc写入数据库,map需要为每个元素创建一个链接而mapPartition为每个partition创建一个链接),则mapPartitions效率比map高的多。
SparkSql或DataFrame默认会对程序进行mapPartition的优化。
MapPartitions的缺点：
如果是普通的map操作，一次function的执行就处理一条数据；那么如果内存不够用的情况下， 比如处理了1千条数据了，那么这个时候内存不够了，那么就可以将已经处理完的1千条数据从内存里面垃圾回收掉，或者用其他方法，腾出空间来吧。
所以说普通的map操作通常不会导致内存的OOM异常。 

但是MapPartitions操作，对于大量数据来说，比如甚至一个partition，100万数据，
一次传入一个function以后，那么可能一下子内存不够，但是又没有办法去腾出内存空间来，可能就OOM，内存溢出。

~~~

## 2. Drive 和 Executo

~~~
所有RDD算子的计算功能都是由Excutor执行
~~~

## 3. Shuffle

~~~
将RDD中一个分区的数据打乱重组到其他不同分区的操作
~~~

## 4. Task 和 Partition

~~~
一个分区划分一个任务，一个任务会被分配到一个excutor
~~~

## 5. reduceByKey 和groupByKey

~~~
reduceByKey 在shuffle之前有combine(预聚合)操作，性能相对groupByKey要好
~~~

## 6. stage 划分

~~~
stage划分根据宽依赖，stage个数=1+shuffle个数
~~~

## 7. 更新map

```scala
/**
 *简化if else 结构
 **/
if(map.contains(v))
	map+=(v->0)
map.update(v,map(v)+1) //if外边
```

## 8. 数组切片

```scala
val l=List(1,2,3,4,5,6,7)
val sl=l.slice(0,l.size-1) //[1,2,3,4,5,6]
val zl=sl.zip(sl.tail) //[(1,2),(2,3),...]
val zl.map((x,y)=>x+"_"+y) //[1_2,2_3,...]
```

