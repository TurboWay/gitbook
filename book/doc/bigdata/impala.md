[TOC]

# 常用命令

tip: impala 依赖于 hive 元数据，并且感知不到 hive 的操作，所以 使用前需要更新表的元数据

| 命令           |  说明  |
| :------------------------- | :------------------ |
| invalidate metadata | 刷新所有表的元数据                   |
| invalidate metadata tablename | 刷新单表元数据 |
| refresh tbname  |  刷新单表元数据    |
| show table stats tablename  |   获取表的统计信息    |
| show column stats tablename  |   获取字段的统计信息    |
| show partitions tablename  |   获取分区表的统计信息    |
| compute stats tablename  |   刷新表统计信息    |

