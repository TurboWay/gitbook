### 安装

```shell
yum install python-setuptools && easy_install pip
pip install shadowsocks
```

### 启动

```shell
ssserver -p 443 -k 'xg123!@#' -m aes-256-cfb -d start
```

### 停止

```shell
ssserver -p 443 -k 'xg123!@#' -m aes-256-cfb -d stop
```

### 添加到开机启动

```shell
vi /etc/rc.d/rc.local

ssserver -p 443 -k 'xg123!@#' -m aes-256-cfb -d start

chmod +x /etc/rc.d/rc.local

reboot
```

tip: 阿里云服务器的话需要配置 安全组规则