## 常用命令

| 命令                         | 说明               |
| ---------------------------- | ------------------ |
| yarn application --help      | 帮助信息           |
| yarn node -list              | 查看节点列表       |
| yarn application -list       | 查看正在运行的 app |
| yarn application -kill appid | 杀死 app           |

```shell
# 杀死所有 app
for i in  `yarn application  -list | awk '{print $1}' | grep application_`; do yarn  application -kill $i; done
```



## CDH 的 YARN 实例作用

| 实例名称          | 作用                                                         |
| ----------------- | ------------------------------------------------------------ |
| ResourceManager   | 资源管理器，负责整个系统的资源管理和分配调度，处理客户端请求 |
| NodeManager       | 节点管理器，分配具体的 Container 给应用，监控并报告 Container 使用信息给 ResourceManager |
| JobHistory Server | 作业历史服务                                                 |



## YARN 的 调度器

| 实例名称                            | 作用                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| FIFO Scheduler                      | 队列调度器，先进先出                                         |
| Capacity Scheduler                  | 容量管理器，使用专门的队列运行小任务，预先占用一定的集群资源 |
| Fair Scheduler（ CDH 默认的调度器） | 公平调度器，动态的调整系统资源，多任务时公平的共享集群资源   |



## YARN工作原理简述

![image-20210303000338144](https://gitee.com/TurboWay/blogimg/raw/master/img/image-20210303000338144.png)

## YARN工作原理详述


![image-20210303000446736](https://gitee.com/TurboWay/blogimg/raw/master/img/image-20210303000446736.png)