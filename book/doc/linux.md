[TOC]

# 1、创建命令

| 命令     | 说明                                                         |
| :------- | :----------------------------------------------------------- |
| mkdir    | 创建文件夹                                                   |
| mkdir -p | 递归创建文件夹                                               |
| touch    | 创建文件                                                     |
| vi       | 编辑文件 (退出先按Esc键，:q! - 强制退出编辑，:wq - 写入后退出) |

# 2、删除命令

| 命令   | 说明           |
| :----- | :------------- |
| rm     | 删除文件       |
| rm -f  | 强制删除文件   |
| rm -r  | 删除文件夹     |
| rm -rf | 强制删除文件夹 |

# 3、查看命令

| 命令                                 | 说明                                                         |
| :----------------------------------- | :----------------------------------------------------------- |
| hostname                             | 查看主机名                                                   |
| cd                                   | 进入文件夹                                                   |
| pwd                                  | 查看当前路径                                                 |
| ll                                   | 逐行展示文件清单（ll -a 展示隐藏文件）                       |
| cat                                  | 查看文件所有内容（一次性输出，cat -A 查看隐藏字符）          |
| more                                 | 翻页查看文件的内容（按空格键翻页，只能往后看）               |
| less                                 | 查看文件的内容（通过上下方向控制显示，q退出，/keyword 查找） |
| head-n 20                            | 查看文件前几行                                               |
| tail -n 20                           | 查看文件后几行（ tail -f 实时查看文件）                      |
| du -sh                               | 查看文件/文件夹大小                                          |
| du -h -x --max-depth=1               | 查看各个文件夹大小                                           |
| ls -lR &#124; grep "^-" &#124; wc -l | 查看文件数                                                   |


# 4、文件复制传输

| 命令                              | 说明                                |
| :-------------------------------- | :---------------------------------- |
| vim                               | 编辑文件（ctrl+b 和 ctrl+f 翻页）   |
| cp                                | 复制文件（cp -r 复制文件夹）        |
| mv                                | 移动文件/文件夹，可以用来重命名     |
| rz                                | 本地上传到 linux （rz -y 上传覆盖） |
| sz                                | linux下载到本地                     |
| scp filename root@ip:/home/root   | 服务器之间传输文件                  |
| scp -p dirname root@ip:/home/root | 服务器之间传输文件（保留源文件的修改时间，访问时间和访问权限。）               |
| scp -r dirname root@ip:/home/root | 服务器之间传输文件夹                |



# 5、压缩解压


tar  -v 显示过程
      -c 打包
      -x 解包
      -z 压缩解压gz

| 命令                                | 说明                     |
| :---------------------------------- | :----------------------- |
| tar -tvf                            | 查看压缩文件内容         |
| tar -cf  work.tar  work             | tar 打包                 |
| tar -xf  work.tar                   | tar 解包                 |
| tar -zcf  work.tar.gz  work         | tar.gz 压缩              |
| tar -zxf  work.tar.gz               | tar.gz 解压              |
| tar -zxf  work.tar.gz  -C /opt/soft | tar.gz 解压到指定路径    |
| tar -tr  work.tar.gz                | 查看压缩包内容           |
| tar -zxf work.tar.gz  work/xxx      | 解压压缩包里面的指定文件 |
| gzip work.txt                       | 压缩成 work.txt.gz |
| gunzip work.txt.gz                  | 解压成 work.txt |
| gunzip -c work.txt.gz               | 输出内容，不解压 |


# 6、vim技巧

| 命令           | 说明                            |
| :------------- | :------------------------------ |
| ctrl+b         | 往上翻页                        |
| ctrl+f         | 往下翻页                        |
| :q!            | 强制退出编辑                    |
| :wq            | 写入后退出                      |
| :%s/dev/prod/g | 全文字符串替换  dev 替换为 prod |
| "$"(Shift+4)   | 跳到行末                        |
| dd             | 删除整行                        |
| gg             | 跳到首行                        |
| :.,$d          | 当前行删到最后一行              |

# 7、记录日志

