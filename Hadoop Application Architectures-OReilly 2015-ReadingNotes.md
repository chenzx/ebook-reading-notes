# Hadoop Application Architectures-OReilly 2015-ReadingNotes.md

## 数据建模
* p23 HBase基于TTL的旧数据清除（合并到新HFile时跳过）
* p25 为了能够在Hive外部使用metastore，一个新项目HCatalog启动了

## 数据移动
* p32 HBase的扫描速度大约是HDFS的1/10-1/8，访问单个记录的时间为毫秒级别
* p36 Flume拦截器？
* p40 小文件：
	* 使用Solr
	* 使用HBase
	* 使用容器格式，如SequenceFiles或Avro
* 可挂载的HDFS
	* Fuse-DFS（会显著影响性能，模型持续性较差？）
	* NFSv3
* Sqoop：Hadoop与关系数据库的批量传输
	* 可能的瓶颈：数据倾斜：Mapper对主键的划分不均匀
	* 2种方法鉴别更新过的行：
		1. Sequence ID
		2. 时间戳
* Flume架构：数据源 --> 拦截器 --> 选择器 --> Channel --> Sink
* p56 Camus: 从Kafka中批量加载数据

## 数据处理
* MapReduce
* Spark
	* RDD
* 抽象层
	* Pig
	* Crunch
		* 核心：Pipeline对象，done()触发流水线的执行
	* Cascading
* Hive
	* 外部表导入：CREATE EXTERNAL TABLE ... FIELDS TERMINATED BY '|' STORED AS TEXTFILE LOCATION 'foo';
	* 收集统计信息（用于CBO？）：ANALYZE TABLE foo COMPUTE STATISTICS
	* 支持各种不同的分布式JOIN：
		* map关联（hash关联）
		* bucketed join
		* sorted buckted merge join
	* EXPLAIN（查询计划）：用户应该养成习惯，查看Hive究竟在背后做了什么
* Impala
	* DataNode：查询规划器 --> 查询协调器 --> 查询执行器
	* 分布式MPP数据库：（参考本书附录）
		* broadcast hash join（将小表复制到所有大表数据所在的节点上，以hashtable形式加载到内存作过滤）
		* partitioned hash join（先hash分区，再分发，每个节点缓存数据集的一个子集）
	* 与Hive不同，Impala后台服务是长期运行的进程
	* 用LLVM编译查询，将查询用到的方法编译为优化的机器码
	* p102 如果查询需要扫描非常多的数据，节点故障不可以强制要求重启恢复查询，推荐使用Hive

## 通用范式
* 依据主键去重
* windowing分析
	* 注意SQL语句里的OVER关键词
* 基于时间序列的更新
	* 利用HBase的版本特性
	* 使用RecordKey-StartTime作为row key
	* 重写HDFS更新整个表
	* 利用分区分开存储当前记录和历史记录

## 图处理
* BSP模型
* Giraph
* Spark GraphX

## 协调调度
* Airbnb Chronos on Mesos？
* OOzie
	* 工作流范式
		* 点对点
		* 扇出（fork-and-join）
		* 分支决策
	* 调度模式
		* 频率
		* 时间／数据触发

## 近实时处理
* p170 lambda架构
* Storm
	* 在要求“仅处理1次”时，2个选择：（1）事务性拓扑；（2）Trident
* Spark Streaming

## 点击流分析
## 欺诈检测
## 数据仓库
