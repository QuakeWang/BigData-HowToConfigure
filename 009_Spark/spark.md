# Spark Configuration

## 1. Local 模式

### 1.1 安装

本地模式，简单来说就是不需要其他任何节点资源就可以在本地执行 Spark 代码的环境。此模式下 Spark 可以做到开箱即用，不依赖于 Hadoop 集群环境，直接在本地启动多个线程，通常是为了方便调试。本地模式分为三类：

- local：只启动一个 executor
- local[k]：启动 k 个executor
- local[*]：启动和 CPU 核数目相同的 executor

```shell
# 解压并重命名
[quakewang@hadoop102 software]$ tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz -C /opt/module
[quakewang@hadoop102 software]$ cd /opt/module
[quakewang@hadoop102 module]$ mv spark-3.0.0-bin-hadoop3.2 spark-local
# 启动 spark-shell
[quakewang@hadoop102 module]$ spark-local/bin/spark-shell 
```

### 1.2 命令行模式

进行上述解压之后，Spark local 模式就已经可以正常运行了，可以使用以下命令进入 spark-shell 命令行进行测试，这里以常见的 WordCount 为例：

```shell
# 创建目标文件，并输入以下内容
[quakewang@hadoop102 module]$ vim spark-local/data/word.txt
hello world
hello spark
spark yarn
spark local
goodbye world

# 启动 spark-shell
[quakewang@hadoop102 module]$ spark-local/bin/spark-shell 
```

```scala
# 执行 WordCount
scala> sc.textFile("data/word.txt").flatMap(_.split(" ")).map((_, 1)).reduceByKey(_ + _).collect
res0: Array[(String, Int)] = Array((hello,2), (yarn,1), (world,2), (goodbye,1), (spark,3), (local,1))
```

### 1.3 提交运行程序

```shell
[quakewang@hadoop102 spark-local]$ bin/spark-submit --class org.apache.spark.examples.SparkPi --master local examples/jars/spark-examples_2.12-3.0.0.jar 10
```

- class 表示要执行程序的主类，此处可以置换为自己需要执行的应用程序；
- master local[2]：部署模式，默认为本地模式，数字表示分配的虚拟 CPU 核数量；
- spark-examples_2.12-3.0.0.jar 运行的应用类所在的 jar 包，实际使用可以置换为自己需要执行的 jar 包；
- 数字 10：表示程序的入口参数，用于设定当前应用的任务数量。

## 2. Standalone 模式

本地模式虽然部署方便，但实际工作中还是要将应用提交到对应的集群中执行，推荐使用 Standalone 和 Yarn 模式，先来看 Standalone 模式。

Standalone 模式是只使用 Spark 自身节点运行的集群模式，而且充分体现了经典的 master-slave 模式。

集群规划：

|       |      hadoop102      | hadoop103 | hadoop104 |
| :---: | :-----------------: | :-------: | :-------: |
| Spark | **Master**   Worker |  Worker   |  Worker   |

### 2.1 解压文件

将 spark-3.0.0-bin-hadoop3.2.tgz 文件上传到 Linux 并解压缩在指定位置。

```shell
[quakewang@hadoop102 software]$ tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz -C /opt/module
[quakewang@hadoop102 software]$ cd /opt/module
[quakewang@hadoop102 module]$ mv spark-3.0.0-bin-hadoop3.2 spark-standalone
```

### 2.2 修改配置文件

1）进入解压缩后路径的 conf 目录，修改 slaves.template 文件名为 slaves

```shell
$ mv slaves.template slaves
```

2）修改 slaves 文件，并添加如下内容：

```
hadoop102
hadoop103
hadoop104
```

3）修改 spark-env.sh.template 文件名为 spark-env.sh

```shell
$ mv spark-env.sh.template spark-env.sh
```

4）修改 spark-env.sh 文件，添加 JAVA_HOME 环境变量和集群对应的 master 节点

```sh
export JAVA_HOME=/opt/module/jdk1.8.0_144
SPARK_MASTER_HOST=hadoop102
SPARK_MASTER_PORT=7077
```

*注意：7077 端口，相当于 hadoop3 内部通信的 8020 端口，此处的端口需要确认自己的 Hadoop
配置*
5）分发 spark-standalone 目录

```shell
[quakewang@hadoop102 module]$ xsync spark-standalone
```

### 2.3 启动集群

1）执行脚本命令

```shell
$ sbin/start-all.sh
```

