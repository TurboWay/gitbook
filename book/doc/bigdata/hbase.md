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
|put 'mytest','id123','cf:key','value' | 新增修改 |
|deleteall 'mytest','id123' | 删除 |
|delete 'mytest','id123','cf:key' | 删除列族中的列 |
|scan 'mytest',{STARTROW=>'id123',ENDROW=>'id125'} | 范围查询 |
|get 'mytest','id123' | 查询一条数据 |
|get 'mytest','id123','cf' | 查询一条数据的指定列族 |
|flush 'mytest' | mem数据刷写到hfile |
|compact 'mytest' | 小合并 |
|major_compact 'mytest' | 大合并，性能消耗多 |
|list_snapshots | 查看快照 |
|snapshot 'mytest', 'mytest_snap' | 创建快照 |
|disable 'mytest'  <br>restore_snapshot 'mytest_snap'  <br>enable 'mytest' | 还原 |
|clone_snapshot 'mytest_snap', 'mytest2' | 通过快照创建一张新的表 |
|delete_snapshot 'mytest_snap' | 删除快照 |


## 运维管理

|  命令    |  说明    |
| :---- | :---- |
|balancer_enabled  | 查看 region 自动平衡启用状态 |
|balance_switch false  | 禁用 region 自动平衡 |
|balance_switch true   | 启用 region 自动平衡 |
|balancer | 立即平衡（需要先启用自动平衡） |
|hbase hbck | 检查所有 region |
|nohup  hbase hbck  2>&1 > hbase_check.log & | 检查所有 region 后台运行 |
|sudo -u'hbase' hbase hbck -repair tablename | 表异常修复，慎用[有风险](https://www.jianshu.com/p/d539e51101ad) |

>repair 命令其实是 hbase hbck -fix 一些组合的快捷方式
fix 命令的一些实践说明 https://www.cnblogs.com/quchunhui/p/9583746.html



## CDH 的 HBase 实例作用

| 实例名称            | 作用                                |
| ------------------- | ----------------------------------- |
| Master              | 管理 table 表 和 RegionServer       |
| RegionServer        | 相应读写请求，管理本节点上的 region |
| HBase Thrift Server | 提供 Hbase Thrift 访问接口          |



## HBase 架构

![image-20210303170441072](https://gitee.com/TurboWay/blogimg/raw/master/img/image-20210303170441072.png)

![image-20210303170519302](https://gitee.com/TurboWay/blogimg/raw/master/img/image-20210303170519302.png)

## HBase 读流程
![image-20210303170536576](https://gitee.com/TurboWay/blogimg/raw/master/img/image-20210303170536576.png)

## HBase 写流程
![image-20210303170547327](https://gitee.com/TurboWay/blogimg/raw/master/img/image-20210303170547327.png)

## HBase 的 flush、compact 原理
![image-20210303170559969](https://gitee.com/TurboWay/blogimg/raw/master/img/image-20210303170559969.png)