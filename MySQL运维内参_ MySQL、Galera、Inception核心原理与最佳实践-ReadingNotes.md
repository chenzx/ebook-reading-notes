# MySQL运维内参_ MySQL、Galera、Inception核心原理与最佳实践

点评：讲解MySQL核心存储源码实现原理的部分不细致，MVCC则未涉及。曾经在O'Reilly的高可用MySQL里讲到的MMM、MHA则已经被5.7官方的GTID和Galera Cluster方案代替了？

* InnoDB不是Monty（MySQL作者）开发的？

## MySQL源代码入门
* 代码目录：libmysqld、sql、storage
* 5.7: 自动为root生成临时密码？--initialize-insecure

## MySQL启动过程
基于简化后的代码讲解。不过代码本身其实说明不了什么问题，略。

## 连接的生命与使命
继续前面的代码讲解风格。

* 每个会话对应一个线程。max_connections
	* 这样看来，InnoDB要支持ACID事务特性，其并发写（TPS）不可能很高。虽然并发查询（QPS）可以很高。
* sql_yacc.yy
	* INSERT INTO的INTO关键字可省略

## MySQL表对象缓存
* 精简的代码片段：... open_table_from_share
* p41 2部分：
	1. SHARE的缓存 ？？
	2. SHARE结构被实例化之后的TABLE实例对象的缓存

## InnoDB初探
* 源码目录：
	* fsp=fs physical？
	* ibuf=insert buffer，新的名字叫做change buffer，这里很容易想到OcceanBase...
	* mtr=mini transaction（MySQL的MVCC事务实现还是没有Oracle的强大，譬如，不宜大事务长时间加锁）
	* que：InnoDB内部实现的一个状态机执行器
	* trx：事务实现，MVCC、回滚段、purge、回滚记录及回滚提交
* mysql系统表是MyISAM存储引擎的表？
	* information_schema: `show create table <tbl-name>`
	* 5.5+ performance_schema
* p54 闩（latch）：物理页面的读写锁？我怎么记得latch相对于Linux内核里的spinlock
* p57 RR（可重复读）隔离级别下，如果一个读事务长期不提交，那么这个事务后面的所有写事务的回滚段都不能释放空间出来
	* 导致ibdata文件撑大
	* 那么Oracle里面是怎么解决这个问题的？？？
	* 回滚段分离出来单独文件存储：innodb_undo_tablespace
* p58 启动时的恢复：先Redo，再Undo（具体不如直接参考Oracle Core那本书～）
* p60 innobase_fast_shutdown：0，1，2
	* 0: 全量的回滚段PURGE、Change Buffer的merge操作、所有日志刷盘、所有Buffer Pool脏数据刷入数据文件
* p63 checkpoint指的就是对LSN环形buffer刷盘的处理

## InnoDB数据字典
* 4个基本的系统表：SYS_TABLES、SYS_COLUMNS、SYS_INDEXES、SYS_FIELDS
* InnoDB用0号表空间0号文件等7号页面来管理字典信息...
* HASH表缓存及LRU链表管理：略
* p74 系统列：Rowid、TRXID、ROLLPTR（应该不支持对于同一行的并发事务吧？）
* InnoDB的库名称是直接存入tablename的 => 想重命名数据库，比较麻烦
* Rowid管理
	* 每分配一个在内存中+1，每256写入一次；重启时会向上256对齐

## InnoDB数据存储结构
* B+树：段、簇（extent）、页面

## InnoDB索引实现原理
* B+树与B树的区别（作者这里讲解似乎有点问题？）那B*树又是什么呢？
* 聚簇索引（primary key）和二级索引
	* p95 唯一索引（unique）导致每次修改都会去检查唯一性，在RR隔离级别下，经常导致死锁
* InnoDB索引的插入过程（略）
* 页面结构管理
	* p117 槽（slot）？一个页面内如何管理多个记录
	* 页面尾部：保存了最新修改的LSN，也是文件头信息中的FIL_PAGE_LSN
	* 页面重组：类似于GC的概念，略

感觉本章作者讲解B+树的物理存储管理机制时，很啰嗦。

## InnoDB记录格式
略

## 揭秘独特的Double Write（重点）
记录的逻辑写转换为物理写，（为保证磁盘IO的事务性，需要写2次？我还是有点疑惑的地方...）

