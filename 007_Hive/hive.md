# Hive Configuration

## 一、Hive 安装部署

1）**把 apache-hive-3.1.2-bin.tar.gz上传到服务器的 /opt/software 目录下**

2）**解压 apache-hive-3.1.2-bin.tar.gz 到 /opt/module/ 目录下面**

`tar -zxvf /opt/software/apache-hive-3.1.2-bin.tar.gz -C /opt/module/`

3）**修改 apache-hive-3.1.2-bin.tar.gz 的名称为 hive**

`mv /opt/module/apache-hive-3.1.2-bin/ /opt/module/hive`

4）**修改 /etc/profile.d/my_env.sh，添加环境变量**

```bash
[quakewang@hadoop102 software]$ sudo vim /etc/profile.d/my_env.sh

# HIVE_HOME
export HIVE_HOME=/opt/module/hive
export PATH=$PATH:$HIVE_HOME/bin

```

刷新环境变量

`source /etc/profile.d/my_env.sh`

5）**解决日志 Jar 包冲突，进入 /opt/module/hive/lib 目录**

`[quakewang@hadoop102 lib]$ mv log4j-slf4j-impl-2.10.0.jar log4j-slf4j-impl-2.10.0.jar.bak`

## 二、Hive 数据源配置到 MySQL

### 1、拷贝驱动

将 MySQL 的 JDBC 驱动拷贝到 Hive 的 lib 目录下。

`[quakewang@hadoop102 lib]$ cp /opt/software/mysql-connector-java-5.1.27.jar /opt/module/hive/lib/`

### 2、配置 MySQL 作为元数据存储

在 $HIVE_HOME/conf 目录下新建 hive-site.xml 文件。

添加如下内容：

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
    </property>

    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>

    <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>

    <property>
    <name>hive.server2.thrift.port</name>
    <value>10000</value>
    </property>

    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>hadoop102</value>
    </property>

    <property>
        <name>hive.metastore.event.db.notification.api.auth</name>
        <value>false</value>
    </property>
    
    <property>
        <name>hive.cli.print.header</name>
        <value>true</value>
    </property>

    <property>
        <name>hive.cli.print.current.db</name>
        <value>true</value>
    </property>
</configuration>
```

## 三、启动 Hive

### 1、初始化源数据库

1）**登陆 MySQL**

`[quakewang@hadoop102 conf]$ mysql -uroot -p123456`

2）**新建 Hive 元数据库**

```mysql
mysql> create database metastore;
mysql> quit;
```

3）**初始化 Hive 元数据库**

`[quakewang@hadoop102 conf]$ schematool -initSchema -dbType mysql -verbose`

### 2启动 hive 客户端

1）**启动 Hive 客户端**

`[quakewang@hadoop102 hive]$ bin/hive`

2）查看一下数据库

```sql
hive (default)> show databases;
OK
database_name
default
```

## 四、修改元数据库字符集

Hive 元数据库的字符集默认为 Latin1，由于其不支持中文字符，故若建表语句中包含中文注释，会出现乱码现象。如需解决乱码问题，须做以下修改。

1）**修改 Hive 元数据库中存储注释的字段的字符集为 utf-8**

（1）字段注释

```mysql
mysql> alter table COLUMNS_V2 modify column COMMENT varchar(256) character set utf8;
```

（2）表注释

```mysql
mysql> alter table TABLE_PARAMS modify column PARAM_VALUE mediumtext character set utf8;
```

2）**修改 hive-site.xml 中 JDBC URL，如下**

```xml
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8</value>
    </property>
```

