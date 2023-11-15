# ClickHouse Configuration

## 1. 准备工作

### 1.1 确保防火墙处于关闭状态

```shell
# 查看防火墙状态
$ sudo systemctl status firewalld
# 关闭防火墙
$ sudo systemctl disable firewalld
```

### 1.2 取消打开文件数限制

1）在 hadoop102 的 `/etc/security/limits.conf` 文件的末尾加入以下内容：

*注意：以下操作会修改 Linux 系统配置，如果操作不当可能导致虚拟机无法启动，建议在执行以下操作之前给三台虚拟机分别打个快照。（快照拍摄需要在关机状态下执行）*

```shell
[quakewang@hadoop102 ~]$ sudo vim /etc/security/limits.conf

* soft nofile 65536 
* hard nofile 65536 
* soft nproc 131072 
* hard nproc 131072
```

2）在hadoop102的 `/etc/security/limits.d/20-nproc.conf` 文件的末尾加入以下内容：

```shell
[quakewang@hadoop102 ~]$ sudo vim /etc/security/limits.d/20-nproc.conf

* soft nofile 65536 
* hard nofile 65536 
* soft nproc 131072 
* hard nproc 131072
```

3）执行同步操作

```shell
[quakewang@hadoop102 ~]$ sudo /home/quakewang/bin/xsync /etc/security/limits.conf
[quakewang@hadoop102 ~]$ sudo /home/quakewang/bin/xsync /etc/security/limits.d/20-nproc.conf
```

### 1.3 安装依赖

集群中所有节点都需要执行如下命令：

```shell
[quakewang@hadoop102 ~]$ sudo yum install -y libtool
[quakewang@hadoop102 ~]$ sudo yum install -y *unixODBC*	
```

### 1.4 取消SELINUX 

1）修改 `/etc/selinux/config` 中的 SELINUX=disabled

```shell
[quakewang@hadoop102 ~]$ sudo vim /etc/selinux/config 
SELINUX=disabled
```

*注意：别改错了*

2）执行同步操作

```shell
[quakewang@hadoop102 ~]$ sudo /home/quakewang/bin/xsync /etc/selinux/config
```

3）重启三台服务器

```shell
$ sudo reboot
```

### 1.5 下载

下载地址：21.9.4: https://github.com/ClickHouse/ClickHouse/releases/tag/v21.9.4.35-stable

```shell
[quakewang@hadoop102 ~]$ cd /opt/module/clickhouse
#common-static
[quakewang@hadoop102 clickhouse]$ wget https://github.com/ClickHouse/ClickHouse/releases/download/v21.9.4.35-stable/clickhouse-common-static-21.9.4.35.tgz
# common-static-dbg
[quakewang@hadoop102 clickhouse]$ wget https://github.com/ClickHouse/ClickHouse/releases/download/v21.9.4.35-stable/clickhouse-common-static-dbg-21.9.4.35.tgz
# server
[quakewang@hadoop102 clickhouse]$ wget https://github.com/ClickHouse/ClickHouse/releases/download/v21.9.4.35-stable/clickhouse-server-21.9.4.35.tgz
# client
[quakewang@hadoop102 clickhouse]$ wget https://github.com/ClickHouse/ClickHouse/releases/download/v21.9.4.35-stable/clickhouse-client-21.9.4.35.tgz
```

## 2. 安装

```shell
# 解压
[quakewang@hadoop102 clickhouse]$ tar -zxvf clickhouse-common-static-21.9.4.35.tgz
# 运行 doinst.sh 
[quakewang@hadoop102 clickhouse]$ ./clickhouse-common-static-21.9.4.35/install/doinst.sh 

[quakewang@hadoop102 clickhouse]$ tar -zxvf clickhouse-common-static-dbg-21.9.4.35.tgz 
[quakewang@hadoop102 clickhouse]$ ./clickhouse-common-static-dbg-21.9.4.35/install/doinst.sh

[quakewang@hadoop102 clickhouse]$ tar -zxvf clickhouse-server-21.9.4.35.tgz
[quakewang@hadoop102 clickhouse]$ ./clickhouse-server-21.9.4.35/install/doinst.sh
 
[quakewang@hadoop102 clickhouse]$ tar -zxvf clickhouse-client-21.9.4.35.tgz
[quakewang@hadoop102 clickhouse]$./clickhouse-client-21.9.4.35/install/doinst.sh
```

在解压 clickhouse-server-21.9.4.35.tgz 并运行 `./clickhouse-server-21.9.4.35/install/doinst.sh`后，ClickHouse 会默认创建一个 default 的用户，让你设置密码，不设置密码可以按回车。

## 3. 启动

```shell
# 查看命令
[quakewang@hadoop102 clickhouse]$ clickhouse --help 
 
# 启动
[quakewang@hadoop102 clickhouse]$ clickhouse start 

# 连接 ClickHouse
[quakewang@hadoop102 clickhouse]$ clickhouse-client

# -m 支持多行语句
clickhouse-client -m
```

## 4. ClickHouse 配置目录

```shell
# 命令目录
$ cd /usr/bin

$ ll |grep clickhouse

# 配置文件目录
$ cd /etc/clickhouse-server/

# 日志目录
$ cd /var/log/clickhouse-server/
 
# 数据文件目录
$ cd /var/lib/clickhouse/
```

## 5. 配置远程访问

ClickHouse 默认不允许远程访问，需要修改配置文件。

```shell
$ cd /etc/clickhouse-server/
 
$ vim config.xml
```

把 <listen_host>::</listen_host> 的注释打开，才能让 ClickHouse 被除本机以外的服务器访问。

修改完成之后，注意重启 `clickhouse restart`。

在这个文件中，有 ClickHouse 的一些默认路径配置，比较重要的

- 数据文件路径：<path>/var/lib/clickhouse/</path>
- 日志文件路径：<log>/var/log/clickhouse-server/clickhouse-server.log</log>