| 命令                         | 说明                         |
| :--------------------------- | :--------------------------- |
| nohup shell > log.txt 2>&1 & | 后台运行程序(忽视挂起信号)，记录所有日志   |
| setid shell  | 后台运行程序(相当于另起会话,所以不受当前会话影响)   |
| cmd > log                    | 覆盖（目标不存在，自动创建） |
| cmd >> log                   | 追加（目标不存在，自动创建） |
| > /dev/null                  | 不记录日志                   |

# 8、文件查找

| 命令                   | 说明         |
| :--------------------- | :----------- |
| find /  -name filename | 查找文件     |
| whereis mysql          | 查找安装路径 |

# 9、系统状态

| 命令                   | 说明               |
| :--------------------- | :----------------- |
| fdisk -l               | 磁盘信息           |
| df -Th                 | 文件空间           |
| top                    | 进程cpu当前状态    |
| du -h -x --max-depth=1 | 查看各个文件夹大小 |
| lsblk                  | 树形查看磁盘       |

# 10、进程管理

| 命令             | 说明                   |
| :--------------- | :--------------------- |
| ps aux           | 查看进程               |
| ps -ef           | 查找进程               |
| kill -9 pid      | 杀死进程               |
| pkill -f keyword | 根据关键词批量杀死进程 |

# 11、用户权限管理

| 命令                                                         | 说明                                         |
| :----------------------------------------------------------- | :------------------------------------------- |
| who                                                          | 查看登录用户                                 |
| useradd xxx                                                      | 创建用户                                 |
| passwd --stdin getway                                        | 强制修改getway用户密码，无视规则             |
| pkill -kill -t  pts/0 （pts/0为w指令看到的用户终端号）       | 踢用户                                       |
| last                                                         | 查看登录历史记录                             |
| chmod 755 test.py                                            | 修改文件权限                                 |
| id  getway                                                   | 查看用户                                     |
| cat /etc/group                                               | 查看用户组                                   |
| cut -d : -f 1 /etc/passwd                                    | 查看所有用户                                 |
| usermod -a -G hdfs getway                                    | 用户增加组（getway用户追加到hdfs组）         |
| gpasswd -d getway hdfs                                       | 将用户从组中删除（getway用户从hdfs组中删除） |
| cat /etc/passwd &#124; grep -v /sbin/nologin &#124; cut -d : -f 1 | 可以登录系统的用户                           |

# 12、定时计划

| 命令        | 说明                                              |
| :---------- | :------------------------------------------------ |
| crontab  -l | 查看定时计划 [网页工具](https://tool.lu/crontab/) |
| crontab -e  | 编辑定时计划                                      |

# 13、网络管理

| 命令                            | 说明         |
| :------------------------------ | :----------- |
| cat /etc/resolv.conf            | 查看网关解析 |
| netstat -ntlp &#124; grep 10000 | 查看端口占用 |


# 14、软件管理

| 命令                                 | 说明                 |
| :----------------------------------- | :------------------- |
| yum remove xxx                       | yum 卸载xxx软件      |
| yum list installed &#124; grep mysql | yum 查看已安装的软件 |
| rpm -qa &#124; grep -i mysql         | rpm 查看已安装的软件 |

# 15、查看环境变量和自定义命令

| 命令                      | 说明                            |
| :------------------------ | :------------------------------ |
| alias list                | 查看自定义命令                  |
| alias work="cd /workpath" | 自定义命令                      |
| echo $HOME                | 查看变量                        |
| cat ~/.bash_profile       | 查看用户环境变量                |
| cat /etc/profile          | 查看系统环境变量                |
| ln -s b a                 | 创建软链接a ，即创建b的快捷方式 |
| curl cip.cc               | 查看本机公网 ip                 |

# 16、常用 shell 脚本

## 循环

```shell 
for page in `seq 1 22`; do
     python execute_spider.py -j d_zhuanli_quanguo_job -p $page -n 6;
done
```

## 正在运行的命令转入后台运行

```shell
# 1.先暂停这条命令并返回客户端。
CTRL+Z  
# 2.让这条shell命令在后台执行。
bg 
# 3.这条命令保证当终端关闭时，Shell脚本不会被杀死。
disown -h  
```


