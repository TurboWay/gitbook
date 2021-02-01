[TOC]

## 常用命令

```shell
# 创建文件夹
hdfs dfs -mkdir /user/test123
# 创建多级文件夹
hdfs dfs -mkdir -p /user/test123/test456
# 删除文件夹
hdfs dfs -rm -r /user/test123
# 查看列表
hdfs dfs -ls /user/hive/
# 上传文件
hdfs dfs -copyFromLocal tmp/caiwei/123.txt /user/hive/udf/
hdfs dfs -put tmp/caiwei/123.txt /user/hive/udf/
# 下载文件
hdfs dfs -get /user/hive/udf/123.txt
# 删除文件
hdfs dfs -rm /user/hive/udf/123.txt
```

## 常用统计命令

```shell
# 统计文件大小
hdfs dfs -du -h /user/hive/warehouse/edw.db/f_gf*
第一列是文件大小
第二列是文件大小 * 副本数  默认3
第三列是查询目录

# 查看hbase数据大小
hdfs dfs -du -s -h /hbase/data/default/

# 查看附件数据大小
hdfs dfs -du -s -h /user/spider/

# 查看hive数据大小
hdfs dfs -du -s -h /user/hive/warehouse/

# 查看附件文件数量
hdfs dfs -count /user/spider/
```

## 常用管理命令

```shell
# 检查集群状态
hdfs fsck /

# 查看缺失块
hdfs fsck -list-corruptfileblocks 

# 删除缺失块
hdfs fsck -delete

# 重平衡
hdfs balancer -threshold 10

# 清空回收站
hdfs dfs -expunge

# 后台运行重平衡
nohup hdfs balancer -threshold 10 > balance.log 2>&1 &

# 查看集群信息
hdfs dfsadmin -report

# 安全模式命令。安全模式是NameNode的一种状态，在这种状态下，NameNode不接受对名字空间的更改（只读）；不复制或删除块。NameNode在启动时自动进入安全模式，当配置块的最小百分数满足最小副本数的条件时，会自动离开安全模式
hdfs dfsadmin -safemode get
hdfs dfsadmin -safemode enter
hdfs dfsadmin -safemode leave
hdfs dfsadmin -safemode wait
```