2）查看集群服务进程

```shell
================hadoop102================
3330 Jps
3238 Worker
3163 Master
================hadoop103================
2966 Jps
2908 Worker
================hadoop104================
2978 Worker
3036 Jps
```

3）访问 Master 资源监控 WebUI 界面`hadoop102:8080`

### 2.4 配置历史服务器

由于 spark-shell 停止掉后，集群监控 hadoop102:4040 页面就看不到历史任务的运行情况，所以
开发时都配置历史服务器记录任务运行情况。
1）修改 spark-defaults.conf.template 文件名为 spark-defaults.conf

```shell
$ mv spark-defaults.conf.template spark-defaults.conf
```

2）修改 spark-default.conf 文件，配置日志存储路径

```conf
spark.eventLog.enabled		true
spark.eventLog.dir			hdfs://hadoop102:8020/spark-history
```

*注意：需要启动 hadoop 集群，HDFS 上的 spark-history 目录需要提前存在*。

```shell
$ sbin/start-dfs.sh
$ hadoop fs -mkdir /directory
```

3）修改 spark-env.sh 文件, 添加日志配置

```sh
export SPARK_HISTORY_OPTS="
-Dspark.history.ui.port=18080
-Dspark.history.fs.logDirectory=hdfs://hadoop102:8020/spark-history
-Dspark.history.retainedApplications=30"
```

- 参数 1 含义：WEB UI 访问的端口号为 18080；
- 参数 2 含义：指定历史服务器日志存储路径；
- 参数 3 含义：指定保存 Application 历史记录的个数，如果超过这个值，旧的应用程序信息将被删除，这个是内存中的应用数，而不是页面上显示的应用数。

4）分发配置文件

```shell
[quakewang@hadoop102 spark-standalone]$ xsync conf
```

5）重新启动集群和历史服务

```shell
[quakewang@hadoop102 spark-standalone]$ sbin/start-all.sh
[quakewang@hadoop102 spark-standalone]$ sbin/start-history-server.sh
```

### 2.5 提交运行程序

```shell
[quakewang@hadoop102 spark-standalone]$ bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop102:7077 \
./examples/jars/spark-examples_2.12-3.0.0.jar \
10
```

## 3. YARN 模式

独立部署（Standalone）模式由 Spark 自身提供计算资源，无需其他框架提供资源。这种方式降低了和其他第三方资源框架的耦合性，独立性非常强。但 Spark 主要是计算框架，而不是资源调度框架，所以本身提供的资源调度并不是它的强项，所以通常会和其他资源调度框架放在一起使用，这里选取 YARN 为例。

### 3.1 解压文件

```shell
[quakewang@hadoop102 software]$ tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz -C /opt/module
[quakewang@hadoop102 software]$ cd /opt/module
[quakewang@hadoop102 module]$ mv spark-3.0.0-bin-hadoop3.2 spark-yarn
```

### 3.2 修改配置文件

1）修改 hadoop 配置文件 `/opt/module/hadoop/etc/hadoop/yarn-site.xml`，并分发

```xml
<!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认
是 true -->
<property>
    <name>yarn.nodemanager.pmem-check-enabled</name>
    <value>false</value>
</property>
<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认
是 true -->
<property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
</property>
```

2）修改 `conf/spark-env.sh`，添加 JAVA_HOME 和 YARN_CONF_DIR 配置

```sh
export JAVA_HOME=/opt/module/jdk1.8.0_144
YARN_CONF_DIR=/opt/module/hadoop/etc/hadoop
```

3）修改 spark-default.conf 配置文件

```conf
spark.master                             yarn
spark.eventLog.enabled                   true
spark.eventLog.dir                       hdfs://hadoop102:8020/spark-history
spark.executor.memory                    1g
spark.driver.memory					   	 1g
spark.yarn.historyServer.address=hadoop102:18080
spark.history.ui.port=18080
```

4）复制 yarn-site.xml 至 conf 目录下

```shell
[quakewang@hadoop102 spark-yarn]$ cp /opt/module/hadoop-3.1.3/etc/hadoop/yarn-site.xml conf/
```

### 3.3 配置历史服务器

同上。

### 3.4 提交运行程序

```shell
[quakewang@hadoop102 spark-yarn]$ bin/spark-submit --class org.apache.spark.examples.SparkPi --master yarn examples/jars/spark-examples_2.12-3.0.0.jar 10
```



