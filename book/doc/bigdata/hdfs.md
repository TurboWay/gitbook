[TOC]

## 常用命令

| 命令                                                         | 说明                 |
| :----------------------------------------------------------- | :------------------- |
| hdfs dfs -mkdir /user/test123                                | 创建文件夹           |
| hdfs dfs -mkdir -p /user/test123/test456                     | 创建多级文件夹       |
| hdfs dfs -rm -r /user/test123                                | 删除文件夹           |
| hdfs dfs -rm  /user/test123/test.txt                         | 删除文件             |
| hdfs dfs -ls /user/hive/                                     | 查看列表             |
| hdfs dfs -ls -R /user/hive/                                  | 递归查看列表         |
| hdfs dfs -put tmp/caiwei/123.txt /user/hive/udf/ <br>hdfs dfs -copyFromLocal tmp/caiwei/123.txt /user/hive/udf/ | 上传文件             |
| hdfs dfs -get /user/hive/udf/123.txt                         | 下载文件             |
| hdfs dfs -du -h /user/hive/warehouse/edw.db/test             | 统计每个文件大小     |
| hdfs dfs -du -s -h /user/hive/warehouse/                     | 查看所有文件的总大小 |
| hdfs dfs -count /user/spider/                                | 统计文件数量         |



## 常用管理命令

| 命令                                                  | 说明                     |
| ----------------------------------------------------- | ------------------------ |
| hdfs fsck /                                           | 检查集群状态             |
| hdfs dfsadmin -report                                 | 查看集群信息             |
| hdfs getconf -namenodes                               | 获取NameNode的节点名称   |
| hadoop checknative                                    | 查看集群是否支持本地压缩 |
| hdfs fsck -list-corruptfileblocks                     | 查看缺失块               |
| hdfs fsck -delete                                     | 删除缺失块               |
| hdfs dfs -expunge                                     | 清空回收站               |
| nohup hdfs balancer -threshold 5 > balance.log 2>&1 & | 后台运行重平衡           |
| hdfs dfsadmin -safemode get                           | 获取安全模式状态         |
| hdfs dfsadmin -safemode enter                         | 进入安全模式             |
| hdfs dfsadmin -safemode leave                         | 退出安全模式             |
| hdfs dfsadmin -safemode wait                          | 等待安全模式结束         |



## CDH 的 HDFS 实例作用

| 实例名称            | 作用                                                         |
| ------------------- | ------------------------------------------------------------ |
| NameNode            | 名称节点，管理集群元数据和处理客户端请求 <br> 集群元数据在内存中，定时备份到 fsimage ，最新操作追加到 edit logs |
| SecondaryNameNode   | 检查点节点，定时从 NN 获取、合并 fsimage 和 edit logs （冷备份） |
| DataNode            | 数据节点，存放集群数据块                                     |
| Failover Controller | 故障恢复控制器，HA模式时，切换 NameNode                      |
| JournalNode         | 日志节点，HA模式时，同步 NameNode 日志                       |
| HttpFS              | 提高 http 接口，直接访问 hdfs 文件                           |
| Balancer            | 数据重平衡                                                   |



## 图解 HDFS 存储原理

### 1. HDFS写数据原理

<div align="center"> <img  src="https://gitee.com/heibaiying/BigData-Notes/raw/master/pictures/hdfs-write-1.jpg"/> </div>

<div align="center"> <img  src="https://gitee.com/heibaiying/BigData-Notes/raw/master/pictures/hdfs-write-2.jpg"/> </div>

<div align="center"> <img  src="https://gitee.com/heibaiying/BigData-Notes/raw/master/pictures/hdfs-write-3.jpg"/> </div>



### 2. HDFS读数据原理

<div align="center"> <img  src="https://gitee.com/heibaiying/BigData-Notes/raw/master/pictures/hdfs-read-1.jpg"/> </div>
