[TOC]

# ETL

## 概述

ETL 的英文全称是 Extract-Transform-Load 的缩写，用来描述将数据从来源迁移到目标的几个过程：
1.Extract 数据抽取，也就是把数据从数据源读出来。
2.Transform 数据转换，把原始数据转换成期望的格式和维度。如果用在数据仓库的场景下，Transform也包含数据清洗，清洗掉噪音数据。
3.Load  数据加载，把处理后的数据加载到目标处，比如数据仓库。

## 常用的 ETL 工具优缺点


| 工具   | 开发者         | 优点                                                         | 缺点                                                     |
| :----- | :------------- | :----------------------------------------------------------- | :------------------------------------------------------- |
| Datax  | 阿里巴巴(开源) | 支持所有常见数据源之间的数据传输；<br>配置简单，日志好看     | 单机运行；<br>社区相对没有那么活跃                       |
| Sqoop  | Apache(开源)   | 分布式，资源调度 on yarn                                     | 仅支持 hadoop 集群数据的导入导出；<br>报错隐晦，排查麻烦 |
| Kettle | 国外(开源)     | 支持所有常见数据源之间的数据传输；<br>可视化拖动配置，操作简单 | 单机运行；<br>似乎比datax慢                              |




# 1. Datax

## Datax 概述

DataX 是阿里巴巴集团内被广泛使用的离线数据同步工具/平台，实现包括 MySQL、Oracle、SqlServer、Postgre、HDFS、Hive、ADS、HBase、TableStore(OTS)、MaxCompute(ODPS)、DRDS 等各种异构数据源之间高效的数据同步功能。

文档说明：
https://github.com/alibaba/DataX

优点：
1、几乎支持所有常见数据源之间的数据传输
2、配置简单，日志信息输出和报错明显，易于使用

缺点：
1、单机运行
2、社区相对没有那么活跃

异常：
win环境日志中文乱码，shell终端执行  chcp 65001
hdfsreader 插件有 bug，读取空文件报错，需要改源码重新编译


## 安装

