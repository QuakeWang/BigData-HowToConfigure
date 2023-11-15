# 001 Hadoop Configuration

Hadoop 所选用的版本为 **hadoop-3.1.3**

## 一、集群部署规划



|      | hadoop102              | hadoop103                        | hadoop104                       |
| ---- | ---------------------- | -------------------------------- | ------------------------------- |
| HDFS | NameNode<br />DataNode | DataNode                         | SecondaryNameNode<br />DataNode |
| YARN | NodeManager            | ResourceManager<br />NodeManager | NodeManager                     |

注 1：NameNode 和 SecondaryNameNode 不要安装在同一台服务器上。

注 2：ResourceManager 也很消耗内存，不要和 NameNode、SecondaryNameNode 配置在同一台服务器上。

## 二、准备工作

### 1）上传、解压 tar 包

进入安装目录：`cd /opt/software/`

解压文件：`tar -zxvf hadoop-3.1.3.tar.gz -C /opt/module/`

### 2）将 Hadoop 添加到环境变量

① 获取 Hadoop 安装路径：`pwd   /opt/module/hadoop-3.1.3`

②打开 /etc/profile.d/my_env.sh 文件：`sudo vim /etc/profile.d/my_env.sh`

③ 在 profile 文件末尾添加 Hadoop 路径：（shitf+g）

```shell
# HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```

④ 分发环境变量文件

`sudo /home/quakewang/bin/xsync /etc/profile.d/my_env.sh`

## 三、配置集群

进入目录：`cd $HADOOP_HOME/etc/hadoop`

### 1、核心配置文件 core-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	<!-- 指定 NameNode 的地址 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop102:8020</value>
	</property>
	<!-- 指定 Hadoop 数据的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/module/hadoop-3.1.3/data</value>
	</property>

	<!-- 配置 HDFS 网页登录使用的静态用户为 quakewang -->
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>quakewang</value>
	</property>

	<!-- 配置该 quakewang(superUser) 允许通过代理访问的主机节点 -->
    <property>
        <name>hadoop.proxyuser.quakewang.hosts</name>
        <value>*</value>
	</property>
	<!-- 配置该 quakewang(superUser) 允许通过代理用户所属组 -->
    <property>
        <name>hadoop.proxyuser.quakewang.groups</name>
        <value>*</value>
	</property>
	<!-- 配置该 quakewang(superUser) 允许通过代理的用户-->
    <property>
        <name>hadoop.proxyuser.quakewang.users</name>
        <value>*</value>
	</property>
</configuration>
```

### 2、HDFS 配置文件 hdfs-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	<!-- nn web 端访问地址-->
	<property>
        <name>dfs.namenode.http-address</name>
        <value>hadoop102:9870</value>
    </property>
    
	<!-- 2nn web 端访问地址-->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop104:9868</value>
    </property>
    
    <!-- 测试环境指定 HDFS 副本的数量 1 -->
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
</configuration
```

### 3、YARN 配置文件 yarn-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	<!-- 指定 MR 走 shuffle -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    
    <!-- 指定 ResourceManager 的地址-->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop103</value>
    </property>
    
    <!-- 环境变量的继承 -->
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
    
    <!-- YARN 容器允许分配的最大最小内存 -->
    <property>
        <name>yarn.scheduler.minimum-allocation-mb</name>
        <value>512</value>
    </property>
    <property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>4096</value>
    </property>
    
    <!-- YARN 容器允许管理的物理内存大小 -->
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>4096</value>
    </property>
    
    <!-- 关闭 YARN 对物理内存和虚拟内存的限制检查 -->
    <property>
        <name>yarn.nodemanager.pmem-check-enabled</name>
        <value>false</value>
    </property>
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>
</configuration>
```

### 4、MapReduce 配置文件 mapred-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	<!-- 指定 MapReduce 程序运行在 YARN 上 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

### 5、配置 workers

`vim /opt/module/hadoop-3.1.3/etc/hadoop/workers`

在该文件增加如下内容：

```
hadoop102
hadoop103
hadoop104
```

### 6、配置历史服务器

为了查看程序的历史运行情况，需要配置一下历史服务器。具体配置步骤如下：

#### 配置 mapred-site.xml

`vim mapred-site.xml`

在该文件里面增加如下配置：

```xml
	<!-- 历史服务器端地址 -->
	<property>
		<name>mapreduce.jobhistory.address</name>
		<value>hadoop102:10020</value>
	</property>

	<!-- 历史服务器 Web 端地址 -->
	<property>
	<name>mapreduce.jobhistory.webapp.address</name>
		<value>hadoop102:19888</value>
	</property>
```

### 7、配置日志的聚集

日志聚集概念：应用运行完成以后，将程序运行日志信息上传到 HDFS 系统上。

日志聚集功能好处：可以方便的查看到程序运行详情，方便开发调试。

注意：开启日志聚集功能，需要重新启动 NodeManager 、ResourceManager 和 HistoryManager。

开启日志聚集功能具体步骤如下：

#### 配置 yarn-site.xml

`vim yarn-site.xml`

在该文件里面增加如下配置：

```xml
	<!-- 开启日志聚集功能 -->
	<property>
		<name>yarn.log-aggregation-enable</name>
		<value>true</value>
	</property>

	<!-- 设置日志聚集服务器地址 -->
	<property>  
		<name>yarn.log.server.url</name>  	<value>http://hadoop102:19888/jobhistory/logs</value>
	</property>

	<!-- 设置日志保留时间为 7 天 -->
	<property>
		<name>yarn.log-aggregation.retain-seconds</name>
		<value>604800</value>
	</property>
```

---

注意：当前配置都是 hadoop102 的服务器上，需要通过脚本同步到集群环境。

`xsync /opt/module/hadoop-3.1.3/`

## 四、启动集群

**如果集群是第一次启动**，需要在hadoop102 节点格式化NameNode（注意格式化之前，一定要先停止上次启动的所有 namenode 和 datanode 进程，然后再删除 data 和 log 数据）

`[quakewang@hadoop102 hadoop-3.1.3]$ bin/hdfs namenode -format`

### 1、启动 HDFS

`[quakewang@hadoop102 hadoop-3.1.3]$ sbin/start-dfs.sh`

### 2、启动 YARN

**在配置了ResourceManager的节点（hadoop103）**

`[quakewang@hadoop103 hadoop-3.1.3]$ sbin/start-yarn.sh`

### 3、Web 端查看 HDFS 的 Web 页面：http://hadoop102:9870/

### 4、Web端查看SecondaryNameNode

（a）浏览器中输入：http://hadoop104:9868/status.html

（b）查看SecondaryNameNode信息

## 五、Hadoop 集群启动脚本

在用户的 bin 目录下创建 Hadoop 集群启动脚本。

`vim myhadoop.sh`

```shell
#!/bin/bash
if [ $# -lt 1 ]
then
    echo "No Args Input..."
    exit ;
fi
case $1 in
"start")
        echo " =================== 启动 Hadoop集群 ==================="

        echo " --------------- 启动 hdfs ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/start-dfs.sh"
        echo " --------------- 启动 yarn ---------------"
        ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/start-yarn.sh"
        echo " --------------- 启动 historyserver ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon start historyserver"
;;
"stop")
        echo " =================== 关闭 Hadoop集群 ==================="

        echo " --------------- 关闭 historyserver ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon stop historyserver"
        echo " --------------- 关闭 yarn ---------------"
        ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/stop-yarn.sh"
        echo " --------------- 关闭 hdfs ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/stop-dfs.sh"
;;
*)
    echo "Input Args Error..."
;;
esac
```

`chmod 777 myhadoop.sh`