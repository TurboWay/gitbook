[TOC]

## 一、环境准备

###  1.0  改密码， 如果有必要的话

```shell
passwd
```

###  1.1  设置好磁盘挂载点  

```shell
fdisk -l     # 查看磁盘大小
df -Th     # 查看磁盘分配的挂载点
mkfs.ext4  /dev/sdb     # 磁盘格式化
mkdir /data      # 创建挂载点          
mount /dev/sdb  /data   # 将磁盘分配到挂载点
vi /etc/fstab   # 自动挂载，不设置的话，重启就需要重新挂载
```

### 1.2 设置联网，如果不能联网的话

```shell
vi /etc/resolv.conf     # 修改dns  
nameserver 114.114.114.114
nameserver 8.8.8.8
```

### 1.3 设置机器名( 改完重启，貌似没有生效，算了先不改了)

```shell
vi /etc/sysconfig/network  # 修改机器名，增加 HOSTNAME=gfdatanode05
```

### 1.4 设置hosts

``` shell
vi /etc/hosts     #  将新增节点加到各台机器hosts
172.16.122.17   gfdatanode01
172.16.122.18   gfdatanode02
172.16.122.19   gfdatanode03
172.16.122.20   gfdatastandby
172.16.122.21   gfmaster
172.16.122.22   gfdatanode05
172.16.122.23   gfdatanode06
172.16.122.24   gfdatanode07
172.16.122.25   gfdatanode08
172.16.122.26   gfdatanode09
```

### 1.5 设置 ssh免密

```shell
ssh-keygen -t rsa    # 生成公钥
cd .ssh
cat id_rsa.pub >> authorized_keys   # 添加公钥免密 
chmod 600 authorized_keys # 必须600权限
# id_rsa.pub 添加到所有机器的 authorized_keys
# known_hosts 顺便也加下
```

### 1.6 关闭SElinux

```shell
getenforce   # 查看是否开启，默认开启
vi /etc/selinux/config    # 修改 SELINUX=disabled， 重启生效
```
### 1.7 关闭防火墙
```shell
service firewalld  status                # 查看防火墙
service firewalld  stop                  # 关闭防火墙      
systemctl disable firewalld.service    # 禁止防火墙开机启动
```

## 二、 Cloudera Manager安装【<a href="/doc/cdh安装.pdf" target="_blank">cdh 安装笔记</a>】

### 2.0   创建CM目录

```shell 
mkdir /opt/cloudera-manager     # 创建CM目录
mkdir /opt/soft        # 创建软件存放目录
```

### 2.1    在master节点分发安装包

```shell
scp /opt/soft/cloudera-manager-centos7-cm5.16.1_x86_64.tar.gz root@gfdatanode05:/opt/soft      # 在master执行scp吧安装包传到新节点
```

### 2.2   安装包解压

```shell       
tar -zxvf /opt/soft/cloudera-manager-centos7-cm5.16.1_x86_64.tar.gz -C /opt/cloudera-manager/
```

### 2.3   修改CM master节点主机名

```shell
sed -i "s/server_host=localhost/server_host=gfdatastandby/g" /opt/cloudera-manager/cm-5.16.1/etc/cloudera-scm-agent/config.ini   
```

### 2.4   创建cloudera-scm账号，这是CM相关服务使用的默认账号

```shell
useradd --system --home=/opt/cloudera-manager/cm-5.16.1/run/cloudera-scm-server/ --no-create-home --shell=/bin/false --comment "Cloudera SCM User" cloudera-scm    # 创建用户
chown -R cloudera-scm:cloudera-scm /opt/cloudera-manager # 修改权限
```

### 2.5  设置系统服务slave

```shell
cp /opt/cloudera-manager/cm-5.16.1/etc/init.d/cloudera-scm-agent /etc/rc.d/init.d/    
chown cloudera-scm:cloudera-scm /etc/rc.d/init.d/cloudera-scm-agent
```

```shell
vim /etc/rc.d/init.d/cloudera-scm-agent   # 修改路径：
注释掉默认路径，修改为如下
#CMF_DEFAULTS=${CMF_DEFAULTS:-/etc/default}
CMF_DEFAULTS=/opt/cloudera-manager/cm-5.16.1/etc/default
```

```shell
# 添加系统启动服务 
chkconfig --add cloudera-scm-agent 
chkconfig --level 35 cloudera-scm-agent on 
chkconfig --list
```

### 2.6   创建软件安装目录 slave 

```shell
mkdir -p /opt/cloudera/parcels
chown -R cloudera-scm:cloudera-scm /opt/cloudera/
```

### 2.7    安装软件 psmisc

```shell
yum install psmisc -y
```

### 2.8     启动agent服务

```shell
service cloudera-scm-agent restart        # 启动服务
service cloudera-scm-agent status -l      # 查看状态
```

## 三、打开CM web界面

3.1  进入 “所有主机”，可以看到 新节点 已经纳入管理

3.2  点击右上角的 向集群添加新主机，不用搜索主机，直接点击下一步，可以看到新节点

3.3 跟着向导，安装选定 Parcel 等待 。。。2,3分钟后完成

3.4  继续跟着向导， 无主机模板（目前没有设），继续...，，然后到完成

其中会有检查主机，遇到问题处理：

问题1：Cloudera 建议将 /proc/sys/vm/swappiness 设置为最大值 10。当前设置为 30

```shell
echo 10 > /proc/sys/vm/swappiness                   # 设置为10
echo 'vm.swappiness=10'>> /etc/sysctl.conf      # 保存配置
```
问题2： IOException thrown while collecting data from host  无法路由到主机

```shell
# 需要关闭防火墙
service firewalld  status                # 查看防火墙
service firewalld  stop                  # 关闭防火墙      
systemctl disable firewalld.service    # 禁止防火墙开机启动
```

问题3：已启用透明大页面压缩，可能会导致重大性能问题

```shell
echo never > /sys/kernel/mm/transparent_hugepage/defrag
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo "echo never > /sys/kernel/mm/transparent_hugepage/defrag" >> /etc/rc.local
echo "echo never > /sys/kernel/mm/transparent_hugepage/enabled" >> /etc/rc.local
chmod +x /etc/rc.d/rc.local   
```


- 参考
[CDH 添加节点教程](https://www.cnblogs.com/chevin/p/10117806.html)
[CDH 安装教程](https://www.aboutyun.com//forum.php/?mod=viewthread&tid=9190&extra=page%3D1&page=1&)





