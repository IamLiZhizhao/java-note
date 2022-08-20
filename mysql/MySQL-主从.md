## 事务
### ACID
（Atomicity、Consistency、Isolation、Durability，即原子性、一致性、隔离性、持久性）

### 一主多从的切换正确性

#### 基于位点的主备切换
当我们把节点 B 设置成节点 A’的从库的时候，需要执行一条 change master 命令：
CHANGE MASTER   TO
MASTER_HOST=$host_name
MASTER_PORT=$port
MASTER_USER=$user_name
MASTER_PASSWORD=$password
MASTER_LOG_FILE=$master_log_name
MASTER_LOG_POS=$master_log_pos

这条命令有这么 6 个参数：
MASTER_HOST、MASTER_PORT、MASTER_USER 和 MASTER_PASSWORD 四个参数，分别代表了主库 A’的 IP、端口、用户名和密码。
最后两个参数 MASTER_LOG_FILE 和 MASTER_LOG_POS 表示，要从主库的 master_log_name 文件的 master_log_pos 这个位置的日志继续同步。而这个位置就是我们所说的同步位点，也就是主库对应的文件名和日志偏移量。

通常情况下，我们在切换任务的时候，要先主动跳过这些错误，有两种常用的方法。
一种做法是，主动跳过一个事务。跳过命令的写法是：
set global sql_slave_skip_counter=1;
start slave;
因为切换过程中，可能会不止重复执行一个事务，所以我们需要在从库 B 刚开始接到新主库 A’时，持续观察，每次碰到这些错误就停下来，执行一次跳过命令，直到不再出现停下来的情况，以此来跳过可能涉及的所有事务。
另外一种方式是，通过设置 slave_skip_errors 参数，直接设置跳过指定的错误。
在执行主备切换时，有这么两类错误，是经常会遇到的：
1062 错误是插入数据时唯一键冲突；
1032 错误是删除数据时找不到行。
因此，我们可以把 slave_skip_errors 设置为 “1032,1062”，这样中间碰到这两个错误时就直接跳过。
这里需要注意的是，这种直接跳过指定错误的方法，针对的是主备切换时，由于找不到精确的同步位点，所以只能采用这种方法来创建从库和新主库的主备关系。


#### 基于 GTID 的主备切换 (推荐)
MySQL 5.6 版本引入了 GTID。
> GTID 的全称是 Global Transaction Identifier，也就是全局事务 ID

基于 GTID 的主备复制的用法。
在 GTID 模式下，备库 B 要设置为新主库 A’的从库的语法如下：
CHANGEMASTERTO
MASTER_HOST=$host_name
MASTER_PORT=$port
MASTER_USER=$user_name
MASTER_PASSWORD=$password
master_auto_position=1
其中，master_auto_position=1 就表示这个主备关系使用的是 GTID 协议。

把现在这个时刻，实例 A’的 GTID 集合记为 set_a，实例 B 的 GTID 集合记为 set_b。接下来，我们就看看现在的主备切换逻辑。
我们在实例 B 上执行 start slave 命令，取 binlog 的逻辑是这样的：
实例 B 指定主库 A’，基于主备协议建立连接。
实例 B 把 set_b 发给主库 A’。
实例 A’算出 set_a 与 set_b 的差集，也就是所有存在于 set_a，但是不存在于 set_b 的 GTID 的集合，判断 A’本地是否包含了这个差集需要的所有 binlog 事务。
a. 如果不包含，表示 A’已经把实例 B 需要的 binlog 给删掉了，直接返回错误；
b. 如果确认全部包含，A’从自己的 binlog 文件里面，找出第一个不在 set_b 的事务，发给 B；
之后就从这个事务开始，往后读文件，按顺序取 binlog 发给 B 去执行。

!!其实，这个逻辑里面包含了一个设计思想：
在基于 GTID 的主备关系里，系统认为只要建立主备关系，就必须保证主库发给备库的日志是完整的。
因此，如果实例 B 需要的日志已经不存在，A’就拒绝把日志发给 B。
这跟基于位点的主备协议不同。基于位点的协议，是由备库决定的，备库指定哪个位点，主库就发哪个位点，不做日志的完整性判断。