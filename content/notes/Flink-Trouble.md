---
title: Flink-Trouble
---

# Flink-Trouble

## 1. 处理时间窗口数据时未添加窗口或者设置处理的时间类型
### 错误现象
```
Caused by: java.lang.RuntimeException: Record has Long.MIN_VALUE timestamp (= no timestamp marker). Is the time characteristic set to 'ProcessingTime', or did you forget to call 'DataStream.assignTimestampsAndWatermarks(...)'?
```
### 解决方案
+ assignTimestampsAndWatermarks
+ env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

## 2. 程序重写(删除算子)导致使用savepoint数据快照无法恢复启动
### 错误现象
```
Failed to rollback to checkpoint/savepoint file:/Users/bulianwei/workspace/project/idea/flink_demo/data/test/savepoint-18c72f-803ce50127dc. Cannot map checkpoint/savepoint state for operator c27dcf7b54ef6bfd6cff02ca8870b681 to the new program, because the operator is not available in the new program. If you want to allow to skip this, you can set the --allowNonRestoredState option on the CLI.
```
### 解决方案
启动时使用 --allowNonRestoredState 或者 -n 跳过删除算子恢复
flink run -s savepointdir -n -c mainclasspath jarpath

## 3. 使用flink stop 停止job时出现无法停止现象
### 错误现象
```
[org.apache.flink.runtime.rest.handler.RestHandlerException: Config key [state.savepoints.dir] is not set. Property [targetDirectory] must be provided.
```
### 解决方案
使用flink stop jobid停止job时会保存数据快照，如果未设置savepointpath会导致无法保存快照，会导致无法停止job
flink stop jobid -p savepointpath

## 4.
### 错误现象
```

```
### 解决方案