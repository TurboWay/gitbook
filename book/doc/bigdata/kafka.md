## 常用命令

kafka 数据是写在本地机器，不是在hdfs上

```shell
cd  /opt/cloudera/parcels/KAFKA/bin

# 查看topic
kafka-topics --bootstrap-server gfdatanode01:9092 --list

# 创建topic
kafka-topics --bootstrap-server gfdatanode01:9092 --create --replication-factor 3 --partitions 1 --topic way

# 查看topic
kafka-topics --bootstrap-server gfdatanode01:9092 --describe --topic way

# 删除topic
kafka-topics --bootstrap-server gfdatanode01:9092 --delete --topic way

# 生产消息
kafka-console-producer --broker-list gfdatanode01:9092,gfdatanode02:9092 --topic way

# 消费消息【查看历史消息 --from-beginning】
kafka-console-consumer --bootstrap-server gfdatanode01:9092 --topic way
```



## CDH 的 KAFKA 实例作用

| 实例名称                 | 作用                                                         |
| ------------------------ | ------------------------------------------------------------ |
| Kafka Broker             | kafka集群中的一个节点                                        |
| Kafka Broker（主控制器） | 包含 controller 的 Kafka Broker。controller 负责管理工作，包括将分区分配给 Broker 和监控 Broker |



## KAFKA 集群架构

![image-20210303105318080](http://pic.turboway.top/blogimg/image-20210303105318080.png)

* topic ：主题。消息数据以 主题 存储。

* partition：分区。主题分为 几个 partition ，每个 partition  存在多个副本，其中一个为 leader，其它为 follower。

* segment file：段文件。每个 partition 目录下的文件被平均切割成大小相等的段文件，每个段文件包含 index 文件（元数据） 和 log文件（消息数据） 。

  
## KAFKA 存储机制

![image-20210303105622604](http://pic.turboway.top/blogimg/image-20210303105622604.png)

* Kafka 把 topic 中一个parition大文件分成多个小文件段，通过多个小文件段，就容易定期清除或删除已经消费完文件，减少磁盘占用。
* 通过索引信息可以快速定位message
* 通过index元数据全部映射到memory，可以避免segment file的IO磁盘操作。
* 通过索引文件稀疏存储，可以大幅降低index文件元数据占用空间大小。



## KAFKA 内核原理

* ISR机制

in-sync replica，就是跟leader partition保持同步的follower partition的数量，只有处于ISR列表中的follower才可以在leader宕机之后被选举为新的leader，因为在这个ISR列表里代表他的数据跟leader是同步的。



* HW&LEO原理

![image-20210303112351414](http://pic.turboway.top/blogimg/image-20210303112351414.png)

![image-20210303112424043](http://pic.turboway.top/blogimg/image-20210303112424043.png)