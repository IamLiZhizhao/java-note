
#### 名词解析：
系统表空间就是用来放系统信息的，比如数据字典什么的，对应的磁盘文件是ibdata1,

数据表空间就是一个个的表数据文件，对应的磁盘文件就是 表名.ibd


buffer pool：缓存池，（在内存中开辟的一整块空间，由引擎利用一些命中算法和淘汰算法负责维护和管理），
change buffer：在缓存池的基础上更进一步，把在内存中更新就能可以立即返回执行结果并且满足一致性约束（显式或隐式定义的约束条件）的记录也暂时放在缓存池中，这样大大减少了磁盘IO操作的几率


视图
在 MySQL 里，有两个“视图”的概念：
一个是 view。它是一个用查询语句定义的虚拟表，在调用的时候执行查询语句并生成结果。创建视图的语法是 create view … ，而它的查询方法与表一样。
另一个是 InnoDB 在实现 MVCC 时用到的一致性读视图，即 consistent read view，用于支持 RC（Read Committed，读提交）和 RR（Repeatable Read，可重复读）隔离级别的实现。
它没有物理结构，作用是事务执行期间用来定义“我能看到什么数据”。



WAL，全称是Write-Ahead Logging， 预写日志系统。指的是 MySQL 的写操作并不是立刻更新到磁盘上，而是先记录在日志上，然后在合适的时间再更新到磁盘上。
这样的好处是错开高峰期。

日志主要分为 undo log、redo log、binlog。
作用分别是 " 完成MVCC从而实现 MySQL 的隔离级别 "、
" 降低随机写的性能消耗（转成顺序写），同时防止写操作因为宕机而丢失 "、
" 写操作的备份，保证主从一致 "。


LSN：
日志逻辑序列号（log sequence number，LSN）的概念。LSN 是单调递增的，用来对应 redo log 的一个个写入点。每次写入长度为 length 的 redo log， LSN 的值就会加上 length。
LSN 也会写到 InnoDB 的数据页中，来确保数据页不会被多次执行重复的 redo log。

GTID：
GTID 的全称是 Global Transaction Identifier，也就是全局事务 ID