## InnoDB日志管理机制
* Buffer Pool
* Redo Log
	* LSN：MTR写入多少log字节，LSN就增长多少（这个有点意思）
	* p155 说白了，日志的作用，就是把随机写变成顺序写，用一个速度更快的写入保证速度较慢的写入的完整性（！）
	* p156 检查点（lsn_checkpoint_up_to）
	* p160 InnoDB的日志是具有逻辑意义的物理日志
		* Type、(table)spaceid、(page)offset、data ——奇怪，这里没有data的长度？
		* Type：MLOG_UNDO_INSERT？
	* 日志刷盘时机*
* REDO日志恢复（略）
* 数据库回滚
	* 《Oracle Core》里谈到Undo日志其实也是用跟Redo一样的格式实现的，不过这里MySQL是怎么实现的就不知道了

## MySQL 5.7中的sys Schema
## 方便的GTID
* GTID = source_id(server_uuid) : sequence_id
* p205 `CHANGE MASTER TO ...`
* p214 不支持CREATE TABLE ... SELECT语句，原因是binlog是基于行模式的复制...

## 半同步复制
* 先同步，timeout的情况下fallback回异步
	* 主库（master）只需要等待至少一个从库收到并flush binlog到relay log即可。

## 5.7多线程复制原理
* p231 所有已经处于prepare阶段的事务，都是可以并行提交到。

## 大量表导致服务变慢的问题
* 最终结论：HASH算法的问题？？？

## 快速删除大表
* ibd，‘硬链接’，... ?

## 2条不同的插入语句导致的死锁（重点）
* p261 隐式锁：事务1发现自己需要更新的记录被其他事务（2）占有了，但还没有上锁的时候，此时1会帮忙给2上锁...
* GAP锁：为了实现RR隔离级别的，保证数据在插入时不会导致幻读；实现依赖于一个heapno的逻辑编号...
	* 关于heapno

## 并发删除同一行时导致的死锁
NOT_GAP锁？？？将隔离级别从默认的RR改为Read-Commited

## 参数SQL_SALVE_SKIP_COUNTER
## binlog中的时间戳
* p286 对于慢查询，锁等待时间不计算在内？

## InnoDB中Rowid对binlog的影响
## Percona XtraBackup
* p302 基于InnoDB自身的崩溃恢复机制？？？

## MySQL分库分表
* 官方：MySQL Connections + MySQL Router + MySQL Fabric？

## MySQL数据安全
* p334 Group Replication基于Paxos实现？——但还是需要人工管理维护的吧？

## 性能拾遗
* p339 5.7.9+，行存储格式由默认COMPACT改为了DYNAMIC
* p348 MySQL CLuster的存储引擎基于NDB？
* p350 传统磁盘每秒可以完成200次IO，而SSD每秒钟可完成高达60万次；SSD已有单盘12TB的容量了...
* p352 MySQL内核的深度定制：AliSQL、TxSQL
	* AWS Aurora：数据库实例与存储分离？？？腾讯云CDB也有类似的版本

## Group Replication（重点）

## Document Store（JSON）
略

## Galera Cluster的设计与实现
* p441 DDL（行数多的时候，大事务）的执行非常危险 => 使用pt-online-schema-change

## Galera参数解析
* wsrep（写集）

## Galera验证方法
## Galera消息传送
## GCache
* 每个节点把最新的写集缓存起来，在需要的时候，如果被选为Donor节点，可以将缓存的最新增量提供给Joiner

## 大话SST／IST（状态传输）

## Donor／Desynced详解
## Galera并发控制（重点）
## Galera流量控制
## Galera Cluster影响单节点执行效率的因素
## grastate.dat文件
## Galera Cluster从库的转移
## Galera Cluster节点与其从库的随意转换
## 业务更新慢，不是Galera引起的
* “为什么之前的MMM没有问题？” => PXC使用老版本的客户端，其auto_commit默认为False，与新版本默认True不一致

## 在线改表引发的Galera Cluster集群死锁
## Inception诞生记（一个SQL审核软件，但似乎没讲清楚具体的实现原理？）
按照我的理解，就是Type Check，和虚拟执行，以检查有无潜在问题？
## Inception安装与使用
## Inception支持选项（这个内容太无聊了）
## Inception备份回滚
## 审核规范
## 参数变量
## 友好的结果集
## 命令集语句
## Inception的菜单
## Inception设计



