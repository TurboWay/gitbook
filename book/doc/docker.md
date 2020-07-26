[TOC]

# 1.docker 常用命令

| 分类 | 命令                                                         | 说明                                                         |
| :--- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 镜像 | docker search mysql:5.7                                      | 搜索镜像                                                     |
|      | docker images                                                | 查看本地下载镜像                                             |
|      | docker pull mysql:5.7                                        | 拉取镜像                                                     |
|      | docker rmi mysql:5.7                                         | 删除镜像                                                     |
| 容器 | docker ps -a                                                 | 查看当前正在运行的容器                                       |
|      | docker ps -n 5                                               | 查看最近运行的5个容器                                        |
|      | docker run hello-world                                       | 新建并启动一个容器，当镜像不存在时，会自动拉取镜像，docker run 可以指定很多参数 |
|      | docker start hello-world                                     | 启动一个已存在的容器                                         |
|      | docker logs hello-world                                      | 查看容器日志                                                 |
|      | docker restart hello-world                                   | 重启容器                                                     |
|      | docker rm hello-world                                        | 删除容器                                                     |
| 交互 | docker exec -it elasticsearch /bin/bash                      | 进入容器（容器必须先启动）                                   |
|      | docker cp temp123.txt elasticsearch:/usr/share/elasticsearch/temp123.txt | 复制文件/文件夹(宿主->容器)                                  |
|      | docker cp elasticsearch:/usr/share/elasticsearch/temp123.txt  temp123.txt | 复制文件/文件夹(容器->宿主)                                  |

# 2.docker 安装常用软件

## 安装 mysql

 ```shell
docker run -itd --name mysql -e MYSQL_ROOT_PASSWORD=password -p 3306:3306 mysql:5.7
 ```

## 安装 postgres

 ```shell
docker run -itd --name postgres -e POSTGRES_PASSWORD=password -p 5432:5432 postgres
 ```

## 安装 sqlserver

 ```shell
docker run -itd -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=utopia.123"  -p 1433:1433 --name sqlserver mcr.microsoft.com/mssql/server:2017-latest
 ```

## 安装 redis

 ```shell
docker run -itd --name redis  -p 6379:6379 redis
 ```

## 安装 mongodb

 ```shell
docker run -itd --name mongo -p 27017:27017 mongo
 ```

## 安装 elasticsearch + kibana + elastichd
| 组件名称      | 安装命令                                                     |
| ------------- | ------------------------------------------------------------ |
| elasticsearch | docker run -id --name elasticsearch -p 9200:9200 -p 9300:9300 elasticsearch:5.6.12 |
| kibana        | docker run -id --name kibana -e ELASTICSEARCH_URL=http://docker.for.win.localhost:9200 -p 5601:5601 kibana:5.6.12 |
| elastichd     | docker run -id --name elastichd -p 9800:9800 --link elasticsearch:demo containerize/elastichd |

### 可视化连接不上 elasticsearch 解决办法：

1.复制容器配置文件
 ```shell
docker cp elasticsearch:/usr/share/elasticsearch/config/elasticsearch.yml  elasticsearch.yml 
 ```
2.修改 elasticsearch.yml 增加如下的跨域访问配置
 ```
http.cors.enabled: true 
http.cors.allow-origin: "*"
 ```
3.更新容器配置文件
 ```shell
docker cp elasticsearch.yml elasticsearch:/usr/share/elasticsearch/config/elasticsearch.yml #
 ```
4.重启容器 
 ```shell
docker restart elasticsearch  
 ```

### elasticsearch 安装插件ik分词器：

1.进入容器
 ```shell
docker exec -it elasticsearch /bin/bash
 ```
2.安装插件
 ```shell
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v5.6.12/elasticsearch-analysis-ik-5.6.12.zip
 ```
3.退出容器
 ```shell
exit
 ```
4.重启容器
 ```shell
docker restart elasticsearch
 ```

# 3.docker win10 环境下常见问题处理

win10 环境下, 安装 docker 需要创建一个 linux 的虚拟环境, 目前有两种方式：Hyper-V 和 WSL2 ; 二者只需要启用一个即可, 其中 WSL2 的性能较好, 优先级较高.

启用 Hyper-V 后, [如何修改虚拟硬盘的存放位置](https://www.cnblogs.com/TurboWay/p/12923814.html)

启用 WSL2 后, [修改虚拟硬盘的存放位置](https://www.cnblogs.com/xhznl/p/13184398.html)

启用 WSL2 后, [启动报错处理](https://www.cnblogs.com/MysticBoy/p/13066611.html)