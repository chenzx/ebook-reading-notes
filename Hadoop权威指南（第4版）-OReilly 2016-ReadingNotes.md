# Hadoop权威指南（第4版）-OReilly 2016-ReadingNotes.md

## 初识Hadoop
## 关于MapReduce
* mapper和reducer
* combiner*

## HDFS
* p48 Hadoop2: HA，备用namenode
* 2种HA共享存储：
	* NFS过滤器
	* QJM（quorum journal manager）
* p73 副本怎么放
	* 由近到远随机放（注意，hadoop不使用DHT机制，靠的是namenode的索引维护）

## YARN
* p83 MapReduce 1 vs YARN
	* RM -> NM（管理监控容器）：YARN vs k8s vs Mesos？

## Hadoop的I/O操作
* p99 deflate／gzip不可切分？？？见鬼（bz2反而可以？）
	* p105 压缩块连续存储，没有特殊标记，无法从数据流任意位置快速定位到下一个块
* 自定义序列化格式：Writable
	* WritableComparator
* 高层次容器封装：SequenceFile
	* MapFile：排序过的SequenceFile，有索引（这让我想起chromium for android里面的pak资源格式）
* 其他：Avro、ORCFile

## MapReduce应用开发
* Configuration API
* MRUnit
	* p159 测试驱动程序：Mini集群？

* 打包作业
* Web界面
* 日志
* 作业调优
* 工作流：JobControl、Oozie

## MapReduce工作机制
* p191 map／reduce任务的JVM会在退出前向其父app master发送错误报告
* p195 shuffle：系统执行排序，将map输出作为输入传给reduce的过程
	* 溢出文件（spill file），combiner
* p202 推测执行
* OutputCommiter *

## MapReduce类型与格式
* 默认情况下，只有1个reducer
* InputFormat
	* 输入分片
* OutputFormat（略）

## MapReduce特性
* 用户自定义计数器
	* p250 动态计数器：enum --> string？
* `排序`是MapReduce的核心技术？
	* p256 通过对键空间进行采样，可较为均匀地划分数据集
	* 辅助排序*
* 连接
	* map端：merge-join？
		* org.apache.hadoop.mapreduce.join.CompositeInputFormat
	* reduce端：hash-join？
* 边数据（side data）分布
	* 分布式缓存（org.apache.hadoop.filecache.DistributedCache）

## 构建Hadoop集群
* 商用机器规格：靠
* Hadoop安全
	* 委托令牌？

## 管理Hadoop
## 关于Avro
## 关于Parquet
* p364 原子类型：int96？？

## Flume
## Sqoop
* p414 BlobRef
* p416 Sqoop会根据目标表的定义生成java类（这个牛）

## Pig
* p432 多查询执行（类似于编译器后端优化中的CSE）
* p452 UDF
* p459 分段复制连接（fragment replicate join）：这不就是Hive里的broadcast hash join嘛

## Hive
* 执行引擎：Tez／Spark的优越性？？
* p482 其他SQL-on-Hadoop：Apache Phoenix：SQL on HBase
* HiveQL
* 分区和桶
* p504 Hive只支持等值连接？
	* RBO -> 0.14+ CBO
* p512 UDAF（聚集函数）

## Crunch
## Spark
## HBase
## ZooKeeper
## 案例学习：医疗公司Cerner的可聚合数据
## 生命数据科学：用软件拯救生命
## Cascading

