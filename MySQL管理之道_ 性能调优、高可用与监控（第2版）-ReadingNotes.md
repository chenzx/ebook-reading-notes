# MySQL管理之道_ 性能调优、高可用与监控（第2版）-读书笔记

## MariaDB架构与历史
* MariaDB：用Percona XtraDB代替InnoDB（前者是后者的增强版，完全兼容后者）
	* 新功能：多源复制、基于表的并行复制
	* 集成更多存储引擎：Aria（增强版MyISAM）、SphinxSE、TokuDB（可看作ARCHIVE的升级版）、Cassandra、CONNECT、SEQUENCE及Spider（分库分表）
	* 5.5: 线程池技术，减少连接建立的开销，适合于高并发短连接应用场景，如秒杀（？）
* 线程池、审计日志等功能在MySQL企业版中，需要付费

## MySQL 5.7与MariaDB 10.1新特性
* p14 OLTP只读模式下，MySQL 5.7有近100w的QPS，比5.6性能高3倍；OLTP读写模式下，近60w TPS
	* 注：官方测试的硬件配置：E7-8890 v3（72核）、512GB内存（靠）
* 5.7 InnoDB存储引擎的提升：
	* 更改索引名字不锁表
	* 在线DDL修改VARCHAR属性不锁表（不过还是建议使用pt-online-schema-change？）
	* 支持中文全文索引
	* Buffer Pool预热改进
	* undo log回滚日志支持在线收缩
	* share.ibd通用表空间
	* innodb_print_all_deadlocks = 1
	* 支持InnoDB只读事务
	* 支持InnoDB表空间数据碎片整理（Facebook contrib）
	* 支持虚拟列（函数索引）：`mod_id int(11) generated always as (id % 10) virtual`, 插入时使用字面量`default`
	* 支持一张表多个INSERT／UPDATE／DELETE触发器
	* 引入线程池（阿里巴巴开源的druid连接池？？？）
	* 支持explain update／delete（数据更新语句里面也可以使用join？）
	* p57 MariaDB／TokuDB：为什么要关闭Transparent Huge Page？？
	* 优化器改进（SQL查询优化与编译器后端优化技术是有共通之处的）
		* 子查询采用半连接优化（MySQL的子查询一向支持不好，这方面没有Oracle做的好，应用开发人员更喜欢写子查询？？？）
			* update／delete仍然不行
		* 优化派生子查询（谓词过滤条件下推）
		* 派生表索引优化（略）
		* 优化排序limit
		* 优化IN条件表达式
		* 优化union all（不创建临时表）
		* 支持索引下推
		* 支持Multi Range Read（MRR）优化，收集主键并排序，把磁盘随机IO改成顺序IO
		* 支持Batched Key Access（BKA）索引优化，同样是把随机磁盘IO转换为顺序IO
		* MariaDB 10.1: 支持Hash Join索引优化（这个地方很容易让我想起Impala里的2种分布式join处理了）
			* Block Nested Loop Hash
			* Block Index Hash Join Batch Key Access Hash
	* 半同步复制改进
		* 《MySQL运维内参》里有讲到，从略
	* GTID复制改进
		* p96 从库上执行操作时，切记先关闭binlog，再执行DML／DDL
	* 5.7从库多线程复制：基于binlog组提交（《MySQL运维内参》里有讲到，多个处于prepare的事务可同时提交，略）
	* slave支持多源（多个主库）复制

## 故障诊断
* innodb_buffer_pool_size：可设置为60%~80%的内存，甚至可将数据库全部放入内存（要是机房突然掉电怎么办？）
* p107 磁盘技术：比较火的Fusion-io？？？
* p109 Linux服务器性能监控：dstat？
* p125 如果是RR默认隔离级别，建议设置binlog_format=ROW。如果是Read-Commited，则ROW与MIXED效果是一样的（...）
* p130 误删除ibdata数据文件，怎么办？（数据库仍然开着，不要杀死mysqld进程，文件内容可从/proc/<pid>/fd恢复）
* p132 update忘加where过滤条件误操作恢复（模拟Oracle闪回）：实质就是从binlog中解析出原来的数据，再人工undo... 矬
	* 为什么不能直接rollback呢？？见鬼

