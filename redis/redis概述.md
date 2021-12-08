### 主要知识点
- Redis 的五种数据结构及其内部实现
- RDB/AOF 持久化
- 主从复制、部分复制、全量复制
- 哨兵、集群、故障转移及其原理
- 最经典的问题：分布式锁

##### Redis 优势
1. 性能极高 – Redis 读的速度是 110000 次 /s, 写的速度是 81000 次 /s 。
2. 丰富的数据类型 - Redis 支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
3. 原子性 - Redis 的单个操作是原子性的。多个操作也支持事务，通过 MULTI 和 EXEC 指令包起来，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的。
4. 其他特性 - Redis 还支持 publish/subscribe 通知，key 过期等特性。
