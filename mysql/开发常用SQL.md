# 常用SQL记录
## DDL
#### 加列
```sql
ALTER TABLE `cho_bill_supplier_base_rule`
ADD COLUMN `rounding_num` decimal(4,2) NOT NULL DEFAULT 0.00 COMMENT '取整系数' ;
```

#### 更改列定义
```sql
ALTER TABLE `cho_bill_supplier_base_rule_detail` 
MODIFY COLUMN `first_part_price` decimal(18,4) NOT NULL DEFAULT '0.0000' COMMENT '首重费用'; 
```

#### 加索引
```sql
ALTER TABLE `cho_bill_carrier_cost` ADD INDEX `idx_composition` ( `vendor_order_no` );
```

#### 修改索引
- mysql中没有真正意义上的修改索引，只有先删除之后在创建新的索引才可以达到修改的目的，原因是mysql在创建索引时会对字段建立关系长度等，只有删除之后创建新的索引才能创建新的关系保证索引的正确性；
> 格式：DROP INDEX 索引名称 ON 表名;
```sql
DROP INDEX login_name_index ON user; 
ALTER TABLE user ADD UNIQUE login_name_index(login_name);
```
> 格式：alter table 表名 drop index 索引名称;
```sql
alter table `cho_bill_supplier_process_rule` drop index `uk_cho_bill_supplier_process_rule`;
alter table `cho_bill_supplier_process_rule` add unique key `uk_cho_bill_supplier_process_rule` (`operation_owner_code`,`province_code`,`city_code`,`region_code`,`node_code_to`,`order_type`,`cost_type`,`pay_type`);
```



## DML

#### 复制列属性
    (例如把id的值复制到freight_line_contract_id列上)
```sql
update frt_freight_line_contract set freight_line_contract_id=id;
```


## QUERY
###### 查询某个元素在列表形式的String是否存在？addressCodes：1,2,5,7
```sql
SELECT * FROM address WHERE FIND_IN_SET('5', addressCodes);
```


######## 要使用查询缓存的语句，可以用 SQL_CACHE 显式指定
```sql
select SQL_CACHE * from T where ID=10;
```

#### 常用命令
##### show status    
查看系统运行的实时状态，便于dba查看mysql当前运行的状态，做出相应优化，动态的，不可认为修改，只能系统自动update。

MariaDB [(none)]> show status like '%conn%';
+--------------------------+----------+
| Variable_name            | Value    |
+--------------------------+----------+
| Aborted_connects         | 101      |   ---- 中断连接数
| Connections              | 11066535 |    ---- 总共的连接数
| Max_used_connections     | 151      |    ----- 曾经的最大连接数
| Ssl_client_connects      | 0        |
| Ssl_connect_renegotiates | 0        |
| Ssl_finished_connects    | 0        |
| Threads_connected        | 10       |    ---- 当前的连接客户端
+--------------------------+----------+

##### show variables    
查看系统参数，系统默认设置或者dba调整优化后的参数，静态的。可以通过set或者修改my.cnf配置文件修改。

MariaDB [(none)]> show variables like '%conn%';
+--------------------------+-----------------+
| Variable_name            | Value           |
+--------------------------+-----------------+
| character_set_connection | utf8            |
| collation_connection     | utf8_general_ci |
| connect_timeout          | 10              |    ---连接数超时时间
| extra_max_connections    | 1               |    ---额外的最大连接数
| init_connect             |                 |
| max_connect_errors       | 10              |    ---允许客户端最大的错误连接数
| max_connections          | 1500            |    ---最大连接数
| max_user_connections     | 0               |
+--------------------------+-----------------+

 max_connect_errors = 10    表示客户端连接数mysql时，如果错误连接数（输入密码错误）10次，之后mysql会自动锁死，防止该客户端再次连接。防止穷举的网络连接攻击。



##### 使用 explain 命令查看语句的执行情况
Extra 字段显示 Using temporary，表示的是需要使用临时表；Using filesort，表示的是需要执行排序操作。

##### SHOW PROCESSLIST 显示哪些线程正在运行。
关键参数：State、Info