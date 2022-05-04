# MySQL Configuration

## 一、准备工作

（1）卸载自带的 Mysql-libs（如果之前安装过 mysql，要全都卸载掉）

`rpm -qa | grep -i -E mysql\|mariadb | xargs -n1 sudo rpm -e --nodeps`

（2）将安装包和 JDBC 驱动上传到 /opt/software，共计 6 个

```
mysql-community-common-5.7.16-1.el7.x86_64.rpm
mysql-community-libs-5.7.16-1.el7.x86_64.rpm
mysql-community-libs-compat-5.7.16-1.el7.x86_64.rpm
mysql-community-client-5.7.16-1.el7.x86_64.rpm
mysql-community-server-5.7.16-1.el7.x86_64.rpm
mysql-connector-java-5.1.27-bin.jar
```

## 二、安装 MySQL

（1）安装 mysql 依赖

```bash
[quakewang@hadoop102 software]$ sudo rpm -ivh mysql-community-common-5.7.16-1.el7.x86_64.rpm
[quakewang@hadoop102 software]$ sudo rpm -ivh mysql-community-libs-5.7.16-1.el7.x86_64.rpm
[quakewang@hadoop102 software]$ sudo rpm -ivh mysql-community-libs-compat-5.7.16-1.el7.x86_64.rpm
```

（2）安装 mysql-client

`[quakewang@hadoop102 software]$ sudo rpm -ivh mysql-community-client-5.7.16-1.el7.x86_64.rpm`

（3）安装 mysql-server

`[quakewang@hadoop102 software]$ sudo rpm -ivh mysql-community-server-5.7.16-1.el7.x86_64.rpm`

（4）启动 mysql

`[quakewang@hadoop102 software]$ sudo systemctl start mysqld`

（5）查看 mysql 密码

`[quakewang@hadoop102 software]$ sudo cat /var/log/mysqld.log | grep password`

## 三、配置 MySQL

配置只要是 root 用户+密码，在任何主机上都能登录 MySQL 数据库。

（1）**用刚刚查到的密码进入 mysql（如果报错，给密码加单引号）**

`[quakewang@hadoop102 software]$ mysql -uroot -p'password'`

（2）**设置复杂密码(由于mysql密码策略，此密码必须足够复杂)**

`mysql> set password=password("Qs23=zs32");`

（3）**更改mysql密码策略**

```mysql
mysql> set global validate_password_length=4;
mysql> set global validate_password_policy=0;
```

（4）**设置简单好记的密码**

```mysql
mysql> set password=password("123456");
```

（5）**进入 msyql 库**

``` mysql
mysql> use mysql
```

（6）**查询 user 表**

```mysql
mysql> select user, host from user;
```

（7）**修改 user 表，把 Host 表内容修改为%**

```mysql
mysql> update user set host="%" where user="root";
```

（8）**刷新**

```mysql
mysql> flush privileges;
```

（9）**退出**

```mysql
mysql> quit;
```