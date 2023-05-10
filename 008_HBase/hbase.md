# HBase Configuration

## 一、准备工作

在安装配置 HBase 之前，需要保证 Hadoop 和 Zookeeper 集群能够正常运行。

1）**把 hbase-2.4.11-bin.tar.gz上传到服务器的 /opt/software 目录下**

2）**解压 hbase-2.4.11-bin..tar.gz 到 /opt/module/ 目录下面**

`tar -zxvf /opt/software/hbase-2.4.11-bin.tar.gz -C /opt/module/`

3）**修改 apache-hive-3.1.2-bin.tar.gz 的名称为 hive**

`mv /opt/module/hbase-2.4.11-bin/ /opt/module/hbase`

4）**修改 /etc/profile.d/my_env.sh，添加环境变量**

```shell
[quakewang@hadoop102 software]$ sudo vim /etc/profile.d/my_env.sh

# HBASE_HOME
export HBASE_HOME=/opt/module/hbase
export PATH=$PATH:$HBASE_HOME/bin
```

刷新环境变量

```
source /etc/profile.d/my_env.sh
```

## 二、配置文件

1）hbase-env.sh 修改内容，可以添加到最后：

`export HBASE_MANAGES_ZK=false`

2）hbase-site.xml 修改内容

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
	<property> 
  	<name>hbase.zookeeper.quorum</name>
    <value>hadoop102,hadoop103,hadoop104</value>
    <description>The directory shared by RegionServers.</description>
  </property>
  <!--
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/export/zookeeper</value>
    <description> 记得修改 ZK 的配置文件,ZK 的信息不能保存到临时文件夹</description>
  </property>
  -->
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://hadoop102:8020/hbase</value>
    <description>The directory shared by RegionServers. </description>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
</configuration>
```

3）regionservers

```txt
hadoop102
hadoop103
hadoop104
```

4）解决 HBase 和 Hadoop 的 log4j 兼容性问题，修改 HBase 的 jar 包，使用 Hadoop 的 jar 包

```sh
[quakewang@hadoop102 hbase]$ mv /opt/module/hbase/lib/client-facing- thirdparty/slf4j-reload4j-1.7.33.jar /opt/module/hbase/lib/client- facing-thirdparty/slf4j-reload4j-1.7.33.jar.bak
```

5）分发集群

```sh
[quakewang@hadoop102 module]$ xsync hbase/
```

## 三、HBase 服务的启动

1）单点启动

```sh
[quakewang@hadoop102 hbase]$ bin/hbase-daemon.sh start master 
[quakewang@hadoop102 hbase]$ bin/hbase-daemon.sh start regionserver
```

2）集群启动/停止

```sh
[quakewang@hadoop102 hbase]$ bin/start-hbase.sh
[quakewang@hadoop102 hbase]$ bin/stop-hbase.sh
```

3）查看 HBase web 页面

启动成功后，可以通过 “host:port” 的方式来访问 HBase 管理页面，例如：http://hadoop102:16010