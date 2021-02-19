[TOC]

## 注意点

 * 表名大小写敏感，要区分大小写
 * 每一行可以有多个列族（建表时指定）
 * 每个列族里面，可以有多个列（列族相当于许多的key-value，每个key就是一个列）


## 常用命令

|  命令    |  说明    |
| :---- | :---- |
|hbase shell  | 进入 cli  |
|list_namespace  |   查看表空间   |
|exists 'mytest' | 表是否存在 |
|create 'mytest', 'cf' | 建表 |
|create 'mytest', {NAME => 'cf', COMPRESSION => 'snappy'} | 建压缩表 |
|enable 'mytest' | 启用表 |
|disable 'mytest' | 禁用表 |
|drop 'mytest' | 删表（需要先禁用表） |
|truncate 'mytest' | 清空表 |
|desc 'mytest' | 查看表 |
|count 'mytest' | 计数 |
|alter 'mytest' , NAME => 'cf2' | 增加列族 |
|alter 'mytest' , { NAME=>'cf2', METHOD=>'delete' } | 删除列族 |
|alter 'mytest', NAME => 'cf', TTL => '86400' | 设置生存期, 24小时后自动删除 |
|alter 'mytest', NAME => 'cf', COMPRESSION  => 'snappy' | 设置为 snappy 压缩 |
|major_compact 'mytest' | 大合并，性能消耗多 |

## 增删改查

|  命令    |  说明    |
| :---- | :---- |
|put 'mytest','id123','cf:key','value' | 新增修改 |
|deleteall 'mytest','id123' | 删除 |
|delete 'mytest','id123','cf:key' | 删除列族中的列 |
|scan 'mytest',{STARTROW=>'id123',ENDROW=>'id125'} | 范围查询 |
|get 'mytest','id123' | 查询一条数据 |
|get 'mytest','id123','cf' | 查询一条数据的指定列族 |


## 运维

|  命令    |  说明    |
| :---- | :---- |
|balancer_enabled  | 查看 region 自动平衡启用状态 |
|balance_switch false  | 禁用 region 自动平衡 |
|balance_switch true   | 启用 region 自动平衡 |
|balancer | 立即平衡（需要先启用自动平衡） |
|hbase hbck | 检查所有 region |
|nohup  hbase hbck  2>&1 > hbase_check.log & | 检查所有 region 后台运行 |
|sudo -u'hbase' hbase hbck -repair | 表异常修复 |
|sudo -u'hbase' hbase hbck -repairHoles | 表异常修复 |

tip: 
修复表，需要 hbase 用户权限
repair 命令其实是 hbase hbck -fix 一些组合的快捷方式
fix 命令的一些实践说明 https://www.cnblogs.com/quchunhui/p/9583746.html




