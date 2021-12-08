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

