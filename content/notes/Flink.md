---
title: Flink
---

# Flink

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

```
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





## 其他

两个流join 必须有等值字段必须都在同一个窗口里面

duplicate key update  mysql数据库的更新插入 合为一条sql

并行度： **算子级别** > **env级别** > **Client级别** > **系统默认级别**

在所有Task共享资源槽点名字相同，默认情况下 （pipline）
同一个job的同一个Task中的多个subTask不能在同一个slot槽中

>具有并行度的subtask 不能在一个slot槽中
对于同一个job，不同Task【阶段】的subTask可以在同一个资源槽中


