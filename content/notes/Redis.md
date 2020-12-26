---
title: Redis
date: 2019-12-26 23:50:21
---

# Redis（Remote Dictionary Server）



## Redis 运行快速的原因

1. 完全基于内存操作

2. 数据结构简单，数据操作也简单

3. 使用多路I/O复用模型

## 数据类型
### String
#### 单条操作
1. 增：set	key	value
2. 查：get	key
3. 删：del	key

#### 多条操作
1. 增：mset	key	value	[key1	value1]
2. 查：mget	key	[key1]

#### 其他命令
1. strlen	key	//获取字符串长度
2. append	key	value	//有则追加，无则新建
3. setnx	key	value	//不存在就设置，存在就不设置
4. incr	key	//自增 1
5. incrby	key	num  //给key的值增加num（int 类型），num 正数则为加，num 为负数 则为减
6. incrbyfloat	key	num	//给key的值增加num（float 类型）
7. decr	key  //自减 1
8. decrby	key	num	//给key的值减num
9. setex	key	second	value	//设置key的值为value存活时间为second秒
10. psetex	key	millisecond	value	//设置key的值为value存活时间为millisecond毫秒

>注：
>字符串值最大值为512m

### Hash
和字符串相似，可理解为字符串厘米套字符串

#### 单条操作
1. 增：hset	key	field	value	
2. 查：hget	key	field
3. 删：hdel	key	field

#### 多条操作
1. 增：hmset	key	field	value	[field1	value2]
2. 查：hmget	key	field	[field1]

#### 其他命令
1. hgetall	key	//获取key的全部的值
2. hlen	key	//获取key的值的数量
3. hexists	key	field	//是否存在field
4. hkeys	key	//所有key的字段（field）
5. hvals	key	//所有key的值 
6. hincrby	key	field	num	//给key的field的值增加num （num 为int值）
7. hincrybyfloat	key	field	num	//给key的field的值增加num （num 为float）
8. hsetnx	key	field	value	//存在不设置，不存在设置

>注：
>1. Hash类型的value只能存字符串，不允许再嵌套其他类型，如果数据为空为Nil
>2. 每个Hash可以存储$2^{32}-1$个键的值对
>3. Hash类型十分贴近对象的数据存储，并且可以灵活添加、删除对象属性。但Hash类型设计并不是存在量而设计的，切记不可滥用，更不可将Hash作为对象列表使用
>4. hgetall操作可以获取全部属性，如果内部field过多，遍历整体数据时效率会降低，很有可能成为数据访问的瓶颈

### String存储对象（Json） VS Hash存储对象
1. String存在对象讲究整体性，以读为主
2. Hash存储对象讲究分散性，以写为主

### List
讲究顺序
#### 添加
1. lpush	key	value	[value1]	//从左添加
2. rpush	key	value	[value1]	//从右添加

#### 获取
1. lrange	key	start	stop
2. lindex	key	index
3. llen	key

#### 获取并移除
1. lpop	key	//从左出
2. rpop	key	//从右出

~~~tex
lpush	list	a	b	c
结果：c b a
===============================
rpush list a	b	c
结果：a b c
===============================
list=[a	b	c]
lpop list
结果：b c
===============================
list=[a b c]
rpop list
结果：a b
===============================
~~~

>注：
>1. list中保存的数据都是String，数据总量是有限的，最多$2^{32}-1$个元素
>2. list具有索引的概念，但是操作数据时通常以队列的形式进行入队出队操作（或以栈的形式进行入栈出栈操作）
>3. 当stop的值为-1时，获取的是全部数据
>4. list对数据进行分页操作，通常第一页的数据使的信息来自list，其他页面的数据通过数据形式进行加载

