
#### 解析：

##### sync_binlog
binlog 写盘状态
可以看到，每个线程有自己 binlog cache，但是共用同一份 binlog 文件。
图中的 write，指的就是指把日志写入到文件系统的 page cache，并没有把数据持久化到磁盘，所以速度比较快。
图中的 fsync，才是将数据持久化到磁盘的操作。一般情况下，我们认为 fsync 才占磁盘的 IOPS。
write 和 fsync 的时机，是由参数 sync_binlog 控制的：
sync_binlog=0 的时候，表示每次提交事务都只 write，不 fsync；
sync_binlog=1 的时候，表示每次提交事务都会执行 fsync；
sync_binlog=N(N>1) 的时候，表示每次提交事务都 write，但累积 N 个事务后才 fsync。
因此，在出现 IO 瓶颈的场景里，将 sync_binlog 设置成一个比较大的值，可以提升性能。在实际的业务场景中，考虑到丢失日志量的可控性，一般不建议将这个参数设成 0，比较常见的是将其设置为 100~1000 中的某个数值。
但是，将 sync_binlog 设置为 N，对应的风险是：如果主机发生异常重启，会丢失最近 N 个事务的 binlog 日志。

##### innodb_flush_log_at_trx_commit
为了控制 redo log 的写入策略，InnoDB 提供了 innodb_flush_log_at_trx_commit 参数，它有三种可能取值：
设置为 0 的时候，表示每次事务提交时都只是把 redo log 留在 redo log buffer 中 ;
设置为 1 的时候，表示每次事务提交时都将 redo log 直接持久化到磁盘；
设置为 2 的时候，表示每次事务提交时都只是把 redo log 写到 page cache。


