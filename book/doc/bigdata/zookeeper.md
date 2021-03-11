[TOC]


cdh zookeeper 命令目录
```shell
find / -name  zkCli.sh
cd /opt/cloudera/parcels/CDH-5.16.1-1.cdh5.16.1.p0.3/lib/zookeeper/bin
sh zkCli.sh 
```

## 常用命令

| 命令                                                         | 说明                 |
| :----------------------------------------------------------- | :------------------- |
| help                     | 获取帮助      |
| ls /                     | 查看节点内容     |
| ls2 /                     | 查看节点详细内容      |
| create /tmp_test 123        | 创建节点      |
| create -e /tmp_test 123        | 创建临时节点（重启或者超时节点会消失）      |
| create -s /tmp_test 123        | 创建带序号的节点      |
| create /tmp_test/123 999        | 创建子节点      |
| get /tmp_test                  | 读取节点      |
| set /tmp_test 456        | 修改节点值      |
| stat /tmp_test        | 查看节点状态      |
| delete /tmp_test                       | 删除节点      |
| rmr /tmp_test                       | 递归删除      |
| history                     | 查看历史执行过的命令      |
| quit                     | 退出      |

