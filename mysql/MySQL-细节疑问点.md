### redo log 和 binlog 是否都是必要的? 我只用其中一种是否可行？

#### 只用binlog
binlog 还是不能支持崩溃恢复的。
> 主要一个不支持的点：binlog 没有能力恢复“数据页”。

场景：连续两个事务，第一个事务已经commit，在第二个事务commit前MySQL 发生了 crash。
因为MySQL写数据是写在内存里的，不保证落盘，所以commit1的数据也可能丢失；但是恢复只恢复binlog失败的也就是commit2的数据，所以数据会丢失。

#### 只用redo log
只从崩溃恢复的角度来讲是可以的。

但在正式的生产库上，binlog 都是开着的。因为 binlog 有着 redo log 无法替代的功能。
- 一个是归档。redo log 是循环写，写到末尾是要回到开头继续写的。这样历史日志没法保留，redo log 也就起不到归档的作用。
- 一个就是 MySQL 系统依赖于 binlog。binlog 作为 MySQL 一开始就有的功能，被用在了很多地方。其中，MySQL 系统高可用的基础，就是 binlog 复制。