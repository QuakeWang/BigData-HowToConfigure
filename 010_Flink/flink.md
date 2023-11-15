# Flink Configuration

## 1. Local 模式

最简单的启动方式，直接本地启动，但并不是集群搭建。本地部署非常简单，直接解压安装包就可以使用，不用进行任何配置；一般用来做一些简单的测试。

具体安装步骤如下：

1. 下载安装包

进入Flink 官网，下载1.14.0 版本安装包 [flink-1.14.0-bin-scala_2.12.tgz](https://archive.apache.org/dist/flink/flink-1.14.0/)，注意此处选用对应 scala 版本为scala 2.12 的安装包。

2. 解压

在 hadoop102 节点服务器上创建安装目录 /opt/module，将 flink 安装包放在该目录下，并执行解压命令，解压至当前目录。

```shell
$ tar -zxvf flink-1.14.0-bin-scala_2.12.tgz -C /opt/module/
```

3. 启动

进入解压后的目录，执行启动命令，并查看进程。

```shell
$ cd flink-1.14.0/

$ bin/start-cluster.sh 
Starting cluster.
Starting standalonesession daemon on host hadoop102.
Starting taskexecutor daemon on host hadoop102.

$ jps
10369 StandaloneSessionClusterEntrypoint
10680 TaskManagerRunner
10717 Jps
```

4. 访问Web UI

启动成功后，访问 http://hadoop102:8081，可以对 flink 集群和任务进行监控管理。

5. 关闭集群

如果想要让 Flink 停止运行，可以执行以下命令：

```shell
$ bin/stop-cluster.sh 
Stopping taskexecutor daemon (pid: 10680) on host hadoop102.
Stopping standalonesession daemon (pid: 10369) on host hadoop102.
```

## 2. 集群模式

Flink 本地启动非常简单，直接执行 start-cluster.sh 就可以了。但是和 Spark Local 模式一样，并不能满足于生产需求。如果我们想要扩展成集群，其实启动命令是不变的，主要是需要指定节点之间的主从关系。

Flink 是典型的 Master-Slave 架构的分布式数据处理框架，其中 Master 角色对应着 JobManager，Slave 角色则对应 TaskManager。集群规划如下：

| 节点服务器 | hadoop102  | hadoop103   | hadoop104   |
| ---------- | ---------- | ----------- | ----------- |
| 角色       | JobManager | TaskManager | TaskManager |

1. 下载并解压安装包

具体操作与上节相同。

2. 修改集群配置

（1）进入 conf 目录下，修改 flink-conf.yaml 文件，修改 jobmanager.rpc.address 参数为 hadoop102，如下所示：

```shell
$ cd conf/

$ vim flink-conf.yaml

# JobManager节点地址.
jobmanager.rpc.address: hadoop102
```

这就指定了 hadoop102 服务器为 JobManager 节点。

（2）修改workers文件，将另外两台节点服务器添加为本 Flink 集群的 TaskManager 节点，具体修改如下：

```shell
$ vim workers 

hadoop103
hadoop104
```

这样就指定了 hadoop103 和 hadoop104 为 TaskManager 节点。

（3）另外，在 flink-conf.yaml 文件中还可以对集群中的 JobManager 和 TaskManager 组件进行优化配置，主要配置项如下：

- jobmanager.memory.process.size：对 JobManager 进程可使用到的全部内存进行配置，包括 JVM 元空间和其他开销，默认为 1600M，可以根据集群规模进行适当调整;
- taskmanager.memory.process.size：对 TaskManager 进程可使用到的全部内存进行配置，包括 JVM 元空间和其他开销，默认为 1600M，可以根据集群规模进行适当调整;
- taskmanager.numberOfTaskSlots：对每个 TaskManager 能够分配的 Slot 数量进行配置，默认为 1，可根据 TaskManager 所在的机器能够提供给 Flink 的 CPU 数量决定。所谓 Slot 就是 TaskManager 中具体运行一个任务所分配的计算资源;
- parallelism.default：Flink 任务执行的默认并行度，优先级低于代码中进行的并行度配置和任务提交时使用参数指定的并行度数量。

3. 分发安装目录

配置修改完毕后，将 Flink 安装目录发给另外两个节点服务器。

```shell
$ xsync flink-1.14.0
```

4. 启动集群

（1）在 hadoop102 节点服务器上执行 start-cluster.sh 启动 Flink 集群：

```shell
$ bin/start-cluster.sh 

Starting cluster.

Starting standalonesession daemon on host hadoop102.

Starting taskexecutor daemon on host hadoop103.

Starting taskexecutor daemon on host hadoop104.
```

（2）查看进程情况：

```shell
[quakewang@hadoop102 flink-1.14.0]$ jps
13859 Jps
13782 StandaloneSessionClusterEntrypoint

[quakewang@hadoop103 flink-1.14.0]$ jps
12215 Jps
12124 TaskManagerRunner

[quakewang@hadoop104 flink-1.14.0]$ jps
11602 TaskManagerRunner
11694 Jps
```

5. 访问 Web UI

启动成功后，同样可以访问 http://hadoop102:8081 对 Flink 集群和任务进行监控管理。

## 3. YARN 模式

YARN 上部署的过程是：客户端把 Flink 应用提交给 YARN 的 ResourceManager，ResourceManager 会向 YARN 的 NodeManager 申请容器。在这些容器上，Flink 会部署 JobManager 和 TaskManager 的实例，从而启动集群。Flink 会根据运行在 JobManger 上的作业所需要的 Slot 数量动态分配 TaskManager 资源。

### 3.1 相关准备和配置

在将 Flink 任务部署至 YARN 集群之前，需要确认集群是否安装有 Hadoop，保证 Hadoop 版本至少在 2.2 以上，并且集群中安装有 HDFS 服务。

具体配置步骤如下：

（1）按照前文所述，下载并解压安装包，并将解压后的安装包重命名为 flink-1.14.0-yarn，本节的相关操作都将默认在此安装路径下执行。

（2）配置环境变量，增加环境变量配置如下：

```shell
$ sudo vim /etc/profile.d/my_env.sh

HADOOP_HOME=/opt/module/hadoop-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export HADOOP_CLASSPATH=`hadoop classpath`
```

（3）启动 Hadoop 集群，包括 HDFS 和 YARN。

```shell
[quakewang@hadoop102 ~]$ start-dfs.sh

[quakewang@hadoop103 ~]$ start-yarn.sh
```

（4）解压 Flink 安装包，并进入 conf 目录，修改 flink-conf.yaml 文件，修改配置。

### 3.2 会话模式部署

YARN 的会话模式与独立集群略有不同，需要首先申请一个YARN会话（YARN session）来启动 Flink 集群。具体步骤如下：

1. 启动集群

（1）启动 Hadoop 集群(HDFS、YARN)。

（2）执行脚本命令向 YARN 集群申请资源，开启一个 YARN 会话，启动 Flink 集群。

```shell
$ bin/yarn-session.sh -nm test
```

可用参数解读：

- -d：分离模式，如果不想让 Flink YARN 客户端一直前台运行，可以使用这个参数，即使关掉当前对话窗口，YARN session 也可以后台运行;
- -jm(--jobManagerMemory)：配置JobManager所需内存，默认单位 MB;
- -nm(--name)：配置在YARN UI界面上显示的任务名;
- -qu(--queue)：指定 YARN 队列名;
- -tm(--taskManager)：配置每个 TaskManager 所使用内存。

注意：Flink1.11.0 版本后不再使用 -n 参数和 -s 参数分别指定 TaskManager 数量和 slot 数量，YARN 会按照需求动态分配 TaskManager 和 slot。所以从这个意义上讲，YARN 的会话模式也不会把集群资源固定，同样是动态分配的。

### 3.3 单作业模式部署

在 YARN 环境中，由于有了外部平台做资源调度，所以也可以直接向 YARN 提交一个单独的作业，从而启动一个 Flink 集群。

（1）执行命令提交作业。

```shell
$ bin/flink run -d -t yarn-per-job -c com.quakewang.wc.StreamWordCount FlinkTutorial-1.0-SNAPSHOT.jar
```

（2）可以使用命令行查看或取消作业，命令如下。

```shell
$ ./bin/flink list -t yarn-per-job -Dyarn.application.id=application_XXXX_YY

$ ./bin/flink cancel -t yarn-per-job -Dyarn.application.id=application_XXXX_YY <jobId>
```

这里的 application_XXXX_YY 是当前应用的ID，<jobId>  是作业的 ID。注意如果取消作业，整个 Flink 集群也会停掉。

### 3.4 应用模式部署

应用模式同样非常简单，与单作业模式类似，直接执行flink run-application命令即可。

（1）执行命令提交作业。

```shell
$ bin/flink run-application -t yarn-application -c com.quakewang.wc.StreamWordCount FlinkTutorial-1.0-SNAPSHOT.jar 
```

（2）在命令行中查看或取消作业。

```shell
$ ./bin/flink list -t yarn-application -Dyarn.application.id=application_XXXX_YY

$ ./bin/flink cancel -t yarn-application -Dyarn.application.id=application_XXXX_YY <jobId>
```

（3）也可以通过 yarn.provided.lib.dirs 配置选项指定位置，将 jar 上传到远程。

```shell
$ ./bin/flink run-application -t yarn-application -Dyarn.provided.lib.dirs="hdfs://myhdfs/my-remote-flink-dist-dir"  hdfs://myhdfs/jars/my-application.jar
```

这种方式下 jar 可以预先上传到 HDFS，而不需要单独发送到集群，这就使得作业提交更加轻量了。