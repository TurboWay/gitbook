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