## 同步复制报错故障处理
* p148 master上更新了一条记录，slave上却找不到（仍然是手工分析处理binlog，疯了）
* slave意外宕机，可能损坏relay-log：找到binlog和POS点，重新同步（5.5 my.cnf：relay_log_recovery=1）
* ? 多台slave上server-id重复（由于直接复制master点my.cnf到slave导致）
* 避免master上执行大事务（老生常谈了）
	* 利用存储过程，把大事务转换为小批量操作....
* p156 binlog_ignore_db引起的同步复制故障：使用replicate-ignore-db=<yourdb>代替

## 性能调优
* 数据类型：选择够用的就行
	* timestamp：默认随行更新而更新？？？（这个特性我怎么之前从来没注意过？不过我之前实际项目开发也只用到4.1/5.0）
	* 5.6: year(2)自动转换为year(4)
	* varchar(5)升级到varchar(10)的底层磁盘存储不变，但decimal(10,1)升级到decimal(10,2)就不行了——后者应该是底层位存储模式不一致
* p192 5.6 online DDL
* 采用合适的锁机制
	* 表锁
	* 行锁
		* p197 只有通过索引条件检索数据，InnoDB才会使用行锁，否则使用表锁
	* 页面锁（粒度介于前两者之间，NDB？）
* 选择合适的事务隔离级别
	* innodb_flush_log_at_trx_commit=0(每隔1秒刷盘)/1（每次事务提交刷盘）/2（仅刷到日志文件）
		* p204 事务提交后，先刷binlog，再刷到redo log,... => 中间发生宕机，可导致主从数据不一致
	* p208 间隙锁：主要是防止幻读。RR隔离级别下，当对数据进行条件／范围检索时，对其范围内也许并不存在的值加锁。RC级别下，没有间隙锁。
* SQL优化与合理利用索引
	* p212 error 1093: 通过子查询删除已查询的记录会报错，可更改子查询引入临时表（更像是MySQL底层实现的bug？？？）
	* p216 类似select count(*)千万不要在主库上执行，因为InnoDB无表级计数器，需要全表扫描一次才能得到汇总
	* p219 避免使用having子句，用where限制记录的数目
	* p222 联合索引要遵循最左原则（这个太不智能了！）
	* p225 当取出数据超过全表的20%，优化器就不会使用索引了；
		* 一条SQL只能有一个索引，如果有多个，优化器选择最优的（！）
		* order by后如果有多个字段排序，顺序要一致，如果一个生序一个降序，会出现Using filesort（性能差）

## 备份与恢复
* p259 取代mysqldump的新工具：mydumper
* p263 热备份与恢复
	* xtrabackuo／innobackupex（后者是前者的perl脚本封装）

## 高可用MHA架构集群管理
* MHA与MMM都是采用Perl编写的（？）

## MySQL架构演进：“一主多从、读写分离”
* p294 HAProxy提供了高可用、负载均衡以及基于TCP／HTTP的应用代理。
* p295 MySQL Proxy一直没有发布GA版本，不能用在生产系统里？？？？
	* MaxScale的GA版本？
	* 作者这里推荐OneProxy，但是OneProxy为什么不支持预编译语句？怎么防止SQL拼接的注入攻击？？见鬼
		* CSDN上曾经有篇文章谈到基于proxy的LB解决方案，技术核心在于SQL语句的语法解析... 另，淘宝也有过类似应用

## Codership Galera Cluster集群架构搭建与管理
* HAProxy结合Galera Cluster实现无单点秒级故障转移*

## OneProxy分库分表的搭建与管理
* 数据访问层TDDL／ZDAL是个什么鬼？
* 注意：分库分表不支持：跨库join、分布式事务XA、存储过程

## Lepus慢日志分析平台搭建与维护

