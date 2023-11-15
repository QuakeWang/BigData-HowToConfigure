# KAFKA

## 一、集群规划

| hadoop102 | hadoop103 | hadoop104 |
| --------- | --------- | --------- |
| zk        | zk        | zk        |
| kafka     | kafka     | kafka     |

## 二、准备工作

### 1、jar 包下载

https://kafka.apache.org/downloads

### 2、解压安装

#### 1）解压安装包

`tar -zxvf kafka_2.11-2.4.1.tgz -C /opt/module/`

#### 2）修改解压后的文件名称

`mv kafka_2.11-2.4.1/ kafka`

#### 3）在 /opt/module/kafka 目录下创建 logs 文件夹

`[quakewang@hadoop102 kafka]$ mkdir logs`

## 三、环境配置

### 1）修改配置文件

```bash
[quakewang@hadoop102 kafka]$ cd config/
[quakewang@hadoop102 config]$ vi server.properties
修改或者增加以下内容：
# broker 的全局唯一编号，不能重复
broker.id=0
# 删除 topic 功能使能
delete.topic.enable=true
# kafka 运行日志存放的路径
log.dirs=/opt/module/kafka/data
# 配置连接 Zookeeper 集群地址
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka
```

### 2）配置环境变量

```bash
[quakewang@hadoop102 module]$ sudo vi /etc/profile.d/my_env.sh

# KAFKA_HOME
export KAFKA_HOME=/opt/module/kafka
export PATH=$PATH:$KAFKA_HOME/bin

[quakewang@hadoop102 module]$ source /etc/profile.d/my_env.sh
```

### 3）集群同步

分发安装包

`[quakewang@hadoop102 module]$ xsync kafka/`

注：分发之后记得配置其他机器的环境变量

**分别在hadoop103 和 hadoop104 上修改配置文件 /opt/module/kafka/config/server.properties 中的 broker.id=1、broker.id=2**

注：broker.id 不得重复

## 四、kafka 群起脚本

```shell
#!/bin/bash

case $1 in
"start"){
    for i in hadoop102 hadoop103 hadoop104
    do
        echo " --------启动 $i Kafka-------"
        ssh $i "/opt/module/kafka/bin/kafka-server-start.sh -daemon /opt/module/kafka/config/server.properties "
    done
};;
"stop"){
    for i in hadoop102 hadoop103 hadoop104
    do
        echo " --------停止 $i Kafka-------"
        ssh $i "/opt/module/kafka/bin/kafka-server-stop.sh stop"
    done
};;
esac
```

增加脚本执行权限

`chmod +x kf.sh`