### Set
#### 命令
1. 增：sadd	key	member	[member1]
2. 查：smembers	key
3. 删：srem	key	member	[member1]
4. 获取总量：scard	key
5. 判定是否存在：sismember	key	member
6. 随机获取（原集合保留）：srandmember	key	[count]
7. 随机获取（原集合不保留）：spop	key
8. 集合交集：sinter	key	key1	key2
9. 集合并集：sunion	key	key1	key2
10. 集合差集：sdiff	key	key1	key2
11. 存储集合交集：sinterstore	destination	key	key1	key2
12. 存储集合并集：sunionstore	destination	key	key1	key2
13. 存储集合差集：sdiffstore	destination	key	key1	key2
14. 集合元素移动：smove	source	destination	member

### Sorted_Set
#### 命令
1. 增：zadd	key	score	member	[score1	member1]
2. 删：zrem	key	member	[member1]
3. 获取全部（正序）：zrange	key	start	stop	[withscores]
4. 获取全部（倒序）：zrevrange	key	start	stop	[withscores]
5. 按条件查（正序）：zrangebyscore	key	min	max	[withscore	limit]
6. 按条件查（倒序）：zrevrangebyscore	key	max	min	[withscore	limit]
7. 按条件删除（索引）：zremrangebyrank	key	start	stop
8. 按条件删除（积分）：zremrangebyscore	key	min	max
9. 获取集合总量：zcard	key	|	zcount	key	min	max
10. 存储集合交集：	zinterstore	destination	numkeys	key	key1
11. 存储集合并集：zunionstore	destination	numkeys	key	key1
12. 获取索引（正序）：zrank	key	member
13. 获取索引（倒序）：zrevrank	key	member
14. score值获取：zscore	key	member
15. score值修改：zincrby	key	num	member

## Key的操作
### 命令
1. 删除：del	key
2. 判断是否存在：exists	key
3. 获取key类型：type	key
4. 指定有效期：
	1.	expire	key	seconds
	2.	pexpire	key	milliseconds
	3.	expireat	key	timestamp
	4.	pexpireat	key	milliseconds-timestamp
5. 获取有效期：
	1.	ttl	key
	2.	pttl	key
6.	设置永久：persist	key
7.	查询key：key	pattern	//*,?,[]
8.	重命名：rename	key	newkey	|	renamenx	key	newkey
9.	对key排序：sort

## 数据库操作
### 命令
1. 选择数据库：select	index
2. 数据移动：move	key	db
3. 数据库大小：dbsize
4. 数据清除：
	1. 单库删除：flushdb
	2. 多库删除：flushall

## 持久化
### RDB（Relational Database）
#### 保存数据
1. 指令（前台）：save	//阻塞 立即保存
2. 指令（后台）：bgsave	//不立即执行
3. 配置：save	second	changes	//用bgsave执行操作

#### 常用配置项
1. 数据文件名称：dbfilename	dump.rdb	//默认
2. 数据保存路径：dir
3. 是否开启压缩：rdbcompression	yes	//默认
4. 是否开启格式检查：rdbchecksum	yes|no //默认no

#### RDB 持久化优点
1. RDB是一个紧凑压缩的二进制文件，存储效率高
2. RDB存储的是Redis在某个时间点的数据快照，非常适用于数据备份全量复制等场景
3. RDB恢复数据速度比AOF快

#### RDB应用
服务器中每X小时执行bgsave备份，并将RDB文件拷贝到远程机器中，用于灾难恢复

#### RDB持久化缺点
1. RDB方式无论是执行命令还是进行配置，无法做到实时持久化，具有较大可能丢失数据
2. bgsave每次运行要执行fork操作创建子进程，要牺牲一些性能
3. Redis的众多版本中未进行RDB文件格式的版本统一，有可能出现各个版本服务器之间数据格式无法兼容
4. 存储数量较大时，效率较低
5. 大数据量下的I／O性能较低
6. 基于fork创建子进程，内存产生额外消耗
7. 宕机带来的数据丢失风险

### AOF（Append Only File）
#### 保存数据策略
每次：always
每秒：everysec
系统控制：no