[环境前置依赖和安装教程](https://www.cnblogs.com/jiangbei/p/10901201.html "环境前置依赖和安装教程")

```shell
# 下载软件包
wget http://datax-opensource.oss-cn-hangzhou.aliyuncs.com/datax.tar.gz
# 解压
tar -zxvf datax.tar.gz
# 测试
cd /opt/datax/bin
python datax.py ../job/job.json
```

## 源码安装(解决 hdfs 插件 bug)

(1)、下载DataX源码：

``` shell
git clone https://github.com/TurboWay/DataX.git
```

(2)、通过maven打包：

``` shell
cd  DataX
mvn -U clean package assembly:assembly -Dmaven.test.skip=true
```

打包成功，日志显示如下：

```
[INFO] BUILD SUCCESS
[INFO] -----------------------------------------------------------------
[INFO] Total time: 08:12 min
[INFO] Finished at: 2015-12-13T16:26:48+08:00
[INFO] Final Memory: 133M/960M
[INFO] -----------------------------------------------------------------
```

打包成功后的DataX包位于 DataX/target/datax/datax/


## 常用命令和模板

```shell
# 可以配置传参, 查看用法
python datax.py --help

# 执行
python ~/datax/bin/datax.py -p "-Dhost=** -Duser=root -Dpwd=utopia2020" pg2mysql.json
```

### hdfs2pg

%accordion%配置模板%accordion%

```json
{
    "job": {
        "setting": {
            "speed": {
                "channel": 3
            }
        },
        "content": [
            {
                "reader": {
                    "name": "hdfsreader",
                    "parameter": {
                        "path": "/user/hive/warehouse/edw.db/f_gf_area_info/*",
                        "defaultFS": "hdfs://nameservices1",
                        "hadoopConfig":{
                              "dfs.nameservices": "nameservices1",
                              "dfs.ha.namenodes.nameservices1": "nn1,nn2",
                              "dfs.namenode.rpc-address.nameservices1.nn1": "172.16.122.21:8020",
                              "dfs.namenode.rpc-address.nameservices1.nn2": "172.16.122.24:8020",
                              "dfs.client.failover.proxy.provider.nameservices1": "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
                        },
                        "column": [
                               {
                                "index": 0,
                                "type": "string"
                               },
                               {
                                "index": 1,
                                "type": "string"
                               },
                               {
                                "index": 2,
                                "type": "string"
                               },
                               {
                                "index": 3,
                                "type": "string"
                               },
                               {
                                "index": 4,
                                "type": "string"
                               },
                               {
                                "index": 5,
                                "type": "string"
                               },
                               {
                                "index": 6,
                                "type": "string"
                               },
                               {
                                "index": 7,
                                "type": "string"
                               },
                               {
                                "index": 8,
                                "type": "string"
                               }
                        ],
                        "fileType": "text",
                        "encoding": "UTF-8",
                        "fieldDelimiter": "\u0001"
                    }
                },
                "writer": {
                    "name": "postgresqlwriter",
                    "parameter": {
                        "username": "spider",
                        "password": "spider",
                        "column": [
                            "area_code","area_name","area_code_lvl1","area_name_lvl1","area_code_lvl2","area_name_lvl2","area_code_lvl3","area_name_lvl3","data_dt"
                        ],
                        "preSql": [
                            "truncate table public.f_gf_area_info;"
                        ],
                        "postSql":[
                            "insert into public.area (area_code,area_name,area_code_lvl1,area_name_lvl1,area_code_lvl2,area_name_lvl2,area_code_lvl3,area_name_lvl3,data_dt,sign) select a.area_code,a.area_name,a.area_code_lvl1,a.area_name_lvl1,a.area_code_lvl2,a.area_name_lvl2,a.area_code_lvl3,a.area_name_lvl3,a.data_dt, case when coalesce(a.area_code_lvl3, '') > '' then 3 when coalesce(a.area_code_lvl2, '') > '' then 2 else 1 end as sign from public.f_gf_area_info a  left join public.area b on a.area_code = b.area_code where b.area_code is null;"
                        ],
                        "connection": [
                            {
                                "jdbcUrl": "jdbc:postgresql://172.16.122.19:5432/spider_db",
                                "table": [
                                    "public.f_gf_area_info"
                                ]
                            }
                        ]
                    }
                }
            }
        ]
    }
}

```

%/accordion%

### pg2pg

%accordion%配置模板%accordion%

```json
{
    "job": {
        "setting": {
            "speed": {
                "channel": 3
            }
        },
        "content": [
            {
                "reader": {
                    "name": "postgresqlreader",
                    "parameter": {
                        "username": "spider",
                        "password": "spider",
                        "connection": [
                            {
                                "querySql": [
                                    "select * from public.area;"
                                ],
                                "jdbcUrl": [
                                    "jdbc:postgresql://172.16.122.19:5432/spider_db"
                                ]
                            }
                        ]
                    }
                },
                "writer": {
                    "name": "postgresqlwriter",
                    "parameter": {
                        "username": "gfecp_dev",
                        "password": "gfecp_dev",
                        "column": [ "*" ],
                        "preSql": [
                            "truncate table public.area;"
                        ],
                        "connection": [
                            {
                                "jdbcUrl": "jdbc:postgresql://172.16.122.19:5432/gfecp_dev",
                                "table": [
                                    "public.area"
                                ]
                            }
                        ]
                    }
                }
            }
        ]
    }
}
```
%/accordion%

### pg2mysql

%accordion%配置模板%accordion%

```json
{
    "job": {
        "setting": {
            "speed": {
                "channel": 1
            }
        },
        "content": [
            {
                "reader": {
                    "name": "postgresqlreader",
                    "parameter": {
                        "username": "spider",
                        "password": "spider",
                        "connection": [
                            {
                                "querySql": [
                                    "select * from public.area;"
                                ],
                                "jdbcUrl": [
                                    "jdbc:postgresql://172.16.122.19:5432/spider_db"
                                ]
                            }
                        ]
                    }
                },
                "writer": {
                    "name": "mysqlwriter",
                    "parameter": {
                        "username": "${user}",
                        "password": "${pwd}",
                        "column": ["*"],
                        "preSql": [
                            "truncate table area;"
                        ],
                        "connection": [
                            {
                                "jdbcUrl": "jdbc:mysql://${host}:3306/spider?useUnicode=true&characterEncoding=utf-8",
                                "table": [
                                    "area"
                                ]
                            }
                        ]
                    }
                }
            }
        ]
    }
}

```
%/accordion%



# 2. Sqoop

## Sqoop 概述

Sqoop 是一款开源的工具，主要用于在 Hadoop(Hive)与传统的数据库(mysql、postgresql...) 间进行数据的传递。独立的Apache项目，可以在CDH中快速部署

优点：
1、支持 （HDFS，HIVE，HBASE） <==> RDBMS
2、分布式, 资源调度 on yarn

缺点：
1、不支持 RDBMS 之间的传输，比如 mysql <> postgre
2、报错隐晦，排查困难

注意点：
1、Sqoop1 和 Sqoop2 不兼容，Sqoop2 不打算用于为生产部署，所以一般都用Sqoop1
2、RDBMS ==> (HIVE, HBASE) 时，需要sqoop机器上有对应的集群服务

## import 导入 RDBMS ===> （HDFS，HIVE，HBASE）

```shell
# 详细传参
sqoop import --help
```

### pg2hdfs

```shell
sqoop import --connect jdbc:postgresql://172.16.122.19:5432/spider_db \
--username spider \
--password spider \
--table area \
--outdir /tmp \
--target-dir "/user/area_test" \
--num-mappers 3 \
--fields-terminated-by "\t"
```

### pg2hive

```shell
sqoop import --connect jdbc:postgresql://172.16.122.19:5432/spider_db \
--username spider \
--password spider \
--table area \
--outdir /tmp \
--num-mappers 3 \
--hive-import \
--fields-terminated-by "\t" \
--hive-overwrite \
--hive-database edw \
--hive-table area \
--null-string '\\N' \
--null-non-string '\\N'
```

### pg2hbase

```shell
sqoop import --connect jdbc:postgresql://172.16.122.19:5432/spider_db \
--username spider \
--password spider \
--query 'select area_code,area_name,sign from area where sign=1 and $CONDITIONS;' \
--outdir /tmp \
--num-mappers 3 \
--columns "area_code,area_name,sign"  \
--column-family "cf" \
--hbase-create-table \
--hbase-row-key "area_code" \
--hbase-table "area" \
--split-by area_code
```



## export 导出 （HDFS，HIVE，HBASE） ===> RDBMS

```shell
# 详细传参
sqoop export --help
```

```shell
sqoop export --connect jdbc:postgresql://172.16.122.19:5432/spider_db \
--username spider \
--password spider \
--export-dir "/user/hive/warehouse/edw.db/f_gf_area_info" \
--table f_gf_area_info \
--outdir /tmp \
--input-fields-terminated-by '\001' \
--input-lines-terminated-by '\n'
```




# 3. Kettle

## Kettle 概述

Kettle是一款国外开源的ETL工具，纯java编写，可以在Window、Linux、Unix上运行， 数据抽取高效稳定。


优点：
1、几乎支持所有常见数据源之间的数据传输
2、可视化拖动配置，操作简单

缺点：
1、单机运行
2、似乎比datax慢

## 安装使用

````shell
# 免安装压缩包 ，国内镜像下载
http://mirror.bit.edu.cn/pentaho/Data%20Integration/

# 启动入口
Spoon.bat：在 Windows 平台上运行 spoon
Spoon.sh：在 Linux、AppleOSX、Solaris 平台上运行Spoon

# 连接数据库需要对应的驱动，放到 data-integration\lib

# 教程笔记 ，基本上和 SSIS 一样
https://www.cnblogs.com/goingforward/p/6419154.html
````


