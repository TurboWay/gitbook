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
| hdfs dfs -mv /user/hive/udf/123.txt   /user/hive/            | 移动文件             |
| hdfs dfs -cp /user/hive/udf/123.txt  /user/hive/udf/456.txt  | 复制文件             |
| hdfs dfs -get /user/hive/udf/123.txt                         | 下载文件             |
| hdfs dfs -getmerge /user/hive/udf/jsonunionUDF.py /user/hive/udf/file_parse.py 123.txt     | 合并下载文件             |
| hdfs dfs -du -h /user/hive/warehouse/edw.db/test             | 统计每个文件大小     |
| hdfs dfs -du -s -h /user/hive/warehouse/                     | 查看所有文件的总大小 |
| hdfs dfs -count /user/spider/                                | 统计文件数量         |
| hdfs dfs -setrep -w 3 /user/hive/udf                         | 修改复制因子         |


## 常用管理命令

| 命令                                                  | 说明                     |
| ----------------------------------------------------- | ------------------------ |
| hdfs fsck /                                           | 检查集群状态             |
| hdfs dfs -df -h /                                     | 查看集群可用空间         |
| hdfs dfsadmin -report                                 | 查看集群信息             |
| hdfs getconf -namenodes                               | 获取NameNode的节点名称   |
| hadoop checknative                                    | 查看集群是否支持本地压缩 |
| hdfs fsck /user/hive/udf -list-corruptfileblocks      | 查看缺失块               |
| hdfs fsck /user/hive/udf  -delete                     | 删除缺失块               |
| hdfs dfs -expunge                                     | 清空回收站               |
| nohup hdfs balancer -threshold 5 > balance.log 2>&1 & | 后台运行重平衡           |
| hdfs dfsadmin -safemode get                           | 获取安全模式状态         |
| hdfs dfsadmin -safemode enter                         | 进入安全模式             |
| hdfs dfsadmin -safemode leave                         | 退出安全模式             |
| hdfs dfsadmin -safemode wait                          | 等待安全模式结束         |
| hdfs oev -i edits_0000000000000000256-0000000000000000363 -o edit1.xml           | 如何查看edits日志文件        |
| hdfs oiv -p XML -i fsimage_0000000000000092691 -o fsimage1.xml                   | 如何查看fsimage文件         |
| hdfs dfsadmin -allowSnapshot /user/admin             | 允许快照         |
| hdfs dfsadmin -disallowSnapshot /user/admin          | 禁止快照         |
| hdfs dfs -createSnapshot /user/admin test            | 创建快照         |
| hdfs dfs -ls /user/admin/.snapshot/test              | 查看快照         |
| hdfs dfs -renameSnapshot /user/admin test test2      | 重命名快照       |
| hdfs dfs -cp /user/admin/.snapshot/test2/12.txt /user/admin/12.txt  | 从快照恢复数据       |
| hdfs dfs -deleteSnapshot /user/admin test2           | 删除快照       |




## CDH 的 HDFS 实例作用

| 实例名称            | 作用                                                         |
| ------------------- | ------------------------------------------------------------ |
| NameNode            | 名称节点，管理集群元数据和处理客户端请求 <br> 集群元数据在内存中，定时备份到 fsimage ，最新操作追加到 edit logs |
| SecondaryNameNode   | 检查点节点，定时从 NN 获取、合并 fsimage 和 edit logs （冷备份） |
| DataNode            | 数据节点，存放集群数据块                                     |
| Failover Controller | 故障恢复控制器，HA模式时，切换 NameNode                      |
| JournalNode         | 日志节点，HA模式时，同步 NameNode 日志                       |
| HttpFS              | 提供 http 接口，直接操作 hdfs 文件；hue 访问 hdfs 需要此实例  |
| Balancer            | 数据重平衡                                                   |



## 图解 HDFS 存储原理

### 1. HDFS写数据原理

![image-20210302140425159](https://gitee.com/TurboWay/blogimg/raw/master/img/image-20210302140425159.png)

![image-20210302140638840](https://gitee.com/TurboWay/blogimg/raw/master/img/image-20210302140638840.png)

![image-20210302140707733](https://gitee.com/TurboWay/blogimg/raw/master/img/image-20210302140707733.png)


### 2. HDFS读数据原理

![image-20210302140751738](https://gitee.com/TurboWay/blogimg/raw/master/img/image-20210302140751738.png)

### 3. HDFS故障类型和其检测方法

![image-20210302140829008](https://gitee.com/TurboWay/blogimg/raw/master/img/image-20210302140829008.png)

![image-20210302140854507](https://gitee.com/TurboWay/blogimg/raw/master/img/image-20210302140854507.png)

**第二部分：读写故障的处理**

![image-20210302140924798](https://gitee.com/TurboWay/blogimg/raw/master/img/image-20210302140924798.png)


**第三部分：DataNode 故障处理**

![image-20210302140949174](https://gitee.com/TurboWay/blogimg/raw/master/img/image-20210302140949174.png)

**副本布局策略**：

![image-20210302141014819](https://gitee.com/TurboWay/blogimg/raw/master/img/image-20210302141014819.png)