#### 配置
1. 是否开启：appendonly	yes|no	//默认no
2. 保存策略：appendfsync	always|everysec|no
3. 重写：
	手动：bgrewriteaof
	自动：
		auto-aof-rewrite-min-size	size
		auto-aof-rewrite-percentage	percentage

### RDB 与 AOF 如何选择
对数据非常敏感，建议使用默认的AOF持久化方案
	AOF策略使用everysec，每秒fsync一次，该策略仍可保持很好性能，出现问题最多丢失一秒内的数据
数据呈现阶段有效性，建议使用RDB持久化方案
	数据可以做到阶段内无丢失，且恢复较快，阶段点数据恢复通常使用RDB方案
>注意：
>AOF文件存储体积较大，恢复速度较慢
>利用RDB使用线紧凑的数据持久化会使Redis性能降低

综合：
1. RDB与AOF选择实际上是在一种权衡，每种都有利有弊
2. 如果不能承受分钟内的数据丢失，对业务数据非常敏感，选用AOF
3. 如果能承受分钟内的数据丢失，且追求大数据集的恢复速度选用RDB
4. 灾难恢复选用RDB
5. 双保险策略，同时开启RDB和AOF，重启后Redis优先使用AOF来恢复数据，降低丢失数据量

## 事务
Redis事务不具有回滚机制

### 命令
1. 开启：multi
2. 结束：exec
3. 中断：discard

### 事务中的错误
#### 命令错误
执行事务过程中输入的命令出现错误，Redis会结束事务不再执行，并报出错误的命令

#### 操作错误
执行事务过程中输入的命令正确，其他操作错误，事务中其他命令正常执行，错误操作报错

## 锁
Redis中锁和事务是相搭配使用的，可解决对key的监控

### 命令
加锁：watch	key	[key1]
解锁：unwatch  //取消掉所有key的监控

## 数据删除策略
当key过期后执行数据删除的策略
### 定时删除（即可删除）
创建一个具有时效性的key时，同时会创建一个定时器来监控该key是否过期，当key过期后立即进行key的删除

优点：节约内存，到时就会进行删除，快速释放占用空间
缺点：CPU压力大影响Redis响应时间和吞吐量
总结：用处理器性能换取存储空间

### 惰性删除
当一个具有实效性的key过期后不会有删除操作，直到下一次调用时会先检查该key是否过期，如果过期则进行删除操作，并返回nil（该key不存在）

优点：节约CPU性能，发现必须删除的时候才会删除
缺点：内存压力大，出现长期占用内存的数据
总结：用存储空间换取处理器性能

### 定期删除
Redis会根据设置的参数，定期对具有时效性的key进行清理工作，它是定时删除和惰性删除的结合者，既不像定时删除会立即进行删除给予CPU压力，也不会像惰性删除给予内存压力

#### 步骤
1. Redis启动服务器初始化时，读取配置server.hz的值（默认为10）
2. 每秒钟执行server.hz次serverCron（）服务（serverCron【服务器级别】->databasesCron【数据库级别】->activeExpireCyle【活跃数据级别】）
3. acitveExpireCyle（）对每个expires[*]（数据库）逐一进行检测，每次执行250ms／server.hz
4. 对某个expires[*]检测时，随机挑选w个key进行检测
	1. 如果key超时，删除key
	2. 如果一轮中删除的kye的数量>w*25%，循环该过程
	3. 如果一轮中删除的可以的数量<=w*25%检查下一个expires[*],0-15（所有的数据库）循环
>W=ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP
>参数current_db用于记录activeExpireCyle进入哪个expires[*]执行

## 数据淘汰策略
当内存到达最大内存限制时进行的数据淘汰策略

数据驱逐淘汰策略配置依据，使用info命令输出相关监控信息，查新缓存hit 命中次数和miss的次数，根据业务调优
### 配置
1. 最大可用内存：maxmemory	//默认为0，一般设置全部内存50%以上
2. 每次选取带删除数据个数：maxmemory-samples	//采用随机获取方式
3. 删除策略：maxmemory-policy	//达到最大内存后，对被选取带数据进行的删除策略

