[TOC]

## 常用配置

```shell
# 每个采集使用独立配置文件
flume-ng agent -n a1 -f spooldir.conf
```

### 端口测试示例： netcat.conf

```shell
# 定义这个agent中各组件的名字 
a1.sources = r1 
a1.sinks = k1 
a1.channels = c1 

# 描述和配置source组件：r1 
a1.sources.r1.type = netcat 
a1.sources.r1.bind = 127.0.0.1 
a1.sources.r1.port = 44444 

# 描述和配置sink组件：k1 
a1.sinks.k1.type = logger 

# 描述和配置channel组件，此处使用是内存缓存的方式 
a1.channels.c1.type = memory 
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 

# 描述和配置source channel sink之间的连接关系 
a1.sources.r1.channels = c1 
a1.sinks.k1.channel = c1
```

### 日志上传到 hdfs 示例：  spooldir.conf

```shell
# 定义各组件的名字
a1.sources = r1 
a1.sinks = k1 
a1.channels = c1 

# 定义sources
##注意：不能往监控目中重复丢同名文件 
a1.sources.r1.type = spooldir 
a1.sources.r1.spoolDir = /root/test
a1.sources.r1.fileHeader = true 

# 定义sink
a1.sinks.k1.type = hdfs 
a1.sinks.k1.channel = c1 
a1.sinks.k1.hdfs.path = hdfs://nameservice1:8020/user/flume/event4
a1.sinks.k1.hdfs.filePrefix = events
a1.sinks.k1.hdfs.round = true 
# HDFS文件滚动周期：10分钟
a1.sinks.k1.hdfs.roundValue = 10 
a1.sinks.k1.hdfs.roundUnit = minute 
a1.sinks.k1.hdfs.rollInterval = 60
a1.sinks.k1.hdfs.rollSize = 2048
a1.sinks.k1.hdfs.rollCount = 1000 
a1.sinks.k1.hdfs.batchSize = 1 
a1.sinks.k1.hdfs.useLocalTimeStamp = true 
# 生成的文件类型，默认是Sequencefile，可用DataStream，则为普通文本 
a1.sinks.k1.hdfs.fileType = DataStream 

# 定义channel 
a1.channels.c1.type = memory 
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 

# 绑定channel 
a1.sources.r1.channels = c1 
a1.sinks.k1.channel = c1
```

