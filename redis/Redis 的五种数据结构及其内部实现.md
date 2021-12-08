### Redis 的五种数据结构及其内部实现
- String
- List
- Hash
- Set
- ZSet

> redis的键(key)都是字符串

#### String(字符串)
- 是redis中最基本的数据类型，类似java中的map，一个key对应一个value。
- String类型是二进制安全的，意思是 redis 的 string 可以包含任何数据。如数字，字符串，jpg图片或者序列化的对象。
- 使用：get 、 set 、 del 、 incr、 decr 等

#### List(链表数组)
List 说白了就是链表（redis 使用双端链表实现的 List），是有序的，value可以重复，可以通过下标取出对应的value值，左右两边都能进行插入和删除数据。

    使用列表的技巧
        lpush+lpop=Stack(栈)
        lpush+rpop=Queue（队列）
        lpush+ltrim=Capped Collection（有限集合）
        lpush+brpop=Message Queue（消息队列）

#### Hash(哈希)
- 本质上是一个Map&lt;map&gt; ，值本身又是一种键值对结构，如 value={{field1,value1},......{fieldN,valueN}}
- 所有hash的命令都是h开头的,如hget、hset、hdel 等
```
127.0.0.1:6379> hset user name1 hao
(integer) 1
127.0.0.1:6379> hset user email1 hao@163.com
(integer) 1
127.0.0.1:6379> hgetall user
1) "name1"
2) "hao"
3) "email1"
4) "hao@163.com"
127.0.0.1:6379> hget user user
(nil)
127.0.0.1:6379> hget user name1
"hao"
127.0.0.1:6379> hset user name2 xiaohao
(integer) 1
127.0.0.1:6379> hset user email2 xiaohao@163.com
(integer) 1
127.0.0.1:6379> hgetall user
1) "name1"
2) "hao"
3) "email1"
4) "hao@163.com"
5) "name2"
6) "xiaohao"
7) "email2"
8) "xiaohao@163.com"
```

#### Set(集合)
- 集合类型也是用来保存多个字符串的元素，但和列表不同的是：集合中  
    1. 不允许有重复的元素，
    2. 集合中的元素是无序的，不能通过索引下标获取元素，
    3. 支持集合间的操作，可以取多个集合取交集、并集、差集。
- 使用：命令都是以s开头的  sset 、srem、scard、smembers、sismember
```
127.0.0.1:6379> sadd myset hao hao1 xiaohao hao
(integer) 3
127.0.0.1:6379> SMEMBERS myset
1) "xiaohao"
2) "hao1"
3) "hao"
127.0.0.1:6379> SISMEMBER myset hao
(integer) 1
```

#### ZSet(有序集合)
- 保留了集合不能有重复成员的特性，区别是，有序集合中的元素是可以排序的，它给每个元素设置一个分数，作为排序的依据。
- 使用： 有序集合的命令都是 以  z  开头    zadd 、 zrange、 zscore

```
127.0.0.1:6379> zadd myscoreset 100 hao 90 xiaohao
  (integer) 2
  127.0.0.1:6379> ZRANGE myscoreset 0 -1
  1) "xiaohao"
  2) "hao"
  127.0.0.1:6379> ZSCORE myscoreset hao
  "100"
```
  