### 检测易失数据集（可能会过期数据server.db[i].expires）
1. volatile-lru：挑选最近最少使用的数据淘汰（最近数据中使用时间离当前最远的数据）。**常用**
2. volatile-lfu：挑选最近使用次数最少的数据淘汰（最近数据中使用次数最少的数据）
3. volatile-ttl：挑选将要过期数据淘汰
4. volatile-random：任意挑选数据淘汰

>ttl：time to live
>lru：least recently	used
>lfu：least frequently	used

### 检测全库数据（所有数据集server.db[i].dict）
1. allkeys-lru：挑选最近最少使用的数据淘汰
2. allkeys-lfu：挑选最近使用次数最少的数据淘汰
3. allkeys-random：任意挑选数据淘汰

### 放弃数据驱逐
no-enviction	//禁止驱逐数据
4.0中默认策略，会引发OOM

## 服务器基本配置
1. 设置服务器守护进程方式：daemonize	yes|no
2. 绑定地址：bing	127.0.0.1
3. 设置服务器端口：port	6379
4. 设置数据库数量：databases	16
5. 设置服务器日志级别：loglevel	debug|verbose|notice|warning
6. 日志文件名称：logfile	端口号.log
7. 设置客户端最大连接数：maxclients	0
8. 客户端闲置最大等待时长：timeout	0

## 高级数据类型
### Bitmaps
标记统计

#### 命令
1. 获取：getbit	key	offset
2. 设置：setbit	key	offset	value	// 0 或 1
3. 交、并、或异
	bitop	op	destkey	key1	key2
	op：
		交：and
		并：or
		非：not
		异或：xor
4. 统计指定key中1的数量：bitcount	key	[start	end]

### HyperLoglog
基数统计
#### 命令
1. 添加：pfadd	key	element	[element1]
2. 统计：pfcount	key	[key1]
3. 合并：pfmerge	destkey	sourcekey	[sourcekey1]

### GEO
距离计算（只计算水平距离）

#### 命令
1. 添加：geoadd	key	longitude	latitude	member	[longitude1	latitude1	member1]
2. 获取：geopos	key	member	[member1]
3. 计算距离：geodist	key	member1	member2	[count]
4. 根据坐标求范围内数据：georadius	key	longitude	latitude	radius	m|km|ft|mi
5. 根据点求范围内的数据：georadusbymember	key	member	radius	m|km|ft|mi
6. 获取指定点对应的hash值：geohash	key	member	[member1]	

## 主从复制
### 创建链接
+ 方式一：客户端发指令：slaveof	masterip masterport
+	方式二：参数启动：redis-server	--slaveof	masterip masterport
+	方式三：服务器配置：slaveof	masterip masterport

### 数据同步 
#### 全量复制
从：发送指令（psync2）
主：执行bgsave
主：第一个slave链接时，创建命令缓冲区
主：生成RDB文件，通过socket发送给slave
从：接收RDB文件，清空自己数据，执行RDB文件恢复过程

#### 部分复制
从：发送命令告知RDB恢复完成
主：发送复制缓冲区信息
从：接收信息，执行bgsavewriteaof，恢复数据

## 哨兵模式
### 配置
sentinel.conf

启动：redis-sentinel

## 集群（cluster）
### 配置
1. 开启：cluster-enabled	yes|no
2. 配置文件名称：cluster-config-file	filename
3. 节点超时时间：cluster-node-timeout	milliseconds
4. master链接slave最小数：cluster-migration-barrier	count

### 命令
1. 查看节点信息：cluster nodes
2. 从一个节点Redis，切换其主节点：cluster	replicate	masterip
3. 新增主节点：cluster meet	ip:port
4. 忽略一个节点：cluster	foeget	id
5. 手动故障转移：cluster	failover

