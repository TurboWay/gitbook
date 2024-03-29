
## [官方文档](https://doris.apache.org/zh-CN/docs/dev/summary/basic-summary/)

### 查看版本
```sql
show variables like 'version%'
```

### 日期格式化

```sql
select date_format(now(),'%Y%m%d');
select date_format(curdate(),'%Y%m%d');
select date_format(date_add(curdate(), interval -1 day),'%Y%m%d');
select date_format(date_add(20220620, interval 6 day),'%Y%m%d');
```

### 特殊限制

- between 隐式转换 '' 转 int 会报错 
- order by 默认限制最大 65535
- 字段名不支持中文
- 中文存储需要占 3 位


### 性能优化


- 稀疏索引   

  > 每张表都有的前缀索引，需要注意建表顺序 、 排序列的顺序

- bloomfilter索引    

  > 适用 store_code / goods_sn/shop_code
  >
  > ```sql
  > ALTER TABLE example_db.my_table SET ("bloom_filter_columns" = ""); -- 删除索引
  > ALTER TABLE example_db.my_table SET ("bloom_filter_columns" = "goods_sn,shop_code"); -- 修改索引
  > ```

- 倒排索引   

  >  适用 is_ht  
  
- 可对维度表设置更多的副本，提升 Join 的性能
  > ```sql
  > ALTER TABLE example_db.my_table SET ("replication_num" = "7");
  > ```
  
### 权限

```sql
-- 创建用户
CREATE USER ods@'%' IDENTIFIED BY 'ods.com.1234';
-- 授权
GRANT SELECT_PRIV, LOAD_PRIV, ALTER_PRIV, CREATE_PRIV, DROP_PRIV   ON ods.* TO 'ods'@'%';
```

### 物化视图

> 限制：不能加过滤条件；不支持表关联；只支持明细模型

- 创建
```sql
create materialized view vfact_shop_inventory_available_day_goods_sum as 
select dateid,dateid2,shop_code,goods_sn,sum(qty) as qty, sum(qtyavailable) as qtyavailable
from fact_shop_inventory_available_day
group by dateid,dateid2,shop_code,goods_sn
```
- 查看进度
```sql
show alter table materialized view from ebi
```
- 查看
```sql
desc  ebi.fact_shop_inventory_available_day  all 
```
- 删除
```sql
drop  materialized view vfact_shop_inventory_available_day_goods_sum on fact_shop_inventory_available_day
```


## 同步落地表

```sql
-- ebi.config_store definition
DROP TABLE config_order_deal_code;
CREATE TABLE `config_order_deal_code` (
`id` bigint  null,
`shop_code` varchar(64) null,
`order_deal_code` varchar(500) null,
`created_by` bigint  null,
`created_time` datetime  null,
`updated_by` bigint  null,
`updated_time` datetime  null
) ENGINE=OLAP
UNIQUE KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`) BUCKETS 4
PROPERTIES (
"replication_num" = "3",
"in_memory" = "false",
"storage_format" = "V2"
);

-- 补序列字段
ALTER TABLE gds_btgoods ENABLE FEATURE "SEQUENCE_LOAD" WITH PROPERTIES ("function_column.sequence_type" = "bigint");
```

## 动态分区
```sql
CREATE TABLE `fact_vip_settlement_day_hour_goods_ads_sum_realtime` (
  `order_type_id` int(11) NOT NULL COMMENT "",
  `dateid` int(11) NOT NULL COMMENT "",
  `add_dateid` int(11) NULL COMMENT "",
  `pay_dateid` int(11) NULL COMMENT "",
  `goods_sn` varchar(240) NOT NULL COMMENT "",
  `is_ht` bigint(20) NOT NULL COMMENT "",
  `salesAmount` decimal(20, 6) NOT NULL COMMENT "",
  `goods_number` int(11) NOT NULL COMMENT "",
  `order_from` varchar(16) NOT NULL COMMENT "",
  `vendor_code` varchar(64) NOT NULL COMMENT "",
  `utc` int(11) NOT NULL COMMENT "",
  `hour` int(11) NULL COMMENT ""
) ENGINE=OLAP
COMMENT "唯品会结算(货号+小时)(实时近30天)"
PARTITION BY RANGE(dateid) ()
DISTRIBUTED BY HASH(dateid)
PROPERTIES
(
    "dynamic_partition.create_history_partition" = "true",
    "dynamic_partition.history_partition_num" = "30",
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-30",
    "dynamic_partition.end" = "1",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "8"
)
```


#### 常用sql

```sql
-- 查看隐藏列
set show_hidden_columns=true
-- 增加数据库配额
alter database testdb set data quota 100t
-- 删除不管分区
set delete_without_partition=true;
-- 设置严格模式
show global variables like '%enable_insert_strict%'
set global enable_insert_strict=true
-- 查看表变更进度
show alter table column order by createtime desc
-- 修改表注释
alter table order_return_goods_history2021 modify comment '换货新单商品明细（2021归档）';
-- 查看表的统计信息收集概况
show table stats ebi.dim_date
-- 查看列的各项统计数据
show column stats ebi.dim_date
```

#### 资源组

```sql
-- 限制用户资源组
SET PROPERTY FOR 'ebi_acc' 'resource_tags.location' = 'acc';

-- 查看用户属性
SHOW PROPERTY FOR ebi_acc

-- 查看分区情况
show partitions from dis_capital_account_occupied_fund

-- 修改副本分布
-- 改表属性(必需)
alter table stm_retail_settle_detail_full set ("default.replication_allocation"="tag.location.default:5");
alter table stm_retail_settle_detail_full set ("default.replication_allocation"="tag.location.default:3, tag.location.acc:2");

-- 非分区表(按需)
alter table stm_retail_settle_detail_full set ("replication_allocation"="tag.location.default:5");
alter table stm_retail_settle_detail_full set ("replication_allocation"="tag.location.default:3, tag.location.acc:2");

-- 分区表(按需)
alter table stm_retail_settle_detail_full MODIFY PARTITION (*) set ("replication_allocation"="tag.location.default:5");
alter table stm_retail_settle_detail_full MODIFY PARTITION (*) set ("replication_allocation"="tag.location.default:3, tag.location.acc:2");

-- 动态分区表(按需)
alter table stm_retail_settle_detail_full set ("dynamic_partition.replication_allocation"="tag.location.default:5");
alter table stm_retail_settle_detail_full set ("dynamic_partition.replication_allocation"="tag.location.default:3, tag.location.acc:2");
alter table stm_retail_settle_detail_full MODIFY PARTITION (*) set ("replication_allocation"="tag.location.default:5");
alter table stm_retail_settle_detail_full MODIFY PARTITION (*) set ("replication_allocation"="tag.location.default:3, tag.location.acc:2");
```

### 修改配置

```shell
# 查看BE配置
http://10.211.6.136:8040/varz

# 动态修改BE配置持久化
curl -X POST http://10.211.6.136:8040/api/update_config?streaming_load_json_max_mb=10240\&persist=true
```



